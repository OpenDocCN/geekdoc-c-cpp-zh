# 10. AVL 树：实现与测试

虽然 Redis 通常被称为键值存储，但 Redis 的“值”部分并不限于普通字符串，列表、哈希表和有序集合都是非常不错的数据结构。由于其丰富的数据结构集，Redis 也被称为“数据结构服务器”。Redis 通常用作内存缓存，在内存中存储数据时，可以自由使用数据结构。Redis 中的有序集合数据结构非常独特且有用。它不仅能够按顺序排序你的数据，还具有按排名查询有序数据的独特功能。如果你将 2000 万条记录放入有序集合，你可以获取排名在第 1000 万的记录，而不必遍历前 1000 万条记录，这是当前 SQL 数据库无法复制的功能。

正如名称“有序集合”所暗示的，它是一种用于排序的数据结构。树、平衡二叉树是存储排序数据的流行数据结构。在各种数据结构中，作者发现 AVL 树特别简单且易于编码，本书将使用 AVL 树来实现有序集合。实际的 Redis 项目使用跳表，这也被认为是易于编码的。

AVL 树的理念是限制左子树和右子树的高度差。子树之间的高度差被限制在最多一个，永远不会达到两个。当在 AVL 树中插入/删除节点时，高度差可以暂时达到两个，然后通过节点旋转来修复。旋转操作是平衡二叉树的基础，也被其他平衡树如 RB 树所使用。旋转后，具有两个子树高度差的节点会恢复到最多一个。

让我们从树节点开始：

```cpp
struct AVLNode {
    uint32_t depth = 0;
    uint32_t cnt = 0;
    AVLNode *left = NULL;
    AVLNode *right = NULL;
    AVLNode *parent = NULL;
};

static void avl_init(AVLNode *node) {
    node->depth = 1;
    node->cnt = 1;
    node->left = node->right = node->parent = NULL;
}
```

这是一个带有额外字段的常规二叉树节点。`depth`字段是树的高度。`cnt`字段是树的大小，这个字段不是 AVL 树特有的，它用于实现基于排名的查询，这将在下一章中解释。

列出一些辅助函数：

```cpp
static uint32_t avl_depth(AVLNode *node) {
    return node ? node->depth : 0;
}

static uint32_t avl_cnt(AVLNode *node) {
    return node ? node->cnt : 0;
}

static uint32_t max(uint32_t lhs, uint32_t rhs) {
    return lhs < rhs ? rhs : lhs;
}

// maintaining the depth and cnt field
static void avl_update(AVLNode *node) {
    node->depth = 1 + max(avl_depth(node->left), avl_depth(node->right));
    node->cnt = 1 + avl_cnt(node->left) + avl_cnt(node->right);
}
```

节点旋转代码：

```cpp
static AVLNode *rot_left(AVLNode *node) {
    AVLNode *new_node = node->right;
    if (new_node->left) {
        new_node->left->parent = node;
    }
    node->right = new_node->left;
    new_node->left = node;
    new_node->parent = node->parent;
    node->parent = new_node;
    avl_update(node);
    avl_update(new_node);
    return new_node;
}

static AVLNode *rot_right(AVLNode *node) {
    // a mirror of the rot_left()
    // code omited...
}
```

`rot_left`操作的可视化：

```cpp
 b           d
 / \         /
a   d  ==>  b
   /       / \
  c       a   c
```

`avl_fix_left`和`avl_fix_right`是用于修复过多高度差的函数：

```cpp
// the left subtree is too deep
static AVLNode *avl_fix_left(AVLNode *root) {
    if (avl_depth(root->left->left) < avl_depth(root->left->right)) {
        root->left = rot_left(root->left);
    }
    return rot_right(root);
}

// the right subtree is too deep
static AVLNode *avl_fix_right(AVLNode *root) {
    if (avl_depth(root->right->right) < avl_depth(root->right->left)) {
        root->right = rot_right(root->right);
    }
    return rot_left(root);
}
```

如果右子树太深，则进行左旋转可以修复它。在左旋转之前，我们可能需要在右子树上进行右旋转，以确保右子树向正确的方向倾斜。以下是可视化：

```cpp
 b           b           d
 / \         / \         / \
a   c  ==>  a   d  ==>  b   c
   /             \     /
  d               c   a
```

`avl_fix`函数在插入/删除操作后修复所有内容。它从最初受影响的节点到根节点。由于旋转可能会改变树的根节点，所以返回根节点。这是我们 AVL 树实现的核心。

