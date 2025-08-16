虚函数可以让成员操作一般化  

看一段代码：  
```C++
#include<iostream>
using namespace std;
class B0{
    public:
        void display(){cout<<"B0::display()"<<endl;}
};
class B1:public B0{
    public:
        void display(){cout<<"B1::display()"<<endl;}
};
class D1:public B1{
    public:
        void display(){cout<<"D1::display()"<<endl;}
};
void fun(B0 *ptr){ptr->display();}
int main(){
    B0 b0; B1 b1; D1 d1; B0 *p;
    p = &b0; fun(p);
    p = &b1; fun(p);
    p = &d1; fun(p);
    system("pause");
    return 0;
}
```
此处的函数都不是虚函数。  

主函数里定义了一个类型为B0的指针p，  

接下来调用指针p的fun函数。  

与Java类似，基类指针调用非虚函数时，不管其指向的是哪一个类型的对象，都是调用基类的重载函数。  

所以输出会是：  

B0::display()  

B0::display()  

B0::display()  

然后如果将display()声明为虚函数，则基类指针指向各个派生类对象时调用display()，调用的就是各个基类中的重载函数。  
