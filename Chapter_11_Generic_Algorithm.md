# Chapter 11 泛型算法

### 泛型算法概述

**何为"泛型"？** 即同样的函数哦可以支持多种对象作为输入。 
**如何实现“泛型”?** 通过迭代器与容器交互，获取区间/数据。 
**泛型算法的优化：** 1. 运行速度快；2. Bug少。 
**泛型算法的性能损失：** 为了兼容不同的容器，不得不损失一定的性能，比如std::find 为了兼容性，使用了逐一遍历的线性方式查找，而std::map::find 则是使用了红黑树的中序查找法，是针对map的特殊结构的更高效的查找算法。 
**泛型算法和类方法的选择：** 如果容器包含了与泛型算法相同名称的方法，则优先使用该方法，因为一个容器如果包含现成的方法，表示这个方法比泛型算法更高效。

### 算法概述

#### 读算法
有返回值，需要给出指定区间的首尾迭代器。
##### std::accumulate
功能：在给定的区间上，对初值 T 进行累计操作，最终返回累计操作的结果。 
`T = std::accumulate(std::begin(), std::end(), T, Op)` 
其中 Op 是 optional，用于指定累计操作, 可以通过函数指针，bind 或者 lambda 表达式指定。如果不提供 Op，std::accumulate 默认按照加法操作。 
##### std::find
功能：在给定区间上，查找是否有等于 T 的元素，如果有元素等于 T, 最终返回等于T的元素的迭代器, 否则返回范围末尾迭代器。 
`Iter = std::find(std::begin(), std::end(), T)` 
其中 Iter 是一个迭代器。
##### std::count
功能：在给定区间上，查找是否有等于 T 的元素，如果有则 Val++, 最后返回Val。
`Val = std::count(std::begin(), std::end(), T)` 
其中 Val 是一个 int 型数据。
#### 写算法
只写算法，无返回值，需要给出指定区间的首尾迭代器。 
读+写算法，有返回值，需要给出读区间的首尾迭代器，以及写区间的首迭代器。
##### std::fill / std::fill_n
功能：在给定区间上，给每个元素赋予相同的值 T，无返回值。 
`std::fill(std::begin(), std::end(), T)` 
`std::fill_n(std::begin(), n, T)` 
使用 std::fill_n 时，需要注意给定区间的范围>=n。 
##### std::transform
功能：在 v1 中给定区间上[std::begin(v1), std::end(v1))，遍历所有元素，并使用 Op 进行变换，将变换的值存入 v2 当中 [std::begin(v2), Iter)，最后返回最后存入 v2 的元素的下一个位置的迭代器 Iter。 
`Iter = std::transform(std::begin(v1), std::end(v1), std::begin(v2), Op)` 
##### std::copy
功能：在 v1 中给定区间上[std::begin(v1), std::end(v1))，遍历所有元素，并将变换的值存入 v2 当中 [std::begin(v2), Iter)，最后返回最后存入 v2 的元素的下一个位置的迭代器 Iter。 
`Iter = std::copy(std::begin(v1), std::end(v1), std::begin(v2))` 
#### 排序算法
##### std::sort
功能：对给定区间上的元素进行排序（缺省为从小到大排序），无返回值。
`std::sort(iter_1, iter_2, Comp)` 
其中，Comp 用于指定比较操作, 可以通过函数指针，bind 或者 lambda 表达式指定。如果不提供 Comp，std::sort 默认按照加法操作。 
##### std::unique
功能：在给定区间上，按照给定的顺序，如果出现连续相同的元素，则删除多余的元素，将后边的元素依次向前移动，返回缩减之后的区间的末尾迭代器。
`Iter = std::unique(std::begin(), std::end())` 
通常情况下，std::unique 与 容器的 erase() 方法一起使用，以避免“越界”索引，如下：
```C++
std::vector<int> v = {1,2,2,3,4}; // v = {1,2,2,1,3,4}
auto last = std::unique(std::begin(v), std::end(v)); // v = {1,2,1,3,4,TBD}, iter 指向 4 后一个位置的地址。
v.erase(last, v.end()) // v = {1,2,1,3,4}
```
通常情况下，std::unique 与 std::sort 一起使用，以真正删除所有重复元素，如下：
```C++
std::vector<int> v = {1,2,2,3,4}; // v = {1,2,2,1,3,4}
std::sort(std::begin(v), std::end(v)); // v = {1,1,2,2,3,4}
auto last = std::unique(std::begin(v), std::end(v)); // v = {1,2,3,4,TBD, TBD}, iter 指向 4 后一个位置的地址。
v.erase(last, v.end()) // v = {1,2,3,4}
```