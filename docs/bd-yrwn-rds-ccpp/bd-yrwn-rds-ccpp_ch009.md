# 08. 数据结构：散列表

本章将填补上一章服务器中的占位代码。我们将首先实现一个散列表。散列表通常是存储未知数量的键值数据且不需要排序的数据结构的明显选择。

有两种散列表：链表法和开放寻址法。它们的主要区别在于冲突解决。开放寻址法在发生冲突时寻找另一个空闲槽位，而链表法只是将冲突的键通过链表分组。由于需要找到空闲槽位，开放寻址法有许多变体，而链表散列表基本上是一个固定的设计。我们服务器中使用的散列表是链表散列表。链表散列表易于编码；它不需要做太多的选择。

我们数据类型的定义：

```cpp
// hashtable node, should be embedded into the payload
struct HNode {
    HNode *next = NULL;
    uint64_t hcode = 0;
};

// a simple fixed-sized hashtable
struct HTab {
    HNode **tab = NULL;
    size_t mask = 0;
    size_t size = 0;
};
```

当散列表的大小是 2 的幂时，索引操作是一个简单的位掩码与哈希码。

```cpp
// n must be a power of 2
static void h_init(HTab *htab, size_t n) {
    assert(n > 0 && ((n - 1) & n) == 0);
    htab->tab = (HNode **)calloc(sizeof(HNode *), n);
    htab->mask = n - 1;
    htab->size = 0;
}

// hashtable insertion
static void h_insert(HTab *htab, HNode *node) {
    size_t pos = node->hcode & htab->mask;
    HNode *next = htab->tab[pos];
    node->next = next;
    htab->tab[pos] = node;
    htab->size++;
}
```

查找子程序只是一个列表遍历：

```cpp
// hashtable look up subroutine.
// Pay attention to the return value. It returns the address of
// the parent pointer that owns the target node,
// which can be used to delete the target node.
static HNode **h_lookup(
    HTab *htab, HNode *key, bool (*cmp)(HNode *, HNode *))
{
    if (!htab->tab) {
        return NULL;
    }

    size_t pos = key->hcode & htab->mask;
    HNode **from = &htab->tab[pos];
    while (*from) {
        if (cmp(*from, key)) {
            return from;
        }
        from = &(*from)->next;
    }
    return NULL;
}
```

删除操作很简单。注意指针的使用如何使得代码简洁。`from` 指针可以是数组中的一个元素，也可以是从一个节点开始的，但代码并没有区分。

```cpp
// remove a node from the chain
static HNode *h_detach(HTab *htab, HNode **from) {
    HNode *node = *from;
    *from = (*from)->next;
    htab->size--;
    return node;
}
```

我们的散列表大小固定，当负载因子过高时，我们需要迁移到一个更大的散列表。在使用 Redis 中的散列表时，有一个额外的考虑。扩容一个大的散列表需要将许多节点移动到新表中，这可能会使服务器暂停一段时间。为了避免一次性移动所有内容，我们保留两个散列表，并逐渐在它们之间移动节点。以下是最终的散列表接口：

```cpp
// the real hashtable interface.
// it uses 2 hashtables for progressive resizing.
struct HMap {
    HTab ht1;
    HTab ht2;
    size_t resizing_pos = 0;
};
```

查找子程序现在帮助进行扩容：

```cpp
HNode *hm_lookup(
    HMap *hmap, HNode *key, bool (*cmp)(HNode *, HNode *))
{
    hm_help_resizing(hmap);
    HNode **from = h_lookup(&hmap->ht1, key, cmp);
    if (!from) {
        from = h_lookup(&hmap->ht2, key, cmp);
    }
    return from ? *from : NULL;
}
```

`hm_help_resizing` 函数是逐渐移动节点的子程序：

```cpp
const size_t k_resizing_work = 128;

static void hm_help_resizing(HMap *hmap) {
    if (hmap->ht2.tab == NULL) {
        return;
    }

    size_t nwork = 0;
    while (nwork < k_resizing_work && hmap->ht2.size > 0) {
        // scan for nodes from ht2 and move them to ht1
        HNode **from = &hmap->ht2.tab[hmap->resizing_pos];
        if (!*from) {
            hmap->resizing_pos++;
            continue;
        }

        h_insert(&hmap->ht1, h_detach(&hmap->ht2, from));
        nwork++;
    }

    if (hmap->ht2.size == 0) {
        // done
        free(hmap->ht2.tab);
        hmap->ht2 = HTab{};
    }
}
```

插入子程序将在表变得太满时触发扩容：

```cpp
const size_t k_max_load_factor = 8;

void hm_insert(HMap *hmap, HNode *node) {
    if (!hmap->ht1.tab) {
        h_init(&hmap->ht1, 4);
    }
    h_insert(&hmap->ht1, node);

    if (!hmap->ht2.tab) {
        // check whether we need to resize
        size_t load_factor = hmap->ht1.size / (hmap->ht1.mask + 1);
        if (load_factor >= k_max_load_factor) {
            hm_start_resizing(hmap);
        }
    }
    hm_help_resizing(hmap);
}

static void hm_start_resizing(HMap *hmap) {
    assert(hmap->ht2.tab == NULL);
    // create a bigger hashtable and swap them
    hmap->ht2 = hmap->ht1;
    h_init(&hmap->ht1, (hmap->ht1.mask + 1) * 2);
    hmap->resizing_pos = 0;
}
```

