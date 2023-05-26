## 一：动态字符串
Redis构架了一种名为简单动态字符串（simple dynamic string,SDS）的抽象类型，作为自己的默认字符串.除了保存数据之外，还可用作缓冲区：AOF模块中的AOF缓冲区，输入缓冲区。
### SDS的定义
```c
sruct sdshdr {
	//记录buff数组中已使用的字节制度程度，不包括'\0'，即字符串程度
	int len;
	//记录未使用字节数量
	int free;
	//字节数组
	char buf[];
}
```
- 遵循C字符串空字符结尾惯例，'\0'的一字节空间不在计算范围内。
- 直接重用了一部分C字符串的函数，例如打印函数（printf）
###  SDS的特点
- C字符串：
	- 使用长度为N+1的字符数组，以'\\0'结尾，不能满足Redis对字符串安全性，效率以及功能方面的要求，字符串长度与底层字符数组空间相关联
	- 本身不记录自身的长度，获取自身长度需要遍历整个字符数组，时间复杂度O(n)。
	- 不记录自身长度还会导致容易导致缓冲区溢出![Cstring缓冲区溢出.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture/202305251026452.png)
		- 若在djccc后面插入字符串，会默认空间足够，导致覆盖别的字符串对象
	- 内存重分配次数比较多，每次增长或缩短买都会进行一次内存重分配操作。
	- 二进制不安全
- SDS：
	- 常数复杂度获取字符串长度
		- 因为结构体内有一个len属性记录了SDS的长度，每次字符串修改后都会自动修改len，这样可以O(1)复杂度获取字符串长度。
	- 杜绝缓冲区溢出
		- 结构体中有一个len和free属性，当需要修改字符串对象时，会让len与free相加，与要修改的字符串长度比较，若足够则在原本基础上修改，否者重新开辟一段内存空间，拓展对象空间。（包括空间预分配）
	- 减少修改字符串时带来的内存重分配次数
		- 增长或缩短都可能会导致内存重分配操作：
			- 情景：
				- 若增长后的长度大于底层字节数组空间，需要内存重分配
				- 若缩短后不再使用那部分空间，由于没有修改数组，空间会一直占用，会产生内存泄漏
				- 若一般情况下不太经常修改程度可以介绍，但Redis频繁修改，所以要减少内存空间重分配的次数。
				- 有空间与分配和惰性空间释放两种策略。
			- 空间预分配
				- 拓展SDS空间之前会判断未使用空间（free）是否足够，若足够，则不需要拓展。
				- 对SDS进行空间拓展的时候，不进会分配所修改必须要的空间，还会分配未使用的空间。
				- 若SDS修改后的长度（即len）将小于1MB，那么将分配与len等长的空间，即len=free，若大于等于1MB，会分配1MB未使用空间，即free=1MB。
			- 惰性空间释放
				- 缩短空间时，不会立即通过内存重分配回收未使用空间，而是会使用free来记录下来，以便在以后增长的时候利用，减少增长时导致的内存重分配，以及本次缩短导致的内存重分配。
				- 通时，SDS提供了相应的API，可以在有需要的时候释放SDS未使用的·空间，不需要担心惰性空间释放策略造成的内存浪费。
				- 另外可能会导致**内存碎片**问题，当SDS的未使用空间散布在SDS字符串的各个位置时，可能会出现无法分配连续内存块的情况，从而导致内存碎片。为了解决这个问题，Redis提供了内存碎片整理（memory defragmentation）功能，可以将SDS字符串的内容移动到连续的内存块中，从而消除内存碎片。但是，内存碎片整理需要消耗额外的时间和资源，因此需要在必要时才进行。
	- 二进制安全
		- buf是字节数组而不是字符数组，是二进制安全的。C的字符串字符数组必须符合某种二进制编码，只能保存文本文件。
		- 不是用这个数组来保存字符的，而是来保存一系列二进制数据，所以保存'\\0'等一些特殊字符都没关系，因为他是通过len而不是'\\0'来判读是否结束的，字节数组也可以保存一些特殊的数据如音频，视频等，不会对其中的数据做任何限制、过滤、或者假设，数据写入是什么样的，读取就是什么样的。
	- 兼容部分C字符串函数
		- 他在结尾用'\\0'是为了保存文本数据的那些SDS可以重用一部分<string.h>库定义的函数，例如对比函数（<string.h>/strcasecmp）、追加函数（将SD作为地个人股参数追加到C字符串后面。

