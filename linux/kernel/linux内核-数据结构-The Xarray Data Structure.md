[toc]

转自：https://kernel.taobao.org/2018/05/The-XArray-data-structure/


# The Xarray Data Structure

有时候，数据结构不符合它接口的描述，有时候反过来说，正确的数据结构它的API设计却不合理。例如在2018年linux.conf.au举办的Kernel miniconf上，Matthew Wilcox演讲提到的，内核的基数树就是后面这种情况。他的回应是，用新的使用方式，去使用旧的基数树，他称之为XArray。

他认为内核的基数树本身很棒，但各个内核子系统却都不太使用它，反而又实现了自己数据结构，做了相同的事。Matthew想转换其它一些子系统的数据结构，让各个子系统都使用基数树。过程中遇到了，他认为不应该遇到的困难。他总结了下问题所在：基数树的API太糟糕了，实际开发中根本不适用。

首先一个问题，“树”这个术语就很有迷惑性。基数树跟传统的，教科书上那种树，并不是很像。举例来说，树上的增加entry的操作，一直都被称为“插入”。但对基数树而言，“插入”并不是字面上发生的事情，尤其是当key已经存在的时候。基数树也支持“异常entry“，光是这个名字，就让用户听着不敢用了。

Wilcox决定改良接口。基数树本身不变，它本身没什么问题。改变的是接口，现在接口暗示用户，把它当做数组来用，而不是当做树来用。因为基数树看起来就像是一个自动增长的数组：一个用unsigned long来索引的指针数组。这种视图，更好地描述了基数树的用途。

基数树还要求用户自己处理锁，而XArray则默认自己处理了锁，简化了使用。基数树的“预加载”机制允许用户获取锁之前先预先分配内存，这个机制在XArray中被取消了，它太复杂又没有太多实际价值。

XArray API被分为两部分，普通API和高级API。后者给用户更多可控性，比如用户可以显式管理锁。API可以用于不同的场景，满足不同的需求。比如Page Cache就可以用XArray。普通API完全在高级API的基础上实现，所以普通API也是高级API的使用范例。

Page Cache也被改为使用XArray了，目前没有已知bug了。Matthew的计划是在4.16的合并窗口里请求合并。

# XArray API的快速浏览

截止本文，最新的XArray patchset版本是第6版https://lwn.net/Articles/744647/，于1月17日发布。其中包含了99个patch，用法可以在patchset里的文档patch里看到。

首先你需要定义一个XArray数组：

```
#include <linux/xarray.h>

DEFINE_XARRAY(array_name);
/* or */
struct xarray array;
xa_init(&array);
```

在XArray里存放一个值：

```
void *xa_store(struct xarray *xa, unsigned long index, void *entry,
               gfp_t gfp);
```

这个函数会把参数给出的entry，放到请求的index这个地方。如果要XArray需要分配内存，会使用给定的gfp来分配。如果成功，返回值是之前存放在index的值。删除一个entry可以通过在这里存放NULL来实现，或者调用

```
void *xa_erase(struct xarray *xa, unsigned long index);
```

xa_store的变体：xa_insert用于存放但不覆盖现有的entry

另一个变体：xa_cmpxchng，只有当存的值和old参数匹配上时，才会将entry存在index处。

```
void *xa_cmpxchg(struct xarray *xa, unsigned long index, void *old,
                 void *entry, gfp_t gfp);
```

xa_insert和xa_cmpxchng都会返回存入的值。

用xa_load()从XArray里取出一个值：

```
void *xa_load(struct xarray *xa, unsigned long index);
```

返回值是存放在index处的值。XArray里，空entry和存入NULL的entry是等价的。因此xa_load不会对空entry有特殊的处理。

非空entry上还可以设置最多3个比特的标签，标签管理函数：

```
void xa_set_tag(struct xarray *xa, unsigned long index, xa_tag_t tag);
void xa_clear_tag(struct xarray *xa, unsigned long index, xa_tag_t tag);
bool xa_get_tag(struct xarray *xa, unsigned long index, xa_tag_t tag);
```

tag的值是XA_TAG_0, XA_TAG_1, XA_TAG_2三者之一。

xa_set_tag用于在index处的entry上设置标签 xa_clear_tag用于清除标签 xxa_get_tag用于返回index处的entry的标签

XArray是很稀疏的，因此一个普遍的准则是，不要进行低效的遍历查找非空项。要查找多个非空项，应该使用这个宏：

```
xa_for_each(xa, entry, index, max, filter) {
    /* Process "entry" */
}
```

在进入循环之前，需要把index设为遍历的起点，max设为遍历的最大index，filter指定需要过滤的tag。

循环执行时，index会被设为当前匹配到的entry。可以在循环里修改index，来改变迭代过程。修改XArray自身也是允许的。

还有其他很多操作XArray的普通API。特殊情况下还可以使用高级API。高级API可以覆盖各种特殊情况，因此大而复杂，也是可以理解的，他比基数树的API好用很多。现在的patchset把很多的基数树的使用者改成XArray的了。没有改过来的，如果Wilcox计划顺利，以后也会改为使用XArray，基数树API则会被完全删除。