```cpp
// fix imbalanced nodes and maintain invariants until the root is reached
static AVLNode *avl_fix(AVLNode *node) {
    while (true) {
        avl_update(node);
        uint32_t l = avl_depth(node->left);
        uint32_t r = avl_depth(node->right);
        AVLNode **from = NULL;
        if (node->parent) {
            from = (node->parent->left == node)
                ? &node->parent->left : &node->parent->right;
        }
        if (l == r + 2) {
            node = avl_fix_left(node);
        } else if (l + 2 == r) {
            node = avl_fix_right(node);
        }
        if (!from) {
            return node;
        }
        *from = node;
        node = node->parent;
    }
}
```

二叉树的插入很简单，只需从根节点向下遍历，直到找到一个空的子树，然后将新节点放置在这里，然后调用`avl_fix`进行维护。

删除操作更复杂。如果目标节点没有子树，则直接删除，如果有一个子树，则用那个子树替换节点。当节点有两个子树时，问题就出现了，我们无法直接删除它，而是删除右子树中的兄弟节点，并将其与分离的兄弟节点交换。以下是删除节点的函数：

```cpp
// detach a node and returns the new root of the tree
static AVLNode *avl_del(AVLNode *node) {
    if (node->right == NULL) {
        // no right subtree, replace the node with the left subtree
        // link the left subtree to the parent
        AVLNode *parent = node->parent;
        if (node->left) {
            node->left->parent = parent;
        }
        if (parent) {
            // attach the left subtree to the parent
            (parent->left == node ? parent->left : parent->right) = node->left;
            return avl_fix(parent);
        } else {
            // removing root?
            return node->left;
        }
    } else {
        // swap the node with its next sibling
        AVLNode *victim = node->right;
        while (victim->left) {
            victim = victim->left;
        }
        AVLNode *root = avl_del(victim);

        *victim = *node;
        if (victim->left) {
            victim->left->parent = victim;
        }
        if (victim->right) {
            victim->right->parent = victim;
        }
        AVLNode *parent = node->parent;
        if (parent) {
            (parent->left == node ? parent->left : parent->right) = victim;
            return root;
        } else {
            // removing root?
            return victim;
        }
    }
}
```

这是从二叉树中删除节点的通用函数，带有 AVL 树特有的`avl_fix`。

有 RB 树经验的读者可能会注意到 AVL 树的实现既小又简单。RB 树节点删除的维护代码比插入复杂得多；而 AVL 树使用相同的函数`avl_fix`进行插入和删除，这种对称性大大减少了编写 AVL 树所需的编码工作量。

AVL 树比我们之前编写的散列表要复杂得多。因此，我们需要投入更多的时间进行测试。测试代码也展示了这些 AVL 树函数的使用。

这里是我们的测试数据类型。如果你不熟悉侵入式数据结构，请阅读散列表章节。

```cpp
struct Data {
    AVLNode node;
    uint32_t val = 0;
};

struct Container {
    AVLNode *root = NULL;
};
```

插入代码：

```cpp
static void add(Container &c, uint32_t val) {
    Data *data = new Data();
    avl_init(&data->node);
    data->val = val;

    if (!c.root) {
        c.root = &data->node;
        return;
    }

    AVLNode *cur = c.root;
    while (true) {
        AVLNode **from =
            (val < container_of(cur, Data, node)->val)
            ? &cur->left : &cur->right;
        if (!*from) {
            *from = &data->node;
            data->node.parent = cur;
            c.root = avl_fix(&data->node);
            break;
        }
        cur = *from;
    }
}
```

这展示了节点的删除过程：

```cpp
static bool del(Container &c, uint32_t val) {
    AVLNode *cur = c.root;
    while (cur) {
        uint32_t node_val = container_of(cur, Data, node)->val;
        if (val == node_val) {
            break;
        }
        cur = val < node_val ? cur->left : cur->right;
    }
    if (!cur) {
        return false;
    }

    c.root = avl_del(cur);
    delete container_of(cur, Data, node);
    return true;
}
```

这里是验证树结构正确性的函数：

