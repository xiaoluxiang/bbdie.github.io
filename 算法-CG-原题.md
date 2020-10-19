#【问题描述】对n个整数进行全排列

【输入形式】在屏幕上输入若干个整数，各数间都以一个空格分隔。

【输出形式】按照顺序每行输出一种排列方式

【样例输入】

 1 2 3

【样例输出】

 [1 2 3]

 [1 3 2]

 [2 1 3]

 [2 3 1]

 [3 2 1]

 [3 1 2]

【样例说明】

 输入：三个整数，分别为1,2,3，以空格分隔。

 输出：按照顺序每行输出一种排列方式，以方括号开始和结尾



```Python
import numpy as np
def perm(ls, k, m):
    if k == m:
        print(ls)
    else:
        for i in range(k, m):
            ls[i], ls[k] = ls[k], ls[i]
            perm(ls, k + 1, m)
            ls[i], ls[k] = ls[k], ls[i]
ls = np.array(input().split(), int)
perm(ls, 0, len(ls))



```



#【问题描述】在有序数组中寻找特定的元素

【输入形式】在屏幕上输入若干个从小到大排列的整数，各数间都以一个空格分隔。再输入要寻找的元素。

【输出形式】若有序数组中存在该特定元素，则输出该元素在数组中的位置；若不存在，则输出0。

【样例1输入】

 1 2 3 4 5 6 7 8 9 10

 3

【样例1输出】

 3

【样例1说明】

 输入：10个有序整数，从1到10，以空格分隔。特定元素为3。

 输出：3，表示特定元素3在有序数组第3个位置。

 【样例2输入】

 1 2 3 4 5 6 7 8 9 10

 11

【样例1输出】

 0

【样例2说明】

 输入：10个有序整数，从1到10，以空格分隔。特定元素为11。

 输出：0，表示特定元素11在有序数组中不存在。



```python 
def Find(A,key):
    left = len(A)
    right = 1
    while right <= left:
        if left==right:
            if key!=A[left-1]:
                return 0

        middle = int((right + left)/2)
        if key < A[middle]:
            left = middle - 1
        elif key > A[middle]:
            right = middle + 1
        else:
            return middle+1
    return 0
if __name__ == "__main__":
    A = input()
    key = int(input())
    A = A.split()
    for i in range(len(A)):
        A[i] = int(A[i])
    left = len(A)
    right = 0
    print(Find(A,key))
```





#问题描述】使用递归合并排序算法对若干整数进行排序

【输入形式】在屏幕上输入若干整数，各数间都以一个空格分隔。

【输出形式】输出每次划分的结果和最终从小到大排序好的结果。

【样例输入】

 48 38 65 97 76 13 27

【样例输出】

[48 38 65 97 76 13 27]

[48 38 65 97]

[48 38]

[48]

[38]

[65 97]

[65]

[97]

[76 13 27]

[76 13]

[76]

[13]

[27]

[13 27 38 48 65 76 97]

【样例说明】

 输入：7个整数，以空格分隔。

 输出：输出每次划分的结果和最终从小到大排序好的结果。起始和结束加方框号。



```python
import numpy as np
def mergesort(seq):
    """归并排序"""
    if len(seq) <= 1:
        return seq
    mid = len(seq) // 2  #求出middle，作为下面递归的middle
    left = mergesort(seq[:mid])
    right = mergesort(seq[mid:])#利用上面求出的middle，递归的处理。


    return merge(left, right)#简写，合并函数

def merge(left, right):
    result = []  # 用来存放已排好序的
    i = 0
    j = 0

    while i < len(left) and j < len(right):#找到最小的元素，放入result，并对列表的下标进行更改。
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])#当某一列表被处理完，而另一个没有处理完，则直接添至result中
            j += 1

    result += left[i:]
    result += right[j:]
    return result



def Fun(a, left, right) :
    print(np.array(a[left: right + 1]))
    if left < right :
        i = (left + right) // 2
        Fun(a, left, i)
        Fun(a, i+1, right)

array =input()
array=array.split()
for i in range(len(array)):#input接受后，对字符串进行处理。成list
    array[i]=int(array[i])
Fun(array,0,len(array)-1)
result = mergesort(array)
print(np.array(result))


```



#【问题描述】每次划分时都以最后一个元素为划分基准，使用快速排序算法对若干整数进行排序。

【输入形式】在屏幕上输入若干整数，各数间都以一个空格分隔。

【输出形式】输出每次划分的基准元素和该数在划分后的数组中的位置，以及从小到大的排序结果。

【样例输入】

 48 38 65 97 76 13 27

【样例输出】

27 1

38 2

65 4

76 5

[13 27 38 48 65 76 97]

【样例说明】

 输入：7个整数，以空格分隔。

 输出：输出每次划分的基准元素和该数在划分后的数组中的位置，以空格分隔。从小到大的排序结果，以空格分隔，起始和结束用方框号。

```python 
import numpy as np


def Partion(a, p, r):
    x = a[r]
    i = p - 1
    for j in range(p, r ):
        if a[j] <= x:
            i = i + 1
            a[i], a[j] = a[j], a[i]
    a[i + 1], a[r] = a[r], a[i + 1]
    return i + 1


def quick_sort(array, low, high):
    if low < high:
        key = Partion(array, low, high)  # 一次排序后返回key的位置，并作为下面的middle。下面是递归。
        print(array[key], key)
        quick_sort(array, low, key - 1)
        quick_sort(array, key + 1, high)


if __name__ == "__main__":
    array = input()
    array = array.split()
    for i in range(len(array)):  # input接受后，对字符串进行处理。成list
        array[i] = int(array[i])
    array = np.array(array)
    quick_sort(array, 0, len(array) - 1)
    array = np.array(array)
    print(array)
```



#【问题描述】每次都是随机选出一个元素为划分基准，在平均情况下线性时间内寻找第i小元素。

【输入形式】在屏幕上输入若干整数，各数间都以一个空格分隔。再输入要寻找的元素是数组从小到大顺序中第几个位置。

【输出形式】数组从小到大顺序中要寻找的那个位置的元素。

【样例输入】

 2 9 8 0 7 10 1 12 3 14 5 13 6 11 4

 3

【样例输出】

 2

【样例说明】

 输入：15个整数，以空格分隔。要寻找第3小元素。

 输出：2，表示第3小元素为2。

```python
import random
import numpy as np

def RandomizedPartition(a, p, r):
    i = int(random.uniform(p, r))
    a[i], a[r] = a[r], a[i]
    x = a[r]
    i = p - 1   
    for j in range(p, r):
        if a[j] <= x:
            i = i + 1
            a[i], a[j] = a[j], a[i]
    a[i + 1], a[r] = a[r], a[i + 1]
    return i+1


def RandomizeSelect(a, p, r, k):
    if p == r:
        return a[p]
    i = RandomizedPartition(a, p, r)
    j = i - p + 1 
    if k < j:
        return RandomizeSelect(a, p, i, k)
    else:
        return RandomizeSelect(a, i + 1, r, k - j)


if __name__ == '__main__':
    a = np.array(input().split(),int)
    p = 0
    r = len(a)
    k =  int(input())
    print (RandomizeSelect(a, p, r - 1, k-1))


```



