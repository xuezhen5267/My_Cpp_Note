# Chapter 11 泛型算法

## 泛型算法

### 泛型算法概述

**何为"泛型"？** 即同样的函数哦可以支持多种对象作为输入。  
**如何实现“泛型”?** 通过迭代器与容器交互，获取区间/数据。  
**泛型算法的优化：** 1. 运行速度快；2. Bug少。  
**泛型算法的性能损失：** 为了兼容不同的容器，不得不损失一定的性能，比如std::find 为了兼容性，使用了逐一遍历的线性方式查找，而std::map::find 则是使用了红黑树的中序查找法，是针对map的特殊结构的更高效的查找算法。  
**泛型算法和类方法的选择：** 如果容器包含了与泛型算法相同名称的方法，则优先使用该方法，因为一个容器如果包含现成的方法，表示这个方法比泛型算法更高效。 

### 算法概述
使用泛型算法，需要引入头文件xxx。
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
### 迭代器

大部分迭代器可以分为以下几类：

1. inputIt 输入迭代器，可读，可递增，可用于 std::find 算法
2. outputIt 输出迭代器，可写，可递增，可用于 std::copy 算法
3. fowardIt 前向迭代器，可读写，可递增，可用于 std::replace 算法
4. bidirIt 双向迭代器，可读写，可递增递减，可用于 std::reverse 算法
5. randomIt 随机访问迭代器，可读写，可增减一个整数，可用于 std::sort 算法

不同算法对最低的迭代器类别的要求可以在泛型算法的定义中看到。
越向下，迭代器的功能越全面。为什么不直接使用功能最全的迭代器？因为泛型算法可能会根据迭代器的类别引入相应的优化。我们通过容器可以定义一个迭代器，系统自动推导迭代器的类别，算法根据迭代器的类别使用不同的实现。

#### 插入迭代器
##### back_insert_iterator / back_interter
back_insert_iterator 与容器的方法 push_back() 近似等价，因此 back_insert_iterator 只能用在 可以使用push_back() 方法的容器上（比如vector，list等），用法如下：
```C++
std::vector<int> v;
std::back_insert_iterator<std::vector<int>> it(v); // 针对容器v，定义了一个 back_insert_iterator 对象，对象名称为it。
for (int i = 0; i < 10; ++i)
    it = i; // 等价于 v.push_back(i);
    // v = {1,2,3,4,5,6,7,8,9,10}
```
既然 back_insert_iterator 跟 push_back() 等价，那么为什么要搞这么个特殊 back_insert_itertor? 使用特殊迭代器来代替常规的迭代器作为泛型算法的输入，可以避免一些错误。  
比如使用 back_insert_iterator 替换 OutputIt 作为 std::fill_n 的输入，可以避免越界写入这样未定义的错误，如下：
```C++
std::vector<int> v;
std::back_insert_iterator<std::vector<int>> it(v)
std::fill_n(it, 4, 3) // v = {3,3,3,3}
std::fill_n(std::back_insert_iterator<std::vector<int>>(v), 4, 3) // 同上一句等价
std::fill_n(std::back_insert(v), 4, 3) // 同上一句等价
```
##### front_insert_iteratorn / front inserter
front_insert_iterator 与容器的方法 pop_back() 近似等价，只能用在可以使用 pop_back() 方法的容器上，比如list。
##### insert_iteratorn / inserter
front_insert_iterator 与容器的方法 insert() 近似等价，只能用在可以使用 insert() 方法的容器上，比如list。  
与 back_interter 和 front inserter 不同，inserter 的定义需要给出2个参数，分别是容器名称和容器的的一个迭代器，插入的位置就是输入的第二个参数的前一个位置，如下：
```C++
std::vector<int> v = {1,2,3};
std::insert_iterator<std::vector<int>> it(v,v.end() - 1); //inserter 需要输入两个参数
std::fill_n(it, 4, 7); // v = {1,2,7,7,7,7,3}
std::fill_n(std::inserter(v,v.end() - 1), 4, 7); // 同上一句等价
```
#### 流迭代器
为什么要引入流迭代器？通过使用流迭代器，我们可以使用泛型算法从流对象读取或者写入数据。
##### istream_iterator
TBD
##### ostream_iterator
TBD
一种用法如下：
```C++
std::vector<int> v = {1, 2, 3, 4, 5};
std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " " ));
// 打印结果为： 1 2 3 4 5
```
解析：std::copy 将元素 1，2，3，4，5 从容且 v 中 copy 到 容器 ostream 中。这个 ostream 容器的元素类型为 int，使用的流是std::cout, 每次输出后使用一个 “ ” 作为间隔符。
#### 反向迭代器
反向迭代器包括std::rend(), std::rbegin(), std::crend(), std::crbegin()。  
反向迭代器在使用的时候，注意比较大小和递增运算保持跟普通迭代器一致，即 rbegin() < rend()，使用 ++ 运算符从 rbegin() 遍历到 rend(), 如下：
```C++
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto back_iter = v.rbegin(); back_iter < v.rend(); ++back_iter)
	std::cout << *back_iter << ' ';
// 打印结果为：5 4 3 2 1
```
另外，rbegin() 指向最后一个元素，rend() 指向第一个元素的前一个位置。
#### 移动迭代器
使用移动迭代器读取元素之后，会将迭代器内的元素清空。
### 并发计算
引入并行算法的作用是更好的提升泛型算法性能，目前可用的并行计算有4个，分别是：
1. std::execution::sequenced_policy::seq 单线程，非SIMD
2. std::execution::sequenced_policy::par 多线程，非SIMD
3. std::execution::sequenced_policy::par_unseq 多线程，SIMD
4. std::execution::sequenced_policy::unseq 单线程，SIMD
SIMD 指的是 signle instruction multiple data 即使用一条指令处理多条数据，需要硬件支持。  
有些泛型算法支持并发计算(std::sort)，有些不支持。  
在泛型算法中使用并发计算策略，如下所示：
```C++
std::vector<int> v = {4,6,3,1,7} ;
std::sort(std::execution::sequenced_policy::par v.begin(), v.end()) ; // 在sort算法中使用多线程非 SIMD 的并行计算策略。
```

