`C`和`C++`有什么区别？
-------
- C++是面向对象的语言，而C是面向过程的语言；
- C++引入`new/delete`运算符，取代了C中的`malloc/free`库函数；
- C++引入引用的概念，而C中没有；
- C++引入类的概念，而C中没有；
- C++引入函数重载的特性，而C中没有

`a`和`&a`有什么区别？
-------
假设数组
```cpp
int a[10] = { 0, }; 
int (*p)[10] = &a;
```
其中：
- a是数组名，是数组首元素地址，+1表示地址值加上一个int类型的大小，如果a的值是0x00000001，加1操作后变为0x00000005。*(a + 1) = a[1]。
- &a是数组的指针，其类型为int (*)[10]（就是前面提到的数组指针），其加1时，系统会认为是数组首地址加上整个数组的偏移（10个int型变量），值为数组a尾元素后一个元素的地址。
- 若`(int *)p` ，此时输出 `*p` 时，其值为`a[0]`的值，因为被转为`int *`类型，解引用时按照int类型大小来读取。

`static`关键字有什么作用？
-------------
- 修饰局部变量时，**使得该变量在静态存储区分配内存**；只能***在首次函数调用中进行首次初始化，之后的函数调用不再进行初始化***；其生命周期与程序相同，但其作用域为局部作用域，并不能一直被访问；
- 修饰全局变量时，**使得该变量在静态存储区分配内存**；**_在声明该变量的整个文件中都是可见的_**，而在文件外是不可见的；
- 修饰函数时，在声明该函数的整个文件中都是可见的，而在文件外是不可见的，从而可以在多人协作时避免同名的函数冲突；
- 修饰成员变量时，**所有的对象都只维持一份拷贝**，可以实现不同对象间的数据共享；不需要实例化对象即可访问；**不能在类内部初始化，一般在类外部初始化，并且初始化时不加static；**
- 修饰成员函数时，**该函数不接受this指针**，只能访问类的静态成员；不需要实例化对象即可访问。

`#define`和`const`有什么区别？
-------------
- 编译器处理方式不同：**#define宏是在预处理阶段展开**，_不能对宏定义进行调试_，而**_const常量是在编译阶段使用_**；
- 类型和安全检查不同：`#define`宏没有类型，**不做任何类型检查，仅仅是代码展开**，可能产生边际效应等错误，而**const常量有具体类型，在编译阶段会执行类型检查**；
- 存储方式不同：#define宏仅仅是代码展开，在多个地方进行字符串替换，不会分配内存，存储于程序的代码段中，而const常量会分配内存，但只维持一份拷贝，存储于程序的数据段中。
- 定义域不同：#define宏不受定义域限制，而const常量只在定义域内有效。

`静态链接`和`动态链接`有什么区别？
----------
- 静态链接是在编译链接时直接将需要的执行代码拷贝到调用处；
> **优点**在于程序在发布时不需要依赖库，可以独立执行，
> 
> **缺点**在于程序的体积会相对较大，而且如果静态库更新之后，所有可执行文件需要重新链接；
- 动态链接是在编译时不直接拷贝执行代码，而是通过记录一系列符号和参数，在程序运行或加载时将这些信息传递给操作系统，操作系统负责将需要的动态库加载到内存中，然后程序在运行到指定代码时，在共享执行内存中寻找已经加载的动态库可执行代码，实现运行时链接；
> **优点**在于多个程序可以共享同一个动态库，节省资源；
> 
> **缺点**在于由于运行时加载，可能影响程序的前期执行性能。

变量的声明和定义有什么区别
----------
变量的定义为变量分配地址和存储空间， 变量的声明不分配地址。**_一个变量可以在多个地方声明， 但是只在一个地方定义_**。加入extern 修饰的是变量的声明，说明此变量将在文件以外或在文件后面部分定义。

**说明**：很多时候一个变量，只是声明**不分配内存空间**，直到具体使用时才初始化，分配内存空间， 如外部变量。
```cpp
int main() 
{
   extern int A;
   //这是个声明而不是定义，声明A是一个已经定义了的外部变量
   //注意：声明外部变量时可以把变量类型去掉如：extern A;
   dosth(); //执行函数
}
int A; //是定义，定义了A为整型的外部变量
```

简述 `#ifdef`、`#else`、`#endif`和`#ifndef` 的作用
-------------
利用`#ifdef`、`#endif`将某程序功能模块包括进去，以向特定用户提供该功能。在不需要时用户可轻易将其屏蔽。
```cpp
#ifdef MATH
#include "math.c"
#endif
```
在子程序前加上标记，以便于追踪和调试。
```cpp
#ifdef DEBUG
printf ("Indebugging......!");
#endif
```
应对硬件的限制。由于一些具体应用环境的硬件不一样，限于条件，本地缺乏这种设备，只能绕过硬件，直接写出预期结果
> **「注意」**：虽然不用条件编译命令而直接用if语句也能达到要求，但那样做目标程序长（因为所有语句都编译），运行时间长（因为在程序运行时间对if语句进行测试）。而采用条件编译，可以减少被编译的语句，从而减少目标程序的长度，减少运行时间。