```cpp
static void avl_verify(AVLNode *parent, AVLNode *node) {
    if (!node) {
        return;
    }

    assert(node->parent == parent);
    avl_verify(node, node->left);
    avl_verify(node, node->right);

    assert(node->cnt == 1 + avl_cnt(node->left) + avl_cnt(node->right));

    uint32_t l = avl_depth(node->left);
    uint32_t r = avl_depth(node->right);
    assert(l == r || l + 1 == r || l == r + 1);
    assert(node->depth == 1 + max(l, r));

    uint32_t val = container_of(node, Data, node)->val;
    if (node->left) {
        assert(node->left->parent == node);
        assert(container_of(node->left, Data, node)->val <= val);
    }
    if (node->right) {
        assert(node->right->parent == node);
        assert(container_of(node->right, Data, node)->val >= val);
    }
}
```

比较 AVL 树内容与预期数据的代码：

```cpp
static void extract(AVLNode *node, std::multiset<uint32_t> &extracted) {
    if (!node) {
        return;
    }
    extract(node->left, extracted);
    extracted.insert(container_of(node, Data, node)->val);
    extract(node->right, extracted);
}

static void container_verify(
    Container &c, const std::multiset<uint32_t> &ref)
{
    avl_verify(NULL, c.root);
    assert(avl_cnt(c.root) == ref.size());
    std::multiset<uint32_t> extracted;
    extract(c.root, extracted);
    assert(extracted == ref);
}
```

不要忘记测试后的清理工作：

```cpp
static void dispose(Container &c) {
    while (c.root) {
        AVLNode *node = c.root;
        c.root = avl_del(c.root);
        delete container_of(node, Data, node);
    }
}
```

我们开始测试用例时从简单的事情做起：

```cpp
    Container c;

    // some quick tests
    container_verify(c, {});
    add(c, 123);
    container_verify(c, {123});
    assert(!del(c, 124));
    assert(del(c, 123));
    container_verify(c, {});

    // sequential insertion
    std::multiset<uint32_t> ref;
    for (uint32_t i = 0; i < 1000; i += 3) {
        add(c, i);
        ref.insert(i);
        container_verify(c, ref);
    }
```

然后我们加入随机的操作：

```cpp
    // random insertion
    for (uint32_t i = 0; i < 100; i++) {
        uint32_t val = (uint32_t)rand() % 1000;
        add(c, val);
        ref.insert(val);
        container_verify(c, ref);
    }

    // random deletion
    for (uint32_t i = 0; i < 200; i++) {
        uint32_t val = (uint32_t)rand() % 1000;
        auto it = ref.find(val);
        if (it == ref.end()) {
            assert(!del(c, val));
        } else {
            assert(del(c, val));
            ref.erase(it);
        }
        container_verify(c, ref);
    }
```

一些更有针对性的测试。给定一个特定大小的树，在每一个可能的位置进行插入/删除操作。

```cpp
static void test_insert(uint32_t sz) {
    for (uint32_t val = 0; val < sz; ++val) {
        Container c;
        std::multiset<uint32_t> ref;
        for (uint32_t i = 0; i < sz; ++i) {
            if (i == val) {
                continue;
            }
            add(c, i);
            ref.insert(i);
        }
        container_verify(c, ref);

        add(c, val);
        ref.insert(val);
        container_verify(c, ref);
        dispose(c);
    }
}

static void test_remove(uint32_t sz) {
    for (uint32_t val = 0; val < sz; ++val) {
        Container c;
        std::multiset<uint32_t> ref;
        for (uint32_t i = 0; i < sz; ++i) {
            add(c, i);
            ref.insert(i);
        }
        container_verify(c, ref);

        assert(del(c, val));
        ref.erase(val);
        container_verify(c, ref);
        dispose(c);
    }
}
```

```cpp
    // insertion/deletion at various positions
    for (uint32_t i = 0; i < 200; ++i) {
        test_insert(i);
        test_remove(i);
    }
```

在这些测试用例的帮助下，作者在编写本章时发现并修复了一些错误。

练习：

1.  虽然我们 AVL 树的代码不多，但这个 AVL 树的实现可能不是非常高效的。我们的代码包含一些多余的指针更新，这可能是优化的一个来源。此外，我们不需要存储平衡的高度值，可以存储高度差。研究和探索高效的 AVL 树实现。

1.  你能创建更多的测试用例吗？本章中提供的测试用例可能不足以覆盖所有情况。

> +   [avl.cpp](https://build-your-own.org/redis/10/avl.cpp.htm)
> +   
> +   [test_avl.cpp](https://build-your-own.org/redis/10/test_avl.cpp.htm)
