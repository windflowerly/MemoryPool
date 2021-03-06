# MemoryPool

一个极简内存池实现(基于First Fit算法, 可扩展)

要一口气预分配大内存来管理


## Example

~~~c
#include <stdio.h>
#include "memorypool.h"

struct TAT
{
    int T_T;
};

int max_mem = 2*GB + 1000*MB + 1000*KB;
int mem_pool_size = 1*GB + 500*MB + 500*KB;

int main()
{
    MemoryPool *mp = MemoryPool_Init(max_mem, mem_pool_size, 1);
    struct TAT *tat = (struct TAT *)MemoryPool_Alloc(mp, sizeof(struct TAT));
    tat->T_T = 2333;
    printf("%d\n", tat->T_T);
    MemoryPool_Free(mp, tat);
    MemoryPool_Clear(mp);
    MemoryPool_Destroy(mp);
    return 0;
}
~~~

## API

- 预定义宏

`KB` `MB` `GB`

- 内存池

`mem_size_t` => `unsigned long long`

`MemoryPool_Init` 参数(`mem_size_t max_mem_pool_size`, `mem_size_t mempoolsize`, `int auto_extend`)

> `mempoolsize`: 内存池字节数

> `auto_extend`: 是否自动扩展 (1->开启自动扩展 0->不开启)

`MemoryPool_Alloc` 行为与系统malloc一致(唔 参数多了一个)

`MemoryPool_Free` 行为与系统free一致(唔 多了个返回值)

~~~c
MemoryPool* MemoryPool_Init(mem_size_t max_mem_pool_size, mem_size_t mempoolsize, int auto_extend);

void* MemoryPool_Alloc(MemoryPool *mp, mem_size_t wantsize);

int   MemoryPool_Free(MemoryPool *mp, void *p);

MemoryPool* MemoryPool_Clear(MemoryPool *mp);

int   MemoryPool_Destroy(MemoryPool *mp);
~~~

- 获取内存池信息

`get_mempool_usage` 获取当前内存池已使用内存比例

`get_mempool_prog_usage` 获取内存池中真实分配内存比例(除去了内存池管理结构占用的内存)

~~~c
double get_mempool_usage(MemoryPool *mp);

double get_mempool_prog_usage(MemoryPool *mp);
~~~

## Update

- 18-1-7 12.53 增加了自动扩展 (内存池耗尽时自动新扩展一个mempoolsize大小的内存)
- 18-5-27 1.10 改进输出信息 增强测试程序(详见main.cpp)
- 19-3-18 11.05 改进格式, 修复潜在bug

## Tips

- 可通过注释`test.c`里的`#include "memorypool.h"`来切换对比系统`malloc` `free`和内存池

- 暂不支持多线程

- 多食用`MemoryPool_Clear`

- 在 **2GB** 数据量 **顺序分配释放** 的情况下比系统`malloc` `free`平均快 **30%-50%** (食用`MemoryPool_Clear`效果更明显)

- `mem_size_t`使用`unsigned long long`以支持4GB以上内存管理(头文件中`MAX_MEM_SIZE`宏定义了最大可管理内存)

- 大量小块内存分配会有 **20%-30%** 内存空间损失(用于存储管理结构体 emmmmm)