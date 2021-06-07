# 字典：
  内存数据库的非常重要的一种结构

## 相关结构：

`typedef struct dictEntry {
    void *key;              //字段元素的键
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;                    //字段元素的值，一个共同体
    struct dictEntry *next; //由于redis的字典是使用哈希表实现的，这个指向具有相同hash值得下一个元素
} dictEntry;`

`typedef struct dictType {
    unsigned int (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;`
这个结构体能够让用户定义自己的一个hash函数、键值复制函数、键的比较函数以及键值的析构函数，不指定的情况下，使用默认情况

`typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;`
字典的哈希表，用来存储字典元素

`typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];               //使用一个长度为2的哈希表，主要用来做rehash用，在做rehash得时候，把ht[0]中的元素重新放到ht[1]中。达到重新分配大小的目的。
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */  // 用来标明已经从表0移动到表1中的表格索引
    int iterators; /* number of iterators currently running */          // 用来记录字典迭代器的数目
} dict;`

`typedef struct dictIterator {
    dict *d;
    long index;
    int table, safe;
    dictEntry *entry, *nextEntry;
    /* unsafe iterator fingerprint for misuse detection. */
    long long fingerprint;
} dictIterator;`

由于字典对于内存数据库很重要（内存数据库本身就是一个大字典），作者使用了比较多得优化手段，确保的在运行过程中由于字典的频繁操作带来的性能瓶颈：
1、哈希表空间扩张策略，在resize哈希表的时候，会根据哈希表当前的大小，扩展到下一个比当前哈希表大，而且大小是2的幂次方。空间的预分配防止频繁的空间分配操作导致性能下降；
2、字典使用哈希表实现，可以实现O(1)时间复杂度的添加和删除；
3、字典添加元素的时候使用头插法，防止添加元素还要遍历链表，降低效率；
4、渐进式哈希，在进行rehash的时候，并不是一次性完成，如果一次性完成，在字典数据量很大的时候，就会使得服务被卡住一段时间。所以作者讲rehash进行分散执行，在往字典中添加、删除、修改等操作的同时，进行一步rehash，
这样就防止了上述的问题。

rehash策略：
 1、没有进行SAVE或者BGSAVE操作的时候，当负载因子超过1的时候就会开始进行rehash
 2、进行SAVE或者BGSAVE操作的时候，负载因子超过5的时候就会进行rehash
