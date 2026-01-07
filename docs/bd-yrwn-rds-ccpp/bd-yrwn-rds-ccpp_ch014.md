# 13. 堆数据结构和 TTL

Redis 的主要用途是作为缓存服务器，一种管理缓存大小的方法是通过显式设置 TTL（生存时间）。TTL 可以使用定时器实现。不幸的是，上一章中的定时器是固定值（使用链表）；因此，需要一个排序数据结构来实现任意和可变的超时；堆数据结构是一个流行的选择。与之前使用的 AVL 树相比，堆数据结构具有使用空间更少的优点。

对堆数据结构的快速回顾：

1.  堆是一个二叉树，打包到一个数组中；树的布局是固定的。父子关系是隐式的，堆元素中不包含指针。

1.  树的唯一约束是父节点不大于其子节点。

1.  元素的值可以更新。如果值发生变化：

    +   其值比之前更大：它可能比其孩子更大，如果是这样，就与最小的孩子交换，这样父子和约束就再次满足。现在其中一个孩子比之前更大，继续这个过程，直到达到叶子节点。

    +   其值更小：同样，将其与其父节点交换，直到达到根节点。

1.  新元素作为叶子节点添加到数组的末尾。保持上述约束。

1.  当从堆中移除一个元素时，用数组中的最后一个元素替换它，然后像更新其值一样维护约束。

代码列表开始：

```cpp
struct HeapItem {
    uint64_t val = 0;
    size_t *ref = NULL;
};

// the structure for the key
struct Entry {
    struct HNode node;
    std::string  key;
    std::string  val;
    uint32_t type = 0;
    ZSet *zset = NULL;
    // for TTLs
    size_t heap_idx = -1;
};
```

堆用于排序时间戳，`Entry` 与时间戳相互链接。`heap_idx` 是相应 `HeapItem` 的索引，`ref` 指向 `Entry`。我们再次使用侵入式数据结构；`ref` 指针指向 `heap_idx` 字段。

父子关系是固定的：

```cpp
static size_t heap_parent(size_t i) {
    return (i + 1) / 2 - 1;
}

static size_t heap_left(size_t i) {
    return i * 2 + 1;
}

static size_t heap_right(size_t i) {
    return i * 2 + 2;
}
```

当孩子小于其父节点时，与父节点交换。注意，在交换时通过 `ref` 指针更新 `heap_idx`。

```cpp
static void heap_up(HeapItem *a, size_t pos) {
    HeapItem t = a[pos];
    while (pos > 0 && a[heap_parent(pos)].val > t.val) {
        // swap with the parent
        a[pos] = a[heap_parent(pos)];
        *a[pos].ref = pos;
        pos = heap_parent(pos);
    }
    a[pos] = t;
    *a[pos].ref = pos;
}
```

与最小的孩子交换类似。

```cpp
static void heap_down(HeapItem *a, size_t pos, size_t len) {
    HeapItem t = a[pos];
    while (true) {
        // find the smallest one among the parent and their kids
        size_t l = heap_left(pos);
        size_t r = heap_right(pos);
        size_t min_pos = -1;
        size_t min_val = t.val;
        if (l < len && a[l].val < min_val) {
            min_pos = l;
            min_val = a[l].val;
        }
        if (r < len && a[r].val < min_val) {
            min_pos = r;
        }
        if (min_pos == (size_t)-1) {
            break;
        }
        // swap with the kid
        a[pos] = a[min_pos];
        *a[pos].ref = pos;
        pos = min_pos;
    }
    a[pos] = t;
    *a[pos].ref = pos;
}
```

`heap_update` 是用于更新位置的堆函数。它用于更新、插入和删除。

```cpp
void heap_update(HeapItem *a, size_t pos, size_t len) {
    if (pos > 0 && a[heap_parent(pos)].val > a[pos].val) {
        heap_up(a, pos);
    } else {
        heap_down(a, pos, len);
    }
}
```