#【问题描述】每次都是优化选出一个元素为划分基准，在线性时间内寻找第i小元素。

【输入形式】在屏幕上输入若干整数，各数间都以一个空格分隔。再输入要寻找的元素是数组从小到大顺序中第几个位置。

【输出形式】数组从小到大顺序中要寻找的那个位置的元素。

【样例输入】

 2 9 8 0 7 10 1 12 3 14 5 13 6 11 4

 3

【样例输出】

 2

【样例说明】

 输入：15个整数，以空格分隔。要寻找第3小元素。

 输出：2，表示第3小元素为2。



```python
import numpy as np

def RandomzediPartition(a, p, r):
    i = int(random.uniform(p, r))
    a[i], a[r] = a[r], a[i]
    x = a[r]
    i = p - 1   
    for j in range(p, r):
        if a[j] <= x:
            i = i + 1
            a[i], a[j] = a[j], a[i]
    a[i + 1], a[r] = a[r], a[i + 1]
    return i+1



def partition(a, p, r,i):
    a[i], a[r] = a[r], a[i]
    x = a[r]
    i = p - 1   
    for j in range(p, r):
        if a[j] <= x:
            i = i + 1
            a[i], a[j] = a[j], a[i]
    a[i + 1], a[r] = a[r], a[i + 1]
    return i+1


def RandomizeSelect(a, p, r, k):
    if p == r:
        return a[p]
    i = RandomizedPartition(a, p, r)
    j = i - p + 1 
    if k < j:
        return RandomizeSelect(a, p, i, k)
    else:
        return RandomizeSelect(a, i + 1, r, k - j)


def select(a,p,r,k):
    if r-p<75:
        a.sort()
        print(a[p+k-1])
        return 0
    for i in range((r-p-4)//5):
        ab=RandomizeSelect(a[p+5*i:p+5*i+5],0,5,3)
        ab,a[p+i]=a[p+i],ab
        x=select(a,p,p+(r-p-4)//5,(r-p-4)//10)
        i=partition(a,p,r,x)
        j=i-p+1
        if k<j:
            return select(a,p,i,k)
        else:
            return select(a,i+1,r,k-j)

if __name__ == '__main__':
    a = np.array(input().split(),int)
    p = 0
    r = len(a)
    k =  int(input())
    select(a, p, r - 1, k)


```



#【问题描述】使用动态规划算法解矩阵连乘问题，具体来说就是，依据其递归式自底向上的方式进行计算，在计算过程中，保存已子问题答案，每个子问题只解决一次，在后面计算需要时只要简单查一下得到其结果，从而避免大量的重复计算，最终得到多项式时间的算法。

【输入形式】在屏幕上输入矩阵连乘个数，和第1个矩阵的行数和第1个矩阵到第n个矩阵的列数，各数间都以一个空格分隔。

【输出形式】矩阵m，其中m(i,j)中存放的是：计算A[i:j](其中1<=i<=j<=n)所需的最少数乘次数。矩阵s，其中s[i][j]记录了断开的位置，即最优的加括号方式应为(A[i:s[i][j]])*(A[s[i][j]+1:j])。矩阵连乘A1...An的最优计算次序。

【样例输入】

 6

 30 35 15 5 10 20 25

【样例输出】

 [[  0 15750 7875 9375 11875 15125]

 [  0   0 2625 4375 7125 10500]

 [  0   0   0  750 2500 5375]

 [  0   0   0   0 1000 3500]

 [  0   0   0   0   0 5000]

 [  0   0   0   0   0   0]]

 [[0 1 1 3 3 3]

 [0 0 2 3 3 3]

 [0 0 0 3 3 3]

 [0 0 0 0 4 5]

 [0 0 0 0 0 5]

 [0 0 0 0 0 0]]

 ((A1(A2A3))((A4A5)A6))

【样例说明】

 输入：矩阵连乘个数为6，第1个矩阵的行数和第1个矩阵到第n个矩阵的列数，以空格分隔。

 输出：矩阵m，s，和矩阵连乘的最优计算次序

```python
import numpy as np



def printoptimalparens(s,i,j):
    if i==j:
        print('A'+str(i),end='')
    else:
        print('(',end='')
        printoptimalparens(s,i,s[i,j])
        printoptimalparens(s,s[i,j]+1,j)
        print(')',end='')

w = int(input()) + 1
p = np.array(input().split(), dtype=int)
s = np.zeros((w, w), dtype=int)
m = np.zeros((w, w), dtype=int)

for i in range(w):
    for n in range(w):
        m[i][n] = 0
for i in range(w):
    for n in range(w):
        s[i][n] = 0
r = 2
n = w - 1
while (r <= n):  # 按列循环
    i = 1
    while (i <= (n - r + 1)):  # 按行循环
        j = i + (r - 1)
        m[i][j] = m[i][i] + m[i + 1][j] + p[i - 1] * p[i] * p[j]
        s[i][j] = i
        k = i + 1
        while (k < j):  # 行内找出最优解，并存入m
            t = m[i][k] + m[k + 1][j] + p[i - 1] * p[k] * p[j]
            if t < m[i][j]:
                m[i][j] = t
                s[i][j] = k
            k += 1
        i += 1
    r += 1

print(m[1:, 1:])

print(s[1:, 1:])

printoptimalparens(s,1,n)

```



#【问题描述】使用动态规划算法解最长公共子序列问题，具体来说就是，依据其递归式

  自底向上的方式依次计算得到每个子问题的最优值。

【输入形式】在屏幕上输入两个序列X和Y，序列各元素数间都以一个空格分隔。

【输出形式】矩阵c，其中c(i,j)中存放的是：序列Xi = {x1, ..., xi}和序列Yj = {y1, ..., yj}的最长公共子序列的长度。序列X和Y的最长公共子序列。

【样例输入】

 A B C B D A B

 B D C A B A

【样例输出】

 [[0 0 0 0 0 0 0]

 [0 0 0 0 1 1 1]

 [0 1 1 1 1 2 2]

 [0 1 1 2 2 2 2]

 [0 1 1 2 2 3 3]

 [0 1 2 2 2 3 3]

 [0 1 2 2 3 3 4]

 [0 1 2 2 3 4 4]]

 BCBA

【样例说明】

 输入：第一行输入序列X的各元素，第二行输入序列Y的各元素，元素间以空格分隔。

 输出：矩阵c，和序列X和Y的最长公共子序列。

