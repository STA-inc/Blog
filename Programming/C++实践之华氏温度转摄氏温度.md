```C++
#include<iostream>
#include<cmath>
#include<iomanip>
using namespace std;
int main(){
    cout<<"输入华氏温度"<<endl;
    double F;
    cin >> F;
    double F2C(double F);
    cout << setprecision(4) << F2C(F);
    system("pause");
}
/**
 * 输入一个华氏温度，要求计算摄氏温度
 * */
double F2C(double F){
    double result = 5*(F-32)/9;
    return result;
}
```
