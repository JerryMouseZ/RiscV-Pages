# lab3 练习题

## 实验要求

1. 描述FIFO页面置换算法下，一个页面从被换入到被换出的过程中，会经过代码里哪些函数/宏的处理（或者说，需要调用哪些函数/宏），并用简单的一两句话描述每个函数在过程中做了什么（**10分**，**一个函数一分**，至少正确指出10个不同的函数分别做了什么，就可以得10分。我们认为只要函数原型不同，就算两个不同的函数。要求指出对执行过程有实际影响,删去后会导致输出结果不同的函数（例如assert）而不是cprintf这样的函数。如果你选择的函数不能完整地体现”从换入到换出“的过程，比如10个函数都是页面换入的时候调用的，或者解释功能的时候只解释了这10个函数在页面换入时的功能，那么**扣2分**）

2. 

   2.1 get_pte()函数中有两段形式类似的代码， 结合sv32，sv39，sv48的异同，解释这两段代码为什么如此相像。（**2分**）

   2.2 目前get_pte()函数将页表项的查找和页表项的分配合并在一个函数里，你认为这种写法好吗？有没有必要把两个功能拆开？（**3分，无标准答案，言之成理即可**）

3. 编程：在我们给出的框架上，填写代码，实现 Clock页替换算法，比较它和FIFO算法的不同。（**15分**）
   - 注：目前这个框架还不存在，但是在下学期OS开课之前应该可以编写好。或者就采取让大家在FIFO框架上修改的方式。