```python
import numpy as np


def RECURSIVE(x, y, c, b):
    m = len(x)
    n = len(y)
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if (x[i - 1] == y[j - 1]):#如果末尾相等，则在其对角值加 1  注意x，y的位置坐标应该是b，c的坐标减一。
                c[i][j] = c[i - 1][j - 1] + 1
                b[i][j] = 1
            elif (c[i - 1][j] >= c[i][j - 1]):#如果末尾不相等，则找其上或下最大值，并赋值给自己
                c[i][j] = c[i - 1][j]
                b[i][j] = 2
            else:
                c[i][j] = c[i][j - 1]
                b[i][j] = 3


def lcs(i, j, x, b):
    if (i == 0 or j == 0):
        return 0
    if b[i][j] == 1:
        lcs(i - 1, j - 1, x, b)
        print(x[i - 1], end='')#放在递归调用之后，从而使得从递归底层向上顺序打印
    elif b[i][j] == 2:
        lcs(i - 1, j, x, b)
    else:
        lcs(i, j - 1, x, b)


if __name__ == '__main__':
    x = np.array(input().split())
    y = np.array(input().split())
    b = np.zeros((len(x) + 1, len(y) + 1), dtype=int)#构建长宽为len(x)+1,len(y)+1矩阵b，c。
    c = np.zeros((len(x) + 1, len(y) + 1), dtype=int)
    RECURSIVE(x, y, c, b)
    print(c)
    i = len(x)
    j = len(y)
    lcs(i, j, x, b)


```



#【问题描述】使用分治递归算法解最大子段和问题，具体来说就是，将序列分为长度相等的左右两段，分别求出这两段的最大子段和，包含左右部分子段的最大子段和，求这三种情况得到的最大子段和的最大值。

【输入形式】在屏幕上输入一个序列元素，包含负整数、0和正整数。

【输出形式】序列的最大子段和，及得到最大子段和时的起始和终止编号。

【样例输入】

 -2 11 -4 13 -5 -2

【样例输出】

 20

 2

 4

【样例说明】

 输入：6个数，元素间以空格分隔。

 输出：序列的最大子段和20，得到最大子段和时的起始编号为2，终止编号为4。

```python
import numpy as np


def judge(sum, leftsum, rightsum):
    if sum >= leftsum:
        if sum > rightsum:
            return 0
        else:
            return 2
    else:
        if leftsum < rightsum:
            return 2
        else:
            return 1


def Maxsub(a, left, right):
    sum = 0
    ri = 0
    li = 0
    if (left == right):  # 判断列表是否被分割完
        if (a[left] > 0):
            sum = a[left]
        else:
            sum = 0

    else:
        center = (left + right) // 2  # center是中间值

        leftsum, llit, lrit = Maxsub(a, left, center)  # 得到左段最大值，用llit（left leftindextruth 缩写替代）和llit记录左段子段最大坐标
        rightsum, rlit, rrit = Maxsub(a, center + 1, right)  # 得到右段最大值，用rlit（right rightindextruth 缩写替代）和rrit记录右端最大子段坐标

        s1 = 0
        lefts = 0
        i = center
        while i >= left:  # 计算从a【center】往左最大值子段和
            lefts = lefts + a[i]
            if (lefts > s1):
                s1 = lefts
                li = i  # 记录向左延伸的最大坐标
            i = i - 1

        s2 = 0
        rights = 0
        for i in range(center + 1, right + 1):  # 计算a【center】往右最大子段值
            rights = rights + a[i]
            if (rights > s2):
                ri = i  # 记录向右延伸的最大坐标
                s2 = rights

        sum = s1 + s2  # 得到列表中间最大值

        if (judge(sum, leftsum, rightsum) == 0):#根据三种情况选择性返回sum及其坐标
            return sum, li, ri
        if (judge(sum, leftsum, rightsum) == 1):
            return sum, llit, lrit
        if (judge(sum, leftsum, rightsum) == 2):
            return sum, rlit, rrit
    return sum ,0,0     #只有分割到底才会使用这个return

a = np.array(input().split(), dtype=int)
# a=np.array(input().split())
# a=[-2 ,11 ,-4 ,13 ,-5 ,-2]
max, right, left = Maxsub(a, 0, len(a) - 1)
print(max)
print(right + 1)
print(left + 1)
#        -2 11 -4 13 -5 -2



```



#【问题描述】使用动态规划算法解最大子段和问题，具体来说就是，依据递归式，按照顺序求得子问题。

【输入形式】在屏幕上输入一个序列元素，包含负整数、0和正整数。

【输出形式】序列的最大子段和，及得到最大子段和时的起始和终止编号。

【样例输入】

 -2 11 -4 13 -5 -2

【样例输出】

 20

 2

 4

【样例说明】

 输入：6个数，元素间以空格分隔。

 输出：序列的最大子段和20，得到最大子段和时的起始编号为2，终止编号为4。

```python 
import numpy as np
def judge(sum, leftsum, rightsum): # 避免查重
    if sum >= leftsum:
        if sum > rightsum:
            return 0
        else:
            return 2
    else:
        if leftsum < rightsum:
            return 2
        else:
            return 1


def Maxsub(a, left, right):
    sum = 0
    ri = 0
    li = 0
    if (left == right):  # 判断列表是否被分割完
        if (a[left] > 0):
            sum = a[left]
        else:
            sum = 0

    else:
        center = (left + right) // 2  # center是中间值

        leftsum, llit, lrit = Maxsub(a, left, center)  # 得到左段最大值，用llit（left leftindextruth 缩写替代）和llit记录左段子段最大坐标
        rightsum, rlit, rrit = Maxsub(a, center + 1, right)  # 得到右段最大值，用rlit（right rightindextruth 缩写替代）和rrit记录右端最大子段坐标

        s1 = 0
        lefts = 0
        i = center
        while i >= left:  # 计算从a【center】往左最大值子段和
            lefts = lefts + a[i]
            if (lefts > s1):
                s1 = lefts
                li = i  # 记录向左延伸的最大坐标
            i = i - 1

        s2 = 0
        rights = 0
        for i in range(center + 1, right + 1):  # 计算a【center】往右最大子段值
            rights = rights + a[i]
            if (rights > s2):
                ri = i  # 记录向右延伸的最大坐标
                s2 = rights

        sum = s1 + s2  # 得到列表中间最大值

        if (judge(sum, leftsum, rightsum) == 0):#根据三种情况选择性返回sum及其坐标
            return sum, li, ri
        if (judge(sum, leftsum, rightsum) == 1):
            return sum, llit, lrit
        if (judge(sum, leftsum, rightsum) == 2):
            return sum, rlit, rrit
    return sum ,0,0     #只有分割到底才会使用这个return

 # 以上为避免被查重用








def maxsum(a, n):
    sum = 0
    b = 0
    index1 = 0  # index1作为坐标起始点
    index2 = 0  # index2作为坐标终止点
    for i in range(0, n - 1):  # 因为传入的是len（a）
        if (b > 0):
            b = b + a[i]
            if b > sum:  # 进行判断子段和是否增加
                sum = b
                index2 = i  # 更新坐标的终止点
        else:
            b = a[i]
            index1 = i  # 更新坐标的起始点

    return sum, index1, index2


a = np.array(input().split(), dtype=int)
# a=[-2 ,11 ,-4 ,13 ,-5 ,-2]
sum, left, right = maxsum(a, len(a))
print(sum)
print(left + 1)  # 对应坐标加一
print(right + 1)
# -2 11 -4 13 -5 -2



```

