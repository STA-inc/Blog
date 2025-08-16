0x00Fibonacci数列  

第一、第二项为1、1  

从第三项开始，每一项是前面两项之和  

0x01程序输入输出  

输入：需要计算的项数num、存储斐波纳契数列前n项的数组指针list  

输出：list  

0x02代码  
```C
#include<iostream>
using namespace std;
int main(){
    int count = 40;
    int * list = (int *)malloc(count);
    void fib(int num,int * list);
    fib(count,list);
    for(int i=0;i<count;i++) cout<<list[i]<<endl; 
    getchar();
}
/**
 * 计算fib数列前n项
 * */
void fib(int num,int * list){
    list[0] = 1;    //定义数列第一项
    list[1] = 1;    //定义数列第二项
    if(num<=2) return ;
    for(int i=2;i<num;i++){
        list[i] = list[i-1]+list[i-2];
    }
}
```