## bind
#### 为什么引入bind？
为什么要使用bind？bind 可以修改一个已有的可调用对象的调用接口，使得其更适合作为泛型算法的输入参数。如下所示，原本MyPredict2是需要2个输入参数的，但是给 copy_if 传递的可调用对象的应该具有一个输入，以及一个 bool 输出。实际上我们使用 bind 将函数MyPredict2 修改为只需要1个输入参数的调用对象，并用数值3固定第二个参数。
```C++
bool MyPredict(int val1, int val2)
{
    return val1 > val2;
}

int main()
{
    std::vector<int> x = {1, 2, 3, 4, 5, 6};
    std::vector<int> y;
    std::copy(x.begin(), x.end(), std::back_inserter(y), std::bind(MyPredict, std::placeholders::_1, 3));
}
// y = {4, 5, 6}
```
解析：因为 y 没有给初值以及大小，所以使用 std::back_inserter(y) 进行插入元素，以防越界访问。
#### bind 使用解析
1. 如果要修改的原函数需要 n 个输入参数，则 bind 需要输入 n+1 个参数。其中第一个参数是原函数名, 后 n 个参数用于给原函数按照顺序传递参数。
2. std::placeholders::_n 用来获取新的可调用对象的第 n 个传入参数。
```C++
bool MyPredict(int val1, int val2)
{
    std::cout << val1 << " > " << val2 << " ?" << std::endl;
    return val1 > val2;
}

int main()
{
    auto MyPredict_2 = std::bind(MyPredict, 10, std::placeholders::_3);
    MyPredict_2(7, 8, 9);
}
// 打印结果为 10 > 9 ?
```
解析：std::bind(MyPredict, 10, std::placeholders::_3) 表示，将数值10传递给原函数 MyPredict 的 val1，将 std::placeholders::_3 传递给原函数的 val2。  val1 的值确定下来了，那么用于确定 val2 的 std::placeholders::_3 的取值是什么呢？  
std::placeholders::_3 表示，将新的可调用对象 MyPredict_2 的第3个输入参数（数值9）传递给 std::placeholders::_3。

