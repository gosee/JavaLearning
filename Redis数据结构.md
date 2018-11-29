# Redis数据结构
## 1 简单动态字符串
简单动态字符串——simple dynamic string，SDS是redis自己构建的一种抽象类型，并将SDS用作redis的默认字符串表示。
### 1.1 SDS的定义
SDS由两部分组成：sds、sdshdr。
* sds是一个char类型的指针，指向buf数组首元素，buf数组是存储字符串的实际位置；
* sdshdr是SDS的头部，为SDS加上一个头部的好处就是为了提高某些地方的效率。
sdshdr结构体中各字段的介绍：
    * len : 已存储的字符串长度；
    * alloc : 能存储的字符串的最大容量，不包括SDS头部和结尾的NULL字符；
    * flags : 标志位，低3位代表了sds头部类型，高5位未用；
    * buf[] : 字节数组，存储字符串；buf数组尾部隐含有一个'\0'，SDS是以len字段来判断是否到达字符串末尾，而不是以'\0'判断结尾。

> 注意sdshdr5没有len和alloc字段，其flags的低3位同样代表头部类型，但高5位代表保存的字符串长度。 

![](https://github.com/gosee/photo/blob/master/20181129155200.png)

### 1.2 SDS与C字符串的区别
C语言使用长度为N+1的字符数组来表示长度为N的字符串，并且字符数组的最后一个元素总是空字符'\0'。
#### 1.2.1 获取字符串长度的复杂度
* 获取一个C字符串的长度，必须遍历整个字符串，直到遇到代表字符串结尾的空字符，复杂度为o(N)
* 获取SDS的长度，只需要访问SDS的len属性，复杂度为o(1)

#### 1.2.2 杜绝缓冲区溢出
C字符串容易造成缓冲区溢出（buffer overflow）。strcat函数可以将src字符串中的内容拼接到dest字符串的末尾。
```C
char *strcat(char *dest, const char *src);
```
如果dest字符串在内存中紧邻的空间存在另一个字符串neighbor，在执行strcat之前并没有为dest分配足够的空间，那么src的内容将被保存到原neighbor的内存空间中，从而导致neighbor保存的内容被意外的修改。
SDS的空间分配策略完全杜绝了发生缓冲区溢出的可能性：当SDS API需要对SDS进行修改时，API会先检查SDS的空间是否满足所需的要求，如果不满足的话，API会自动将空间扩展至执行修改所需的大小，然后才执行修改的操作。

```C
/* Append the specified binary-safe string pointed by 't' of 'len' bytes to the
 * end of the specified sds string 's'.
 *
 * After the call, the passed sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call. */
sds sdscatlen(sds s, const void *t, size_t len) {
    size_t curlen = sdslen(s);

    //为sds的len字段增加addlen个字节，剩余空间不足时会引起空间重新分配
    s = sdsMakeRoomFor(s,len);
    if (s == NULL) return NULL;
    //没找到实现方法，猜测是内存复制，将t复制到s的当前内存空间长度之后
    memcpy(s+curlen, t, len);
    //重新设置sds的长度
    sdssetlen(s, curlen+len);
    //最后一个字节保存空字符
    s[curlen+len] = '\0';
    return s;
}
```

#### 1.2.3 减少修改字符串时带来的内存重分配的次数
因为C字符串的长度和底层数组的长度存在关联，因此每次增长或缩短一个C字符串，程序总要对保存这个C字符串的数据进行一次内存重分配操作：
*  如果程序执行的是增长字符串的操作，比如拼接（append），那么在操作之前，程序需要通过内存重分配来扩展底层数组的空间大小。否则就会产生**缓冲区溢出**
*  如果程序执行的是缩短字符串的操作，比如截断（trim），那么在操作之后，程序需要通过内存重分配来释放字符串不再使用的那一部分空间。否则会产生**内存泄漏**

为了避免C字符串的这种缺陷，SDS通过未使用空间解除了字符串长度与底层数组长度的关联。通过未使用空间，SDS实现了空间预分配和惰性空间释放。
* **空间预分配**

空间预分配用于优化SDS的字符串增长操作：当需要将SDS的len增加addlen个字节时，如果SDS剩余空间足够，则什么都不用做。如果剩余空间不够，则会分配新的内存空间，并且采用预分配。新长度newlen为原len+addlen，若newlen小于1M，则为SDS分配新的内存大小为2*newlen；若newlen大于等于1M，则SDS分配新的内存大小为newlen  + 1M。

```C
// 为sds的len字段增加addlen个字节，剩余空间不足时会引起空间重新分配
sds sdsMakeRoomFor(sds s, size_t addlen) {
    void *sh, *newsh;
    size_t avail = sdsavail(s);
    size_t len, newlen;
    char type, oldtype = s[-1] & SDS_TYPE_MASK;
    int hdrlen;

    /* Return ASAP if there is enough space left. */
    if (avail >= addlen) return s; // sds剩余空间足够

    len = sdslen(s);
    sh = (char*)s-sdsHdrSize(oldtype);
    newlen = (len+addlen); // sds剩余空间不够，新的len为len+addlen
    
    // 下面两步实现空间预分配
    if (newlen < SDS_MAX_PREALLOC) // 新长度小于1M，则len设为2*(len+addlen)大小
        newlen *= 2;
    else
        newlen += SDS_MAX_PREALLOC; // 新长度大于1M，则len设为 len+1M 大小

    type = sdsReqType(newlen); // 新len对应的sds头部

    /* Don't use type 5: the user is appending to the string and type 5 is
     * not able to remember empty space, so sdsMakeRoomFor() must be called
     * at every appending operation. */
    if (type == SDS_TYPE_5) type = SDS_TYPE_8;

    hdrlen = sdsHdrSize(type);
    if (oldtype==type) {
        newsh = s_realloc(sh, hdrlen+newlen+1);
        if (newsh == NULL) return NULL;
        s = (char*)newsh+hdrlen;
    } else {
        /* Since the header size changes, need to move the string forward,
         * and can't use realloc */
        newsh = s_malloc(hdrlen+newlen+1);
        if (newsh == NULL) return NULL;
        memcpy((char*)newsh+hdrlen, s, len+1);
        s_free(sh);
        s = (char*)newsh+hdrlen;
        s[-1] = type;
        sdssetlen(s, len);
    }
    sdssetalloc(s, newlen);
    return s;
}
```

* **惰性空间释放**

惰性空间释放用于优化SDS的字符串缩短操作：当清空一个SDS时，并不真正释放其内存，而是设置len字段为0即可，这样当之后再次使用到该SDS时，可避免重新分配内存，从而提高效率；当要缩短SDS保存的字符串时，程序也不会重新分配内存，而是设置len字段为新的值newLen。

```C
void sdsclear(sds s) {
    sdssetlen(s, 0);
    s[0] = '\0';
}
```

#### 1.2.4 二进制安全
* C字符串中的字符必须符合某种编码，并且除了字符串的末尾之外，字符串里面不能出现空字符，这些限制使得C字符串只能保存文本数据，而不能保存像图片、音频、视频、压缩文件这样的二进制文件。
* SDS的API都是二机制安全的，所有SDS API都会以处理二进制的方式处理SDS存放在buf数组里的数据。这也是我们将SDS的buf属性成为字节数组的原因——Redis不是用这个数组来保存字符，而是用它来保存一系列二进制数据。

#### 1.2.5 兼容部分C字符串函数
* SDS遵循C字符串以空字符结尾的惯例，因此SDS可以在有需要时重用<string.h>函数库

#### 1.2.6 总结
| C字符串 | SDS |
| --- | --- |
| 获取字符串长度的复杂度为o(N) | 获取字符串长度的复杂度为o(1) |
| API是不安全的，可能会造成缓冲区溢出 | API是安全的，不会造成缓冲区溢出 |
| 修改字符串长度N次必然需要执行N次内存重分配 | 修改字符串长度N次最多需要执行N次内存重分配 |
| 只能保存文本数据 | 可以保存文本或二进制数据 |
| 可以使用所有<string.h>库中的函数 | 可以使用一部分<string.h>库中的函数 |

