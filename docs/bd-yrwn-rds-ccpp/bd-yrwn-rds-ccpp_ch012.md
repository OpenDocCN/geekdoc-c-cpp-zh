# 11. AVL 树和有序集合

基于上一章中的 AVL 树，可以轻松地添加有序集合数据结构。结构定义：

```cpp
struct ZSet {
    AVLNode *tree = NULL;
    HMap hmap;
};

struct ZNode {
    AVLNode tree;
    HNode hmap;
    double score = 0;
    size_t len = 0;
    char name[0];
};

static ZNode *znode_new(const char *name, size_t len, double score) {
    ZNode *node = (ZNode *)malloc(sizeof(ZNode) + len);
    assert(node);   // not a good idea in real projects
    avl_init(&node->tree);
    node->hmap.next = NULL;
    node->hmap.hcode = str_hash((uint8_t *)name, len);
    node->score = score;
    node->len = len;
    memcpy(&node->name[0], name, len);
    return node;
}
```

有序集合是一系列`(score, name)`对的有序列表，支持通过排序键或`name`进行查询或更新。它是 AVL 树和散列表的组合，对节点属于两者，这展示了侵入式数据结构的灵活性。`name`字符串嵌入在节点对的末尾，希望节省一些空间开销。

树插入函数大致与上一章中看到的测试代码相同：

```cpp
// insert into the AVL tree
static void tree_add(ZSet *zset, ZNode *node) {
    if (!zset->tree) {
        zset->tree = &node->tree;
        return;
    }

    AVLNode *cur = zset->tree;
    while (true) {
        AVLNode **from = zless(&node->tree, cur) ? &cur->left : &cur->right;
        if (!*from) {
            *from = &node->tree;
            node->tree.parent = cur;
            zset->tree = avl_fix(&node->tree);
            break;
        }
        cur = *from;
    }
}
```

`zless`是用于比较两个对的辅助函数：

```cpp
// compare by the (score, name) tuple
static bool zless(
    AVLNode *lhs, double score, const char *name, size_t len)
{
    ZNode *zl = container_of(lhs, ZNode, tree);
    if (zl->score != score) {
        return zl->score < score;
    }
    int rv = memcmp(zl->name, name, min(zl->len, len));
    if (rv != 0) {
        return rv < 0;
    }
    return zl->len < len;
}

static bool zless(AVLNode *lhs, AVLNode *rhs) {
    ZNode *zr = container_of(rhs, ZNode, tree);
    return zless(lhs, zr->score, zr->name, zr->len);
}
```

插入/更新子程序：

```cpp
// update the score of an existing node (AVL tree reinsertion)
static void zset_update(ZSet *zset, ZNode *node, double score) {
    if (node->score == score) {
        return;
    }
    zset->tree = avl_del(&node->tree);
    node->score = score;
    avl_init(&node->tree);
    tree_add(zset, node);
}

// add a new (score, name) tuple, or update the score of the existing tuple
bool zset_add(ZSet *zset, const char *name, size_t len, double score) {
    ZNode *node = zset_lookup(zset, name, len);
    if (node) {
        zset_update(zset, node, score);
        return false;
    } else {
        node = znode_new(name, len, score);
        hm_insert(&zset->hmap, &node->hmap);
        tree_add(zset, node);
        return true;
    }
}

// lookup by name
ZNode *zset_lookup(ZSet *zset, const char *name, size_t len) {
    // just a hashtable look up
    // code omitted...
}
```

这里是排序集的主要用途：范围查询。

```cpp
// find the (score, name) tuple that is greater or equal to the argument,
// then offset relative to it.
ZNode *zset_query(
    ZSet *zset, double score, const char *name, size_t len, int64_t offset)
{
    AVLNode *found = NULL;
    AVLNode *cur = zset->tree;
    while (cur) {
        if (zless(cur, score, name, len)) {
            cur = cur->right;
        } else {
            found = cur;    // candidate
            cur = cur->left;
        }
    }

    if (found) {
        found = avl_offset(found, offset);
    }
    return found ? container_of(found, ZNode, tree) : NULL;
}
```

范围查询只是一个常规的二元树查找，随后是偏移操作。偏移操作使得有序集合特殊，它不是常规的二元树遍历。

让我们回顾一下`AVLNode`：

```cpp
struct AVLNode {
    uint32_t depth = 0;
    uint32_t cnt = 0;
    AVLNode *left = NULL;
    AVLNode *right = NULL;
    AVLNode *parent = NULL;
};
```

它有一个额外的`cnt`字段（树的大小），这在上一章中没有解释。它被`avl_offset`函数使用：

