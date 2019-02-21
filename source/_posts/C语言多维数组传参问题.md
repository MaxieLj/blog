---
title: C语言多维数组传参问题
date: 2017-02-12 15:18:35
tags: c语言
categories: c语言
toc: true
---

刷题遇到个问题，需要传递一个二维数组作为实参。函数如下

```
int* spiralOrder(int** matrix, int matrixRowSize, int matrixColSize) {
}
```

遇到的问题是如果我直接`int matrix[3][3]`声明，然后传递参数，在`spiralOrder`无法用matrix[3][3]的形式调用。代码如下

```c
int* spiralOrder(int** matrix, int matrixRowSize, int matrixColSize) {
    int* ret;
    printf("matrix:%d\n",*(*(matrix+0)+2));
    printf("matrix:%d\n", matrix[2][2]);
    return  ret;
}

int main(){
    int c[3][3] = { {1,2,3} , {4,5,6}, {7,8,9}};

    spiralOrder((int**)c, 3, 3);
}
```
在`spiralOrder`打印的时候报错如下：

![image](/photo/img/c语言多维数组传参问题/bug1.png)
当然c primer也明确指出，数组当做实参传递时，当做指针处理。所以说上述问题在于把数组当成二重指针处理当然没法处理了。以为数组可以当做指针，其实二维数组也是可以当做指针。
那么二维数组如何作为实参传递呢？
## 方法一
> 多维数组以指向 0 号元素的指针方式传递。多维数组的 元素本身就是数组。除了第一维以外的所有维的长度都是元素类型的一部分，必须明确指定.---C++ Primer

### 代码
```
void spiralOrder1(int matrix[][3], int matrixRowSize, int matrixColSize) {
    printf("matrix:%d\n", matrix[0][0]);
    printf("matrix:%d\n", matrix[1][1]);
    printf("matrix:%d\n", matrix[2][2]);
    return ;
}
int main(){
    int c[3][3] = { {1,2,3} , {4,5,6}, {7,8,9}};
    spiralOrder1(c, 3, 3);
}
```
因为形参是`(int matrix[][3]`所以编译器知道它按照二维数组的方式寻址。

### 返回结果
![image](/photo/img/c语言多维数组传参问题/result2.png)

## 方法二——指针的形式

```c
void spiralOrder1(int* matrix, int matrixRowSize, int matrixColSize) {
    printf("matrix:%d\n", matrix[0]);
    return ;
}

int main(){
    int c[3][3] = { {1,2,3} , {4,5,6}, {7,8,9}};
    spiralOrder1((int*)c, 3, 3);
}
```
![image](/photo/img/c语言多维数组传参问题/result1.png)

所以当形参是`int*`的情况下，二维数组可以通过[i]的形式访问，也可以通过*(i*j+j)(规整的二维数组)的方式访问，其中i、j可以通过传参的形式传入。
**原因是直接定义的数组是在程序的堆栈区，数据占用连续的空间，所以可以使用上述方式寻址。**


## 方法三
如果上述都不满足需求的话，还有一种方式——通过二级指针的形式。

### 代码
```
int* spiralOrder(int** matrix, int matrixRowSize, int matrixColSize) {
    int* ret;
    printf("matrix:%d\n",*(*(matrix+0)+2));
    printf("matrix:%d\n", matrix[2][2]);
    return  ret;
}
int main(){
    int** a = (int**)malloc(sizeof(int*)*9);
    int** current = a;
    for(int i = 0; i < 3; i++)
    {
        int *tmp = (int*)malloc(sizeof(int)*3);
        *a = tmp;
        a++;
        for(int j = 0; j < 3; j++)
        {
            (*tmp) = j+i;
            tmp++;
        }
    }
    spiralOrder(current, 3, 3);
}
```

### 返回结果

![image](/photo/img/c语言多维数组传参问题/result3.png)
返回结果
如果形参是二级指针的形式，可以通过`[][]`的形式访问数组，当然这里必须要控制二维数组范围。也可以通过`*(*(matrix+0)+2))`方式访问。是否可以像上一个方法中提到的 通过连续的内存地址 去访问？

### 代码

```c
int* spiralOrder(int** matrix, int matrixRowSize, int matrixColSize) {
    int* ret;
    printf("matrix:%d\n", *(*(matrix+2)));
    printf("matrix:%d\n", *(*(matrix+4)));
    printf("matrix:%d\n", matrix[0]);
    return  ret;
}
int main(){
    int** a = (int**)malloc(sizeof(int*)*9);
    int** current = a;
    for(int i = 0; i < 3; i++)
    {
        int *tmp = (int*)malloc(sizeof(int)*3);
        *a = tmp;
        a++;
        for(int j = 0; j < 3; j++)
        {
            (*tmp) = j+i;
            tmp++;
        }
    }
    spiralOrder(current, 3, 3);
}
```

返回结果
![image](/photo/img/c语言多维数组传参问题/result4.png)

首先它是一个二维数组以一维数组的形式访问打印出来是一个内存地址。为什么`*(*(matrix+2))`打印出来是对的数值，而`*(*(matrix+4))`却是内存地址呢？ 因为**malloc动态申请出来的数组是在系统的远堆上（far heap）,元素不是连续的，导致无法按照连续内存访问**

## 最后一个问题

那么二维数组和二级指针是什么关系呢？？？

首先声明一个二维数组和一个二维指针，我们把它打印出来
### 代码
```c
int main(){
    int c[3][3] = { {1,2,3} , {4,5,6}, {7,8,9}};
    int** a = (int**)malloc(sizeof(int*)*9);
    int** current = a;
    for(int i = 0; i < 3; i++)
    {
        int *tmp = (int*)malloc(sizeof(int)*3);
        *a = tmp;
        a++;
        for(int j = 0; j < 3; j++)
        {
            (*tmp) = j+i;
            tmp++;
        }
    }
    printf("current%p\n", current);
    printf("current0%p\n", current[0]);
    printf("c:%p\n", c);
    printf("c:%p\n", c+1);
    printf("c0:%p\n", c[0]);
    printf("c0:%p\n", c[0]+1);
    printf("c00:%d\n", c[0][0]);
    printf("c00:%p\n", &c[0][0]);
}
```
### 返回结果

![image](/photo/img/c语言多维数组传参问题/result5.png)

首先数组`c`存储的是一个占用两个int大小对象的地址。`c[0]`存储的是一个占用一个int大小对象的地址。因为`c`与`c[0]`存储的都是数组元素的首地址所以所以`c`与`c[0]`所存地址相同，但是他们所存储对象的大小不同。从输出的**c+1**和**c[0]+1**就可以看出来（这里的+1是指按照一个对象长度寻址）

再看看malloc申请出来的二级指针，压根`current` 与`current[0]`所存储的地址不一样，所以肯定无法像直接申请数组按照连续内存寻址找到所需数值。