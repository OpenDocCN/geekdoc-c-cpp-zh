# 14. 线程池与异步任务

自从引入有序集合数据类型以来，我们的服务器存在一个缺陷：键的删除。如果一个有序集合的大小非常大，释放其节点可能需要很长时间，并且在销毁键的过程中服务器会停滞。这可以通过使用多线程将析构函数从主线程中移除来轻松修复。

首先，我们介绍“线程池”，字面上是一个线程池。池中的线程从队列中消费任务并执行它们。使用 `pthread` API 编写多生产者多消费者队列是微不足道的。（尽管在我们的情况下只有一个生产者。）

相关的 `pthread` 原语是 `pthread_mutex_t` 和 `pthread_cond_t`；它们分别被称为互斥锁和条件变量。如果你对它们不熟悉，建议在阅读本章之后获取一些关于多线程的教育。（例如 `pthread` API 的 manpages、操作系统教材、在线课程等。）

这里是对两个 `pthread` 原语的简要介绍：

+   队列被多个线程（生产者和消费者）访问，因此显然需要互斥锁的保护。

+   消费者线程在空闲时应该处于睡眠状态，只有在队列不为空时才被唤醒，这是条件变量的工作。

线程池数据类型定义如下：

```cpp
struct Work {
    void (*f)(void *) = NULL;
    void *arg = NULL;
};

struct TheadPool {
    std::vector<pthread_t> threads;
    std::deque<Work> queue;
    pthread_mutex_t mu;
    pthread_cond_t not_empty;
};
```

`thread_pool_init` 用于初始化和启动线程。`pthread` 类型通过 `pthread_xxx_init` 函数初始化，`pthread_create` 通过目标函数 `worker` 启动线程。

```cpp
void thread_pool_init(TheadPool *tp, size_t num_threads) {
    assert(num_threads > 0);

    int rv = pthread_mutex_init(&tp->mu, NULL);
    assert(rv == 0);
    rv = pthread_cond_init(&tp->not_empty, NULL);
    assert(rv == 0);

    tp->threads.resize(num_threads);
    for (size_t i = 0; i < num_threads; ++i) {
        int rv = pthread_create(&tp->threads[i], NULL, &worker, tp);
        assert(rv == 0);
    }
}
```

消费者代码：

```cpp
static void *worker(void *arg) {
    TheadPool *tp = (TheadPool *)arg;
    while (true) {
        pthread_mutex_lock(&tp->mu);
        // wait for the condition: a non-empty queue
        while (tp->queue.empty()) {
            pthread_cond_wait(&tp->not_empty, &tp->mu);
        }

        // got the job
        Work w = tp->queue.front();
        tp->queue.pop_front();
        pthread_mutex_unlock(&tp->mu);

        // do the work
        w.f(w.arg);
    }
    return NULL;
}
```

生产者代码：

```cpp
void thread_pool_queue(TheadPool *tp, void (*f)(void *), void *arg) {
    Work w;
    w.f = f;
    w.arg = arg;

    pthread_mutex_lock(&tp->mu);
    tp->queue.push_back(w);
    pthread_cond_signal(&tp->not_empty);
    pthread_mutex_unlock(&tp->mu);
}
```

解释：

1.  对于生产者和消费者，队列访问代码被 `pthread_mutex_lock` 和 `pthread_mutex_unlock` 包围，一次只有一个线程可以访问队列。

1.  在消费者获取互斥锁后，检查队列：

    +   如果队列不为空，从队列中获取一个工作，释放互斥锁并执行工作。

    +   否则，释放互斥锁并进入睡眠状态，睡眠可以被条件变量唤醒。这通过单个 `pthread_cond_wait` 调用完成。

1.  在生产者将工作放入队列后，生产者调用 `pthread_cond_signal` 唤醒一个可能正在睡眠的消费者。

1.  在消费者从 `pthread_cond_wait` 中唤醒后，互斥锁会自动再次被持有。消费者在醒来后必须再次检查条件，如果条件（非空队列）不满足，则返回睡眠状态。

