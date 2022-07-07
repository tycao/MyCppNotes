# C++基础：callback对象与回调函数

在C++中，**设计回调函数**有多种方式，主要是**可调用对象**的使用。

C++中的可调用对象有如下几种：
- 函数、
- 函数指针、
- lambda表达式、
- 仿函数、
- bind表达式。

C++中实现回调函数有
- 函数指针（传统C++）、
- 仿函数（现代C++）、
- std::function（现代C++），

依次介绍各自用法。
## 1. 函数指针
**函数指针**不同于函数，函数指针的类型取决于其***返回值和形参列表***，与函数名无关。

函数指针**本质上是一个指针，指向的是函数**。另外，函数指针也可以作为形参类型。

#### 1.1 声明函数指针：
```cpp
void (*func1)(int);
void *(*func2)(int *);
```
函数指针的声明关键是**由内而外**地阅读，上面代码中func1是一个指向函数的指针，它接受一个int型参数，返回void；func2也是一个函数指针，它接受一个int*，并返回void*。

#### 1.2 初始化函数指针：
```cpp
int add(int v1, int v2){
    return v1 + v2;
}
int (*func1)(int, int);
func1 = add_num; //或者写成func1 = &add_num;
printf("addNum:%d\n", func1(1,2)); //addNum:3
```
#### 1.3 函数指针简化声明（使用typedef或using）：
```cpp
// 1、使用typedef简化
typedef int (*FunType)(int, int);

//2、使用using简化
// using FunType = int(*)(int, int);

//3、函数指针作为形参
void testFuncPointer(FunType func1, int v1, int v2) {
    if (func1 != nullptr) {
        int result = func1(v1, v2);
        printf("testFuncPointer:%d \n",result );
    }
}

FunType func2 = add_num;
testFuncPointer(func2, 10,20); //testFuncPointer:30
```

## 2.lambda表达式
lambda表达式是内部匿名函数对象，几种常用的声明格式为：
```cpp
auto lambFunc = [捕获变量列表] (参数列表) -> 返回类型  { /*lambda函数体 */ };
auto lambFunc = [捕获变量列表] (参数列表)  { /*lambda函数体 */ };
auto lambFunc = [捕获变量列表] {  /*lambda函数体 */  };
```
**注意**：使用`auto`关键字来自动推断 lambda 表达式的返回类型。

`[]`表示lambda表达式的开始，`()`成为参数列表，与普通函数类型。其中参数列表和返回值类型可以省略。

#### 2.1 基本用法
- 一个基本的lambda表达式如下：
```cpp
auto printFunc = []() {
    printf("hello world\n");
};

//等价于
//void printFunc() {
//    printf("hello world\n");
//}

//lambda表达式调用与普通函数一样
printFunc(); 
```
- 带参数的lambda表达式：
```cpp
auto add = [] (int a, int b) {
   cout << "Sum = " << a + b;
};

  // call the lambda function
add(100, 78); //Sum = 178 
```
- 具有返回值的lambda表达式：
    - 当返回值类型是一种，声明lambda表达式可省略返回值类型
    - 当返回值类型不止一种，必须明确声明lambda表达式类型
```cpp
auto add = [] (int a, int b) {
    // always returns an 'int'
    return a + b;
};

auto operation = []  (int a, int b,  string op) -> double {
    if (op == "+") {
        // returns integer value
        return a + b;
    } 
    // returns double value
    return a - b;
};

printf("Return :%d \n", add(50,51));
printf("Returns :%f\n",operation(22,33,"+")); 
```
#### 2.2 捕获列表
lambda表达式可以使用lambda表达式域外的变量，这些变量通过捕获变量列表提供。捕获变量列表是以**逗号分隔的参数列表**，包含零个或多个捕获变量，捕获列表多种捕获方式，分别是 **_按值捕获、按引用捕获、隐式按值捕获、隐式按引用捕获_** 等。

