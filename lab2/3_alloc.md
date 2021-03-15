# lab2 3/n 页面分配算法

我们在`default_pmm.c`定义了一个first fit为算法的pmm_manager类型的结构体，并实现它的接口
first_fit分配算法需要维护一个查找有序（地址按从小到大排列）空闲块（以页为最小单位的连续地址空间）的数据结构，而双向链表是一个很好的选择。为此kern/mm/memlayout.h中定义了一个` free_area_t `数据结构作为空闲双向链表的头，并根据此进行对空闲块的管理。
```c
  list_entry_t free_list;         // the list header   空闲块双向链表的头
  unsigned int nr_free;           // # of free pages in this free list  空闲块的总数（以页为单位）
```
在`default_pmm.c`中需要实现`pmm_manager`中的以下几个方法: `default_init,default_init_memmap,default_alloc_pages, default_free_pages`.
其中`init`函数主要用于初始化`memlayout.h`中提出管理空闲块的两个数据结构。
而`init_memmap`函数将根据每个物理页帧的情况来建立空闲页链表，且空闲页块应该是根据地址高低形成一个有序链表。

根据上述变量的定义，`default_init_memmap`可大致实现如下：
```c
static void
default_init_memmap(struct Page *base, size_t n) {
    assert(n > 0);
    struct Page *p = base;
    for (; p != base + n; p ++) {
        assert(PageReserved(p));
        p->flags = p->property = 0;
        set_page_ref(p, 0);
    }
    base->property = n;
    SetPageProperty(base);
    nr_free += n;
    if (list_empty(&free_list)) {
        list_add(&free_list, &(base->page_link));
    } else {
        list_entry_t* le = &free_list;
        while ((le = list_next(le)) != &free_list) {
            struct Page* page = le2page(le, page_link);
            if (base < page) {
                list_add_before(le, &(base->page_link));
                break;
            } else if (list_next(le) == &free_list) {
                list_add(le, &(base->page_link));
            }
        }
    }
}

```
如果要分配一个页，那要考虑哪些呢？这里就需要考虑实现`default_alloc_pages`函数，注意参数n表示要分配n个页。另外，需要注意实现时尽量多考虑一些边界情况，这样确保软件的鲁棒性。比如
```c
if (n > nr_free) {
return NULL;
}
```
这样可以确保分配不会超出范围。也可加一些 assert函数，在有错误出现时，能够迅速发现。比如 n应该大于0，我们就可以加上
```c
assert(n \> 0);
```
这样在n<=0的情况下，ucore会迅速报错。firstfit需要从空闲链表头开始查找最小的地址，通过list_next找到下一个空闲块元素，通过le2page宏可以更加链表元素获得对应的Page指针p。通过p->property可以了解此空闲块的大小。如果>=n，这就找到了！如果< n，则list_next，继续查找。直到list_next== &free_list，这表示找完了一遍了。找到后，就要重新组织空闲块，然后把找到的page返回。所以default_alloc_pages可大致实现如下：
```c
static struct Page *
default_alloc_pages(size_t n) {
    assert(n > 0);
    if (n > nr_free) {
        return NULL;
    }
    struct Page *page = NULL;
    list_entry_t *le = &free_list;
    while ((le = list_next(le)) != &free_list) {
        struct Page *p = le2page(le, page_link);
        if (p->property >= n) {
            page = p;
            break;
        }
    }
    if (page != NULL) {
        list_entry_t* prev = list_prev(&(page->page_link));
        list_del(&(page->page_link));
        if (page->property > n) {
            struct Page *p = page + n;
            p->property = page->property - n;
            SetPageProperty(p);
            list_add(prev, &(p->page_link));
        }
        nr_free -= n;
        ClearPageProperty(page);
    }
    return page;
}
```
`default_free_pages`函数的实现其实是`default_alloc_pages`的逆过程，不过需要考虑空闲块的合并问题。这里就不再细讲了。注意，上诉代码只是参考设计，不是完整的正确设计。更详细的说明位于lab2/kernel/mm/default_pmm.c的注释中。希望同学能够顺利完成本实验的第一部分。

看完了first fit算法，该lab需要我们自己实现best fit算法。

目前为止的代码可以在[这里](https://github.com/Liurunda/riscv64-ucore/tree/lab2/lab2)找到，遇到困难可以参考。