#### bind 默认使用传值的方式传递参数
```C++
void Proc(int& par)
{
    ++par;
}

int main()
{
    int x = 0;
    auto b = std::bind(Proc, x);
    b();
    std::cout << x << std::endl;
}
// 打印结果为0
```
解析：当调用对象 b 的时候，x 通过传值的方式拷贝到 b 的内部数据，(假定这个数据叫x')。x' 通过传引用的方式跟 Proc 中的 par 链接起来，Proc 中 par 执行递增之后，b 内部的数据 x' 也会递增。但是 x 并不会递增，因为 x 和 x' 是传值拷贝。

那么如何实现 bind 的传引用？使用 std::ref() 或者 std::cref(), 如下：
```C++
void Proc(int& par)
{
    ++par;
}

int main()
{
    int x = 0;
    auto b = std::bind(Proc, std::ref(x));
    b();
    std::cout << x << std::endl;
}
// 打印结果为1
```
解析：x 与 b 的内部数据 x' 是通过使用 std::ref() 来建立传引用的关系，当 x' 递增以后， x 也会递增由0变为1。
#### bind_front
TBD
## Lambda 表达式
lambda 表达式在编译器内部是通过类来实现的。
#### Lambda 表达式的简单用法
```C++
auto lam = [](int val) -> bool
{
    return val > 3;
};

std::cout << lam(5) << std::endl;
```
解析： `(int val)` 表示可调用对象 x 的参数列表。  
`-> bool` 表示显示地指定调用对象的返回类型，实际上可以不写 `-> bool` 来使编译器可以隐式地推到返回类型。需要注意的是，想让编译器自动推导返回类型，必须保持函数体内所有含有 return 语句返回的类型必须一致，如果多个 return 语句返回的类型不一致，比如一个 return 返回 double 类型数据，同一个函数体内还有另一个 retrun 返回 float 类型的数据，那么就必须显式地指定调用对象的返回类型。   
`{return val > 3;}` 表示可调用对象 x 的结构体，需要注意 {} 后有分号。
### Lambda 表达式的捕获
我们说 lambda 表达式在编译器内部是通过类来实现的, 而 lambda 表达式的捕获在编译器内部是通过定义成员变量来实现的。

如果我们想使用 Lambda 表达式来比较两个值的大小关系，可以使用 **传参** ，如下所示：
```C++
int x = 10;
auto lam = [](int val1, int val2) 
{
    return val1 > val2;
};

std::cout << lam(5, x) << std::endl;
```
也可以使用 **捕获** ,如下所示：
```C++
int x = 10;
auto lam = [x](int val1) 
{
    return val1 > x;
};

std::cout << lam(5) << std::endl;
```
解析：使用捕获，可以将传参列表长度控制在1，并将变量 x 以“成员变量”的方式定义在 Lambda 表达式内部， 这样我们就能在函数体内使用变量 x。
#### 值捕获，引用捕获，混合捕获
值捕获某些变量：`auto lam = [x, y](int val1)` 

值捕获全部变量：`auto lam = [=](int val1)` 

引用捕获某些变量：`auto lam = [&x, &y](int val1)` 

引用捕获全部变量：`auto lam = [&](int val1)` 

混合捕获某些变量：`auto lam = [&x, y](int val1)` 

很和捕获全部变量：`auto lam = [&, y](int val1)`，`auto lam = [x, =](int val1)` 
#### this 捕获
为什么要使用 this 捕获？ this 捕获用来捕获一些比较特殊的变量，比如一个变量即不是局部自动变量，也不是局部静态变量，也不是全局变量，如下：
```C++
struct Str
{
    auto fun()
    {
        int val = 3;
        auto lam = [va, x]()
        {
            return val > x;
        };
        retrun lam();
    }
    int x;
};
```
解析：上述代码编译会报错，因为变量 x 即不是局部自动变量（不在fun函数的函数体内），也不是局部静态变量（没有用 static 声明），也不是全局变量。

那么如何捕获 变量 x 呢？ 使用 this 捕获如下：
```C++
struct Str
{
    auto fun()
    {
        int val = 3;
        auto lam = [va, this]()
        {
            return val > x;
        };
        retrun lam();
    }
    int x;
};
```
解析：在某个类定义的对象的成员函数使用关键字 this 时，this 表示指向这个类定义的对象的指针。如果我们使用类 Str 定义了一个对象 s, 那么如果 s.fun() 中的使用了 this 关键字， 则 this 是一个指向 s 的指针。  
在 Lambda 表达式中使用 this 捕获，可以直接捕获整个对象（结构体），自然也能捕获到定义在内部的成员变量 x。

#### 初始化捕获（since C++14)
在捕获的 [ ] 使用初始化: `auto lam = [z = x + y](int val1)`
#### *this 捕获（since C++17)
为什么要引入 *this 捕获？ 因为 this 捕获类似于引用捕获，而捕获的对象如果被销毁了，则这个捕获是未定义的，如下：
```C++
struct Str
{
    auto fun()
    {
        int val = 3;
        auto lam = [val, this]()
        {
            return val > x;
        };
        retrun lam();
    }
    int x;
};

auto wrapper()
{
    Str s;
    return s.fun()
}

int main()
{
    auto L = wrapper();
    L();
}
```
解析: wrapper() 等价于其返回的 s.fun(), 而 s.fun() 等价于其返回的 lam()。所以 `auto L = wrapper();` 可以看作等价于 `auto L = [val, this](){retrun val > x;};`。 问题在于对象 s 是定义在 函数 wrapper() 内部的，而当主函数执行 L() 的时候，其捕获的 s 已经被销毁了，因此上述代码是危险的。

因此，C++17 引入了 *this 捕获， 即采用值捕获的方式，将整个对象拷贝到 lambda 表达式的函数体内，这样即使 this 指向的对象销毁了，也没问题。

*this 捕获的用法与 this 捕获完全一样，只是加了解引用。

