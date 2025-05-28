# `string`类型

[C++ string类方法大全 - 博客园](https://www.cnblogs.com/dongsheng/articles/2685114.html)

标准库中的类型

```cpp
#include <string>
#include <iostream> //typedef std::basic_string<char> string;
using namespace std;
typedef string String; //给类型起别名

int main(){
    string s1; //使用默认构造函数：没有参数或参数有默认值
    //这里s1默认是空字符串
    String s2("Hello"); //普通构造函数
    s1="HHHHH"; //赋值运算符
    String s3(s1); //拷贝构造函数

    //可以自行cout看看
    //string的其他初始化方法：
    string s4("this is a C_string",10);
    //first: argument C string
    //second: number of characters
    //作用是从first这个字符串中取前second来构造一个字符串

    string s5(s4, 6, 4);
    //1 C++ string
    //2 start position
    //3 number of characters
    //作用是从s4这个字符串第六个字符开始取4个字符构成一个字符串

    string s6(15,'*');
    //1 number characters
    //2 character itself
    //作用是 生成一个字符串，由15个'*'组成

    string s7(s4.begin(),s4.end()-5);
    //1 start iterator(迭代器)
    //2 end iterator
    //作用是 构成一个字符串，从s4这个字符串的开始位置到它末尾位置往前推5个那个位置

    string s8 = "hhhhhh";
    //you can instantiate string with assignment
    //就是普通的生成赋值

    string s9 = s1+"hello"+s2;
    //结果是string类型的对象（变量）

    return 0;

}
```

访问字符串：

```cpp
#include <string>
#include <iostream>
using namespace std;
int main(){
    string s = "hell";
    string w = "worl!";
    s = s +w; //s += w;
    //访问字符串：
    //方法一：下标
    for(int ii=0;ii!=s.size();ii++)
        cout << ii << "" << s[ii] << endl;
    cout << endl;
    //方法二：迭代器
    string::const_iterator cii; //数据类型：常量迭代器
    //若是string::iterator cii;则是非常量（普通的）迭代器，可以修改
    //比如说可以这样修改*cii='A'
    int ii=0;
    for(cii = s.begin();cii!=s.end();cii++) //指向字符串的起始位置
        cout << ii++ << " " << *cii << endl;
    return 0;

}
```

## 向量`vector`

`vector` 向量 用于表示一组数据（相当于数组，但其类型和大小是任意的）

`vector`是类模板

```cpp
#include <vector>
#include <iostream>
using namespace std;
int main(){
    vector<double> student_marks; //类模板<>就这样用的qwq
    int num_students;
    cout << "Number of students:" << endl;
    cin >> num_students;

    student_marks.resize(num_students); //重新改变大小，一开始大小默认的是0；
    for(vector<double>::size_type i=0;i<num_students;i++){ //i的数据元素类型是vector<double>这个类的作用域中的size_type这样一个类型，这个类型比较安全，像整形一样
        cout << "Enter marks for student #" << i+1 << ":" << endl;
        cin >> student_marks[i]; //可用下标访问
    }

    cout << endl;
    for(vector<double>::iterator it = student_marks.begin();it!=student_marks.end();it++) //用迭代器输出 为什么要使用迭代器呢？迭代器的作用用于遍历容器中的每个对象而不用考虑容器中对象的个数，而且保护数据不显示的表现出来。
        cout << *it << endl;

    return 0;
}
``` 