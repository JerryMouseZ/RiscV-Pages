# 项目组成和执行流
## 项目组成
```
lab2
├── Makefile
├── kern
│   ├── debug
│   │   ├── assert.h
│   │   ├── kdebug.c
│   │   ├── kdebug.h
│   │   ├── kmonitor.c
│   │   ├── kmonitor.h
│   │   ├── panic.c
│   │   └── stab.h
│   ├── driver
│   │   ├── clock.c
│   │   ├── clock.h
│   │   ├── console.c
│   │   ├── console.h
│   │   ├── intr.c
│   │   └── intr.h
│   ├── init
│   │   ├── entry.S
│   │   └── init.c
│   ├── libs
│   │   └── stdio.c
│   ├── mm
│   │   ├── best_fit_pmm.c
│   │   ├── best_fit_pmm.h
│   │   ├── default_pmm.c
│   │   ├── default_pmm.h
│   │   ├── memlayout.h
│   │   ├── mmu.h
│   │   ├── pmm.c
│   │   └── pmm.h
│   ├── sync
│   │   └── sync.h
│   └── trap
│       ├── trap.c
│       ├── trap.h
│       └── trapentry.S
├── lab2.md
├── libs
│   ├── atomic.h
│   ├── defs.h
│   ├── error.h
│   ├── list.h
│   ├── printfmt.c
│   ├── readline.c
│   ├── riscv.h
│   ├── sbi.c
│   ├── sbi.h
│   ├── stdarg.h
│   ├── stdio.h
│   ├── string.c
│   └── string.h
└── tools
    ├── boot.ld
    ├── function.mk
    ├── gdbinit
    ├── grade.sh
    ├── kernel.ld
    ├── kernel_nopage.ld
    ├── sign.c
    └── vector.c

10 directories, 51 files
```
### 链表
`libs/list.h`：定义了通用双向链表结构以及相关的查找、插入等基本操作，这是建立基于链表方法的物理内存管理（以及其他内核功能）的基础。其他有类似双向链表需求的内核功能模块可直接使用 list.h 中定义的函数。

### 物理内存管理器
`kern/mm/pmm.c(h)`:物理内存管理器结构的实现，定义了内存管理相关函数。

### 默认的页面分配算法
`kern/mm/default_pmm.c(h)`: 以first fit为算法的具体的内存管理函数的实现，还包括实现的检查。

### Best Fit 页面分配算法
`kern/mm/best_fit_pmm.c(h)`: 需要自己实现的以best fit为算法的具体内存管理函数的实现。

## 执行流
1.虚拟地址的进入：在OpenSBI结束之后，OpenSBI 代码放在 [0x80000000,0x80200000) 中，内核代码放在以 0x80200000 开头的一块连续物理内存中。在进入`kern_init`之前，先分配**4KiB**内存给三级页表。我们假定内核大小不超过**1GiB**，因此通过一个大页，将虚拟地址区间 [0xffffffffc0000000,0xffffffffffffffff] 映射到物理地址区间 [0x80000000,0xc0000000)，而我们只需要分配一页内存用来存放三级页表，并将其最后一个页表项(这个虚拟地址区间明显对应三级页表的最后一个页表项)，进行适当设置,即可成功在访问虚拟地址完成虚拟地址与物理地址之间的映射，之后即可用虚拟地址设置内核栈并跳转到`kern_init`运行。

2.`kern_init`->`pmm_init`进行物理内存管理初始化->`init_pmm_manager`绑定pmm_manager的实例->`page_init`计算从kernel的内存结尾到物理内存结尾间所有内存所需的管理物理内存的结构体Page数量，并将这些结构体对应页设置为保留的，然后计算除去kernel所占内存和管理物理内存的结构体所占内存之后的空闲内存开始位置，将这个开始位置对应页面和所需的页面数作为参数传递给->` init_memmap`根据每个物理页帧的情况来建立空闲页链表->`check_alloc_page`函数调用`pmm_manager`实例的check函数检查实现的`alloc_pages`和`free_pages`的正确性。

