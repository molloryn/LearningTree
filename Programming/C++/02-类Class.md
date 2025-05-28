# 类Class

是在C的`struct`类型基础上，增加了"成员函数"

C的`struct`可将一个概念或实体的所有属性组合在一起，描述同一类对象的共同属性

C++使得`struct`不但包含数据，还包含函数（方法）用于访问或修改类变量（对象）

PS：C++中的类更经常用`class`实现。`struct`更适合看成是一个数据结构的实现体，`class`更适合看成是一个对象的实现体。区别：[C++中struct和class的区别 - 博客园](https://www.cnblogs.com/starfire86/p/5367740.html)

```cpp
struct Date
{
    int d,m,y;
    void init(int dd,int mm, int yy){
        d=dd;m=mm;y=yy;
    }
    void print(){
        cout << y << "-" << m << "-" << d << endl;
    }
    Date& add(int dd){ //成员函数返回"自引用"（*this）
        d = d+dd;
        return *this;
        //this 是指向调用这个函数的类型对象指针
        //*this 就是调用这个函数的那个对象
        //这个成员函数返回的是"自引用"，即调用这个函数对象本身
        //通过返回自引用，可以连续调用这个函数
        //day.add(3);
        //day.add(3).add(7);
        //返回的就是那个对象，那个对象被修改，而不是复制品（引用）
    }
    Date& operator+=(int dd){
        d = d+dd;
        return *this;
    } //类中的运算符重载
}; //Date 类
int main(){
    Date day; //定义一个类Date对象day
    day.print(); //通过类Date对象day调用类Date的print方法
    day.init(4,6,1999);
    //也可以day.d = 4;之类的；
    day.add(3);
    day+=3; //使用运算符重载后,实际上是day.operator+=(3);
    day.add(3).add(5);
    (day+=3)+=5;
    day.print();
    return 0;
}
```

另外：

```cpp
struct Date
{
    int d,m,y; //若是类，这里称为成员
    Date(){ //......
    
    } //同名函数，函数重载，参数不一样就行。
    Date(int dd,int mm, int yy){
        d=dd;m=mm;y=yy;
    } //和类名一样，且没有返回类型，则称为"构造函数"
    void print(){
        cout << y << "-" << m << "-" << d << endl;
    }
    //如果想要不定义多个 而又想让它能识别无参数、一个参数、两个参数、三个参数，那么可以给它个默认值：
    Date(int dd=1,int mm=1,int yy=1900){
        d=dd;m=mm;y=yy;
    }

}; //Date 类
int main(){
    Date day2; //如果不写Date(){}那么会报错，提示没有默认构造函数
    Date day(4,6,1999); //定义一个类的变量时，会自动调用构造函数。构造函数 ，是一种特殊的方法。主要用来在创建对象时初始化对象， 即为对象成员变量赋初始值，总与new运算符一起使用在创建对象的语句中。特别的一个类可以有多个构造函数 ，可根据其参数个数的不同或参数类型的不同来区分它们 即构造函数的重载。
    day.print();
    return 0;
}
```

例子二：

```cpp
#define _CRT_SECURE_NO_WARNINGS //若是不定义这个，使用strcpy会提示不安全
#include<iostream>
#include<cstring>
struct student{
    char *name;
    int age;
    student(char *n = "no_name",int a=0){
        //不能直接 name=n;
        int len = strlen(n);
        name = new char[len+1];
        strcpy(name,n); //C语言的字符拷贝
        age=a;
        std::cout << "constructor!" << name << std::endl;
    } //这是构造函数
    virtual ~student(){ //析构函数，没有参数，前面有个波浪线 还有virtual关键字（貌似是虚函数，得另外学习学习）
        std::cout << "destructor!" << name << std::endl;
        delete[] name; //防止内存泄漏
    } //执行看看析构函数的调用顺序
};
void f()
{
    student stu1;
    student stu2("wang");
    student stu3("zhang",23);
    //函数执行完后，这些变量都会被销毁，会自动调用一个析构函数，若自己不写，自动生成的析构函数不会帮你释放被分配的内存，这样会导致内存泄漏
    std::cout << stu1.name << "\t" << stu1.age << std::endl;
    std::cout << stu2.name << "\t" << stu2.age << std::endl;
    std::cout << stu3.name << "\t" << stu3.age << std::endl;
}
int main(int argc, char const *argv[])
{
    f();
    //...
    return 0;
}
```

`class`与`struct`的区别：

`class`定义的类的成员默认都是私有的`private`，外部函数无法访问类对象成员或类成员函数

接口：`public`的公开成员（一般是成员函数）称为这个类的对外接口，外部函数只能通过这些接口访问类对象`private`等非`public`的包含内部细节，不对外公开，从而可以封装保护类对象

```cpp
#define _CRT_SECURE_NO_WARNINGS //若是不定义这个，使用strcpy会提示不安全
#include<iostream>
#include<cstring>
class student{ //如果改成class，外部函数f()会无法访问，因为成员默认都是私有的，除非加上public，这样能起到保护作用
    //struct默认是public 所以可以被外部函数访问
    private: //默认是私有的，其实可以不写
        char *name;
        int age;
    public: //若是不写，f()将无法访问构造函数
        //public里的所有成员，称作student这个类的接口（对外界公开的）
        char *get_name(){return name;}
        int get_age(){return age;}
        void set_name(char *n="new_name"){
            //这里有个问题，修改的话，会比原来的长或短，那么要释放再分配新空间；
            delete[] name;
            int len=strlen(n);
            name = new char[len+1];
            strcpy(name,n);
        }
        void set_age(int new_age){age=new_age;}
        student(char *n = "no_name",int a=0){
            //不能直接 name=n;
            int len = strlen(n);
            name = new char[len+1];
            strcpy(name,n); //C语言的字符拷贝
            age=a;
            std::cout << "constructor!" << name << std::endl;
        } //构造函数
        virtual ~student(){ //析构函数
            std::cout << "destructor!" << name << std::endl;
            delete[] name; //防止内存泄漏
        } //执行看看析构函数的调用顺序
};
void f()
{
    student stu1; //每次定义一个类的对象，若是类中没有定义构造函数，系统会自动定义一个什么都没有的构造函数 即student (){} 所以若是class类，不改为public，将无法调用构造函数
    student stu2("wang");
    student stu3("zhang",23);
    //std::cout << stu1.name << "\t" << stu1.age << std::endl;
    //若是继续使用stu1.name来访问成员，是不行的，因为name和age都是private成员
    //想要访问和修改，都需自己再定义一个函数来获得
    std::cout << stu2.get_name() << "\t" << stu2.get_age() << std::endl;
    std::cout << stu3.get_name() << "\t" << stu3.get_age() << std::endl;
    //stu1.age=15 //这样会报错，因为是private成员
    stu1.set_age(20);
    stu1.set_name("Li");
}
int main(int argc, char const *argv[])
{
    f();
    return 0;
}
```

## 拷贝

拷贝构造函数、赋值运算符（函数）

拷贝构造函数：定义一个类对象时使用同类型的另外对象初始化

赋值运算符：一个对象赋值给另外一个对象

```cpp
student s;
//...对s的各种操作
student m(s); //这就是拷贝构造函数（m是新对象名称）
```

或

```cpp
student k;
k = s; //这就是赋值运算符
```

大部分情况编译器自行生成的拷贝构造函数、赋值运算符函数没有问题，但有时也会出现一些状况

```cpp
#include <iostream>
#include <cstdlib>
using namespace std;
struct student{
    char *name;
    int age;
    student (char *n = "name",int a = 0){
        name = new char[100];
        strcpy(name,n);
        age = a;
    }
    virtual ~student(){
        delete[] name;
    }
};
int main(int argc, char const *argv[])
{
    student s;
    student m(s); //（硬拷贝）直接调用编译器自动生成的拷贝构造函数可能会出错
    cout << s.name << ", age" << s.age << endl << endl;
    cout << m.name << ", age" << m.age << endl << endl;
    //若使用编译器自动生成的拷贝构造函数，则会出错，因为这种拷贝是硬拷贝，直接让m.name = s.name(地址给它了)所以此时m.name 和s.name指向的是同一块内存
    //此时调用析构函数时，delete[] m.name;后delete[] s.name 释放了两次同一块内存，因此报错

    student k("John",26);
    s = k; //也是直接s.name = k.name 出现与上面的情况一样
    cout << k.name << ", age " << k.age << endl << endl;

    return 0;
}

//因此我们要定义自己的拷贝构造函数或是赋值运算符

struct student{
    char *name;
    int age;
    student (char *n = "name",int a = 0){
        name = new char[100];
        strcpy(name,n);
        age = a;
    }
    student (const student &s){
        //name = s.name//这叫硬拷贝
        name = new char[100];
        strcpy(name,s.name);
        age = s.age;
        cout << "拷贝构造函数" << endl;
    }
    student& operator= (const student &s){
        //name = s.name//这叫硬拷贝
        name = new char[100];
        strcpy(name,s.name);
        age = s.age;
        cout << "赋值运算符" << endl;
        return *this;
    } //调用这个函数时候 比如s=k 可以写成s.operator=(k)

    virtual ~student(){
        delete[] name;
    }
};
```

## 类体外定义方法（成员函数）

必须在类定义中声明，类体外要有类作用域，否则就是全局外部函数了！

```cpp
#include <iostream>
using namespace std;
class Date{
    int d,m,y;
    public:
        void print(); //声明一下
        Date(int dd = 1,int mm = 1,int yy = 1900){
            d = dd; m = mm; y = yy;
            cout << "构造函数" << endl;
        }
        ~Date(){cout << "析构函数" << endl;} //析构函数名是~和类名，且不带参数，没有返回类型
        //目前不需要做任何释放工作，因为构造函数没有申请资源，所以实际上可以不用写
};
void Date::print(){
    cout << y << "-" << m << "-" << d << endl;
} //使用 Date::这个类的作用域来说明它不是全局函数，而是Date这个类的成员函数（方法）
int main()
{
    Date day;
    day.print();
    return 0;
}
```

## 类模板

我们可以将一个类变成"类模板"或"模板类"，正如一个模板函数一样

例子：

动态数组：

```cpp
#include <iostream>
#include <cstdlib>
#include <string>
using namespace std;
class Array{
    int size;
    double *data;
    public:
        Array(int s){
            size = s;
            data = new double[s];
        }
        ~Array(){
            delete[] data;
        }
        double& operator[](int i){
            if(i<0||i>=size){
                cerr << endl << "Out of bounds" << endl;
                //Cerr 一个ostream对象，关联到标准错误，通常写入到与标准输出相同的设备。默认情况下，写到cerr的数据是不缓冲的。Cerr通常用于输出错误信息与其他不属于正常逻辑的输出内容。
                exit(EXIT_FAILURE);
                //不用exit用throw也可
                throw "Out of bounds";
            }
            else
                return data[i];
        }
};
int main(int argc, char const *argv[])
{
    int n;
    cin >> n;
    Array t(n);
    t[0]=45; //如果没有使用operator重载运算符，那么就需要t.data[0]来使用了

    return 0;
}

//这个动态数组的缺点是只能存放double类型的数据，若是要存其他类型，就得另写，那么使用模板类，可以很好的解决这个问题
template<class T> //将所有的double都换成T
class Array{
    int size;
    T *data;
    public:
        Array(int s){
            size = s;
            data = new T[s];
        }
        ~Array(){
            delete[] data;
        }
        T &operator[](int i){
            if(i<0||i>=size){
                cerr << endl << "Out of bounds" << endl;
                exit(EXIT_FAILURE);
                throw "Out of bounds";
            }
            else
                return data[i];
        }
};
int main()
{
    int n;
    cin >> n;
    Array<int> t(n); //类模板后加入两个箭头 写int就告诉编译器 把T替换为int
    t[0]=45;
    int m;
    cin >> m;
    Array<string> d(m); //string是c++的字符串类型，如果使用C的头文件是cstring
    d[0]="Hello";
    return 0;
}
```

## 类的继承和派生

`Inheritance`继承（`Derivation`派生）：一个派生类（`derived class`）从1个或多个父类（`parent class`）/ 基类（`base class`）继承，即继承父类的属性和行为，但也有自己的特有属性和行为。如：

```cpp
#include <iostream>
#include <string>
using namespace std;
class Employee{ //定义一个雇员类
    string name;
    public:
        Employee(string n); //声明构造函数
        void print(); //声明一个print方法（行为）
};
class Manager:public Employee{ //雇员基础上派生出经理这个类，除了拥有雇员那些基础的属性和行为外，另有自己的属性和行为。Mangager继承了雇员这个类中的属性和行为，不需要再定义 name和print
    //public继承是公有继承(public) 公有继承的特点是基类的公有成员和保护成员作为派生类的成员时，它们都保持原有的状态，而基类的私有成员仍然是私有的，不能被这个派生类的子类所访问。 还有其他的继承方式，具体自行查询
    int level;
    public:
        Manager(string n,int l=1);
        void print(); //若是基类的print不满足要求（就是无法打印level）可以自己再重载它；
};
Employee::Employee(string n):name(n) //初始化成员列表
{
    //那个:name(n)写在外面相当于
    //name = n;写在里面
} //在类体外定义方法要加上类的作用域
void Employee::print(){
    cout << name << endl;
}

Manager::Manager(string n,int l):Employee(n),level(l){
    //写在外面那些相当于构造函数参数传递给那些成员
    //即相当于写在里面的：
    //name = n;//那个Employee(n)其实是调用了基类的构造函数
    //level = l;
}
//派生类的构造函数只能描述它自己的成员和其直接基类的初始式，不能直接对基类的成员进行初始化
//例：
//Manager::Manager(string n, int l):name(n),level(l){} //这样会报错，因为name(n)这是对基类的成员进行了初始化

void Manager::print(){
    cout << level << "\t";
    //若是写cout << name << endl;那么会报错，因为无法访问基类的私有成员
    Employee::print();
}
int main(int argc, char const *argv[])
{
    Manager m("Zhang");
    Employee e("Li");
    m.print();
    e.print();
    Employee *p = &e; //还可以定义指针变量
    p->print();
    Manager *q = &m;
    q->print();
    return 0;
}
```

## 虚函数和多态性

虚函数`Virtual Functions`

派生类的指针可以自动转化为基类的指针，用一个指向基类的指针分别指向基类对象和派生类对象，并2次调用

```cpp
#include <iostream>
#include <string>
using namespace std;

class Employee{ //定义一个雇员类
    string name;
    public:
        Employee(string n); //声明构造函数
        void print(); //声明一个print方法（行为）
};
class Manager:public Employee{
    int level;
    public:
        Manager(string n,int l=1);
        void print();
};

Employee::Employee(string n):name(n){
}

void Employee::print(){
    cout << name << endl;
}

Manager::Manager(string n,int l):Employee(n),level(l){
}

void Manager::print(){
    cout << level << "\t";
    Employee::print();
}

int main(int argc, char const *argv[])
{
    Manager m("Zhang");
    Employee e("Li");
    m.print();
    e.print();
    /*
    Employee *p = &e;
    p->print();
    Manager *q = &m;
    q->print();
    */

    Employee employees[100]; //若是没有给构造函数一个默认值，它会报错

    Employee* employees[100];int num=0;
    employees[num] = &e; num++;
    employees[num] = &m; num++;
    //这样就不会报错，employees这个数组存的是employee这个类的对象的地址
    //employees类型是employee* 这个类的指针类型
    //但是&m这个是manager* 这个指针类型，为什么可以存呢？
    //因为manager是employee派生出来的，派生类的指针可以自动转化为基类的指针
    return 0;
}
```

```cpp
int main()
{
    Employee *p;
    Manager m("Zhang",2);
    Employee e("Li");
    p = &e;
    p->print();
    p = &m;
    p->print(); //p是employee的指针类型，但是它指向的是manager这个派生类的地址，调用print（）这个方法是employee的方法而不是manager的函数，如果要让p根据它指向的实际对象来调用相应的方法，那么需要将这个print（）方法变成虚函数（主要是在基类中声明这个函数为虚函数，在派生类中声明不声明无大碍）
    return 0;
}
//下面是声明它为虚函数
class Employee{
    string name;
    public:
        Employee(string n);
        virtual void print(); //声明其为虚函数
};
class Manager:public Employee{
    int level;
    public:
        Manager(string n,int l=1);
        virtual void print(); //派生类可以不用声明
};

//这样的行为称为多态性
```

## 多重继承

我们可以从一个类派生出多个不同的类

```cpp
class Employee{
    //...
    public:
        virtual void print();
};
class Manager: public Employee{ //从employee派生出manager这个类，当然manager还可以派生出另一个类
    //...
    public:
        void print();
};
class Secretary: public Employee{ //从employee基类派生出secretary类
    //...
};
```

我们也可以从多个不同的类派生出一个类来，这称为多重派生(`Multiple inheritance`)

```cpp
class One{
    //class internals;
};

class Two{
    //class internals;
};
class MultipleInheritance:public One, public Two{ //从One、和Two两个类派生出来，两个基类之间用逗号隔开
    //class internals;
};
```

## 纯虚函数(`pure virtual function`) 和抽象类(`abstract base class`)

函数体=0的虚函数称为"纯虚函数"，包含纯虚函数的类称为"抽象类"；

```cpp
#include <string>
class Animal //This Animal is an abstract base class
{
    protected:
        std::string m_name;
    public:
        Animal(std::string name):m_name(name){

        }
        std::string getName(){return m_name;}
        virtual const char* speak()=0; //note that speak is now a pure virtual function
};
int main(int argc, char const *argv[])
{
    Animal a; //错误，抽象类不能实例化（不能定义抽象类的对象（变量））
    return 0;
}
//从抽象类派生的类型如果没有继承实现所有的纯虚函数，则仍然是抽象类；
class Cow:public Animal
{
    public:
        Cow(std::string name):Animal(name){

        }
        //We forgot to redefine speak
};
int main(int argc, char const *argv[])
{
    Cow cow("Frank"); //仍然错：因为Cow仍然是抽象类
    return 0;
}
//像下面这样实现所有纯虚函数就没问题了，Cow就不是抽象类了
class Cow:public Animal
{
    public:
        Cow(std::string name):Animal(name){

        }
        virtual const char* speak(){return "Moo";}
};
int main(int argc, char const *argv[])
{
    Cow cow("Frank");
    return 0;
}
//抽象类通常作为一个接口，这个接口一般不涉及到具体的实现，当然也可以有，可是不能定义这个抽象类的对象
``` 