## 二：链表
当一个列表键包含了数量比较多的元素或者列表中包含的元素都是比较长的字符串时，redis就会使用链表作为列表键的底层实现。 除了链表键之外，发布与订阅、慢查询、监视器等功能也用到了链表，redis服务器本身还用链表来构建客户端输出缓冲区。
链表的实现：
- 每个链表节点使用一个adlist.h/listNode结构来表示：
```c
typedef struct listNode {
	//前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值
    void *value;
} listNode;
```
- 多个listNode组成双端链表，通过adlist.h/list来持有链表
```c
typedef struct list {
	//表头节点
    listNode *head;
    //表尾节点
    listNode *tail;
    //节点复制函数
    void *(*dup)(void *ptr);
    //节点释放函数
    void (*free)(void *ptr);
    //节点对比函数
    int (*match)(void *ptr, void *key);
    //链表所包含的节点数量
    unsigned long len;
} list;
```
- Redis的链表实现的特性：
	- 双端： 有prev和next指针，获取某个节点的前置节点和后置节点的负责的均为O(1)
	- 无环：表头节点的prev和表尾结点的next均指向null，对链表的访问以NULL为终
	- 带表头和尾指针：通过list的head和tail指针，获取表头表尾复杂度尾O(1)
	- 带链表长度计数器：list里的len，对list持有节点数量进行统计。
	- 多态：链表节点使用void* 指针保存节点值，可以通过list结构中的dup、free、match三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值

## 三：字典
字典在Redis中应用非常广泛，Redis的数据库就是使用字典来作为底层实现的。此外还是哈希键的底层实现之一，当一个哈希键包含的键值对比较多或者键值对中的元素都是比较长的字符串时，Redis就会使用字典作为哈希键的底层实现。
- 字典的实现：
	Redis的字典是使用哈希表作为底层实现，哈希表里面可以有多个哈希节点，每个哈希节点保存了一个键值对。
	- 哈希表：
		- Redis字典使用的哈希表有dict.h/dictht定义
		- table是一个数组，每个元素是指向dict.h/dictEntry结构体的指针，每个dictEntry结构保存了一个键值对
		- size记录了哈希表的大小（table数组大小）
		- used记录了哈希表目前已有节点的数量
		- sizemask等于size-1，和哈希值一起决定了键应该放到table的哪个索引上
	```c
	typedef struct dictht{
		//哈希表数组
		dictEntry **table;
		//哈希表大小
		unsigned long size;
		//哈希表大小掩码，用于计算索引值
		//总是等于size-1
		unsigned long sizemask;
		//该哈希表已有的节点的数量
		unsigned long used;
	} dictht
	```
	- 哈希节点：
		- k保存键，v保存值，值可以是一个指针、uint64_t整数或int64_t整数。
		- next指向下一个哈希节点，当哈希值相同的时候，在dictht哈希数值（table）的指针进行头插法。以此来解决键冲突的问题。
	```c
	typedef struct dictEntry{
		//键
		void *key；
		//值
		union {
			void *val;
			uint63_t u64;
			int64_t s64;
		}v;
		//下一个哈希节点
		struct dictEntry *next;
	}dictEntry;
	```
	- 字典：
		- Redis中字典由dict.h/dict结构表示
		- type和privdata是为了针对不同类型的的键值对创建的多态字典而设置的。
		- type是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为不同字典设置不同类型操作函数。
		- privdata保存了需要传给那些特定函数的可选参数
		- **ht**是一个包含了两个dictht哈希表的数组，一般情况下字典只使用ht[0]，ht[1]只有在对ht[0]进行rehash的时候会使用
		- trehashidx用来标志rehash的进度，若没有进行rehash，则trehashidx为-1
		```c
		typedef struct dict{
			//
			dictType *type;
			//所有数据
			void *privdata;
			//哈希表
			dictht ht[2];
			//rehash索引
			//当rehash不在进行时，值为-1
			long trehashidx; /* rehashing not in progress if rehashidx == -1 */
		}

		typedef struct dictType {  
	    uint64_t (*hashFunction)(const void *key);  
	    void *(*keyDup)(dict *d, const void *key);  
	    void *(*valDup)(dict *d, const void *obj);  
	    int (*keyCompare)(dict *d, const void *key1, const void *key2);  
	    void (*keyDestructor)(dict *d, void *key);  
	    void (*valDestructor)(dict *d, void *obj);  
	    int (*expandAllowed)(size_t moreMem, double usedRatio);  
	    unsigned int no_value:1;  
	    unsigned int keys_are_odd:1;  
	    size_t (*dictEntryMetadataBytes)(dict *d);  
	    size_t (*dictMetadataBytes)(void);  
	    void (*afterReplaceEntry)(dict *d, dictEntry *entry);  
	} dictType;
		```
