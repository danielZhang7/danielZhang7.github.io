---
layout:     post
title:      PHP的static
subtitle:   浅谈 static关键字
date:       2017-07-20
author:     Zq
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - PHP
---


## 前言

了解下静态，对PHP的static做下简单的总结


## 正文

首先，在php中对static关键字的使用在

- 静态变量
- 类的静态属性，方法

### 静态变量

先由一段代码引出吧：

```objc
static $num = 0;

function checkStatic()
{
    static $num = 1;
    return $num;
}

function checkStaticGlobal()
{
    global $num;
    return $num++;
}

echo checkStatic(); //1

echo $num;  //0

echo checkStaticGlobal(); //0

echo $num;  //1
```

这里`static`作为静态变量，在`checkStatic`方法中，又重新定义了静态变量`static $num`，但是并没有对外层的`$num`产生影响，首先说明了静态变量的访问范围。
而后在`checkStaticGlobal `方法中，对`$num`使用`global`全局化，才使其全局统一，之后结果发生了改变。

实际使用中，静态变量：需要一个数据对象为整个类而非某个对象服务,同时又力求不破坏类的封装性,即要求此成员隐藏在类的内部，对外不可见。
对比auto变量，静态变量的存储方式为静态存储方式，在程序运行过程中，一直占用这些存储空间（在程序整个运行期间都不释放），也可以认为是其内存地址不变，直到整个程序运行结束。静态变量虽在程序的整个执行过程中始终存在，但是在它作 用域之外不能使用，而auto自动变量，即动态局部变量，属于动态存储类别，占动态存储空间，函数调用结束后即释放。

静态变量可以在任何可以申请的地方申请，一旦申请成功后，它将不再接受其他的同样申请。
静态变量并不是说其就不能改变值，不能改变值的量叫*常量*。 其拥有的值是可变的 ，而且它会保持最新的值。说其静态，是因为它不会随着函数的调用和退出而发生变化。即上次调用函数的时候，如果我们给静态变量赋予某个值的话，下次函数调用时，这个值保持不变。

静态局部变量的初始化表达式必须是一个常量或者常量表达式。即使局部静态变量定义时没有赋初值，系统会自动赋初值0（对数值型变量）或空字符（对字符变量）；静态变量的初始值为0。

之后我们再区分下局部静态变量、全局静态变量:
当多次调用一个函数且要求在调用之间保留某些变量的值时，可考虑采用静态局部变量。虽然用全局变量也可以达到上述目的，但全局变量有时会造成意外的副作用（主要是变量的作用域造问题成的），因此仍以采用局部静态变量为宜。【局部静态变量占用内存时间较长，并且可读性差，因此，除非必要，尽量避免使用局部静态变量。】

static全局变量(外部变量)声明，称静态全局变量，无`static`关键字则称全局变量。
全局变量本身就是静态存储方式，静态全局变量当然也是静态存储方式。

- 非静态全局变量的作用域是整个源程序，当一个源程序由多个源文件组成时，非静态的全局变量在各个源文件中都是有效的。
- 静态全局变量则限制了其作用域， 即只在定义该变量的源文件内有效，在同一源程序的其它源文件中不能使用它。

所有的全局变量都是静态变量，而局部变量只有定义时加上类型修饰符static，才为局部静态变量。
分配内存位置：
全局数据区，而不是在堆栈中分配，所以不会导致堆栈溢出

数据区分配：

- 栈 存放变量名(会有一部分的整型,浮点型,字符串存放在此处)
- 堆 存放变量值
- 全局区 存放静态变量,全局变量
- 代码区 存放要执行的函数及方法
- 文字常量区 存放常量等



### 类的静态属性，方法

看看下一段代码：

```
class parentClass
{
    static $name = 'parentName';

    public static function use_self()
    {
        echo self::$name;
        self::runOut();
        return true;
    }

    public static function use_static()
    {
        static::runOut();
        return true;
    }

    public static function runOut()
    {
        echo ' from parent';
        return true;
    }
}

class childClass extends parentClass
{
    public static function runOut()
    {
        echo ' from child';
        return true;
    }
}

var_dump(parentClass::use_self());   //parentName from parent
echo PHP_EOL;
var_dump(childClass::use_static());  // from child
```

先说说基础：

- 声明类属性或方法为静态，就可以不实例化类而直接访问。静态属性不能通过一个类已实例化的对象来访问（但静态方法可以）。
- 静态属性不可以由对象通过 -> 操作符来访问。
- 用静态方式调用一个非静态方法会导致一个 E_STRICT 级别的错误。
- 静态方法可以调用静态属性，禁止调用非静态属性。
- 静态方法内不允许`$this`调用【由于静态方法不需要通过对象即可调用，所以伪变量`$this`在静态方法中不可用。】
- 类中调用静态方法，可以用`self::`和`static::`调用

对于输出：
`self::`的调用则是调用自身的runOut方法，而`static::`调用的则是继承后的覆盖后的方法。

实际使用中，static关键字用于类的所有对象共有的变量和方法的上下文中。 因此，应该提取可以在类的多个实例之间共享的任何逻辑并将其放入静态方法中。
如果一个方法与他所在类的实例对象无关，那么它就应该是静态的，而不应该把它写成实例方法。所以所有的实例方法都与实例有关，既然与实例有关，那么创建实例就是必然的步骤。
其它文件中可以定义相同名字的函数，不会发生冲突； 

### 总结归纳

- 1.对于静态变量而言：本地化（名字冲突）、初始化=0、唯一共享性（静态区）。对于类静态成员变量：
 - （1）属于整个类，可以直接通过类名访问而不用通过实例
 - （2）必须初始化，类内static声明，类外初始化（不可以再加static）
- 2.对于类静态成员函数而言
 - （1）没有this指针，仅能访问静态成员变量和静态成员函数，不能声明为虚函数
 - （2）常用于多线程中的子类。



### 参考：

- [php中static关键字的作用](https://www.php.cn/php-weizijiaocheng-372124.html)

- [Static Function in PHP](https://www.geeksforgeeks.org/static-function-in-php/)


