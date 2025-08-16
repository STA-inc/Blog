0x00题目  

设圆半径r=1.5,圆柱高h=3，求圆周长、圆面积、圆球体积、圆球面积、圆柱体积  

用cin输入数据，取小数点后两位数字.  

0x01代码  
```C++
#include<iostream>
#include<cmath>
#include<iomanip>
using namespace std;
int main(){
    cout<<"输入圆半径与圆柱高，空格隔开"<<endl;
    double r,h;
    cin >> r >> h;
    void caculate(double r,double h);
    caculate(r,h);
    system("pause");
}
/**
 * 计算圆周长、圆面积、圆球表面积、圆球体积、圆柱体积
 * */
void caculate(double r,double h){
    const double PI = atan(1.)*4.; //定义圆周率
    double yuanD = PI*2*r;            
    double yuanS = PI*r*r;
    double ballS = 4*PI*r*r;
    double balla = 4,ballb = 3;
    double ballV = ((balla/ballb))*PI*r*r*r;
    double zhuV = PI*r*r*h;
    cout << "圆周长:" << setprecision(3) << yuanD << endl;
    cout << "圆面积:" << setprecision(3) << yuanS << endl;
    cout << "圆球表面积:" << setprecision(4) << ballS << endl;
    cout << "圆球体积:" << setprecision(4) << ballV << endl;
    cout << "圆柱体积:" << setprecision(4) << zhuV << endl;
}
```
