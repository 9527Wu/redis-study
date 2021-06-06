sds: Simple Dynamic String

`typedef char* sds;`
结构：这里面有个小技巧，sdshdr最后一个字节数组，在实例化的时候不占用内存，但是可以作为一个hook，之后的sds都会通过这个指针获得对应的结构体的地址，同事能够做到sds与c中的字符串做兼容,sds也能够使用部分c语言的字符串函数（C语言中的字符串长度以空字符'\0'作为结束标志，而sds没有这个限制，所以中间含有空字符的sds使用一些c语字符串函数就会有问题）。
`struct sdshdr {
    unsigned int len;
    unsigned int free;
    char buf[];
};`
len字段：其中结构中的len表示存储的字符串长度，c中获取字符串长度的时间复杂度死O(n)，而这个的时间复杂度是O(1)，避免在使用过程中频繁使用获取字符串长度导致效率低；
free字段：free表示buf中可用的空间长度；
buf字段：buf后面指向保存的字符串地址。

`#define SDS_MAX_PREALLOC (1024*1024)`

`sds sdsMakeRoomFor(sds s, size_t addlen) {`
    `struct sdshdr *sh, *newsh;`
    `size_t free = sdsavail(s);`
    `size_t len, newlen;`
    `if (free >= addlen) return s;`
    `len = sdslen(s);`
    `sh = (void*) (s-(sizeof(struct sdshdr)));`
    `newlen = (len+addlen);`
    `if (newlen < SDS_MAX_PREALLOC)`
        `newlen *= 2;`
    `else`
        `newlen += SDS_MAX_PREALLOC;`
    `newsh = zrealloc(sh, sizeof(struct sdshdr)+newlen+1);`
    `if (newsh == NULL) return NULL;`
    `newsh->free = newlen - len;`
    `return newsh->buf;`
`}`
sds通过这个函数进行空间拓展，在增加addlen长度的时候，给sds进行预分配长度，总是会多分配处一定空间，防止sds频繁拓展空间每次都要重新分配空间导致性能瓶颈，根据代码，分配后的总长度是newlen，如果newlen小于1M的长度，就讲sds的字符串空间拓展成newlen*2，否则sds的字符串空间拓展成newlen+1M的大小。

sds在拼接过程中，会自动拓展空间，防止了空间溢出问题。

sds的空间回收：sds拓展了空间之后，如果拓展的空间没用，不会立即回收，使用了延迟回收，只有在回收sds或者主动调用sds空间调整接口，才会进行sds空间的回收或者调整。
