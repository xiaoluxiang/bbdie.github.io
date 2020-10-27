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
2. 对于一维二维数组，我们可以通过sizeof来取到其长度；（重点在于数组作为参数传递时，无法在其他函数中使用sizeof函数）
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



## 堆内存和文件操作

堆内存通过malloc请求，得到的是地址块的首地址，单个则为指针，多个理解为数组。

这里的文件操作只是做了一些小小了解：文件打开（模式）与关闭，文件fputc/fputs/fgetc/fgets的用法，文件指针的偏移。

```c
// 堆内存和读写文本方法
# include <stdio.h>
# include <stdlib.h>
int main(){
    // 申请堆内存，
    int *a = (int *) malloc(sizeof(int));
    *a=1;
    printf("%d",*a);
    free(a);

    //指针 堆内存 数组三合一
    int *b = (int *) malloc(sizeof(int)*10);
    //此处如何理解b:理解成数组a[]中的a,其后的访问方法就是b[i];
    for(int i = 0; i<10; i++){
        printf("%d\n",b[i]);
    }
    free(b);
    

    // 文件操作
    FILE * fp = fopen("./txt.txt","a+");
    
    char c = 'c';
    char d[] = "\nzhelishifputsaffect\n";
    //测试方法：读 - 写 -（移动）- 再读
    printf("%c",fgetc(fp));


    fputc(c,fp);
    fputs(d,fp);

    
    
    //fp 位置移动
    /*
    int fseek(FILE * stream, long offset, int fromwhere);
    【参数】stream为文件指针，offset为偏移量，fromwhere为指针的起始位置。

    SEEK_SET：从距文件开头 offset 位移量为新的读写位置；
    SEEK_CUR：以目前的读写位置往后增加 offset 个位移量；
    SEEK_END：将读写位置指向文件尾后再增加 offset 个位移量。
    */
    fseek(fp,-4,SEEK_CUR);
    printf("%c",fgetc(fp));
    fclose(fp);
    return 1;
}
```



## 字符串操作

> 记得引入头文件：\#icnlude<string.h\>

1. 使用 `strlen` 取字符串的长度；

2. 使用 `strchr` 取某个字符第一次出现的位置（指针）；

3. 使用 `strcpy` 将一个字符串复制到另一个字符串中；

4. 使用 `strcat` 将一个字符串追加到另一个字符串中。

   ```c
   #include <stdio.h>
   #include <string.h>
   #define MAX 1000
   
   void main() {
       char s[MAX] = "hello world";
       char t[MAX] = "";
   
       int len = strlen(s); // len is 11
       char *pl = strchr(s, 'l'); // (pl-s) is 2
       strcpy(t, s); // t is "hello world"
       strcat(t, s); // t is "hello worldhello world"
   }
   ```

   

## 条件和循环

条件：

1. if/else  

   ```c
   if(){
     Exp;
   }
   else{
     Exp;
   }
   ```

   

2. switch/case  其一般格式：

   ```c
   switch(表达式)
   {
       case 常量表达式1:语句1;
       case 常量表达式2:语句2;
       ...
       default:语句n+1;
   }
   ```

   

3. Exp? Exp1: Exp2;

   ```c
   Exp? Exp1: Exp2;
   ```

   

循环

1. while

   ```c
   while(Exp)
   {
      Exp;
   }
   ```

2. for

   ```c
   for(exp1;exp2;exp3){
     
   }
   ```

3. do-while

   ```c
   do{
     Exp;
   }while(Exp);
   ```




## 输入输出

1. 输入

   - scanf() //读取字符串时以空格为分隔，遇到空格就认为当前字符串结束了，所以无法读取含有空格的字符串。
   - getchars()  = scarf("%d", &var); 用法：var = getchars();
   - gets(var )  //认为空格也是字符串的一部分，只有遇到回车键时才认为字符串输入结束，所以，不管输入了多少个空格，只要不按下回车键，对 gets() 来说就是一个完整的字符串。

   ```c
   
   ```

   

2. 输出

   ```c
   
   ```




## 二维数组函数参数传递

二维数组在函数参数传递中，需要至少指定第二维的长度。int a\[2]\[7];总共三种方法：

​	// 下面俩个方法，仅仅是函数声明的参数写法不一样。都可以正常使用 a\[]\[] 访问（细微往下考究）

1. int example(int a\[]\[7], int n, int m); // 正常的逻辑

   调用方法：example(a, 2, 7);

2. int example(int (*a)[7], int n, int m); //这里是将数组看出每个有七个元素的一位数组

   调用方法：example(a, 2, 7); 

   

3. int example(int * a, int n, int m); // 利用了二维数组也是顺序存储特性，*(a + i\*m + j);来访问特定值

   调用方法：example(&a\[0]\[0], 2, 7); //传入首地址

```c
// 代码示例：
//
//  debug_my.c
//  zt-2019-1
//
//  Created by lushixiang on 2020/10/26.
//  Copyright © 2020 lushixiang. All rights reserved.
//

#include <stdio.h>
//采用常规方法
int example1(int a[][5], int n, int m){
    for(int i = 0; i<n; i++){
        for(int j = 0; j < m; j++){
            printf("%d",a[i][j]);
        }
    }
    printf("\n这里example1\n");
    return 0;
}

//常规方法小变形，但是没有必要
int example2(int(*a)[5], int n, int m){
    for(int i = 0; i < n;i++){
        for(int j = 0; j<m; j++){
            printf("%d",a[i][j]);
        }
    }
    printf("\n这里example2\n");
    return 0;
}

//把二维数组视作一位数组
int example3(int *a, int n, int m){
    for(int i=0; i<n; i++){
        for(int j= 0; j<m; j++){
            printf("%d",*(a+i*m+j));
        }
    }
    printf("\n这里example3\n");
    return 0;
}

int main(){
    int a[3][5] = {1,2,3,4,5,6,7,8,9,0};
    example1(a, 3, 5);
    example2(a,3, 5);
    example3(&a[0][0] , 3, 5);
    return 0;
}

```



复习结束🔚睡觉😪