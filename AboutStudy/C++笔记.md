写入“endl”的效果是结束当前行，并将与设备关联的缓冲区（buffer）中的内容刷到设备中。
缓冲刷新操作可以保证到目前为止程序所产生的所有输出都真正写入输出流中，
而不是仅停留在内存中等待写入流。

- 引用传递
引用传递方式是在函数定义时在形参前面加上引用运算符“&”。
在函数被调用时，参数传递的内容不是实参的值，而是实参的地址，即将实参的地址放到C++为形参分配的内存空间中，
因此形参的任何操作都是对地址所对应的内存空间进行操作，所以都会改变相应实参的值。
在C++底层实现中，引用传递与指针是一致的。

- 默认参数
当一个函数既有定义又有声明时，形参的默认值必须在声明中指定，而不能在定义中指定。
只有当函数没有声明时，才可以在函数定义中指定形参的默认值。

- 宏
在宏展开时将对宏定义的内容进行原样替换。 
注意①：是原样替换
注意②：凡是被双引号括起来的字符，系统都不会进行宏代换，而是直接输出其中的字符，不进行任何改变。

- 函数指针
指针类型就是函数原型的类型，指针指向函数的入口地址。原型定义时：表示方法直接用*p替换方法名，由于函数指针指向
函数的入口地址，同函数名，所以可以直接将函数指针替换函数方法名。
- 指针的函数
指针与是函数返回值类型。

- 结构体
使用typedef说明一个结构体类型名后再用新类型名来定义变量
包含成员函数的结构体，可以在结构体中添加成员函数，并在其他地方，通过此结构体的对象，可调用此成员函数。

默认情况下结构体的成员是公有（public）的，而默认情况下类的成员是私有（private）的
类与结构体的区别除了使用关键字“class”和“struct”不同之外，更重要的是在成员的访问控制方面有所差异。
成员函数与一般函数定义不同的是多了类名（）和域运算符（::），它们是用来指明所定义的函数属于哪个类，
在类外定义成员函数的话这是必需的。

- 拷贝构造函数
拷贝构造函数的作用是用一个已经存在的对象来初始化该类的新对象，用户可根据需要定义拷贝构造函数，
也可由系统生成一个默认的拷贝构造函数。

- 析构函数
一般系统的默认析构（do nothing）就可以满足要求，但如果对象在完成操作前需要做内部处理，则应显式地定义析构函数

- 友元函数
友元函数非成员函数，因此不能直接引用对象成员的名字，也不能通过this指针引用对象的成员，
必须通过参数传递进来的对象名或对象指针来引用该对象的成员。
firend double distanceX(Point &p1, Point &p2);
firend double distanceY(Point *p1, Point *p2);

- 友元成员
某函数是某个类的成员函数，同时，是另一个类的友元函数，则称这个函数为友元成员。

==============================================================================================
- 友元类
友元关系是单向的，且不可传递。
- 派生类
派生继承有三种方式，需要了解。
- 多态
需要了解
- 虚函数

- 纯虚函数

- 抽象类

- 数组指针与指针数组
int (*p) [5]  数组的指针：一个指针，指向数组第0个元素的地址。由于指针类型为数组指针，且在声明时已经明确指定数组长度，那么指针步进长度为数组大小，亲测。 sizeof(array) = 20
int * p[5]    指针类型的数组：数组中每一个元素都是一个整型指针，所以每个指针步进长度为整型所占用空间。 
注：指针的步进取决于指针的类型。

- 在cpp文件中，使用///注释只能在函数内部，否则报错

- 数组、vector、array对比
  数组与array都存储在栈中，但vector在堆中。array与vector都是数组的替代品，更安全，更方便。
  eg：两个array可以直接赋值，经测试，赋值后二者地址不同，的确申请了空间。
  vector
  注：在C++与c语言中都不检测数据越界，语法上a2[-2]不会产生编译的错误，但是我使用的确出错了！！！编译器报错！！！或是debug模式编译器报错。所以需要自己关注，当然了C++提供了at方法。

TODO
关于函数实参、形参、指针等方式入栈总结。

提示：编写程序时不要忽视编译器的警告

` `


