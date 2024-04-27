---
title: 二叉排序树
date: 2024-04-28 06:03:09
tags:
---

## 二叉排序树是啥
是一个具有以下性质的二叉树：
- 左子树小于根节点
- 右子树大于根节点
- 中序遍历二叉树得到有序数列

## 如何插入新节点
二叉排序树是一个**动态**树，节点的生成是动态的。生成二叉排序树的过程就是无序数列变为有序数列的过程。这个过程是：

1. 查找树，查找失败的时候插入


2. 指针返回插入位置（左子树或右子树）

代码：
```c
Status insert_bst(BinaryTree *T, ElemType e)
{
    KeyType key;
    BinaryTree p;

    if (!search_bst(T, e.key, NULL, p)) //查找失败
    {
        BinaryTree s = (BinaryTree)malloc(sizeof(BinaryTreeNode));
        s->data = e;
        s->Lchild = s->Rchild = NULL;
        
        if (!p) 
            T = s; //新节点变为新的根节点(空树的时候)
        else if (LQ(e.key, p->data.key))
        {
            p->Lchild = s; //新节点为左子树
        }
        else
        {
            p->Rchild = s; //新节点为右子树
        }
        return 0;
    }
    else
    {
        return -1; //查找成功，不插入
    }
}
```

查找树的代码如下：
```c
Status search_bst(BinaryTree T, KeyType key, BinaryTree parents, BinaryTree *ptr)
{
    if (!T)
    {
        ptr = parents;
        return -1;
    }
    else if (EQ(key, T->data.key)) // 查找成功
    {
        ptr = T;
        return 0;
    }
    else if (LT(key, T->data.key))
    {
        return search_bst(T->Lchild, key, T, &ptr); //查找左子树
    }
    else
    {
        return search_bst(T->Rchild, key, T, &ptr); //查找右子树
    }
}
```

## 如何删除节点

删除节点分以下情况：

- 节点的**左右子树为空**的时候  ---  修改父节点指针
- 节点的**左子树或右子树为空**的时候  --- 令节点的左子树或右子树成为父节点的左子树或右子树
- 节点的**左右子树不为空**的时候 --- 前驱代替被删节点，删去前驱。前驱的左子树变为父节点的右子树，右变为父的左子树。

代码如下：
```c
Status delete_bst(BinaryTree *T, KeyType key)
{
    if (!T)
    {
        return -1;
    }
    else
    {
        if (EQ(key, (*T)->data.key))
        {
            return delete (&T);
        }
        else if (LT(key, (*T)->data.key))
        {
            /* code */
            return delete_bst((*T)->Lchild, key);
        }
        else
        {
            return delete_bst((*T)->Rchild, key);
        }
    }
}


Status delete(BinaryTree *p)
{
    if (!(*p)->Rchild)
    {
        BinaryTree q = p;
        p = (*p)->Lchild;
        free(q);
    }
    else if (!(*p)->Lchild)
    {
        BinaryTree q = p;
        p = (*p)->Rchild;
        free(q);
    }
    else // 左右子树均不为空
    {
        BinaryTree q = p;
        BinaryTree s = (*p)->Lchild;
        while (s->Rchild)
        {
            q = s;
            s = s->Rchild;
        } // 转左，然后向右走到头（找到直接前驱）

        (*p)->data = s->data; // s指向被删节点的前驱

        if (q != p)
        {
            // 重接右子树
            q->Rchild = s->Lchild;
        }
        else
        {  //重接左子树
            q->Lchild = s->Lchild;
        }
        free(s);
    }
}
```
所谓“前驱”，指的是在中序遍历数列中，节点的直接前驱。

为什么转左，向右走到头就能找到直接前驱呢？
因为这样可以找到被删节点左子树中最大的节点，在中序遍历数列中即比被删节点小，即被删节点前驱。

## 静态查找

二叉排序树是动态查找，理解这个玩意还是有难度的。静态查找还是比较好理解的：

比如：
```c
// 遍历
int search_sq(SStable ST, KeyType key) {
    int i;
    ST.elem[0].key = key; // 设置监视哨
    for (size_t i = ST.length; ST.elem[i].key != key; i--);
    return i;
}

// 二分查找
int search_bin(SStable ST, KeyType key) {
    int low = 1;
    int high = ST.length;

    while (low <= high) {
        int mid = low + high / 2;
        if (EQ(key, ST.elem[mid].key)) {
            return mid;
        } else if (LT(key, ST.elem[mid].key)) {
            high = mid - 1;
        } else {
            low = mid + 1;
        }
    }

    return 0;
}
```

用到的数据类型：
```c
#include <stdlib.h>
#include <string.h>

// 关键字类型
typedef void *KeyType;
typedef void *InfoType;
typedef int Status;

// 对数值类关键字的比较
#define EQ(a, b) ((a) == (b))
#define LT(a, b) ((a) < (b))
#define LQ(a, b) ((a) <= (b))

// 对字符串类关键字的比较
#define EQ(a, b) (!strcmp((a), (b)))
#define LT(a, b) (strcmp((a), (b)) < 0)
#define LQ(a, b) (strcmp((a), (b)) <= 0)

// 数据元素类型
typedef struct ElemType {
    /* data */
    KeyType key;
    InfoType otherinfo;
} ElemType;

//顺序表
typedef struct SStable {
    /* data */
    ElemType *elem;
    int length;

} SStable;

typedef struct BinaryTreeNode {
    ElemType data;
    struct BinaryTreeNode *Lchild;
    struct BinaryTreeNode *Rchild;
} BinaryTreeNode;

//二叉树
typedef BinaryTreeNode *BinaryTree;
```

难理解的是二叉排序树的删除，先得需要知道中序遍历是啥。然后再用直接前驱（或直接后继）来代替被删节点。