#【问题描述】使用动态规划算法解凸多边形最优三角剖分问题，具体来说就是，依据递归式，按照顺序求得子问题，使得该三角剖分中诸三角形上权之和为最小。

【输入形式】在屏幕上输入凸多边形顶点个数和顶点坐标。

【输出形式】最优三角剖分后的三角形顶点。

【样例输入】

 7

 8 26

 0 20

 0 10

 10 0

 22 12

 27 21

 15 26

【样例输出】

 012

 234

 024

 456

 046

【样例说明】

 输入：顶点个数为7，每一行为一个顶点坐标，以空格分隔。

 输出：每一行为顺序产生的最优三角剖分后的三角形顶点。



```python
'''import numpy as np


def Minweightriangulation(a, n, m, s):
    r = 2

    while (r <= n):  # 按列循环
        i = 1
        while (i <= (n - r + 1)):  # 按行循环
            j = i + r - 1
            print(j)
            m[i][j] = m[i + 1][j] + sanjiaoquan(a, i - 1, i, j)
            s[i][j] = i
            k = i + 1
            while (k < j):  # 行内找出最优解，并存入m
                t = m[i][k] + m[k + 1][j] + sanjiaoquan(a, i - 1, k, j)
                if t < m[i][j]:
                    m[i][j] = t
                    s[i][j] = k
                k += 1
            i += 1
        r += 1
    print(m)
    print(s)


def sanjiaoquan(l, i, k, j):
    a = np.linalg.norm(l[i] - l[k])
    b = np.linalg.norm(l[k] - l[j])
    c = np.linalg.norm(l[i] - l[j])
    return a + b + c


if __name__ == '__main__':
    n = int(input())
    a = np.zeros((n + 1, 2), dtype=int)
    for i in range(n):
        a[i] = np.array(input().split(), dtype=int)
    m = np.zeros((n + 1, n + 1), dtype=np.double)
    s = np.zeros((n + 1, n + 1), dtype=np.double)

    Minweightriangulation(a, n, m, s)'''

'''
7
8 26
0 20
0 10
10 0
22 12
27 21
15 26
'''
import numpy as np


def printoptimalparens(s, i, j):  # 修改后的traceback去打印相应坐标
    if i == j:
        return
    else:
        printoptimalparens(s, i, s[i, j])
        printoptimalparens(s, s[i, j] + 1, j)
        print(i - 1,end='')
        print(s[i][j],end='')
        print(j,end='')
        print('')

def sanjiaoquan(l, i, k, j):  # 计算三角三边权值总合
    a = np.linalg.norm(l[i] - l[k])
    b = np.linalg.norm(l[k] - l[j])
    c = np.linalg.norm(l[i] - l[j])
    return a + b + c


w = int(input()) + 1

a = np.zeros((w, 2), dtype=int)  # 存入各点坐标
for i in range(w - 1):
    a[i] = np.array(input().split(), dtype=int)
s = np.zeros((w, w), dtype=int)  # 初始化s 存入最优解坐标位置
m = np.zeros((w, w), dtype=int)

r = 2
n = w - 1
while (r <= n):  # 按列循环
    i = 1
    while (i < (n - r + 1)):  # 按行循环
        j = i + (r - 1)
        m[i][j] = m[i + 1][j] + sanjiaoquan(a, i - 1, i, j)
        s[i][j] = i
        k = i + 1
        while (k < j):  # 行内找出最优解，并存入m
            t = m[i][k] + m[k + 1][j] + sanjiaoquan(a, i - 1, k, j)
            if t < m[i][j]:
                m[i][j] = t
                s[i][j] = k
            k += 1
        i += 1
    r += 1

printoptimalparens(s, 1, n - 1)




```

#【问题描述】使用动态规划算法解0-1背包问题，具体来说就是，依据递归式，按照顺序求得子问题，使得选择合适物品装入背包可使这些物品的重量总和不超过背包容量，且价值总和最大。

【输入形式】在屏幕上输入背包容量、物品数量、每件物品价值和重量。

【输出形式】最优解时所选物品编号。

【样例输入】

 10

 5

 6 3 5 4 6

 2 2 6 5 4

【样例输出】

 1 2 5

【样例说明】

 输入：背包容量10、物品数量5、每件物品价值6, 3, 5, 4, 6和重量2, 2, 6, 5, 4。

 输出：最优解时选择物品编号为1,2,5。

```python
import numpy as np


def bag(n, c, w, v,res):
    for j in range(c + 1):
        res[0][j] = 0
    for i in range(1, n + 1):   #双重循环处理二维列表
        for j in range(1, c + 1):
            res[i][j] = res[i - 1][j]
            if j >= w[i - 1] and res[i][j] < res[i - 1][j - w[i - 1]] + v[i - 1]:   # 判断选择价值最大值
                res[i][j] = res[i - 1][j - w[i - 1]] + v[i - 1]
    return res


def show(n, c, w, res):
    x = [0 for i in range(n)]
    j = c
    for i in range(1, n + 1):   # 如果此点的值来自第I加，则将其记为TRUE
        if res[i][j] > res[i - 1][j]:
            x[i - 1] = True
            j -= w[i - 1]   # 要改变再寻找的真值
    for i in range(n):
        if x[i]:    # 在x列表中遍历真值，并将结果打印出来
            print(i+1, end=' ')


if __name__ == '__main__':
    c = int(input())

    n = int(input())

    v = np.array(input().split(), dtype=int)  # 输入价值

    w = np.array(input().split(), dtype=int)  # 输入质量

    res = [[0 for j in range(c + 1)] for i in range(n + 1)] # 将列表初始化，res列表多增加一行一列。

    res = bag(n, c, w, v,res)
    show(n, c, w, res)


'''
10
5
6 3 5 4 6
2 2 6 5 4
'''


```



#【问题描述】使用贪心算法求解Huffman编码问题，具体来说就是，根据每个字符的出现频率，使用最小堆构造最小优先队列，构造出字符的最优二进制表示，即前缀码。在程序开始说明部分，简要描述使用贪心算法求解Huffman编码问题的算法过程。

【输入形式】在屏幕上输入字符个数和每个字符的频率。

【输出形式】每个字符的Huffman编码。

【样例输入】

 6

 45 13 12 16 9 5

【样例输出】

a 0

b 101

c 100

d 111

e 1101

f 1100

【样例说明】

 输入：字符个数为6，a至f每个字符的频率分别为：45, 13, 12, 16, 9, 5。

 输出：每个字符对应的Huffman编码。

