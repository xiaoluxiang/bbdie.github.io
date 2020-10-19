#  c语言简要复习笔记

> 参考资料：[简要C语言复习](https://xieguanglei.github.io/blog/post/c-language-review-notes.html)  [菜鸟enum教程](https://www.runoob.com/cprogramming/c-enum.html) 

## 枚举enum

C语言中的枚举可以简单理解为一组被宏定义后的变量组，

1. 组名可以作为一个类型，去声明变量来指向组内成员，

2. 组内成员值不可更改，

3. 组内成员可以通过值来跳跃指向（神奇）

```c
#include<stdio.h>
int main(){
  
  enum day{mon,tu=2,wen,thu,fri,sat,sun}Day;
  //这里区别下结构体，这里定义的Day这个变量。
  //这里的数值也有一定讲究，前一个被定义值后面逐个加一，mon = 0, wen = 3
  enum day Ti;//变量声明
  
  //变量访问赋值
  Ti = mon;
  printf("%d\n",Ti);//0
  
  
  int a = 2;
  Ti = enum day(a);//显式类型转换，Ti = tu;
  Ti = 3; //隐式类型转换
  printf("%d\n",Ti == wen);
  //true
  
}
```



## 结构体 struct typedef

结构体定义数据结构形式，常常搭配着typedef使用

1. 注意结构体变量的

```c
# include<stdio.h>

typedef struct Lnode //定义结构体
{
    int a;
    float b;
    char c;
}Lnode,*Ln;//这里是重点，可以直接typedef Lnode和*Ln

int main(){
    //结构体 使用
    Lnode L = {1,2.0,'c'}; //结构体初始化
  	//对于结构体变量名来说，需要用.来访问结构体内部变量
    printf("%d,%f,%c\n",L.a, L.b, L.c); 

    Ln N = &L;
  	//对于结构体指针来说，需要使用->来访问结构体内部变量
    printf("%d,%f,%c",N->a, N->b, N->c);
  
    return 1;
}
```