1. 按值捕获
- 按值捕获类似于函数按值传递的形参，在lambda表达式创建时被复制到lambda表达式域内。
- 在表达式域外修改此变量时，也不影响表达式内部的值。
- lambda表达式按值捕获的外部变量，在表达式内部不能修改。
```cpp
int var1 = 100;
auto value_lambda = [var1] () {
    //var1++; //error, cannot assign to a variable captured by copy in a non-mutable lambda
    printf("value_lambda : %d\n",var1);
};
var1++;
value_lambda();				//value_lambda : 100
printf("var1:%d \n", var1); //var1:101  
```
2. 按引用捕获：按引用捕获外部变量时，需要在变量前添加&符号。
```cpp
int var2 = 23;
auto value_lambda = [&var2] () {
    var2++;
    printf("value_lambda : %d\n",var2);
};
value_lambda();     //value_lambda : 24
var2 *= 10;
printf("var2:%d \n", var2);  //var2:240  
```
3. 隐式按值捕获：按值捕获时，需要将域外的多个变量以逗号分割写在捕获列表中，而隐式按值捕获则不需要写多个捕获变量，只需在[]中写上=即可。
```cpp
float var3 = 13.0;
float var4 = 15.0;
auto value_lambda = [=]{
	return var3 * var4;
};
printf("value:%f\n", value_lambda()); //value:195.000000 
```
4. 隐式按引用捕获：与隐式按值捕获类似，隐式按引用捕获则是在[]中写上&即可。
```cpp
float var3 = 13.0;
float var4 = 15.0;
auto value_lambda = [&]{
    return var3 * var4;
};

var3 -= 10;
printf("value:%f\n", value_lambda()); //value:45.000000 
```
5. 除了上面常用的四类捕获方式外，还有混合捕获方式，即引用捕获和值捕获搭配使用。
- [] ——不捕获变量
- [var,....] ——按值或引用捕获多个变量
- [=] ——按值捕获外部所有变量
- [&] ——按引用捕获外部所有变量
- [this] ——按值捕获this指针
- [=, &var1] ——除var1变量按引用捕获外，其他变量按值捕获
- [&, =var1] ——除var1变量按值捕获外，其他变量按引用捕获
## 3.仿函数
**仿函数（英文Functor**）是一个像函数一样工作的C++类，其本质上是函数对象。仿函数的类必须重载`operator()`操作符，操作符的参数可以是多个。
- 仿函数可以存储状态
- 仿函数和函数指针的调用方式相同
```cpp
class Functor1{
public:
    void operator()(const std::string& str) {
        mStr += str;
        printf("Functor1::operator() >> %s\n", str.c_str());
    }
    std::string mStr;
};

Functor1 f;
f("hello functor");	//Functor1::operator() >> hello functor
f(" test ");		//Functor1::operator() >> hello functor test  
```
上述代码说明了如何实现functor，以及functor状态的存储。

仿函数和函数指针的相同调用方式，可以参考STL的for_each函数：
```cpp
template <typename InputIterator, typename Func>
void for_each(InputIterator first, InputIterator last, Func f) {
    while (first != last) f(*first++);
}
```
for_each函数的最后一个参数f可以是一个**函数指针**，也可以是一个重载了`operator()` 的**functor**。

## 4. bind表达式
std::bind是一个标准函数对象，是通用的函数适配器。std::bind将函数作为输入并返回一个新函数对象。
#### 4.1 使用要点
- 要点一：使用格式：
```
    auto newCallbackObj = std::bind(callbackObj, args_list);
```
表示调用newCallbackObj时，内部会调用callbackObj，并将args_list作为callbackObj的形参。
- 要点二：std::placeholders使用
使用std::bind时，可能搭配std::placeholders使用。std::placeholders是占位符，可以出现在args_list中，表示传递给callbackObj函数的参数由newCallbackObj对应的第n个参数指定。

std::placeholders::_1代表callbackObj中_1对应的参数由newCallbackObj的第一个参数指定、std::placeholders::_2代表callbackObj中_2对应的参数由newCallbackObj的第二个参数，std::placeholders::_n依次类推。

- 要点三：bind绑定类成员函数时，第一个参数表示对象的成员函数的指针，第二个参数表示对象的地址，因为对象的成员函数需要使用this指针。
- 要点四：std::bind返回值是一个新的函数对象，可以直接赋值给std::function对象。
#### 4.2 常用用法
下面简单介绍std::bind几种常用用法：

**用法一**：使用std::bind绑定一个函数。
```cpp
int add(int f, int s) {
    printf("first param :%d  second param:%d \n", f, s);
    return f + s;
}

auto addFunc = std::bind(&add, std::placeholders::_1, std::placeholders::_2);
printf("addFunc >> %d\n", addFunc(4,5));

// 输出结果
first param :4  second param:5 
addFunc >> 9
```
**addFunc** 是一个新的函数对象，等价于add。当调用addFunc时，将在内部调用add函数，_1表示add函数的第一个参数由addFunc1第一个参数指定，_2表示add函数的第二个参数由addFunc1第二个参数指定。