```python
'''
简述贪心算法求解Huffman编码问题的算法过程
    构造哈夫曼树
        1：声明一个结点类，类中只包含参数，类似于C中的结构体。类中的参数有 字符名，字符的权值，右孩子，左孩子
        2：将用户输入的字符及其权值来实例化结点类的对象，并将这些类放在堆中
        3：取最小堆中的俩个类的权值最小的类的作为父节点的左右孩子 // 贪心算法体现点
        4：循环做第3步，直至列表总仅有一个类，哈夫曼树构造完成
    递归读取编码
        1：采用递归的方式，从根节点往叶子结点读
        2：每次读取左孩子则记0，读取右孩子则记1
        3：最后顺序打印出各字符的编码结果
'''

import numpy as np

global List  # 设置全局变量用来以存各字符的哈夫曼编码，存储单位为coding类
List = []  # 初始化全局变量


class coding():
    def __init__(self, Str, Coding):  # coding类 存储字符及其编码
        self.Str = Str
        self.Coding = Coding


class Node():  # 根据步骤一：声明结点类
    def __init__(self, name=None, value=None):
        self._name = name  # 字符
        self._value = value  # 字符的权值
        self._left = None  # 左孩子
        self._right = None  # 右孩子


# 哈夫曼树类
class HuffmanTree():
    # 根据Huffman树的思想：以叶子节点为基础，反向建立Huffman树
    def __init__(self, char_weights, n):  # 根据步骤二：实例化结点类
        self.a = [Node(part[0], part[1]) for part in char_weights]  # 利用for循环解析列表中的列表。并将解析结果传入node类来创建结点

        while len(self.a) != 1:  # 构造最小堆，将最小俩结点pop出，重新再构造
            self.a.sort(key=lambda node: node._value, reverse=True)
            c = Node(value=(self.a[-1]._value + self.a[-2]._value))  # c永远指向根结点
            c._left = self.a.pop(-1)
            c._right = self.a.pop(-1)
            self.a.append(c)
        self.root = self.a[0]
        self.b = list(range(n))  # self.b用于保存每个叶子节点的Haffuman编码,range的值只需要不小于树的深度就行


def pre(self, tree, length):  # 利用递归遍历哈夫曼树
    node = tree
    if (not node):  # 叶子结点左右孩子为NULL，以此作为递归终止信号
        return
    elif node._name:  # 只有叶子结点的name属性是非空的，作为记录字符
        cod = coding(node._name, self.b[:length])
        List.append(cod)
        List.sort(key=lambda cod: cod.Str)
        return
    self.b[length] = 0  # 向左遍历即编码记0
    pre(self, node._left, length + 1)
    self.b[length] = 1  # 向右遍历即编码记1
    pre(self, node._right, length + 1)


n = int(input())  #记录字符个数
a = 97
char_weights = []   # 待记录字符和权值
power = np.array(input().split(), dtype=int)
for i in range(n):
    char_weights.append([chr(a + i), power[i]]) # 构建如下的char_weights
# char_weights = [['a', 45], ['b', 13], ['c', 12], ['d', 16], ['e', 9], ['f', 5]]

tree = HuffmanTree(char_weights, n)  #构造哈夫曼树
pre(tree, tree.root, 0)  # 遍历哈夫曼树记录编码结果

for i in range(len(List)):   # 解析全局变量List打印出符合要求的结果，List中存储的都是coding类
    print(List[i].Str, end=' ')
    for j in range(len(List[i].Coding)):
        print(List[i].Coding[j], end='')
    print()
'''
a 0

b 101

c 100

d 111

e 1101

f 1100'''



```



#【问题描述】Dijkstra算法解决的是带权重的有向图上单源最短路径问题。所有边的权重都为非负值。设置顶点集合S并不断地作贪心选择来扩充这个集合。使用最小堆数据结构构造优先队列。

【输入形式】在屏幕上输入顶点个数和连接顶点间的边的权矩阵。

【输出形式】从源到各个顶点的最短距离及路径。

【样例输入】

5

0 10 0 30 100

0 0 50 0 0

0 0 0 0 10

0 0 20 0 60

0 0 0 0 0

【样例输出】

10: 1->2

50: 1->4->3

30: 1->4

60: 1->4->3->5

【样例说明】

 输入：顶点个数为5。连接顶点间边的权矩阵大小为5行5列，位置[i,j]上元素值表示第i个顶点到第j个顶点的距离，0表示两个顶点间没有边连接。

 输出：每行表示源1到其余各顶点的最短距离及路径。

```python
# 1：每次在图中寻找距离最小值（使用最小堆）
# 2：将距离最小的那个顶点收纳，并更新定点到其他各点的距离
# 3：重复做1，2，直至所有的顶点都被收纳
# 4：打印出所有的路径


import numpy as np
import heapq
global Max_value
Max_value = 9999


def DJSTL(Graph, dist):
    dist1 = dist.copy
    print(dist1)
    collect = np.zeros(n, dtype=int)  # 记录节点是否被收录，全部初始为0
    while True:
        if dist == []:
            break

        heapq.heapify(dist)
        Min = heapq.heappop(dist)

        for i in dist1:
            if Min == dist1[int(i)]:
                break
        collect[i] = 1  # 找到最小的节点

        for k in Graph[i]:
            if k != 9999:
                if collect[k] == 0:
                    if dist1[i] + Graph[i, k] < dist1[k]:
                        dist1[k] = dist1[i] + Graph[i, k]
                        path[k] = i
        dist = dist1.copy
        for i in range(-n,0):
            if collect[abs(i)] == 0:
                del dist[abs(i)]

    print(dist1)
    print(path)

def prit(q):  # 指定打印路径
    x = len(q)
    for v in range(x):
        if (x != 1):
            print(q[v] + 1, end='')
            if (v != x - 1):
                print('->', end='')
            else:
                print('')


def result(cost, path):  # 按要求格式打印
    for q in range(1, len(cost)):
        print(cost[q], end=': ')
        for p in path:
            if (path.index(p) == q):
                prit(p)


def Djstl(Graph):  # 单源最短路径返回路径长度和路径列表
    collect = [0]  # 列表中0，1代表有无被收录
    cost = [Max_value] * len(Graph)
    cost[0] = 0
    parents = [[]] * len(Graph)  # 初始化路径列表
    parents[0] = [0]

    while len(collect) < len(Graph):  # 循环直到所有的顶点都被收录到：collect的长度和a的长度一致
        Min_value = Max_value + 1
        col = -1
        row = -1

        for i in collect:
            cd = [x for x in range(n) if x not in collect]
            for j in cd:
                if Graph[i][j] + cost[i] < Min_value:  # 如何被收录的顶点导致其他顶点路径变小，则更新其他顶点路径长度
                    Min_value = Graph[i][j] + cost[i]
                    row = i
                    col = j

        if col == -1:
            break
        if row == -1:
            break
        collect.append(col)  # 收录顶点
        cost[col] = Min_value
        parents[col] = parents[row][:]  # 记录此路径上一个顶点
        parents[col].append(col)
    return cost, parents


if __name__ == '__main__':

    n = int(input())  # 顶点个数

    Graph = np.zeros((n, n), dtype=int)  # 存储图的信息
    for i in range(n):
        Graph[i, :] = np.array(input().split(), dtype=int)

    for j in range(n): # 处理图的信息，将0变为最大值Max-value
        for t in range(n):
            if j != t:
                if Graph[j, t] == 0:
                    Graph[j, t] = Max_value

    cost, path = Djstl(Graph)
    result(cost, path)

    '''costs
[6, 0, 5, 10, 15, 10, 8]
parents
[2, -1, 1, 1, 5, 0, 0]

'''
'''
【样例输入】

5

0 10 0 30 100
0 0 50 0 0
0 0 0 0 10
0 0 20 0 60
0 0 0 0 0


【样例输出】

10: 1->2

50: 1->4->3

30: 1->4

60: 1->4->3->5
'''


```

