跳跃链表：平均查询和插入时间复杂度O(lgn)，性能与平衡树的性能相近，但是在实现上相较于平衡树实现难度低，是代替平衡树的一个很好的替代。所以在redis中，作者对于有序集合的实现，选择了跳跃链表。跳跃链表在redis中的应用主要体现在两方面：zset底层的实现以及在集群节点中作为内部的数据结构。

## 相关结构：
元素节点：level表示表示节点的层级，层级最多层级32，backward字段指向节点的前一个节点，forward指向节点的下一个节点。

`typedef struct zskiplistNode {
    robj *obj;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned int span;
    } level[];
} zskiplistNode;`

跳表：header不存数据，只作为一种哨兵，level存储当前的最大层级，length存储链表长度，避免频繁获取长度需要频繁遍历导致性能下降

`typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;`

有序集合：

`typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;`

元素查询范围结构：min、max存储边界的最小值和最大值，minex、maxex表示做判断的时候是否包括边界值。

`typedef struct {
    double min, max;
    int minex, maxex; /* are min or max exclusive? */
} zrangespec;`

level值获取函数：在 0~ffff之前的随机值，在四分之一范围内加一，最大是32，这个随机函数可以改善？

`#define ZSKIPLIST_P 0.25`
`int zslRandomLevel(void) {
    int level = 1;
    while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level<ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}`

可以改成下面的？这样可以一定程度上减少循环次数

`int zslRandomLevel(void) {
    int level = 1;
    while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF) && level < ZSKIPLIST_MAXLEVEL)
        level += 1;
    return level;
}`