**用法二**：假设有一种场景，需要将第一个参数固定为15， 只允许用户设置第二个参数。
```cpp
auto addFunc2 = std::bind(&add, 15, std::placeholders::_1);
//等价于
//std::function<int(int)> addFunc2 = std::bind(&add, 15, std::placeholders::_1);
printf("addFunc2 >> %d\n",  addFunc2(5));

// 输出结果：
first param :15  second param:5 
addFunc2 >> 20
```
当调用addFunc2(5)时，内部调用add函数，add的第一个参数始终是15，`add`的**第二个参数**由`addFunc2`的**第一个参数**指定，此时是5。

**用法三**：重新排列参数顺序。
```cpp
auto addFunc4 = std::bind(&add, std::placeholders::_2, std::placeholders::_1);
//等价于
//std::function<int(int,int)> addFunc4 = std::bind(&add, std::placeholders::_2, std::placeholders::_1);
printf("addFunc4::%d\n", addFunc4(13,14));

//输出结果
first param :14  second param:13 
addFunc4::27
```
当我们调用addFunc4(13,14)时，等效于调用add(14, 13)。即`add`的**第一个参数**由`addFunc4`的**第二个参数**指定，`add`的**第二个参数**由`addFunc4`的**第一个参数**指定。

**用法四**：适配类的成员函数。
```cpp
class Test1{
public:
    void func_3(int a, int b, int c){
        printf("Test1::printNum :%d %d %d\n", a, b, c);
    }
};

Test1 t;
// printNum等价于std::function<void(int,int,int)>
auto printNum = std::bind(&Test1::func_3, &t, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3);
printNum(1,2,3);

std::function<void(int,int,int)> printNum2 = std::bind(&Test1::func_3, &t, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3);
printNum2(4,5,6);

//输出结果：
Test1::printNum :1 2 3
Test1::printNum :4 5 6
```
## 5. std::function
std::function是一个通用的函数包装器，可以存储可调用对象，如**函数指针、lambda表达式、bind表达式、仿函数，类成员函数、类成员变量**。使用`std::function`可以实现**函数回调**目的。
```cpp
//全局函数
void print_num(int i) {
    printf("print_num >> %d\n", i);
}

class Foo{
public:
    Foo(int num):mNum(num){}
    void addNum(int i) const {
        printf("Foo::addNum  >> %d\n", (mNum+i));
    }
    int mNum;
};

//仿函数：定义struct，实现operator()操作符
struct PrintNum{
    void operator() (int num) {
        printf("PrintNum::operator() >> %d\n", num);
    }
};

// 可回调对象：函数、函数指针（全局函数和类成员函数）、lamda表达式、bind表达式、仿函数（也称函数对象）
void learnSTDFunction(){
    //1. 存储全局函数
    std::function<void(int)> f_print_num = print_num;
    f_print_num(-100);

    //2. 存储lamda表达式
    std::function<void()> f_print_num_20 = [](){ print_num(20); };
    f_print_num_20();

    //3. 存储std::bind的返回结果
    std::function<void()> f_print_num_130 = std::bind(print_num, 130);
    f_print_num_130();

    //4. 存储仿函数（函数对象）
    std::function<void(int)> f_diaplay_obj = PrintNum();
    f_diaplay_obj(22);

    //5. 类成员函数，注意这里是函数的地址，告之具体指向类中的哪个函数
    std::function<void(const Foo&, int)> f_display_add = &Foo::addNum;
    const Foo foo(321);
    f_display_add(foo, 12);

    //6.包一层lamda函数调用类成员函数
    std::function<void(const Foo&, int)> f_display_add2 = [](const Foo& foo, int i){
        foo.addNum(i);
    };
    const Foo foo1(555);
    f_display_add2(foo1, 10);

    //7. 指向一个类成员函数和对象
    std::function<void(int)> s_display_add3 = std::bind(&Foo::addNum, foo1, std::placeholders::_1);
    s_display_add3(20);

    //8. 存储成员变量指针
    std::function<int(const Foo&)> s_diaplay_num = &Foo::mNum;
    printf("call to a data member accessor:%d\n", s_diaplay_num(999));
}
```

- [x] [C++基础：callback对象与回调函数](https://zhuanlan.zhihu.com/p/508260957?utm_source=wechat_session&utm_medium=social&utm_oi=1135453983547121664&s_r=0)