#【问题描述】Prim算法解决的是带权重的无向图上连接所有顶点的耗费最小的生成树。

【输入形式】在屏幕上输入顶点个数和连接顶点间的边的权矩阵。

【输出形式】从源到各个顶点的最短距离及路径。

【样例输入】

8

0 15 7 0 0 0 0 10

15 0 0 0 0 0 0 0

7 0 0 9 12 5 0 0 

0 0 9 0 0 0 0 0

0 0 12 0 0 6 0 0

0 0 5 0 6 0 14 8

0 0 0 0 0 14 0 3

10 0 0 0 0 8 3 0

【样例输出】

15: 1<-2

7: 1<-3

9: 1<-3<-4

6: 1<-3<-6<-5

5: 1<-3<-6

3: 1<-3<-6<-8<-7

8: 1<-3<-6<-8

【样例说明】

 输入：顶点个数为8。连接顶点间边的权矩阵大小为8行8列，位置[i,j]上元素值表示第i个顶点到第j个顶点的距离，0表示两个顶点间没有边连接。

 输出：每行表示其余各顶点的值及其到起始点1的路径。



```python
import numpy as np
import heapq
global Max_value
Max_value = 9999


def prim(Graph, start):
    collect = [start]  # 已找到最短路径的节点
    cost = [Max_value] * len(Graph)
    cost[start] = 0
    parents = [[]] * len(Graph)# 初始化路径列表
    parents[start] = [start]

    while len(collect) < len(Graph):  # 当已找到最短路径的节点小于n时
        Min_value = Max_value + 1
        col = -1
        row = -1
        for f in collect:
            cd = [x for x in range(n) if x not in collect]
            for i in cd:
                if a[f][i] < Min_value:
                    Min_value = a[f][i]
                    row = f
                    col = i
        if col == -1:
            break
        if row == -1:
            break  # 若没找出最小值且节点还未找完，说明图中存在不连通的节点

        collect.append(col)
        cost[col] = Min_value
        parents[col] = parents[row][:]
        parents[col].append(col)
    return cost, parents

def DJSTL(Graph, dist):
    dist1 = dist.copy
    print(dist1)
    collect = np.zeros(n, dtype=int)  # 记录节点是否被收录，全部初始为0
    while True:
        if dist == []:
            break

        heapq.heapify(dist)
        Min = heapq.heappop(dist)

        for i in dist1:
            if Min == dist1[int(i)]:
                break
        collect[i] = 1  # 找到最小的节点

        for k in Graph[i]:
            if k != 9999:
                if collect[k] == 0:
                    if dist1[i] + Graph[i, k] < dist1[k]:
                        dist1[k] = dist1[i] + Graph[i, k]
                        path[k] = i
        dist = dist1.copy
        for i in range(-n,0):
            if collect[abs(i)] == 0:
                del dist[abs(i)]

    print(dist1)


def result(cost):
    for i in range(1, len(cost)):
        print(str(cost[i]) + ':', end=' ')
        for d in parents:
            if parents.index(d) == i:
                prit(d)


def prit(d):
    for i in range(len(d)):
        if len(d) != 1:
            print(d[i] + 1, end='')
            if i != len(d) - 1:
                print('<-', end='')
            else:
                print()


if __name__ == '__main__':
    n = int(input())    # 顶点个数

    a = np.zeros((n, n), dtype=np.int)  # 存储图
    for r in range(n):
        a[r, :] = np.array(input().split(), dtype=np.int)

    for i in range(n):  # 将题中所给的非直接联通的改成Max-value
        for j in range(n):
            if (i != j and a[i, j] == 0):
                a[i, j] = Max_value

    cost, parents = prim(a, 0)
    result(cost)

'''
15: 1<-2

7: 1<-3

9: 1<-3<-4

6: 1<-3<-6<-5

5: 1<-3<-6

3: 1<-3<-6<-8<-7

8: 1<-3<-6<-8



0 15 7 0 0 0 0 10
15 0 0 0 0  0 0 0
7 0 0 9 12 5 0 0 
0 0 9 0 0 0 0 0
0 0 12 0 0 6 0 0
0 0 5 0 6 0 14 8
0 0 0 0 0 14 0 3
10 0 0 0 0 8 3 0
'''



```



#【问题描述】采用宽度优先搜索算法求解TSP问题，并在搜索过程中，使用界限条件（当前结点已经走过的路径长度要小于已求得的最短路径）进行“剪枝”操作（不再对后续结点进行遍历），从而提高搜索效率。采用queue模块中的队列（Queue）来实现宽度优先搜索。

【输入形式】在屏幕上输入顶点个数和连接顶点间的边的邻接矩阵。

【输出形式】最优值和其中一条最优路径。

【样例输入】

4

0 30 6 4

30 0 5 10

6 5 0 20

4 10 20 0

【样例输出】

[1]

[1, 2]

[1, 3]

[1, 4]

[1, 2, 3]

[1, 2, 4]

[1, 3, 2]

[1, 3, 4]

[1, 4, 2]

[1, 4, 3]

[1, 2, 3, 4]

[1, 2, 4, 3]

[1, 3, 2, 4]

[1, 3, 4, 2]

[1, 4, 2, 3]

[1, 4, 3, 2]

25: [1, 3, 2, 4]

【样例说明】

 输入：顶点个数为4。连接顶点间边的邻接矩阵大小为4行4列，位置[i,j]上元素值表示第i个顶点到第j个顶点的距离，0表示两个顶点间没有边连接。

 输出：最优值为25，最优路径为[1, 3, 2, 4]。

