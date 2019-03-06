---
lsyout: post
title: <咸鱼书> 数组与指针
date: May 29, 2018 10:14 PM
excerpt: 这些内容分别对应咸鱼书第IV, IX, X章,在此处放在一起来解释.其实这还是一个老生常谈的问题, 数组与指针, 指针与数组, 傻傻分不清楚. 今天之际就彻底撕开数组与指针之间的那层小小联系吧.另外说实话,咸鱼书后面提供的程序员面试的秘密,真的太难了,看的人脑子疼.
categories:
- C/C++
tags:
- Expert_C_Programming
toc: true
comments: true
---

*数组与指针, 真的是C中, 甚至于C++中也是让人头大的问题, 脑子疼*

*但是越难的东西就越要去干他一发,不是吗?*

## 一, 声明与定义

声明与定义我们之前一直都在说,都是实际上这两个的区别还是很大的.

主要表现在以下方面:

**定义: 定义实际上是特殊的声明, 他告诉编译器有这个变量及其类型,并且为他分配空间**

**声明: 声明则是表明存在这个变量,它的定义在别处.比如extern,表示变量在别处**

我们常见的:

```cpp
extern char a[];      // 这是合法的, 表明a数组定义在别处,所以不需要提供数组长度
```

## 二, 左右值的问题

这其实也是个历史遗留问题, 很多时候我们都说过左右值,但是从没有详细的谈过

记得最早扯到左右值, 其实是在< C Primer Plus > 中的"="运算符中,其声明:

只有可修改的左值,可以放在赋值运算符的左侧,可修改的左值是C中新定义的概念

> 左值: 指的是可以获取到地址的内容, 所以编译期一定要获取到期内容

> 右值: 只需要获取到它的值即可.只有运行时才能取得值.
> 
> 所以全局变量在赋值时,要求是const experssion(常量表达式)
>
> 数组的维数也是这么要求的,不过在C99推出VLA后,这些都是过去式了.

*那么,C中可修改的左值到底是为了什么呢?*

**它的存在是为了遏制数组名赋值的滥用,后面会提到,数组名会被编译器改为首元素的地址**

**bingo! 数组名也是左值(可以获取到其地址),但它并不是可修改的左值,可修改的左值就是为处理此情况**

## 三, array与pointer的访问模式

看下面的代码:

```cpp
char a[10] = "Linux";

char *a = "Linux";

// 访问模式
char temp = a[4];
```

上面的代码中,两种类型肯定不能同时存在!

这里,我们只是来说说编译器的思维模式,如何来进行访问:

**对于数组, 取得a的地址, 然后根据下标i以及数据类型,获取偏移量, 直接拿到即可**

**对于指针, 其实只有8个字节(64位), 那么,访问时,a[4] #=> *(a+4), 先取出a中的内容**

**然后 a中的地址 + sizeof(类型) * 下标 => 地址, 取出内容**

所以说,指针访问多了一次dereference, 多访问了一次, 在现在来看是无法影响效率的.但是指针更加灵活.

## 四,交叉访问会出现什么结果?

*此处的交叉访问是我自己拟出来的一个概念, 主要是下面两种情况:*

### 1. 定义为指针, 声明为数组.

定义为指针,声明为数组,也是不少人干过的事, 会发生什么样的结果.我们先看看这张图:

上面这幅图就表明了,定义为指针,声明为数组:

**会导致,访问时, 将地址解释为数据,牛头不对马嘴, 此处将地址部分解释为ASC II 字符,显然是不正确的.

**严重情况下,修改地址, 这个指针的提领内容就永远也访问不到了**

### 2. 定义为数组, 声明为指针

同上面,先来看看图:

上面这幅图就表明了,定义为数组,声明为指针:

**会导致, 访问时, 将数组内容解释为地址,然后去这个地址提领操作, 显然也是不正确的,**

## 五, array与pointer到底何时相同,何时不同?

上面说了这么多,我们就直截了当的说清,到底什么时候数组与指针相同,什么时候指针不能等同数组!

总结如下:

**1. 数组总是可以写为指针的访问形式, 这是基于底层实现的基地址 + 偏移量的原因**

**2. 在进行函数传递参数的时候, 数组一律被转化为指针,这是为了效率考虑,而且大多数时候**

** 我们真的不需要完整的数组内容可可拷贝, 只是对其中一部分内容感兴趣**

**3. 另外一点,就是数组名表示数组首元素地址**

上面这些,就是数组与指针相同的时候,其他时候都是不同的.

**编译器看变量的时候, 一个变量就是地址, 一个指针就是地址的地址**

## 六, 实际上,为什么会发生混淆?

这个锅,要甩在出版社身上了!

众所周知, < the C Programming Language > 中提到数组与指针

> As format parementers in function definition
> (翻页)
> char s[] 
> is same as 
> char *s;

OK, 就是这么简单地一件事, 当然锅不能全甩在出版社身上了,开个玩笑而已.

## 七, 专门来说一说字符串指针与其他的指针

简单地说,字符指针是个骚东西.

**因为,它可以根据后面的NUL来标识终止.这样的话,就不需要额外的变量来控制边界访问**

比如: 

```cpp
int main(int argc, char *argv[])
```

其中便是直接通过终止字符来标识, 不然一般二维指针,是需要两个变量标识的;

另外一点就是,使用字符指针,会自动分配空间.

```cpp
char *str = "linux";     // OK

struct test *f = {3.14, ...}; // Error , 需要分配空间
```

另外对于指针数组进行赋值:

```cpp
char *str[] = {
    {"Linux"},
	{"Xiyou"},
};                     // OK

int *integer[] = {
    {1, 2, 3},
	{4, 5, 6},
};                    // Error
```

其他类型的指针数组赋值, 安心用数组名赋值好吧 (滑稽;

实际上所谓的字符串,只是一个字符指针,但是其他的类型,可都是需要分配空间的呀! ! !

## 八, 一段深刻的代码

下面这段代码,可以帮助你搞清楚,数组与指针之间的问题

```cpp
#include <stdio.h>
#include <stdlib.h>

char *str = "NieR:Automata";

void func_array(char array[])
{
    printf("%p, %p, %p\n",&array, &(array[0]), &(array[1]));
}

void func_pointer(char *ptr)
{
    printf("%p, %p, %p, %p\n",&ptr, &(ptr[0]), &(ptr[1]), ++ptr);
}

int main(void)
{
    func_array(str);
    func_pointer(str);
    printf("%p, %p, %p\n",&str[0], &(str[0]), &(str[1]));

    return 0;
}


#=> 
0x7ffdf69e1978, 0x400630, 0x400631
0x7ffdf69e1978, 0x400631, 0x400632, 0x400631
0x400630, 0x400630, 0x400631
```

我们现在来解释输出结果:

首先,因为用的是指针,所以第1,2行第1列与后面不同, 后面的地址都是str在静态区的内容

**而前面就是实际上str指针的地址, 这就是提领的意义**

另外后面,的不同,是因为操作的非原子性, 可以让睡一会儿,或者上个锁处理.

**另外,整天说传值, 传址, 实际上哪有传地址,地址其实也是值,函数参数传递就是进行值传递!**

## 九, 多维指针与Iliffc向量(指针数组)

之前很多人整天说**二维数组与二维指针相同, 头给你打烂**

实际上,,二维指针与二维数组区别很大,另外加上一个Iliffc向量(便是数组指针)

具体情况是下面这样:

- 二维数组: int a[4][5]  #=> 20x4 个字节

- 指针数组: int *a[5] #=> 5x8 个字节

- 二维指针: int **a #=> 8个字节

他们进行访问的方法也就不同了:

- 二维数组直接在基地址上进行偏移量的求取即可.

- 指针数组在一次偏移量上找到对应指针,然后提领到其他内存位置, 在进行一次偏移量求取

- 所以,二维指针进行量此提领,然后去找偏移量找到数据.

那么,在存取效率上,有什么问题?

**推荐使用数组指针, 因为数组指针可以保证字符串个数,但是对于每个字符串不限制长度**

**这样就可以大幅度减少系统中的内存开销,节省空间.**

**一般情况下,我们尽量能不使用字符串拷贝,就不拷贝,而是引用它的指针,因为拷贝的开销太大了**

**但是,从另一方面来说,锯齿状数组,即Iliffc向量,因为大小不同,会将数据存在不同的页面上**

**另一方面考虑来说, 不停的换页,会严重降低效率**

说到这里,其实让我想起来了当年小组面试,康康抛出来的一个问题:

*如何动态分配二维数组 ?*

今天我就再来说一说:

两种方法:

一.

1> 循环两次分配,先分配一维数组的i个指针的空间, 

2> 然后分配二维数组中的j个指针空间.

```cpp
// 目标分配char a[m][n]

char **a = NULL;
a = (char **)malloc(sizeof(char *) * m);
for (int i = 0; i < m; ++i)
    a[i] = (char *)malloc(sizeof(char) * n);
```

二,

另外一种办法,就是直接分配总大小的空间,然后再去分配一维数组指针,缺点是整个全部分配

优点是可以分配出连续的内存来

```cpp
char **a = NULL;
a = (char **)malloc(sizeof(char *) * m);
a[0] = (char *)malloc(sizeof(char) * m * n);
for (int i  = 0; i < m; ++i)
    a[i] = a[0] + n * i * sizeof(char);
```

一次性分配,也只是指最后一步的时候,一次性分配,前面还是一样的

切记:**多维数组的动态分配类似于递归的形式,一层一层分配进去, 一层一层释放出来,严格按照栈的顺序**

十, 数组转换为指针形式是非递归的

上面说过,数组是可以转换为指针的,切记,**这是非递归的**

**意思便是: 二维数组,会变成指针数组,而非二维指针**

所以,我们可以说,**main()的原型参数是指针数组,而并非函数指针,这样便可以减少空间开销(锯齿形数组**

***

PS: 

就说点其他的话吧, 最近真的是脑子疼,没一点劲

各种各样的恶心事,就不说了,康康刚才和我聊了一会儿

让我感受到了时间的紧迫性,

然而我整天都是在逃避,哈哈,可笑得很.整天作逼清高,就他妈一个王八蛋

玩锤子 !

好了,丧这些就够了,在丧下去就真的完了,已经菜到忘了多维数组的动态内存分配了,靠

菜菜菜,有什么办法呢?

干他妈一票大的!

明天, 不,今天开始吧.

C的内容到此为止,后面就是C++, 服务器, 内核(按重要性次序)

反正怎么说,咸鱼书后面的内容,燃我看得很害怕, 大公司面试的问题,是真的可怕,这还是20多年前的问题

真的是感受到恐惧,唉,以前就是个瓜皮,现在开始尽力弥补吧

有这样一句话:

> 生命转瞬即逝, 没时间丧      - 单读

所以,明天,不是今天开始,大干一场!

May 30, 2018 12:39 AM