```cpp
// offset into the succeeding or preceding node.
// note: the worst-case is O(log(n)) regardless of how long the offset is.
AVLNode *avl_offset(AVLNode *node, int64_t offset) {
    int64_t pos = 0;    // relative to the starting node
    while (offset != pos) {
        if (pos < offset && pos + avl_cnt(node->right) >= offset) {
            // the target is inside the right subtree
            node = node->right;
            pos += avl_cnt(node->left) + 1;
        } else if (pos > offset && pos - avl_cnt(node->left) <= offset) {
            // the target is inside the left subtree
            node = node->left;
            pos -= avl_cnt(node->right) + 1;
        } else {
            // go to the parent
            AVLNode *parent = node->parent;
            if (!parent) {
                return NULL;
            }
            if (parent->right == node) {
                pos -= avl_cnt(node->left) + 1;
            } else {
                pos += avl_cnt(node->right) + 1;
            }
            node = parent;
        }
    }
    return node;
}
```

在节点中嵌入大小信息后，我们可以确定偏移目标是否在子树中。偏移操作分为两个阶段：首先，如果目标不在子树中，则沿着树向上移动，然后向下移动树，缩小距离直到遇到目标。无论偏移量有多长，最坏情况都是`O(log(n))`，这比逐个移动到后续节点进行偏移要好（最佳情况为`O(offset)`）。实际的 Redis 项目使用类似的跳表技术。

现在停止并测试新的`avl_offset`函数是个好主意。

```cpp
static void test_case(uint32_t sz) {
    Container c;
    for (uint32_t i = 0; i < sz; ++i) {
        add(c, i);
    }

    AVLNode *min = c.root;
    while (min->left) {
        min = min->left;
    }
    for (uint32_t i = 0; i < sz; ++i) {
        AVLNode *node = avl_offset(min, (int64_t)i);
        assert(container_of(node, Data, node)->val == i);

        for (uint32_t j = 0; j < sz; ++j) {
            int64_t offset = (int64_t)j - (int64_t)i;
            AVLNode *n2 = avl_offset(node, offset);
            assert(container_of(n2, Data, node)->val == j);
        }
        assert(!avl_offset(node, -(int64_t)i - 1));
        assert(!avl_offset(node, sz - i));
    }

    dispose(c.root);
}
```

目前，我们已经实现了有序集合的主要功能。让我们将有序集合类型添加到我们的服务器中。

```cpp
enum {
    T_STR = 0,
    T_ZSET = 1,
};

// the structure for the key
struct Entry {
    struct HNode node;
    std::string  key;
    std::string  val;
    uint32_t type = 0;
    ZSet *zset = NULL;
};
```

代码的其余部分被认为是平凡的，将在代码列表中省略。

添加 Python 脚本以测试新命令：

```cpp
CASES = r'''
$ ./client zscore asdf n1
(nil)
$ ./client zquery xxx 1 asdf 1 10
(arr) len=0
(arr) end
# more cases...
'''

import shlex
import subprocess

cmds = []
outputs = []
lines = CASES.splitlines()
for x in lines:
    x = x.strip()
    if not x:
        continue
    if x.startswith('$ '):
        cmds.append(x[2:])
        outputs.append('')
    else:
        outputs[-1] = outputs[-1] + x + '\n'

assert len(cmds) == len(outputs)
for cmd, expect in zip(cmds, outputs):
    out = subprocess.check_output(shlex.split(cmd)).decode('utf-8')
    assert out == expect, f'cmd:{cmd} out:{out}'
```

练习：

1.  `avl_offset`函数使我们能够通过排名查询排序集，现在反过来，给定 AVL 树中的一个节点，找到其排名，最坏情况为`O(log(n))`。（这是`zrank`命令。）

1.  另一个有序集合应用：计算一个范围内的元素数量。（也具有`O(log(n))`的最坏情况。）

1.  `11_server.cpp`文件已经包含了一些有序集合命令，尝试添加更多。

> +   [11_client.cpp](https://build-your-own.org/redis/11/11_client.cpp.htm)
> +   
> +   [11_server.cpp](https://build-your-own.org/redis/11/11_server.cpp.htm)
> +   
> +   [avl.cpp](https://build-your-own.org/redis/11/avl.cpp.htm)
> +   
> +   [avl.h](https://build-your-own.org/redis/11/avl.h.htm)
> +   
> +   [common.h](https://build-your-own.org/redis/11/common.h.htm)
> +   
> +   [hashtable.cpp](https://build-your-own.org/redis/11/hashtable.cpp.htm)
> +   
> +   [hashtable.h](https://build-your-own.org/redis/11/hashtable.h.htm)
> +   
> +   [test_cmds.py](https://build-your-own.org/redis/11/test_cmds.py.htm)
> +   
> +   [test_offset.cpp](https://build-your-own.org/redis/11/test_offset.cpp.htm)
> +   
> +   [zset.cpp](https://build-your-own.org/redis/11/zset.cpp.htm)
> +   
> +   [zset.h](https://build-your-own.org/redis/11/zset.h.htm)