```python
'''
1：根据图生成树（节点使用类替代，类有三个属性，self.name：此节点编号，self.child:此节点的子节点，self.power:到此节点的路径长度）
2: 根据 1 中生成的树，读树，如果遇到到当前节点的路径长加到子节点路径长大于已知最优解，则停止，return；
3：按照格式，打印出最优解，最优路径

'''

import numpy as np


class Node():
    def __init__(self, name, child, power, parent):
        self.name = name  # name 表示此地点编号
        self.child = child  # chlid 是的列表用来存储子节点 Node
        self.power = power  # power 是个数值用来存储路径长度 即到此点的路径长度
        self.parent = parent


def buildTree(root, Graph):
    # print('正在进行', root.name, '子节点的判断')
    nd = len(Graph[1:2, :][0])
    for i in range(nd):
        if Graph[root.name][i]:
            ChildNode = Node(i, [], Graph[root.name, i:i + 1][0], root)

            Graph[:, root.name] = 0  # 任何子节点将不能再回父节点

            root.child.append(ChildNode)  # 向父节点添加刚实例化的子节点

            # print('这是', root.name, '的子节点编号', root.child[len(root.child) - 1].name)
            GraphCopy = Graph.copy()
            # print('这是处理后将再次递归的Graph：', Graph, '\n', '将被', i + 1, '使用', end='\n\n')

            buildTree(ChildNode, GraphCopy)
    if root.child == []:
        # print('这是结束递归时的Graph', Graph)
        return


def Correct(close1):  # 我对每个子节点编号是0 1 2 3，打印时全加一。
    for i in range(len(close1)):
        close1[i] += 1
    return close1


def ReadTree(root, power):
    child = []
    powerNext = []
    for i in range(len(root)):  # 父节点列表
        for j in root[i].child:  # 单个父节点的子节点J
            child.append(j)
            powerNext.append(power[i] + j.power)

    childCp = child.copy()

    for i in range(len(child)):
        a = [0]
        while (childCp[i].name):
            a.insert(1, childCp[i].name)
            childCp[i] = childCp[i].parent
        print(Correct(a))

    if child == []:
        for i in range(len(power)):
            if MIN[0] > power[i] + Graph[root[i].name][0]:
                MIN[0] = Graph[root[i].name][0] + power[i]
                childCp = root.copy()
                cost[0]=[0]
                while (childCp[i].name):
                    cost[0].insert(1, childCp[i].name)
                    childCp[i] = childCp[i].parent
        return
    ReadTree(child, powerNext)


MIN = [9999]  # 全局变量，记录最优解
cost = [[0]]  # 记录最优值
n = int(input())

'''Graph = [
    [0, 30, 6, 4],
    [30, 0, 5, 10],
    [6, 5, 0, 20],
    [4, 10, 20, 0]
]'''
Graph = np.zeros((n, n), dtype=int)  # 记录路径图
for i in range(n):
    Graph[i, :] = np.array(input().split(), dtype=int)

Graph = np.array(Graph)
GraphCp = Graph.copy()  # 复制图

root = Node(0, [], 0, 0)  # 实例化子节点

buildTree(root, GraphCp)  # 建树
print([1])
ReadTree([root], [0])

print(MIN[0], end=': ')
print(Correct(cost[0]))

'''
[1]

[1, 2]

[1, 3]

[1, 4]

[1, 2, 3]

[1, 2, 4]

[1, 3, 2]

[1, 3, 4]

[1, 4, 2]

[1, 4, 3]

[1, 2, 3, 4]

[1, 2, 4, 3]

[1, 3, 2, 4]

[1, 3, 4, 2]

[1, 4, 2, 3]

[1, 4, 3, 2]

25: [1, 3, 2, 4]


'''
'''

4
0 30 6 4
30 0 5 10
6 5 0 20
4 10 20 0


'''



```



#【问题描述】采用深度优先搜索算法求解TSP问题，并在搜索过程中，使用界限条件（当前结点已经走过的路径长度要小于已求得的最短路径）进行“剪枝”操作（不再对后续结点进行遍历），从而提高搜索效率。采用queue模块中的栈（LifoQueue）来实现深度优先搜索。

【输入形式】在屏幕上输入顶点个数和连接顶点间的边的邻接矩阵。

【输出形式】最优值和其中一条最优路径。

【样例输入】

4

0 30 6 4

30 0 5 10

6 5 0 20

4 10 20 0

【样例输出】

[1]

[1, 2]

[1, 2, 3]

[1, 2, 3, 4]

[1, 2, 4]

[1, 3]

[1, 3, 2]

[1, 3, 2, 4]

[1, 3, 4]

[1, 4]

[1, 4, 2]

[1, 4, 2, 3]

[1, 4, 3]

25: [1, 3, 2, 4]

【样例说明】

 输入：顶点个数为4。连接顶点间边的邻接矩阵大小为4行4列，位置[i,j]上元素值表示第i个顶点到第j个顶点的距离，0表示两个顶点间没有边连接。

 输出：最优值为25，最优路径为[1, 3, 2, 4]。

```python
'''
1：根据图生成树（节点使用类替代，类有三个属性，self.name：此节点编号，self.child:此节点的子节点，self.power:到此节点的路径长度）
2: 根据 1 中生成的树，读树，如果遇到到当前节点的路径长加到子节点路径长大于已知最优解，则停止，return；
3：按照格式，打印出最优解，最优路径

'''

import numpy as np


class Node():
    def __init__(self, name, child, power):
        self.name = name  # name 表示此地点编号
        self.child = child  # chlid 是的列表用来存储子节点 Node
        self.power = power  # power 是个数值用来存储路径长度 即到此点的路径长度


def buildTree(root, Graph):
    # print('正在进行', root.name, '子节点的判断')
    nd = len(Graph[1:2, :][0])
    for i in range(nd):
        if Graph[root.name][i]:
            ChildNode = Node(i, [], Graph[root.name, i:i + 1][0])

            Graph[:, root.name] = 0  # 任何子节点将不能再回父节点

            root.child.append(ChildNode)  # 向父节点添加刚实例化的自节点

            # print('这是', root.name, '的子节点编号', root.child[len(root.child) - 1].name)
            GraphCopy = Graph.copy()
            # print('这是处理后将再次递归的Graph：', Graph, '\n', '将被', i + 1, '使用', end='\n\n')

            buildTree(ChildNode, GraphCopy)
    if root.child == []:
        # print('这是结束递归时的Graph', Graph)
        return


def read(root, close, power):
    close.append(root.name)  # 将当前的点加入到close中

    closeCp = close.copy()
    print(Correct(closeCp))  # 按照要求打印close路径
    if len(close) >= n:
        if power + Graph[root.name][0] < MIN[0]:  # 如果所有节点均被遍历，并且路径长小于最优解，便更新最优解和最优路径
            cost[0] = Correct(close.copy())
            MIN[0] = power + Graph[root.name][0]

        return

    if power + root.child[0].power >= MIN[0]:  # 剪枝 如果路径长度加到子节点路径长度则返回
        return

    for i in root.child:  # 遍历子节点
        powercopy = power
        powercopy += i.power
        closecopy = close.copy()
        read(i, closecopy, powercopy)
    return


def Correct(close1):  # 我对每个子节点编号是0 1 2 3，打印时全加一。
    for i in range(len(close1)):
        close1[i] += 1
    return close1


MIN = [9999]  # 全局变量，记录最优解
cost = [[]]  # 记录最优值
n = int(input())

'''Graph = [
    [0, 30, 6, 4],
    [30, 0, 5, 10],
    [6, 5, 0, 20],
    [4, 10, 20, 0]
]'''
Graph = np.zeros((n, n), dtype=int)  # 记录路径图
for i in range(n):
    Graph[i, :] = np.array(input().split(), dtype=int)

Graph = np.array(Graph)
GraphCp = Graph.copy()  # 复制图

root = Node(0, [], 0)  # 实例化子节点

buildTree(root, GraphCp)  # 建树

read(root, [], 0)  # 读树

print(MIN[0], end=': ')
print(cost[0])

'''
4
0 30 6 4
30 0 5 10
6 5 0 20
4 10 20 0
'''

'''
[1]

[1, 2]

[1, 2, 3]

[1, 2, 3, 4]

[1, 2, 4]

[1, 3]

[1, 3, 2]

[1, 3, 2, 4]

[1, 3, 4]

[1, 4]

[1, 4, 2]

[1, 4, 2, 3]

[1, 4, 3]

25: [1, 3, 2, 4]


'''



```