![字典数据结构.drawio.png](https://cdn.jsdelivr.net/gh/mydy930657303/djcPicture/202305261429753.png)
- 哈希算法及特点
	- 将新的键值对要插入到字典里时，根据键值对的键计算出哈希值和索引值，然后根据索引值放到哈希数表数组的指定索引上。
		- hash=hash->type->hsahFunction(key);//计算哈希值
		- index=hash & dict->ht[x].sizemask;//计算索引值
		- 当字典被用作数据库的底层实现，或者哈希键的底层实现时，Reis使用MurmurHash2算法来计算键的哈希值。
	- 哈希冲突：
		- 当两个或多个减被分到同一个索引上时，称发生了哈希冲突
		- 通过头插法，把新的节点插入到链表的表头位置。
	- rehash：
		- 随着操作的不断执行，哈希表保存的键值对会逐渐的增多或减少，为了让哈希表的负载因子维持在一个合理的范围内，当哈希表保存的键值对太多或太少时，对哈希表的大小进行相应的拓展或收缩（哈希表数组，dicht的dictENtry ** table大小）。
		- 哈希表索引过少的问题：
			- 内存利用率低：哈希表的索引空间不足时，会导致每个哈希桶（bucket）中的链表较长，增加了冲突和遍历的开销。这会占用更多的内存空间，并且影响了哈希表的性能。
			- 查询效率低：哈希表的索引空间不足时，查找特定键的效率较低。需要遍历较长的链表来找到目标键，增加了查找的时间复杂度。
		- 哈希表索引过多的问题：
			- 内存开销大：哈希表的索引空间过多会占用更多的内存空间，导致内存的浪费。
			- 内存碎片化：过多的索引空间可能会导致内存碎片化问题，使得内存的连续可用空间变少。
			- 
		- rehash步骤：
			1. 为字典的ht[1]哈希表分配空间，空间大小取决于要执行的操作以及[0]当前包含的键值对数量(即ht[0].used属性的值)
				-  若为拓展操作，这ht[1]大小为第一个大于等于ht[0].used* 2   的2^n的数
				-  若为收缩，这ht[0]大小为第一个大于等于ht[0].used   的2^n的数
			2. 将保存在ht[0]中所有键值对rehash到ht[1]上，在过程中，并没有重新分配内存或复制节点数据。节点仍然保持原有的地址和数据内容，只是将节点的指针从 `ht[0]` 桶中的位置修改为 `ht[1]` 桶中的位置。
			3. 当rt[0]包含的所有键值对转移到ht[1]之后，释放ht[1],将ht[1]设置为ht[0],并在ht[1]新创建一个空白的哈希表，为下次rehash做准备
	- 执行拓展或收缩的条件：
		- 负载因子=哈希表已保存节点数/哈希表大小
		- 拓展（满足任意一个）：
			- 服务器目前没有执行BGSAVE或者BGREWRITEAOP命令。并且哈希表的负载因子大于等于1
			- 服务器正在执行BGSAVE或者BGREWRITEAOP命令并且哈希表负载因子大于等于5
			- 原因：在执行BGSAVE或者BGREWRITEAOP命令时，Redis需要创建当前服务器进程的子进程，而大多数操作系统都采用**写时复制**技术来优化子进程的使用效率，所以提供所需的负载因子来避免进行哈希拓展操作，这样可以避免不必要的内存写入，最大限度地节约内存。
		- 当负载因子小于0.1时进行收缩操作。
		- 关于写时复制：Redis在执行BGSAVE或BGREWRITEAOF命令时，会fork出一个子进程来生成rdb文件或重写AOF文件，采用写时复制技术，父子进程共享同一片内存区域，当任一进程企图对这片内存区域进行写入操作时，会把将要写入的那一部分内存页复制一份进行写入，其他内存页依旧共享。在字典扩容期间，父进程要迁移数据，不可避免的会有大量内存写入操作，为了减少内存页过多的复制，而提高了扩容的门限，这是出于节省内存考虑的。至于为什么Redis字典收缩时不用考虑写时复制？我认为原因有两个：一是收缩条件是键值对个数小于哈希表长度的10%，有意设置的这么低，就是为了不会造成过多的页分离；二是收缩操作完成会释放一部分内存，本身目的就是节省内存的。所以两个原因综合起来，不用考虑写时复制对收缩的影响。参考[[基础/其他/写时复制|写时复制]]