移除键的子程序。没有什么有趣的。

```cpp
HNode *hm_pop(
    HMap *hmap, HNode *key, bool (*cmp)(HNode *, HNode *))
{
    hm_help_resizing(hmap);
    HNode **from = h_lookup(&hmap->ht1, key, cmp);
    if (from) {
        return h_detach(&hmap->ht1, from);
    }
    from = h_lookup(&hmap->ht2, key, cmp);
    if (from) {
        return h_detach(&hmap->ht2, from);
    }
    return NULL;
}
```

散列表实现已完成。让我们将其添加到服务器中。再次查看 `struct HNode`，这个结构不包含数据，我们实际上如何使用它呢？答案是“侵入式数据结构”：

```cpp
// the structure for the key
struct Entry {
    struct HNode node;
    std::string  key;
    std::string  val;
};
```

而不是让我们的数据结构包含数据，散列表节点结构被嵌入到有效载荷数据中。这是在 C 中创建通用数据结构的标准方式。除了使数据结构完全通用外，这种技术还有减少不必要的内存管理的优势。结构节点不是单独分配的，而是有效载荷数据的一部分，数据结构代码不拥有有效载荷，而只是组织数据。如果你从教科书中学习数据结构，这可能是一个全新的概念，可能使用 `void *` 或 C++ 模板，甚至是宏。

列出 `do_get` 函数以查看如何使用侵入式数据结构：

```cpp
// The data structure for the key space.
static struct {
    HMap db;
} g_data;

static uint32_t do_get(
    std::vector<std::string> &cmd, uint8_t *res, uint32_t *reslen)
{
    Entry key;
    key.key.swap(cmd[1]);
    key.node.hcode = str_hash((uint8_t *)key.key.data(), key.key.size());

    HNode *node = hm_lookup(&g_data.db, &key.node, &entry_eq);
    if (!node) {
        return RES_NX;
    }

    const std::string &val = container_of(node, Entry, node)->val;
    assert(val.size() <= k_max_msg);
    memcpy(res, val.data(), val.size());
    *reslen = (uint32_t)val.size();
    return RES_OK;
}

static bool entry_eq(HNode *lhs, HNode *rhs) {
    struct Entry *le = container_of(lhs, struct Entry, node);
    struct Entry *re = container_of(rhs, struct Entry, node);
    return lhs->hcode == rhs->hcode && le->key == re->key;
}
```

`hm_lookup` 函数返回一个指向 `HNode` 的指针，而 `HNode` 是 `Entry` 的一个成员，我们需要进行一些指针运算来将这个指针转换为 `Entry` 指针。在 C 项目中，`container_of` 宏通常用于此目的：

```cpp
#define container_of(ptr,  type,  member)  ({  \
  const  typeof(  ((type  *)0)->member  )  *__mptr  =  (ptr);  \
  (type  *)(  (char  *)__mptr  -  offsetof(type,  member)  );})
```

`do_set` 和 `do_del` 都是微不足道的。

```cpp
static uint32_t do_set(
    std::vector<std::string> &cmd, uint8_t *res, uint32_t *reslen)
{
    (void)res;
    (void)reslen;

    Entry key;
    key.key.swap(cmd[1]);
    key.node.hcode = str_hash((uint8_t *)key.key.data(), key.key.size());

    HNode *node = hm_lookup(&g_data.db, &key.node, &entry_eq);
    if (node) {
        container_of(node, Entry, node)->val.swap(cmd[2]);
    } else {
        Entry *ent = new Entry();
        ent->key.swap(key.key);
        ent->node.hcode = key.node.hcode;
        ent->val.swap(cmd[2]);
        hm_insert(&g_data.db, &ent->node);
    }
    return RES_OK;
}

static uint32_t do_del(
    std::vector<std::string> &cmd, uint8_t *res, uint32_t *reslen)
{
    (void)res;
    (void)reslen;

    Entry key;
    key.key.swap(cmd[1]);
    key.node.hcode = str_hash((uint8_t *)key.key.data(), key.key.size());

    HNode *node = hm_pop(&g_data.db, &key.node, &entry_eq);
    if (node) {
        delete container_of(node, Entry, node);
    }
    return RES_OK;
}
```

练习：

1.  当哈希表的负载因子过高时，我们的哈希表会触发扩容，那么当负载因子过低时，我们也应该缩小哈希表吗？缩小操作可以自动执行吗？

> +   [08_server.cpp](https://build-your-own.org/redis/08/08_server.cpp.htm)
> +   
> +   [hashtable.cpp](https://build-your-own.org/redis/08/hashtable.cpp.htm)
> +   
> +   [hashtable.h](https://build-your-own.org/redis/08/hashtable.h.htm)