将堆添加到我们的服务器：

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
} g_data;
```

更新、添加和删除堆中的定时器。只需在更新数组中的一个元素后调用 `heap_update` 即可。

```cpp
// set or remove the TTL
static void entry_set_ttl(Entry *ent, int64_t ttl_ms) {
    if (ttl_ms < 0 && ent->heap_idx != (size_t)-1) {
        // erase an item from the heap
        // by replacing it with the last item in the array.
        size_t pos = ent->heap_idx;
        g_data.heap[pos] = g_data.heap.back();
        g_data.heap.pop_back();
        if (pos < g_data.heap.size()) {
            heap_update(g_data.heap.data(), pos, g_data.heap.size());
        }
        ent->heap_idx = -1;
    } else if (ttl_ms >= 0) {
        size_t pos = ent->heap_idx;
        if (pos == (size_t)-1) {
            // add an new item to the heap
            HeapItem item;
            item.ref = &ent->heap_idx;
            g_data.heap.push_back(item);
            pos = g_data.heap.size() - 1;
        }
        g_data.heap[pos].val = get_monotonic_usec() + (uint64_t)ttl_ms * 1000;
        heap_update(g_data.heap.data(), pos, g_data.heap.size());
    }
}
```

删除 `Entry` 时删除可能的 TTL 定时器：

```cpp
static void entry_del(Entry *ent) {
    switch (ent->type) {
    case T_ZSET:
        zset_dispose(ent->zset);
        delete ent->zset;
        break;
    }
    entry_set_ttl(ent, -1);
    delete ent;
}
```

`next_timer_ms` 函数被修改为使用空闲定时器和 TTL 定时器。

```cpp
static uint32_t next_timer_ms() {
    uint64_t now_us = get_monotonic_usec();
    uint64_t next_us = (uint64_t)-1;

    // idle timers
    if (!dlist_empty(&g_data.idle_list)) {
        Conn *next = container_of(g_data.idle_list.next, Conn, idle_list);
        next_us = next->idle_start + k_idle_timeout_ms * 1000;
    }

    // ttl timers
    if (!g_data.heap.empty() && g_data.heap[0].val < next_us) {
        next_us = g_data.heap[0].val;
    }

    if (next_us == (uint64_t)-1) {
        return 10000;   // no timer, the value doesn't matter
    }

    if (next_us <= now_us) {
        // missed?
        return 0;
    }
    return (uint32_t)((next_us - now_us) / 1000);
}
```

向 `process_timers` 函数添加 TTL 定时器：

```cpp
static void process_timers() {
    // the extra 1000us is for the ms resolution of poll()
    uint64_t now_us = get_monotonic_usec() + 1000;

    // idle timers
    while (!dlist_empty(&g_data.idle_list)) {
        // code omitted...
    }

    // TTL timers
    const size_t k_max_works = 2000;
    size_t nworks = 0;
    while (!g_data.heap.empty() && g_data.heap[0].val < now_us) {
        Entry *ent = container_of(g_data.heap[0].ref, Entry, heap_idx);
        HNode *node = hm_pop(&g_data.db, &ent->node, &hnode_same);
        assert(node == &ent->node);
        entry_del(ent);
        if (nworks++ >= k_max_works) {
            // don't stall the server if too many keys are expiring at once
            break;
        }
    }
}
```

这只是检查堆的最小值并移除键。注意，我们限制了每个事件循环迭代中过期的键的数量；限制是必要的，以防止在一次性有太多键过期时服务器停滞。

更新和查询 TTL 的命令添加起来很简单：

```cpp
static void do_expire(std::vector<std::string> &cmd, std::string &out) {
    int64_t ttl_ms = 0;
    if (!str2int(cmd[2], ttl_ms)) {
        return out_err(out, ERR_ARG, "expect int64");
    }

    Entry key;
    key.key.swap(cmd[1]);
    key.node.hcode = str_hash((uint8_t *)key.key.data(), key.key.size());

    HNode *node = hm_lookup(&g_data.db, &key.node, &entry_eq);
    if (node) {
        Entry *ent = container_of(node, Entry, node);
        entry_set_ttl(ent, ttl_ms);
    }
    return out_int(out, node ? 1: 0);
}
```

```cpp
static void do_ttl(std::vector<std::string> &cmd, std::string &out) {
    Entry key;
    key.key.swap(cmd[1]);
    key.node.hcode = str_hash((uint8_t *)key.key.data(), key.key.size());

    HNode *node = hm_lookup(&g_data.db, &key.node, &entry_eq);
    if (!node) {
        return out_int(out, -2);
    }

    Entry *ent = container_of(node, Entry, node);
    if (ent->heap_idx == (size_t)-1) {
        return out_int(out, -1);
    }

    uint64_t expire_at = g_data.heap[ent->heap_idx].val;
    uint64_t now_us = get_monotonic_usec();
    return out_int(out, expire_at > now_us ? (expire_at - now_us) / 1000 : 0);
}
```

练习：

1.  基于堆的定时器向服务器添加 `O(log(n))` 操作，这可能会成为足够多键的数量时的瓶颈。你能想到针对大量定时器的优化方法吗？

1.  真实的 Redis 并不使用排序来实现过期，找出它是如何实现的，并列出两种方法的优缺点。

> +   [13_server.cpp](https://build-your-own.org/redis/13/13_server.cpp.htm)
> +   
> +   [avl.cpp](https://build-your-own.org/redis/13/avl.cpp.htm)
> +   
> +   [avl.h](https://build-your-own.org/redis/13/avl.h.htm)
> +   
> +   [common.h](https://build-your-own.org/redis/13/common.h.htm)
> +   
> +   [hashtable.cpp](https://build-your-own.org/redis/13/hashtable.cpp.htm)
> +   
> +   [hashtable.h](https://build-your-own.org/redis/13/hashtable.h.htm)
> +   
> +   [heap.cpp](https://build-your-own.org/redis/13/heap.cpp.htm)
> +   
> +   [heap.h](https://build-your-own.org/redis/13/heap.h.htm)
> +   
> +   [list.h](https://build-your-own.org/redis/13/list.h.htm)
> +   
> +   [test_heap.cpp](https://build-your-own.org/redis/13/test_heap.cpp.htm)
> +   
> +   [zset.cpp](https://build-your-own.org/redis/13/zset.cpp.htm)
> +   
> +   [zset.h](https://build-your-own.org/redis/13/zset.h.htm)
