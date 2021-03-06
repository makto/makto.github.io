---
layout: note
---

## 第2章 信息的表示和处理

- 二值信号能够很容易地被表示、存储和运输，相应的电子电路非常简单和可靠
- 把位组合在一起，再加上某种解释，即给不同的可能位赋予含义，我们就能够表示任何有限集合的元素
- 研究三种最重要的数字表示：无符号（unsigned），补码（two's-complement），浮点数（floating-point）
- 计算机的表示法是用有限数量的位来对一个数字编码，当结果太大以致不能表示时，某些运算就会溢出（overflow）
- C++和C使用完全相同的数字表示和运算标准
- C语言的演变：1973年第一版，89年ANSI C，90年ISO C90,99年ISO C99
- 在gcc中可以通过-std命令行选项来指定编译所依据的版本

### 2.1 信息存储
- 大多数计算机使用8位的块（字节byte）作为最小的可寻址的存储器单位（而不是访问单独的位）
- 机器级程序将存储器视为一个非常大的字节数组，即 **虚拟存储器**（virtual memory）
- 存储器的每个字节都由一个唯一的数字来标识，称为它的 **地址**（address）
- 所有可能地址的集合称为 **虚拟地址空间**（virtual address space）
- C语言中一个指针的值都是某个存储块的第一个字节的虚拟地址
- 每个程序对象都可以简单地视为一个字节块，那么程序本身就是一个字节序列

#### 十六进制表示法
- 十六进制表示位模式，既简短，又容易转化为二进制
- 用十六进制书写，一个字节的值域是00~FF
- 十进制和十六进制的相互转换，二进制和十六进制的相互转换，直接十六进制的加减法

#### 字
- 字长（word size）指明整数和指针数据的标称大小（nominal size）
- 虚拟空间以这样的一个字（word）来编码，所以字长决定的最重要系统参数就是虚拟地址空间的最大大小
- 字长为w **位**的机器，虚拟地址的范围为0~2^w-1，程序最多访问2^w个 **字节**（因为字节是最小可寻址单位）
- 32位字长的机器虚拟地址空间最大4GB

#### 数据大小
- C语言对不同数据类型的数字范围设置了下界，却没有上界
- 指针类型使用机器的全字长
- 应力图使程序在不同机器和编译器上是可移植的，其中一个方面就是使程序对不同数据类型的确切大小不敏感
    - 试用sizeof而不是一个固定的值，是向编写在不同机器类型上可移植的代码迈进了一步

#### 寻址和字节顺序
- 几乎所有机器上，多字节对象都被存储为连续的字节序列，对象的地址为所使用的字节中最小的地址
- 最低有效字节排列在最前面的方式，称为小端法（little endian）；反之成为大端法（big endian）
- 这两个术语来自《格列佛游记》，原意是指到底应该从哪一端打开半熟的鸡蛋
- 字节顺序在以下情况下会变得可见，从而产生问题：
    - 导致网络传输中出现字节反序的问题，因此必须遵守网络标准中关于字节顺序的规则，以一种共同的中间形式来传输数据
    - 阅读表示整数数据的字节序列（指令的字节级表示）时，对小端机器需要将字节反序来阅读
    - 强制类型转换时（以一种数据类型引用另一种数据类型的对象）
- [示例程序][show-bytes]

#### 表示字符串
- C语言中字符串被编码为一个以null（值为0）字符结尾的字符数组
- 十进制数字x的ASCII码正好是0x3x
- 文本数据比二进制数据具有更强的平台独立性
- 在Linux下输入`man ascii`命令，可以看到ASCII字符码表

#### 表示代码
- 不同的机器类型使用不同的且不兼容的指令和编码方式，因此二进制代码是不兼容的

#### 布尔代数简介
- 1850前后George Boole创立布尔代数，20世纪创立信息论的Claude Shannon建立了布尔代数和数字逻辑之间的联系
- 用位向量来表示有限集合，则布尔运算|和&分别对应于集合的并和交，而~对应于集合的补
- 异或运算^可交换，可结合，而且a^a=0

#### C语言中的位级运算
- |或，&与，~取反，^异或
- 能运用到任何“整型”数据类型上
- 每个元素都是它自身的加法逆元（a^a=0）
    - 据此可以实现如下的函数，来完成交换两个变量的值

        ```C
        void inplace_swap(int *x, int *y){
            *y = *x ^ *y;
            *x = *x ^ *y;
            *y = *x & *y;}
        ```
        
        [参见代码][inplace-reverse]
- 位级运算的一个常见用法就是实现掩码运算，比如掩码0xFF结合与运算就表示取出一个字的低位字节，[示例代码][mask-operation]
- 表达式~0将生成一个全1的掩码，而且可移植

#### C语言中的逻辑运算
- ||或，&&与，!非
- 跟位级运算的功能完全不同，只有在参数被限定为0或1时，才有相同行为
- 惰性求值，比如a&&5/a不会造成被0除，p&&*p++不会造成引用空指针
- 用逻辑运算和位级运算来实现x==y这个表达式的效果：`!(x^6)`

#### C语言中的移位运算
- 左移运算`<<`，在右边补0
- 右移运算`>>`
    - 逻辑右移，在左端补0
    - 算数右移，在左端补最高有效位的值
    - C标准没有明确规定，但几乎所有的编译器都对有符号数使用算数右移，对无符号数试用逻辑右移
- 移位量k应该小于数据的位数w（如果超过数据位数，则取余后再移位，即移动k%w位——但C标准对此没有保证，应避免使用）
- 移位运算可从左至右结合，即`x<<j<<k`等价于`(x<<j)<<k`，其优先级小于加法