条件变量的使用需要更多的解释：`pthread_cond_wait` 函数始终在一个循环中检查条件。这是因为条件可能在唤醒的消费者抓取互斥锁之前被其他消费者更改；互斥锁不会从信号者转移到即将被唤醒的消费者！如果你看到没有循环使用的条件变量，那可能是一个错误。

一个具体的序列，帮助你理解条件变量的使用：

1.  生产者发出信号。

1.  生产者释放互斥锁。

1.  一些消费者获取互斥锁并清空队列。

1.  消费者从生产者的信号中醒来并获取互斥锁，但队列是空的！

注意，`pthread_cond_signal` 不需要被互斥锁保护，释放互斥锁后进行信号也是正确的。

线程池已完成。让我们将其添加到我们的服务器中：

```cpp
// global variables
static struct {
    HMap db;
    // a map of all client connections, keyed by fd
    std::vector<Conn *> fd2conn;
    // timers for idle connections
    DList idle_list;
    // timers for TTLs
    std::vector<HeapItem> heap;
    // the thread pool
    TheadPool tp;
} g_data;
```

在 `main` 函数内部：

```cpp
    // some initializations
    dlist_init(&g_data.idle_list);
    thread_pool_init(&g_data.tp, 4);
```

`entry_del` 函数被修改：它将大排序集的销毁放入线程池。线程池仅用于大型操作，因为多线程也有一些开销。

```cpp
// deallocate the key immediately
static void entry_destroy(Entry *ent) {
    switch (ent->type) {
    case T_ZSET:
        zset_dispose(ent->zset);
        delete ent->zset;
        break;
    }
    delete ent;
}

static void entry_del_async(void *arg) {
    entry_destroy((Entry *)arg);
}

// dispose the entry after it got detached from the key space
static void entry_del(Entry *ent) {
    entry_set_ttl(ent, -1);

    const size_t k_large_container_size = 10000;
    bool too_big = false;
    switch (ent->type) {
    case T_ZSET:
        too_big = hm_size(&ent->zset->hmap) > k_large_container_size;
        break;
    }

    if (too_big) {
        thread_pool_queue(&g_data.tp, &entry_del_async, ent);
    } else {
        entry_destroy(ent);
    }
}
```

练习：

1.  信号量通常被介绍为多线程原语，而不是条件变量和互斥锁。尝试使用信号量实现线程池。

1.  一些有趣的练习，帮助你进一步理解这些原语：

    1.  使用信号量实现互斥锁。（简单）

    1.  使用条件变量实现信号量。（简单）

    1.  仅使用互斥锁实现条件变量。（中等）

    1.  既然你知道这些原语在某种程度上是等价的，为什么你应该选择其中一个而不是另一个？

> +   [14_server.cpp](https://build-your-own.org/redis/14/14_server.cpp.htm)
> +   
> +   [avl.cpp](https://build-your-own.org/redis/14/avl.cpp.htm)
> +   
> +   [avl.h](https://build-your-own.org/redis/14/avl.h.htm)
> +   
> +   [common.h](https://build-your-own.org/redis/14/common.h.htm)
> +   
> +   [hashtable.cpp](https://build-your-own.org/redis/14/hashtable.cpp.htm)
> +   
> +   [hashtable.h](https://build-your-own.org/redis/14/hashtable.h.htm)
> +   
> +   [heap.cpp](https://build-your-own.org/redis/14/heap.cpp.htm)
> +   
> +   [heap.h](https://build-your-own.org/redis/14/heap.h.htm)
> +   
> +   [list.h](https://build-your-own.org/redis/14/list.h.htm)
> +   
> +   [thread_pool.cpp](https://build-your-own.org/redis/14/thread_pool.cpp.htm)
> +   
> +   [thread_pool.h](https://build-your-own.org/redis/14/thread_pool.h.htm)
> +   
> +   [zset.cpp](https://build-your-own.org/redis/14/zset.cpp.htm)
> +   
> +   [zset.h](https://build-your-own.org/redis/14/zset.h.htm)
