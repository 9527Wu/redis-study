# 双向链表

## 链表节点
`typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;`

## 链表迭代器
`typedef struct listIter {
    listNode *next;
    int direction;
} listIter;`

## 链表结构
`typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;`

list结构中的head、tail分别指向双向列表的头部和尾部，非环状，所以header的prev总是NULL，tail的next总是NULL；
len字段保存双向链表的长度，与sds相似的技巧，使得获取链表长度的操作时间复杂为O(1)，防止频繁的获取长度的操作造成性能问题，而且没用使用len字段记录的话，获取链表的长度耗时会随着链表的增大而不断延长；
dup、free、match函数指针可以让用户指定自己的复制、释放和匹配操作，如果用户没有指定，则用默认的操作。