### 2.2 整数表示
#### 整型数据类型
- 整型数据类型，表示有限范围的整数
- C标准定义了各个类型的最小的取值范围，且正负数范围对称
- 但实际实现中，不同大小所能表示的值的范围根据机器的字长和编译器而有所不同，而且取值范围不对称，负数比正数的范围大1

#### 无符号数的编码
- 整数的二进制表示即其无符号的表示
- w位无符号数表示的范围是0~2**w-1

#### 补码编码
- 最高位解释为负权
- 与无符号数的区别是，最高位的权重取反即可
- C标准并没有要求用补码来表示有符号整数
- ISO C99标准在文件stdint.h中引入了另一类整数类型，声明形如intN_t和unitN_t指定的是N位有符号和无符号整数
- 除了补码，还有反码（Ones' Complement）和原码（Sign-Magnitude）李娜高中表示有符号数的方法
- Two's Complement（补码）的命名，是因为-x的位表示是2**w-x的原码表示
- Ones' Complement的（反码）命名，是因为-x的位表示是[11..1]-x的原码表示
- 利用上面两行对定义的解读，可以方便地将有符号数的位形式转化为十进制，不用做麻烦的减法计算

#### 有符号数和无符号数之间的转换
- 同样字长的有符号数和无符号数之间相互转换，数值可能会改变，位模式不变
- 有符号数x对应的无符号值x' = X*2^w + x，其中X为x的位模式中最高位的值（0或1）
- 无符号数u对应的有符号值u' = -U*2^w + u，其中U为u的位模式中最高位的值（0或1）
- 这部分的插图真的非常棒

#### C语言中的有符号数和无符号数
- C中大多数数字都默认是有符号的，要创建一个无符号常量，必须加上后缀字符U或u，如12345U或0x1A2Bu
- 显式强制类型转换、隐式强制类型转换（通过直接赋值、printf等）的位模式都不变
- 运算中同时出现有符号和无符号数的话，会隐式地将有符号转换为无符号数
    - 如-1<0U，其值为0，因为-1转换为unsigned类型后会成为最大的无符号数
- C中的TMin32不能直接写成-2147483648，而要写成-2147483647-1，这是补码表示的不对称性带来的

#### 扩展一个数字的位表示
- 无符号数转换为更大的数据类型，需进行零扩展，即在前面加0
- 有符号数转换为更大的数据类型，需进行符号扩展，即在前面加最高位的副本
- C标准要求，先进行位数扩展，再进行符号转换

    ```C
    short sx = -12345;
    unsigned uy = sx;
    ```
    先对sx的位模式进行符号扩展，再转换为unsigned

#### 截断数字
#### 关于无符号数和有符号数的建议
- 无符号数和有符号数进行运算会进行隐式类型转换，容易导致不易发现的错误
- 无符号数减法结果为负时，会不正确地变为一个很大的正数，要注意这种错误
- 避免这种错误的一种方法就是绝不是用无符号数，但当我们不使用其数字意义时，无符号数还是非常有用的

### 2.3 整数运算

#### 无符号加法
- 无符号加法等价于计算的和再模上2^w
- C不会将溢出作为错误而发信号，但我们可以做出判断：s=x+y，如果s<x或s<y时，则发生了溢出
- 阿贝尔群（Abelian group）的概念：对无符号数的加法运算，每个无符号数都有一个加法逆元，0对应0，其他x对应2^w-x

#### 补码加法
- 两个数的w位补码之和与无符号数之和有完全相同的位级表示
- 发生负溢出时，结果为非负，值为x+y+2^w
- 发生正溢出时，结果为负，值为x+y-2^w
- 本质上就是位级运算，然后做截断
- 两个负数相加得到正数，说明发生负溢出；两个正数相加得到负数，说明发生正溢出

#### 补码的非
- 补码加上也能形成一个阿贝尔群，因此(x+y)-x=y永远成立，不管x+y是否溢出
- TMin的加法逆元为其自身
- 这样就能解释为什么TMin32在C中要写成-2147483647-1
- 有符号数的非和无符号数的非的位模式是相同的
- 补码非的简便求法有两种：
    - 对每一位求补，然后结果加1，如0101先转换为1010，再加1变为1011
    - 最右边的1的左边的所有位取非，如1100的非是0100

#### 无符号和补码的乘法
- 溢出的情况，通过截断来处理
- 虽然无符号和补码两种乘法的积的完整位表示不同，但截断后的位级表示是相同的
- 所以，w位数字上的无符号运算和补码运算都是同构的

#### 乘以常数 | 除以2的幂
- 整数乘法指令相当慢，所以编译器使用了一种重要的优化，试着用移位运算和加法运算的组合来替代乘以常数因子的乘法
- 无论是无符号数还是有符号数，表达式x<<k等价于x*2^k，即使溢出结果也一样
- 整数除法需要30个或更多的时钟周期，乘法需要10个或更多的时钟周期，加法、减法、位级运算及移位只需要1个时钟周期


### 2.4 浮点数

#### 二进制小数

### 联想
- 64位机器日益普及，于是很多之前隐藏的对字长的依赖性就会显现，变成错误，比如假设int型对象能被用来存储指针，在64位机器上会出问题（反过来也一样）
- 可以用长度为3的位向量来表示基于RGB的8种颜色，这就是8位色的名称来源？那16、32位色如何得到？
- 异或和或的相同之处是，0对应的位都不变
- 无符号数的减法利用的就是阿贝尔群的逆元概念和模运算？
- 整数除法规定向下取整，与移位运算的效果相同，这是巧合还是特意？

[show-bytes]: https://github.com/makto/codinC/blob/master/show_bytes.c
[inplace-reverse]: https://github.com/makto/codinC/blob/master/inplace_swap.c
[mask-operation]: https://github.com/makto/codinC/blob/master/mask_operation.c
