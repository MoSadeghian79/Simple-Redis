# Redis: Mini, Simple Clone of Redis

Redis clone is a simple but powerfull database. It's a mini practical  project to sail through C programming language and some of the most important data structures including:

- AVL tree
- Difference List
- ZSet
- Heap

# Structure

## avl.h

Avl tree core description are in this file. We define depth, left, right and parent for each node. To get value for each node we use container_of definition.

```c
struct AVLNode {  
    uint32_t depth = 0;  
    uint32_t cnt = 0;  
    AVLNode *left = NULL;  
    AVLNode *right = NULL;  
    AVLNode *parent = NULL;  
};  
  
inline void avl_init(AVLNode *node) {  
    node->depth = 1;  
    node->cnt = 1;  
    node->left = node->right = node->parent = NULL;  
}  
  
AVLNode *avl_fix(AVLNode *node);  
AVLNode *avl_del(AVLNode *node);  
AVLNode *avl_offset(AVLNode *node, int64_t offset);
```

## heap.h

In our redis clone we use heap to store TTL of each key.

```c
struct HeapItem {  
    uint64_t val = 0;  
    size_t *ref = NULL;  
};  
  
void heap_update(HeapItem *a, size_t pos, size_t len);

```

## zset.h

Based on our avl tree, we build our zset which is like redis zset and is a kind of sorted set.

```c
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
  
bool zset_add(ZSet *zset, const char *name, size_t len, double score);  
ZNode *zset_lookup(ZSet *zset, const char *name, size_t len);  
ZNode *zset_pop(ZSet *zset, const char *name, size_t len);  
ZNode *zset_query(  
        ZSet *zset, double score, const char *name, size_t len, int64_t offset  
);  
void zset_dispose(ZSet *zset);  
void znode_del(ZNode *node);
```

## list.h

Difference list with an efficient O(1) concatenation operation and conversion to a linked list in time proportional to its length.

```c
struct DList {  
    DList *prev = NULL;  
    DList *next = NULL;  
};  
  
inline void dlist_init(DList *node) {  
    node->prev = node->next = node;  
}  
  
inline bool dlist_empty(DList *node) {  
    return node->next == node;  
}  
  
inline void dlist_detach(DList *node) {  
    DList *prev = node->prev;  
    DList *next = node->next;  
    prev->next = next;  
    next->prev = prev;  
}  
  
inline void dlist_insert_before(DList *target, DList *rookie) {  
    DList *prev = target->prev;  
    prev->next = rookie;  
    rookie->prev = prev;  
    rookie->next = target;  
    target->prev = rookie;  
}
```

## thread_pool.h

Represent worker pool for our database with store tasks in a queue and get them done in order.

```c
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
  
void thread_pool_init(TheadPool *tp, size_t num_threads);  
void thread_pool_queue(TheadPool *tp, void (*f)(void *), void *arg);

```