#【问题描述】采用优先队列搜索算法求解TSP问题，用一最小堆来存储活结点表，其优先级是结点的当前费用。并在搜索过程中，使用界限条件（当前结点已经走过的路径长度要小于已求得的最短路径）进行“剪枝”操作（不再对后续结点进行遍历），从而提高搜索效率。采用heapq模块来实现最小堆。

【输入形式】在屏幕上输入顶点个数和连接顶点间的边的邻接矩阵。

【输出形式】搜索过程，最优值和其中一条最优路径。

【样例输入】

4

0 30 6 4

30 0 5 10

6 5 0 20

4 10 20 0

【样例输出】

[1]

[1, 4]

[1, 3]

[1, 3, 2]

[1, 4, 2]

[1, 4, 2, 3]

[1, 3, 2, 4]

[1, 4, 3]

[1, 3, 4]

[1, 2]

25: [1, 4, 2, 3]

【样例说明】

 输入：顶点个数为4。连接顶点间边的邻接矩阵大小为4行4列，位置[i,j]上元素值表示第i个顶点到第j个顶点的距离，0表示两个顶点间没有边连接。

 输出：在整个算法过程中的先后搜索路径，最优值为25，最优路径为[1, 4, 2, 3]。

```python
'''
1：根据图生成树（节点使用类替代，类有三个属性，self.name：此节点编号，self.child:此节点的子节点，self.power:到此节点的路径长度）
2: 根据 1 中生成的树，读树，如果遇到到当前节点的路径长加到子节点路径长大于已知最优解，则停止，return；
3：按照格式，打印出最优解，最优路径

'''

import numpy as np
import heapq as hq


class Node():
    def __init__(self, name, child, power, parent):
        self.name = name  # name 表示此地点编号
        self.child = child  # chlid 是的列表用来存储子节点 Node
        self.power = power  # power 是个数值用来存储路径长度 即到此点的路径长度
        self.parent = parent

    def __lt__(self, other):
        return self.power < other.power


def buildTree(root, Graph):
    # print('正在进行', root.name, '子节点的判断')
    nd = len(Graph[1:2, :][0])
    for i in range(nd):
        if Graph[root.name][i]:
            ChildNode = Node(i, [], Graph[root.name, i:i + 1][0], root)

            Graph[:, root.name] = 0  # 任何子节点将不能再回父节点

            root.child.append(ChildNode)  # 向父节点添加刚实例化的自节点

            # print('这是', root.name, '的子节点编号', root.child[len(root.child) - 1].name)
            GraphCopy = Graph.copy()
            # print('这是处理后将再次递归的Graph：', Graph, '\n', '将被', i + 1, '使用', end='\n\n')

            buildTree(ChildNode, GraphCopy)
    if root.child == []:
        # print('这是结束递归时的Graph', Graph)
        return


def read(open):
    MIN = 9999
    while open:
        p = 0
        

        open.sort(key=lambda node: node.power)  # 将列表排序并弹出最小值


        if len(open) >= 2:
            if open[0].power == open[1].power:
                a = open.pop(1)
            else:
                a = open.pop(0)
        else:
            a = open.pop(0)
        # 节点是否是叶子节点
        if a.child == []:
            if MIN > a.power + Graph[a.name][0]:  # 看弹出来的节点是否是叶子节点，是则更新存储最小值
                MIN = a.power + Graph[a.name][0]
                p = 1
        # 节点是否是倒数第二并且距离大于当前最优解
        else:
            if a.child[0].child == []:
                if a.power + a.child[0].power + Graph[a.child[0].name][0] > MIN:
                    spent = [0]

                    # 打印弹出来的节点路径
                    while (a.name):
                        spent.insert(1, a.name)
                        a = a.parent
                    print(Correct(spent), end='\n')
                    # 结束当前循环
                    continue

        if a.power > MIN:
            spent = [0]
            while (a.name):
                spent.insert(1, a.name)
                a = a.parent
            print(Correct(spent), end='\n')
            continue

        # 更新open表 也更新新加节点的power值
        for i in a.child:
            i.power += i.parent.power
            hq.heappush(open, i)

        spent = [0]
        while a.name:
            spent.insert(1, a.name)  # 打印弹出来的节点路径
            a = a.parent

        print(Correct(spent), end='\n')
        if p == 1:
            cost[0] = spent

        # Pop = []
        # for i in range(len(open)):
        #   if (open[i].power > MIN):  # 进行剪枝
        #      Pop.append(i)
        # Pop.sort(reverse=True)
        # for i in Pop:
        #   print(open[i].name)
        #  open.pop(i)

    return MIN


def Correct(close1):  # 我对每个子节点编号是0 1 2 3，打印时全加一。
    for i in range(len(close1)):
        close1[i] += 1
    return close1


cost = [[]]  # 记录最优值
n = int(input())

'''Graph = [
    [0, 30, 6, 4],
    [30, 0, 5, 10],
    [6, 5, 0, 20],
    [4, 10, 20, 0]
]'''
Graph = np.zeros((n, n), dtype=int)  # 记录路径图
for i in range(n):
    Graph[i, :] = np.array(input().split(), dtype=int)

Graph = np.array(Graph)
GraphCp = Graph.copy()  # 复制图

root = Node(0, [], 0, 0)  # 实例化子节点

buildTree(root, GraphCp)  # 建树

M = read([root])  # 读树

print(M, end=': ')
print(cost[0])

'''
4
0 30 6 4
30 0 5 10
6 5 0 20
4 10 20 0

[1]
[1, 2]
[1, 4]
[1, 2, 3]
[1, 2, 4]
[1, 3]
[1, 4, 3]
[1, 2, 4, 3]
[1, 3, 4]
[1, 2, 3, 4]
[1, 4, 2]
[1, 3, 2]
[1, 4, 3, 2]
[1, 3, 4, 2]
13: [1, 2, 3, 4]
'''




```

