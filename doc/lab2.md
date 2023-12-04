# Lab 2: Memory Management
在这个实验中需要给操作系统写内存管理的代码。内存管理主要有两部分。  
第一部分是内核的物理内存申请和释放，操作的最小单位是一个物理页（4KB）。  
第二部分是虚拟内存，虚拟地址的映射以及用户程序的物理寻址。

将代码切换到lab2分支
```bash
git checkout -b lab2 origin/lab2
git merge lab1  
```
合并lab1的代码到lab2,如果出现冲突需要先解决冲突在合并。lab2新增的代码
- `inc/memlayout.h`
- `kern/pmap.c`
- `kern/pmap.h`
- `kern/kclock.h`
- `kern/kclock.c`



## Part1: Physical Page Management
操作系统必须实时跟踪哪一部分的物理RAM是空闲的和可用的。JOS是以页的粒度管理内存，所以它可以使用MMU映射和保护每一块分配的内存。  
写一个物理页分配器。在实现虚拟内存之前需要完成物理页的分配器，因为页表管理的代码需要分配物理内存存在页表中。

#### Exercise1:
> 实现 `kern/pmap.c` 中下面的函数：
>- `boot_alloc()` //  分配n bytes大小的空间，空间大小以PAGE大小向上取整。
>- `mem_init()`  (only up to the call to `check_page_free_list(1)`)
>- `page_init()` // 将可使用的内存以链表的形式连接在一起。
>- `page_alloc()`  // 分配一个空闲page
>- `page_free()`   // 与上面的功能相反，收回一个空闲页。  
>
>`check_page_free_list()` 和 `check_page_alloc()` 用于测试物理页分配器。启动JOS观察 `check_page_alloc()` 是否成功。可以添加自己的 `assert()` 验证假设是否成功。


物理内存分布：

![](./images/lab2-e1-1.drawio.png)






