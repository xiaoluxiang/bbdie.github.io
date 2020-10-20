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

1. 注意结构体变量的typedef后的命名，以及结构体指针的声明
2. 结构体变量和结构体指针的访问内部变量的方法
3. 待定

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

## 函数 

等待被调用的代码块，地址传入的需要加*去访问他地址所存的值。

1. 函数结构

   - 函数（返回）类型声明，int void float等，配合着return一致。

   - 函数参数，参数分为俩类，形参（值传入），实参（地址传入） int a, int &a;

   - 函数内部变量无法带出，仅能带出return 后的信息。

2. 函数位置

   - 函数在main外部（同文件内），要么需要在main前全部都写出来，要么需要在main前需要声明这个函数，在后面补上。

   - 函数在其他文件，则需要在文件头文件引用部分，添加include “***.c"

   - ```c
     //作为其他文件被引用
     #include<stdio.h>//这里需要添加头文件，否则编译器警告
     int prov(){
         printf(" head 文件被调用，且此函数成功执行");
         return 1;
     }
     ```

   - ```c
     # include "head.c" //引入其他文件
     #include <stdio.h>
     # include "head.c"
     #include <stdio.h>
     
     int passin(int a, float * b){
             printf("这是函数内部的a:%d\n", a+=1);
             printf("这是函数内部的b:%f\n", *b+=1.0);
             return 1;
         }
     
     int main(){
         // 值传入和地址传入
         int a = 1;
         float b = 1.0;
     
         passin(a,&b);
         printf("这是main函数的a:%d\n", a);
         printf("这是main函数内部的b:%f\n", b);//这里有一个很大的易错点就是地址传入的地址，需要*b访问哦
     
         prov();//直接调用其他函数
         return 1;
     }
     int main(){
         prov();//直接调用其他函数
         return 1;
     }
     ```

## 指针和数组

如何理解指针：

1. 指针在类型声明中，函数参数引用中等。应该使用int * a / int * a[]；
2. 指针作为变量引用过程中，区分为三种状态：
   - ap本身是一个指针类型的变量，表示其值即为其所指向的地址；
   - *ap表示取其值（地址）的值；
   - &ap表示取变量本身的地址；

如何理解数组

1. 数组为空间地址上连续的一段，每固定长度表示一个元素。且各个元素种类需要保持一致；
2. 对于一维二维数组，我们可以通过sizeof来取到其长度；
3. 数组变量名指向其首地址；

```c
//数组和指针
# include<stdio.h>
//数组遍历函数
int printeveryone(int a[]){
    
    for(int i=0;i<10;i++){
        printf("%d\n",a[i]);
    }
    return 1;
}

int main(){
    int a[10]={0,1,2};
    int b[8][9] = {0,1,2,3};
    printeveryone(a);

    //一维数组size获取:
    printf("这里是一维数组长度：%d\n",sizeof(a)/sizeof(a[0]));
    
    //二维数组size获取:
    /*sizeof(array[0][0])为一个元素占用的空间，
    sizeof(array[0])为一行元素占用的空间，
    sizeof(array)为整个数组占用的空间*/
    printf("这里是二维数组列长：%d\n",sizeof(b[0])/sizeof(b[0][0]));
    printf("这里是二维数组的宽度：%d\n",sizeof(b)/sizeof(b[0]));

    //指针部分
    int * ap = &a;
    //指针实现遍历整个数组
    for(int i = 0;i < sizeof(a)/sizeof(a[0]);i++){
        printf("指针遍历中:%d,%d,%d\n",*ap,ap,&ap);
        *(ap++);
    }

    
}
```

