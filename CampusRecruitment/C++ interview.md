# 一. 基础知识
### 1.常用linux命令 
ls：列出当前目录中的文件和子目录 / pwd：显示当前工作目录的路径 / mkdir：创建新目录 / rmdir：删除空目录 / rm：删除文件或目录 / cp：复制文件或目录 / mv：移动或重命名文件或目录 / grep：通常与管道命令一起使用，对于一些命令的输出进行筛选 / cat：连接和显示文件内容 / more/less：逐页显示文本文件内容 / head/tail：显示文件的前几行或后几行 / grep：在文件中搜索指定文本 / ps：显示当前运行的进程 / kill：终止进程 /  top : 查看操作系统的信息，如进程、cpu占有率、内存信息等/ free：查看内存使用情况
### 2. 大小端
union 的成员数据共⽤内存，并且⾸地址都是低地址⾸字节。Int i= 1时：⼤端存储1放在最⾼位，⼩端存储1放在最 低位。当读取char ch时，是最低地址⾸字节，⼤⼩端会显示不同的值
```
union EndianTest { int a; char b; };
int a = 0x12345678; int *c = &a; 
c[0] == 0x12 ⼤端模式  
c[0] == 0x78 ⼩段模式 
低地址 -> 高地址
78 56 34 12(小端)
↑
ch读取这里 (0x78)
```
### 3. gdb调试命令
b 某行打断点/ r 运行程序/ c 从中断处继续执行/ n 单独执行 不进入函数 s 单步 但是进入函数 / info b 查看断点 / bt 查看栈帧 / print 打印变量  /info threads  查看线程状态 /thread  线程ID 切换到某个线程
### 4. 动态库与静态库区别
静态库 libxx.a（win libxx.lib）动态库 libxx.so(win libxx.dll）
动态库 运行时链接 ，静态库编译时 链接
静态库优点 ：发布程序无需提供静态库，移植时方便，运行时速度相对较快，缺点是可执行文件较大，消耗内存，库改变时，程序需要重新编译
动态库更加节省内存 且库改变时无需重新编译程序，但是发布程序时，得提供动态库
### 5.线程和进程 
# 二. C++八股
### 1. 智能指针
###### auto_ptr->unique_ptr：
实现独占式拥有或严格拥有概念，保证同⼀时间内只有⼀个智能指针可以指向该对象。它对于避免资源泄露特别有⽤
###### shared_ptr：
==采用引用计数实现共享式拥有概念==。多个智能指针可以指向相同对象，该对象和其相关资源会在最后一个引用被销毁时候释放。它使用引用计数来表明资源被几个指针共享。例如，赋值时，计数将加 1，而指针过期时，计数将减 1。仅当最后一个指针过期时，才调用 delete
###### weak_ptr：
用于解决 shared_ptr 的循环引用问题。当两个对象相互引用时，它们的引用计数都不会变为 0，导致内存泄漏，解决办法：把其中⼀个改为weak_ptr
weak_ptr 只是提供了对管理对象的⼀个访问⼿段。weak_ptr 设计的⽬的是为配合 shared_ptr ⽽引⼊的⼀种智 能指针来协助 shared_ptr ⼯作，它只可以从⼀个 shared_ptr 或另⼀个 weak_ptr 对象构造,，它的构造和析构不会 引起引⽤记数的增加或减少

野指针 ：野指针指向的位置是随机的不可知的 
产生原因： 1 指针变量未初始化或随便赋值  2 指针释放后未置空（内存被释放，但是指针变量中的值还在） 3 指针操作超出了变量的作用域（函数返回了局部变量的地址或者引用，因为局部变量出了作用域就被释放了）避免方法： 一定要初始化（置为nullptr），释放后置为nullptr

悬空指针：指针指向的内存空间已被释放或不再有效
```
1.这里 free 掉的是 p 的内存空间，并不是变量 p
int *p = new int(5);
cout<<"*p = "<<*p<<endl;
free(p); // p 在释放后成为悬空指针
p = NULL; // 非悬空指针

2.变量 tmp 的作用范围为最近的一层括号内，在括号外引用便超出了变量的作用范围
int *p;
{
	int tmp = 10;
	p = &tmp;
}//p 在此处成为悬空指针

3.在函数 getVal 执行完后，局部变量的内存空间会被释放，而这里 p 指向了函数内的局部变量，p 便成为了悬空指针，可以将 tmp 变为 static 的
int* getVal() {
    int tmp = 10;
    return &tmp;
}
 
int main()
{
    int *p = getVal(); //悬空指针
    cout<<"*p = "<<*p<<endl;
    return 0;
}
```
### 2. static、const、move、extern、final、override关键字
##### const
**const应用到函数**：函数内部不能改变变量，通常用于参数为指针或引⽤的情况
**const 在类中的⽤法**：const 成员变量：只在某个对象⽣命周期内是常量，⽽对于整个类⽽⾔是可以改变的，多个对象的const数据成员可以不同。所以不能在类的声明中初始化 const 数据成员， 因为类的对象在没有创建时候，编译器不知道 const 数据成员的值是什么。const 数据成员的初始化只能在类的构 造函数的初始化列表中进⾏。/  const 成员函数：const 成员函数的主要⽬的是防⽌成员函数修改对象的内容。要注 意，const 关键字和 static 关键字对于成员函数来说是不能同时使⽤的，因为 static 关键字修饰静态成员函数不含 有 this 指针，即不能实例化，const 成员函数⼜必须具体到某⼀个函数
**const myclass a**：==只能调用常量const函数==
##### static   控制变量的存储⽅式和可⻅性
修饰类：如果 C++ 中对类中的某个函数⽤ static 修饰，则表示该函数属于⼀个类⽽不是属于此类的任何 特定对象；如果对类中的某个变量进⾏ static 修饰，则表示该变量以及所有的对象所有，存储空间中只存在⼀个副本
静态成员被该类的所有成员共享
```
class A{}; sizeof(A) = 1; //空类在实例化时得到⼀个独⼀⽆⼆的地址，所以为 1. 
class A{virtual Fun(){} }; 
sizeof(A) = 4(32bit)/8(64bit) //当 C++ 类中有虚函数的时候，会有⼀ 个指向虚函数表的指针（vptr） 
class A{static int a; }; sizeof(A) = 1;
class A{int a; }; sizeof(A) = 4; 
class A{static int a; int b; }; sizeof(A) = 4;
```
##### final和override关键字
在C++中提供了对类继承和函数重写行为的额外控制。final关键字可以保护基类不被修改，防止滥用继承，而override关键字可以明确标识派生类中对基类的虚函数的重写，并进行编译时类型检查
### 3. 常量表达式
```
constexpr int N = 5; // N 变成了⼀个只读的值 
int arr[N]; // OK

constexpr int getFive(){ return 5; } 
int arr[getFive() + 1];
```
### 4. C++中有四种强制类型转换
- static_cast：明确指出类型转换，⼀般建议将隐式转换都替换成显示转换，因为没有动态类型检查，上⾏转换 （派⽣类->基类）安全，下⾏转换（基类->派⽣类） 不安全，所以主要执⾏⾮多态的转换操作； 
- dynamic_cast：专⻔⽤于派⽣类之间的转换，type-id 必须是类指针，类引⽤或 void*，对于下⾏转换是安全 的，当类型不⼀致时，转换过来的是空指针，⽽static_cast，当类型不⼀致时，转换过来的是错误意义的指针，可能造成⾮法访问等问题。 
- const_cast：专⻔⽤于 const 属性的转换，去除 const 性质，或增加 const 性质， 是四个转换符中唯⼀⼀个 可以操作常量的转换符。 
- reinterpret_cast：不到万不得已，不要使⽤这个转换符，⾼危操作。使⽤特点： 从底层对数据进⾏重新解释，依赖具体的平台，可移植性差； 可以将整形转 换为指针，也可以把指针转换为数组；可以在指针和引⽤ 之间进⾏肆⽆忌惮的转换
### 5. 内存对齐、内存泄漏
经过内存对⻬之后，CPU 的内存访问速度⼤⼤提升
有的 CPU 遇到未进⾏内存对⻬的处理直接拒绝处理，不是所有的硬件平台都能访问任 意地址上的任意数据，某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。所以内存 对⻬还有利于平台移植

内存泄漏简单的说就是申请了⼀块内存空间，使⽤完毕后没有释放掉。 它的⼀般表现⽅式是程序运⾏时间越 ⻓，占⽤内存越多，最终⽤尽全部内存，整个系统崩溃。由程序申请的⼀块内存，且没有任何⼀个指针指向它，那 么这块内存就泄漏了
Linux 中使⽤ swap 命令观察还有多少可⽤的交换空间，在⼀ 两分钟内键⼊该命令三到四次，看看可⽤的交换区是否在减少
### 6. 栈和堆的区别
栈 ：
由编译器进⾏管理，在需要时由编译器⾃动分配空间，在不需要时候⾃动回收空间，⼀般保存的是局部变量和函数参数等。 
连续的内存空间，在函数调⽤的时候，⾸先⼊栈的主函数的下⼀条可执⾏指令的地址，然后是函数的各个参数。 char * fun(char * p) {…} // 函数fun char * (pf)(char * p); // 函数指针pf pf = fun; // 函数指针pf指向函数fun pf(p); // 通过函数指针pf调⽤函数fun ⼤多数编译器中，参数是从右向左⼊栈（原因在于采⽤这种顺序，是为了让程序员在使⽤C/C++的“函数参数⻓度可 变”这个特性时更⽅便。如果是从左向右压栈，第⼀个参数（即描述可变参数表各变量类型的那个参数）将被放在 栈底，由于可变参的函数第⼀步就需要解析可变参数表的各参数类型，即第⼀步就需要得到上述参数，因此，将它 放在栈底是很不⽅便的。）本次函数调⽤结束时，局部变量先出栈，然后是参数，最后是栈顶指针最开始存放的地 址，程序由该点继续运⾏，不会产⽣碎⽚。 
栈是⾼地址向低地址扩展，栈低⾼地址，空间较⼩
*堆* ：
由程序员管理，需要⼿动 new malloc delete free 进⾏分配和回收，如果不进⾏回收的话，会造成内存泄漏的问题。 不连续的空间，实际上系统中有⼀个空闲链表，当有程序申请的时候，系统遍历空闲链表找到第⼀个⼤于等于申请 ⼤⼩的空间分配给程序，⼀般在分配程序的时候，也会空间头部写⼊内存⼤⼩，⽅便 delete 回收空间⼤⼩。当然如果有剩余的，也会将剩余的插⼊到空闲链表中，这也是产⽣内存碎⽚的原因。 
堆是低地址向⾼地址扩展，空间较⼤，较为灵活

### 7. 内联函数
### 8. 引用和指针
从编译的⻆度来讲，程序在编译时分别将指针和引⽤添加到符号表上，符号表中记录的是变量名及变量所对应地址
指针变量在符号表上对应的地址值为指针变量的地址值
引⽤在符号表上对应的地址值为引⽤对象的地址值 （与实参名字不同，地址相同）
==符号表==⽣成之后就不会再改，因此指针可以改变其指向的对象（指针变量中的值可以改），⽽引⽤对象则不能修改
### 9. C++的内存管理
栈：由编译器管理分配和回收，存放局部变量和函数参数
堆：由程序员管理，需要⼿动 new malloc delete free 进⾏分配和回收，空间较⼤，但可能会出现内存泄漏和空闲碎⽚的情况
全局/静态存储区：分为初始化和未初始化两个相邻区域，存储初始化和未初始化的全局变量和静态变量
常量存储区：存储常量，⼀般不允许修改
代码区：存放程序的⼆进制代码。
### 10. C++指针参数传递 引用参数传递

### 11. new跟malloc区别
new = malloc+初始化
```
listnode* dummynode = new listnode(0);
```
因为对于⾮内部数据类型⽽⾔，光⽤ malloc／free ⽆法满⾜动 态对象的要求。对象在创建的同时需要⾃动执⾏构造函数，对象在消亡以前要⾃动执⾏析构函数。由于 mallo／ free 是库函数⽽不是运算符，不在编译器控制权限之内，不能够把执⾏的构造函数和析构函数的任务强加于 malloc／free，所以有了 new／delete 操作符
### 12. volatile 和 extern 关键字
volatile 三个特性 
易变性：在汇编层⾯反映出来，就是两条语句，下⼀条语句不会直接使⽤上⼀条语句对应的 volatile 变量的寄存器 内容，⽽是重新从内存中读取 
不可优化性：volatile 告诉编译器，不要对我这个变量进⾏各种激进的优化，甚⾄将变量直接消除，保证程序员写 在代码中的指令，⼀定会被执⾏。 
顺序性：能够保证 volatile 变量之间的顺序性，编译器不会进⾏乱序优化。 

extern 在 C 语⾔中，修饰符 extern ⽤在变量或者函数的声明前，⽤来说明 “此变量/函数是在别处定义的，要在此处引⽤”。为啥要⽤ extern？因为⽤ extern 会加速程序的编译过程，这样能节省时间
1 extern置于变量或者函数声明前表示变量或者函数定义在别的文件中。2 extern 变量表示声明一个变量，表示该变量是全局变量，保存在静态存储区；3extern “C”的作用是是为了在C++代码中调用C语言代码

### 13. 多态、==虚函数的实现原理==
就是向不同的对象发送同⼀个消息，不同对象在接收时会产⽣不同的⾏为（即⽅法）。即⼀个接⼝，可以实现多种 ⽅法。 多态与⾮多态的实质区别就是函数地址是早绑定还是晚绑定的。如果函数的调⽤，在编译器编译期间就可以确定函 数的调⽤地址，并产⽣代码，则是静态的，即地址早绑定。⽽如果函数调⽤的地址不能在编译器期间确定，需要在 运⾏时才确定，这就属于晚绑定
多态其实⼀般就是指继承加虚函数实现的多态，对于重载来说，实际上基于的原理是，编译器为函数⽣成符号表时的不同规则，重载只是⼀种语⾔特性，与多态⽆关，与⾯向对象也⽆关，但这⼜是 C++中增加的新规则，所以也算属于 C++，所以如果⾮要说重载算是多态的⼀种，那就可以说：多态可以分为静态多态和动态多态。 
静态多态其实就是重载，因为静态多态是指在编译时期就决定了调⽤哪个函数，根据参数列表来决定； 
动态多态是指通过⼦类重写⽗类的虚函数来实现的，因为是在运⾏期间决定调⽤的函数，所以称为动态多态， ⼀般情况下我们不区分这两个时所说的多态就是指动态多态。 动态多态的实现与虚函数表，虚函数指针相关。
```C++
class Animal {
public:
    virtual void speak() { cout << "Animal sound" << endl; }
    virtual void eat() { cout << "Eating" << endl; }
    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void speak() override { cout << "Woof!" << endl; }
    void eat() override { cout << "Eating dog food" << endl; }
};

// 内存布局：
// Animal对象: [vptr] → vtable[&Animal::speak, &Animal::eat, &Animal::~Animal]
// Dog对象:    [vptr] → vtable[&Dog::speak, &Dog::eat, &Animal::~Animal]
```

### 14. 动态编译与静态编译
静态编译，编译器在编译可执⾏⽂件时，把需要⽤到的对应动态链接库中的部分提取出来，连接到可执⾏⽂件中去，使可执⾏⽂件在运⾏时不需要依赖于动态链接库；
动态编译，可执⾏⽂件需要附带⼀个动态链接库，在执⾏时，需要调⽤其对应动态链接库的命令。所以其优点⼀⽅⾯是缩⼩了执⾏⽂件本身的体积，另⼀⽅⾯是加快了编译速度，节省了系统资源。缺点是哪怕是很简单的程序，只 ⽤到了链接库的⼀两条命令，也需要附带⼀个相对庞⼤的链接库；⼆是如果其他计算机上没有安装对应的运⾏库， 则⽤动态编译的可执⾏⽂件就不能运⾏

### 15. STL ⾥  [[resize和 reserve的区别]] 
resize()：改变当前容器内含有元素的数量(size())，eg: vectorv; v.resize(len);v的size变为len,如果原来v的size⼩于 len，那么容器新增（len-size）个元素，元素的值为默认为0.当v.push_back(3);之后，则是3是放在了v的末尾，即 下标为len，此时容器是size为len+1； 
reserve()：改变当前容器的最⼤容量（capacity）,它不会⽣成元素，只是确定这个容器允许放⼊多少对象，如果 reserve(len)的值⼤于当前的capacity()，那么会重新分配⼀块能存len个对象的空间，然后把之前v.size()个对象通 过 copy construtor 复制过来，销毁之前的内存
### 16. 重载和重写，重定义的区别
重载 
翻译⾃ overload，是指同⼀可访问区内被声明的⼏个具有不同参数列表的同名函数，依赖于 C++函数名字的修饰 会将参数加在后⾯，可以是参数类型，个数，顺序的不同。==根据参数列表==决定调⽤哪个函数，重载不关⼼函数的返回类型。 
重写 
翻译⾃ override，派⽣类中重新定义⽗类中除了函数体外完全相同的虚函数，注意被重写的函数不能是 static 的， ⼀定要是虚函数，且其他⼀定要完全相同。要注意，重写和被重写的函数是在不同的类当中的，重写函数的访问修饰符是可以不同的，尽管 virtual 中是 private 的，派⽣类中重写可以改为 public。 
重定义（隐藏） 
派⽣类重新定义⽗类中相同名字的⾮ virtual 函数，参数列表和返回类型都可以不同，即⽗类中除了定义成 virtual 且完全相同的同名函数才不会被派⽣类中的同名函数所隐藏（重定义），可以理解成发生在继承中的重载
### 17. 构造函数 析构函数
```
struct ListNode {
	int val;
	ListNode *next;
	ListNode() : val(0), next(nullptr) {}//构造函数
	ListNode(int x) : val(x), next(nullptr) {}
	ListNode(int x, ListNode *next) : val(x), next(next) {}
};

class Line { 
public: 
	void setLength( double len ); 
	double getLength( void ); 
	Line(); // 这是构造函数声明 
	~Line(); // 这是析构函数声明 
private: 
	double length; 
}; 
Line::~Line(void) { 
	cout << "Object is being deleted" << endl; 
}
```
构造函数为什么⼀般不定义为虚函数：
	虚函数机制依赖于虚函数表（vtable），而构造函数的作用正是创建对象和初始化虚函数表
[[析构函数⼀般写成虚函数的原因]]
	防止内存泄漏
```C++
#include <iostream>
using namespace std;

class Base {
public:
    // ✅ 虚析构函数
    virtual ~Base() {
        cout << "Base destructor" << endl;
    }
    
    // ❌ 如果不是虚函数：
    // ~Base() {
    //     cout << "Base destructor" << endl;
    // }
};

class Derived : public Base {
public:
    ~Derived() {
        cout << "Derived destructor" << endl;
    }
};

int main() {
    Base* ptr = new Derived();
    
    delete ptr;  // 如果析构函数是虚函数：
                 // 输出: Derived destructor → Base destructor
                 
                 // 如果析构函数不是虚函数：
                 // 输出: Base destructor (内存泄漏！)
    return 0;
}
```


### 18. [[C++ STL容器]]中迭代器的作⽤，有指针为何还要迭代器

### C++duo'xian'c

# 三. 算法
### 排序
### 1. 数组
##### 双指针、滑动窗口、前缀和 
## 2. 链表
##### 虚拟头节点、反转链表、链表相交、==环形链表==
## 3. 哈希
##### set去重、map映射
利用unorderd map、set易于查找、去重等特点
##### 数组作为哈希表、set作为哈希表(数组交集)、map(key, i)
## 4. 字符串
[翻转字符串里的单词](https://leetcode.cn/problems/reverse-words-in-a-string/description/)
==KMP==
## 5. 队列
priority_queue用法（[[C++自定义比较规则]]）
```
	class Cmp {
	public:
	    bool operator()(const Node &a, const Node &b) {
	        return a.size == b.size ? a.price > b.price : a.size < b.size;
	    }
	};
    priority_queue<Node, vector<Node>, Cmp> priorityQueue;
```
滑动窗口最大值(deque)、求前 K 个高频元素(大根堆)
## 6. 二叉树
## 7. 图
#### 1. dfs
```C++
#include <iostream>
#include <vector>
using namespace std;
vector<vector<int>> result; // 收集符合条件的路径
vector<int> path; // 1节点到终点的路径

void dfs (const vector<vector<int>>& graph, int x, int n) {
    // 当前遍历的节点x 到达节点n 
    if (x == n) { // 找到符合条件的一条路径
        result.push_back(path);
        return;
    }
    for (int i = 1; i <= n; i++) { // 遍历节点x链接的所有节点
        if (graph[x][i] == 1) { // 找到 x链接的节点
            path.push_back(i); // 遍历到的节点加入到路径中来
            dfs(graph, i, n); // 进入下一层递归
            path.pop_back(); // 回溯，撤销本节点
        }
    }
}

int main() {
    int n, m, s, t;
    cin >> n >> m;

    // 节点编号从1到n，所以申请 n+1 这么大的数组
    vector<vector<int>> graph(n + 1, vector<int>(n + 1, 0));

    while (m--) {
        cin >> s >> t;
        // 使用邻接矩阵 表示无线图，1 表示 s 与 t 是相连的
        graph[s][t] = 1;
    }

    path.push_back(1); // 无论什么路径已经是从0节点出发
    dfs(graph, 1, n); // 开始遍历

    // 输出结果
    if (result.size() == 0) cout << -1 << endl;
    for (const vector<int> &pa : result) {
        for (int i = 0; i < pa.size() - 1; i++) {
            cout << pa[i] << " ";
        }
        cout << pa[pa.size() - 1]  << endl;
    }
}
```

#### 2. bfs
```C++
int dir[4][2] = {0, 1, 1, 0, -1, 0, 0, -1}; // 表示四个方向
// grid 是地图，也就是一个二维数组
// visited标记访问过的节点，不要重复访问
// x,y 表示开始搜索节点的下标
void bfs(vector<vector<char>>& grid, vector<vector<bool>>& visited, int x, int y) {
    queue<pair<int, int>> que; // 定义队列
    que.push({x, y}); // 起始节点加入队列
    visited[x][y] = true; // 只要加入队列，立刻标记为访问过的节点
    while(!que.empty()) { // 开始遍历队列里的元素
        pair<int ,int> cur = que.front(); que.pop(); // 从队列取元素
        int curx = cur.first;
        int cury = cur.second; // 当前节点坐标
        for (int i = 0; i < 4; i++) { // 开始想当前节点的四个方向左右上下去遍历
            int nextx = curx + dir[i][0];
            int nexty = cury + dir[i][1]; // 获取周边四个方向的坐标
            if (nextx < 0 || nextx >= grid.size() || nexty < 0 || 
	            nexty >= grid[0].size()) continue;  // 坐标越界了，直接跳过
            if (!visited[nextx][nexty]) { // 如果节点没被访问过
                que.push({nextx, nexty});  // 队列添加该节点为下一轮要遍历的节点
                visited[nextx][nexty] = true; // 只要加入队列立刻标记，避免重复访问
            }
        }
    }

}
```

# 四. 编译器开发经验
添加pass
首先，是编写pass，LLVM提供了多种pass类，均集成自Pass class，一般常用的是modulepass和functionpass，关于ModulePass（将整个程序看作一个单元进行处理，可以引用添加删除function）FunctionPass（可以修改和分析Function的行为，每个函数间是相互独立的，相互没有影响，还有machinefunctionpass（处理机器指令）
大致步骤如下：
1）首先，新建pass文件，把基本框架搭建起来，并在target.h里面声明，用于创建一个实例（createXXXPass），target.h里面声明initializeXXXpass函数用于初始化注册这个pass
2）再将这个pass注册到pass注册表里面，具体位置在XXXTargetMachine.cpp里面，并将pass添加至特定的阶段，比如说寄存器分配前等，指令选择前等
3）最后将该pass文件添加至cmakelists.txt文件即可
pass文件基本框架：
定义你的pass类，继承自MachineFunctionPass，写你pass类的构造函数来initializeXXXPass，然后重写MachineFunctionPass类中的虚函数runOnMachineFunction，在重写的函数中实现你pass的功能
### 1. SSA、Phi、Machine Outliner、Moudle>function>basic block、MLIR-AI编译器
### 2. llvm数据流分析
