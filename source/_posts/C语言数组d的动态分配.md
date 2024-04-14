---
title: C语言动态数组的创建
date: 2024-04-14 21:59:19
tags:
---

## 内存地址不连续

创建动态数组，如果是用for循环嵌套的话，创建出来的数组地址不连续，比如：
```c++
#include <iostream>

#define ROW 2
#define COL 3

//-----------分配出来的数组地址不连续-----------------

void delete_array(int **p, size_t dim1);

int **create_array(size_t dim1, size_t dim2);

/**
 * 测试
 * @return
 */
int main() {
    int **m;
    m = create_array(ROW, COL);
    // 初始化
    for (size_t i = 0; i < ROW; i++)
        for (size_t j = 0; j < COL; j++) {
            m[i][j] = i + j;
        }
    // 打印
    for (int i = 0; i < ROW; i++) {

        printf("m[%d] = %p\n", i, m[i]);

        for (int j = 0; j < COL; j++) {
            printf("&m[%d][%d] = %p; m[%d][%d] = %d\n", i, j, &(m[i][j]), i, j, m[i][j]);
        }

        printf("\n");
    }

    delete_array(m, ROW);
    return 0;
}


int **create_array(size_t dim1, size_t dim2) {
    int **p;

    if ((p = (int **) malloc(dim1 * sizeof(int *))) == NULL)
        return NULL;
    for (int i = 0; i < dim1; i++) {
        if ((p[i] = (int *) malloc(dim2 * sizeof(int))) == NULL)
            return NULL;
    }

    return p;
}

void delete_array(int **p, size_t dim1) {
    for (int i = 0; i < dim1; i++) {
        free(p[i]);
    }

    free(p);
}
```

运行结果,行与行之间地址不连续：
```
 m[0] = 0000018e68931460
 &m[0][0] = 0000018e68931460; m[0][0] = 0
 &m[0][1] = 0000018e68931464; m[0][1] = 1
 &m[0][2] = 0000018e68931468; m[0][2] = 2
 
 m[1] = 0000018e68931480
 &m[1][0] = 0000018e68931480; m[1][0] = 1
 &m[1][1] = 0000018e68931484; m[1][1] = 2
 &m[1][2] = 0000018e68931488; m[1][2] = 3
```

内存不连续导致 `*(*(a+i)+j)` 的值就不等于 `a[i][j]` 了。

## 让它连续

修改create_array函数：
```c++
int **create_array(size_t dim1, size_t dim2) {
    int **p;

    if ((p = (int **) malloc(dim1 * sizeof(int *))) == NULL)
        return NULL;

    if ((p[0] = (int *) malloc(dim1 * dim2 * sizeof(int))) == NULL)
        return NULL;

    for (size_t i = 1; i < dim1; i++)
        p[i] = p[i - 1] + dim2; //

    return p;
}
```

测试：
```c++
#include <iostream>

#define ROW 3
#define COL 2

int **create_array(size_t dim1, size_t dim2);

void delete_array(int **m);

int main() {
    int **m = create_array(ROW, COL);

    for (int i = 0; i < ROW; i++)
        for (int j = 0; j < COL; j++) {
            m[i][j] = i + j;
        }

    int *start = *m;
    int *const end = start + ROW * COL;

    for (; start != end; start++)
        printf("%p -> %d\n", start, *start);

    delete_array(m);
    return 0;
}

int **create_array(size_t dim1, size_t dim2) {
    int **p;

    if ((p = (int **) malloc(dim1 * sizeof(int *))) == NULL)
        return NULL;

    if ((p[0] = (int *) malloc(dim1 * dim2 * sizeof(int))) == NULL)
        return NULL;

    for (size_t i = 1; i < dim1; i++)
        p[i] = p[i - 1] + dim2; // Loc（A[i]）=Loc(A[1])+(i-1)*len;

    return p;
}

void delete_array(int **m) {
    free(m[0]);
    free(m);
}
```

运行结果：
```
0000010618531470 -> 0
0000010618531474 -> 1
0000010618531478 -> 1
000001061853147c -> 2
0000010618531480 -> 2
0000010618531484 -> 3
```
当然，这种方法得知道数组地址的计算公式：`Loc（A[i]）=Loc(A[1])+(i-1)*len`;

## 创建多维数组

多维数组空间的分配同理：
```c++
float ***create_multi_array(int dim1, int dim2, int dim3) {

    float ***ptr = (float ***) malloc(dim1 * sizeof(float **) +
                                      dim1 * dim2 * sizeof(float *) +
                                      dim1 * dim2 * dim3 * sizeof(float));

    for (size_t i = 0; i < dim1; ++i) {
        ptr[i] = (float **) (ptr + dim1) + i * dim2; //
        for (size_t j = 0; j < dim2; ++j)
            ptr[i][j] = (float *) (ptr + dim1 + dim1 * dim2) + i * dim2 * dim3 + j * dim3;
    }

    return ptr;
}
```
