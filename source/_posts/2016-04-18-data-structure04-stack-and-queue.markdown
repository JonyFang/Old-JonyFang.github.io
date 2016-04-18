---
layout: post
title: "数据结构浅析（四）：栈和队列"
date: 2016-04-18 14:30:23 +0800
comments: true
categories: 
---

什么是栈？
栈有哪些存储方式？
栈有哪些应用？
<br>
什么是队列？
队列有哪些存储方式？
<br>
本篇总结了这几个问题
{:.info}


* list element with functor item
{:toc}


<!-- more -->

#		1.栈

##		1.1.栈的定义

栈（stack）是限定仅在表尾（栈顶 top）进行插入和删除操作的后进先出的线性表。

push、pop 操作在栈顶进行。

```
ADT 栈(stack)
Data
    同线性表。元素具有相同的类型，相邻元素具有前驱和后继关系。
Operation
    InitStack(*S):    初始化操作，建立一个空栈S。
    DestroyStack(*S): 若栈存在，则销毁它。
    ClearStack(*S):   将栈清空。
    StackEmpty(S):    若栈为空，返回true，否则返回false。
    GetTop(S, *e):    若栈存在且非空，用e返回S的栈顶元素。
    Push(*S, e):      若栈S存在，插入新元素e到栈S中并成为栈顶元素。
    Pop(*S, *e):      删除栈S中栈顶元素，并用e返回其值。
    StackLength(S):   返回栈S的元素个数。
endADT
```

----


## 	1.2.	栈的顺序存储结构及实现

栈的结构定义

```
/* SElemType类型根据实际情况而定，这里假设为int */
typedef int SElemType;    
typedef struct
{
    SElemType data[MAXSIZE];
    /* 用于栈顶指针 */
    int top;              
}SqStack; 
```

![栈普通3种状态](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E6%A0%88%E6%99%AE%E9%80%9A3%E7%A7%8D%E7%8A%B6%E6%80%81.png)

现在有一个栈，StackSize是5，则栈普通情况、空栈和栈满的情况、空栈和栈满的情况示意图如上

进栈(push)：

```
/* 插入元素e为新的栈顶元素 */
Status Push(SqStack *S, SElemType e)
{
    /* 栈满 */
    if (S->top == MAXSIZE - 1)    
    {
        return ERROR;
    }
    /* 栈顶指针增加一 */
    S->top++;                     
    /* 将新插入元素赋值给栈顶空间 */
    S->data[S->top] = e;          
    
    return OK;
}
```

出栈(pop):

```
/* 若栈不空，则删除S的栈顶元素，用e返回其值，
   并返回OK；否则返回ERROR */
Status Pop(SqStack *S, SElemType *e)
{
    if (S->top == -1)
        return ERROR;
    /* 将要删除的栈顶元素赋值给e */
    *e = S->data[S->top];    
    /* 栈顶指针减一 */
    S->top--;  
                  
    return OK;
}
```
----


##		1.3.两栈共享空间

![两栈共享空间](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E4%B8%A4%E6%A0%88%E5%85%B1%E4%BA%AB%E7%A9%BA%E9%97%B4.png)

数组有两个端点，两个栈有两个栈底，让一个栈的栈底为数组的始端，即下标为0处，另一个栈为数组的末端，即下标为数组长度n-1处。这样，两个栈如果增加元素，就是两端点向中间延伸。（只针对两个具有相同数据类型的栈）

栈1为空时，即top1等于-1时；栈2为空时，即top2等于n时；栈满时，即top1+1==top2时。

两栈共享空间的结构的代码:

```
/* 两栈共享空间结构 */
typedef struct
{
    SElemType data[MAXSIZE];
    int top1;    /* 栈1栈顶指针 */
    int top2;    /* 栈2栈顶指针 */
} SqDoubleStack;
```

对于两栈共享空间的push方法，除了要插入元素值参数外，还需要有一个判断是栈1还是栈2的栈号参数stackNumber。插入元素的代码如下：

```
/* 插入元素e为新的栈顶元素 */
Status Push(SqDoubleStack *S, SElemType e, 
int stackNumber)
{
    /* 栈已满，不能再push新元素了 */
    if (S->top1 + 1 == S->top2)    
        return ERROR;
    /* 栈1有元素进栈 */
    if (stackNumber == 1)          
        /* 若栈1则先top1+1后给数组元素赋值 */
        S->data[++S->top1] = e;    
    /* 栈2有元素进栈 */
    else if (stackNumber == 2)     
        /* 若栈2则先top2-1后给数组元素赋值 */
        S->data[--S->top2] = e;    
        
    return OK;
}
```

对于两栈共享空间的pop方法，参数就只是判断栈1栈2的参数stackNumber，代码如下：

```
/* 若栈不空，则删除S的栈顶元素，用e返回其值，
   并返回OK；否则返回ERROR */
Status Pop(SqDoubleStack *S, SElemType *e, int stackNumber)
{
    if (stackNumber == 1)
    {
        /* 说明栈1已经是空栈，溢出 */
        if (S->top1 == -1)
            return ERROR;
        /* 将栈1的栈顶元素出栈 */
        *e = S->data[S->top1--];    
    }
    else if (stackNumber == 2)
    {
        /* 说明栈2已经是空栈，溢出 */
        if (S->top2 == MAXSIZE)
            return ERROR;           
        /* 将栈2的栈顶元素出栈 */
        *e = S->data[S->top2++];    
    }
    
    return OK;
}
```

## 	1.4.栈的链式存储结构（链栈）及实现

链栈的结构代码如下：

```
typedef struct StackNode
{
    SElemType data;
    struct StackNode *next;
} StackNode, *LinkStackPtr;

typedef struct LinkStack
{
    LinkStackPtr top;
    int count;
} LinkStack;
```

进栈：

```
/* 插入元素e为新的栈顶元素 */
Status Push(LinkStack *S, SElemType e)
{
    LinkStackPtr s 
      = (LinkStackPtr)malloc(sizeof(StackNode));
    s->data = e;
    /* 把当前的栈顶元素赋值给新结点的直接后继，如图中① */
    s->next = S->top;    
    /* 将新的结点s赋值给栈顶指针，如图中② */
    S->top = s;          
    S->count++;
    
    return OK;
}
```

出栈：

```
/* 若栈不空，则删除S的栈顶元素，用e返回其值，
   并返回OK；否则返回ERROR */
Status Pop(LinkStack *S, SElemType *e)
{
    LinkStackPtr p;
    if (StackEmpty(*S))
        return ERROR;
    *e = S->top->data;
    /* 将栈顶结点赋值给p，如图③ */
    p = S->top;               
    /* 使得栈顶指针下移一位，指向后一结点，如图④ */
    S->top = S->top->next;    
    /* 释放结点p */
    free(p);                  
    S->count--;
    
    return OK;
}
```

----



# 	2.栈的应用－－递归

## 2.1.斐波那契数列（Fibonacci）实现

问：如果兔子在出生两个月后，就有繁殖能力，一对兔子每个月能生出一对小兔子来。假设所有兔都不死，那么一年以后可以繁殖多少对兔子呢？

分析：拿新出生的一对小兔子分析一下：第一个月小兔子没有繁殖能力，所以还是一对；两个月后，生下一对小兔子数共有两对；三个月以后，老兔子又生下一对，因为小兔子还没有繁殖能力，所以一共是三对……

依次类推可以列出下表：

![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0%E5%88%97-tuzi.png)

表中数字1，1，2，3，5，8，13……构成了一个序列。这个数列有个十分明显的特点，前面相邻两项之和，构成了后一项

如下图：
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E4%B8%A4%E9%A1%B9%E4%B9%8B%E5%92%8C%EF%BC%9D%E4%B8%8B%E4%B8%80%E9%A1%B9.png)

对应的数学函数：
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%85%94%E5%AD%90%E8%AE%A1%E7%AE%97%E5%85%AC%E5%BC%8F.png)

打印出前40位的斐波那契数列数:

```
int main()
{
    int i;
    int a[40];
    a[0] = 0;
    a[1] = 1;
    printf("%d ", a[0]);
    printf("%d ", a[1]);
    for (i = 2; i < 40; i++)
    {
        a[i] = a[i - 1] + a[i - 2];
        printf("%d ", a[i]);
    }
    return 0;
}
```

其实我们的代码，如果用递归来实现，还可以更简单:

```
/* 斐波那契的递归函数 */ 
int Fbi(int i)
{
    if (i < 2)
        return i == 0 ? 0 : 1;
    /* 这里Fbi就是函数自己，它在调用自己 */
    return Fbi(i - 1) + Fbi(i - 2);    
}
int main()
{
    int i;
    for (i = 0; i < 40; i++)
        printf("%d ", Fbi(i));
    return  0;
}
```

相比较迭代的代码，是不是干净很多。
那么这段递归是怎么执行的呢？我们来模拟代码种的 Fbi(i) 函数当 i=5 的执行过程，如下图：

![Fbi(5)执行过程](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_Fbi(5)%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

## 2.2.递归定义

迭代和递归的区别是：
迭代使用的是循环结构，递归使用的是选择结构。递归能使程序的结构更清晰、更简洁、更容易让人理解，从而减少读懂代码的时间。但是大量的递归调用会建立函数的副本，会耗费大量的时间和内存。迭代则不需要反复调用函数和占用额外的内存。

递归与栈有什么关系？
这得从计算机系统的内部说起，前面我们已经看到递归是如何执行它的前行(Fbi(i))和退回(return)阶段的。递归过程退回的顺序是它前行顺序的逆序。在退回过程中，可能要执行某些动作，包括恢复在前行过程中存储起来的某些数据。

这种存储某些数据，并在后面又以存储的逆序恢复这些数据，以提供之后使用的需求，显然很符合栈这样的数据结构，因此，编译器使用栈实现递归就没什么好惊讶的了。

简单的说，就是在前行阶段，对于每一层递归，函数的局部变量、参数值以及返回地址都被压入栈中。在退回阶段，位于栈顶的局部变量、参数值和返回地址被弹出，用于返回调用层次中执行代码的其余部分，也就是恢复了调用的状态。


#		3.栈的应用－－四则运算表达式求值

## 3.1.后缀（逆波兰）表示法应用

对于“9+(3-1)×3+10÷2”，如果要用后缀表示法应该是什么样子：“9 3 1-3*+102/+”，这样的表达式称为后缀表达式，叫后缀的原因在于所有的符号都是在要运算数字的后面出现。

举例：
后缀表达式：9 3 1-3*+10 2/+

规则：从左到右遍历表达式的每个数字和符号，遇到是数字就进栈，遇到是符号，就将处于栈顶两个数字出栈，进行运算，运算结果进栈，一直到最终获得结果。

1．初始化一个空栈。此栈用来对要运算的数字进出使用；
2．后缀表达式中前三个都是数字，所以9、3、1进栈；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F01.png)
3．接下来是“-”，所以将栈中的1出栈作为减数，3出栈作为被减数，并运算3-1得到2，再将2进栈；
4．接着是数字3进栈；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F02.png)
5．后面是“*”，也就意味着栈中3和2出栈，2与3相乘，得到6，并将6进栈；
6．下面是“+”，所以栈中6和9出栈，9与6相加，得到15，将15进栈；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F03.png)
7．接着是10与2两数字进栈；
8．接下来是符号“/”，因此，栈顶的2与10出栈，10与2相除，得到5，将5进栈；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F04.png)
9．最后一个是符号“+”，所以15与5出栈并相加，得到20，将20进栈，如图4-9-5的左图所示。10．结果是20出栈，栈变为空
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F05.png)

很顺利的解决了计算的问题，那么如何让“9+(3-1)×3+10÷2”转化为“9 3 1-3+10 2/+”呢？

## 3.2.中缀表达式转后缀表达式

“9+(3-1)×3+10÷2”这样平时所用的标准四则运算表达式，因为所有的运算符号都在两数字之间，所以叫做中缀表达式。

中缀表达式“9+(3-1)×3+10÷2”转化为后缀表达式“9 3 1-3*+10 2/+”

规则：从左到右遍历中缀表达式的每个数字和符号，若是数字就输出，即成为后缀表达式的一部分；若是符号，则判断其与栈顶符号的优先级，是右括号或优先级不高于栈顶符号（乘除优先加减）则栈顶元素依次出栈并输出，并将当前符号进栈，一直到最终输出后缀表达式为止。

1．初始化一空栈，用来对符号进出栈使用；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F01.png)
2．第一个字符是数字9，输出9，后面是符号“+”，进栈；
3．第三个字符是“(”，依然是符号，因其只是左括号，还未配对，故进栈；
4．第四个字符是数字3，输出，总表达式为93，接着是“-”，进栈；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F02.png)
5．接下来是数字1，输出，总表达式为 9 31，后面是符号“)”，此时，我们需要去匹配此前的“(”，所以栈顶依次出栈，并输出，直到“(”出栈为止。此时左括号上方只有“-”，因此输出“-”。总的输出表达式为 9 3 1-；
6．紧接着是符号“×”，因为此时的栈顶符号为“+”号，优先级低于“×”，因此不输出，“*”进栈。接着是数字3，输出，总的表达式为 9 3 1-3；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F03.png)
7．之后是符号“+”，此时当前栈顶元素“”比这个“+”的优先级高，因此栈中元素出栈并输出（没有比“+”号更低的优先级，所以全部出栈），总输出表达式为9 3 1-3+。然后将当前这个符号“+”进栈。也就是说，前6张图的栈底的“+”是指中缀表达式中开头的9后面那个“+”，而图4-9-9左图中的栈底（也是栈顶）的“+”是指“9+(3-1)×3+”中的最后一个“+”；
8．紧接着数字10，输出，总表达式变为9 31-3*+10。后是符号“÷”，所以“/”进栈；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F04.png)
9．最后一个数字2，输出，总的表达式为9 31-3+10 2。如图4-9-10的左图所示。10．因已经到最后，所以将栈中符号全部出栈并输出。最终输出的后缀表达式结果为93 1-3+10 2/+”；
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E8%BD%AC%E5%90%8E%E7%BC%80%E8%A1%A8%E8%BE%BE%E5%BC%8F05.png)



----

#	4.队列

##		4.1.队列定义

队列（queue）是一种先进先出（First In First Out）的线性表，简称FIFO。只允许在一端进行插入操作，而在另一端进行删除操作的线性表。允许插入的一端称为队尾，允许删除的一端称为队头。

```
ADT 队列(Queue)
Data
    同线性表。元素具有相同的类型，相邻元素具有前驱和后继关系。
Operation
    InitQueue(*Q):    初始化操作，建立一个空队列Q。
    DestroyQueue(*Q): 若队列Q存在，则销毁它。
    ClearQueue(*Q):   将队列Q清空。
    QueueEmpty(Q):    若队列Q为空，返回true，否则返回false。
    GetHead(Q, *e):   若队列Q存在且非空，用e返回队列Q的队头元素。
    EnQueue(*Q, e):   若队列Q存在，插入新元素e到队列Q中并成为队尾元素。
    DeQueue(*Q, *e):  删除队列Q中队头元素，并用e返回其值。
    QueueLength(Q):   返回队列Q的元素个数
endADT
```

##		4.2.循环对列

### 4.2.1.循环对列

首先了解下，什么是假溢出：
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%BE%AA%E7%8E%AF%E5%AF%B9%E5%88%9701.png)
假设这个队列的总个数不超过5个，但目前如果接着入队的话，因数组末尾元素已经占用，再向后加，就会产生数组越界的错误，可实际上，我们的队列在下标为0和1的地方还是空闲的。我们把这种现象叫做“假溢出”。

解决假溢出的办法就是后面满了，就再从头开始，也就是头尾相接的循环。这种头尾相接的顺序存储结构就是循环队列。

此时问题出来了，空队列时，front等于rear，现在当队列满时，也是front等于rear，那么如何判断此时的队列究竟是空还是满呢？

- 办法一是设置一个标志变量flag，当front==rear，且flag=0时为队列空，当front==rear，且flag=1时为队列满。
- 办法二是当队列空时，条件就是front=rear，当队列满时，我们修改其条件，保留一个元素空间。也就是说，队列满时，数组中还有一个空闲单元。例如图4-12-8所示，我们就认为此队列已经满了，也就是说，我们不允许图4-12-7的右图情况出现。
![](http://7xob7d.com1.z0.glb.clouddn.com/stack_and_queue_%E5%BE%AA%E7%8E%AF%E5%AF%B9%E5%88%9702.png)

问题又来了，第二种方法，由于rear可能比front大，也可能比front小，所以尽管它们只相差一个位置时就是满的情况，但也可能是相差整整一圈。
若队列的最大尺寸为QueueSize，那么队列满的条件是(rear+1)%QueueSize==front（取模“%”的目的就是为了整合rear与front大小为一个问题）。

另外，当rear>front时，此时队列的长度为rear-front。但当rear<front时，队列长度分为两段，一段是QueueSize-front，另一段是0+rear，加在一起，队列长度为rear-front+QueueSize。
因此通用的计算队列长度公式为：
 
```
(rear-front+QueueSize)%QueueSize
```

有了这些讲解，现在实现循环队列就不难了。

循环队列的顺序存储结构代码如下：

```
/* QElemType类型根据实际情况而定，这里假设为int */
typedef int QElemType;    
/* 循环队列的顺序存储结构 */
typedef struct
{
    QElemType data[MAXSIZE];
    /* 头指针 */
    int front;            
    /* 尾指针，若队列不空，
       指向队列尾元素的下一个位置 */
    int rear;             
} SqQueue;
```

循环队列的初始化代码如下：

```
/* 初始化一个空队列Q */
Status InitQueue(SqQueue *Q)
{
    Q->front = 0;
    Q->rear = 0;    
    
    return OK;
}
```

循环队列求队列长度代码如下：

```
/* 返回Q的元素个数，也就是队列的当前长度 */
int QueueLength(SqQueue Q)
{
    return (Q.rear - Q.front + MAXSIZE) % MAXSIZE;
}
```

循环队列的入队列操作代码如下：

```
/* 若队列未满，则插入元素e为Q新的队尾元素 */
Status EnQueue(SqQueue *Q, QElemType e)
{
    /* 队列满的判断 */
    if ((Q->rear + 1) % MAXSIZE == Q->front)    
        return ERROR;
    /* 将元素e赋值给队尾 */
    Q->data[Q->rear] = e;                       
    /* rear指针向后移一位置， */
    Q->rear = (Q->rear + 1) % MAXSIZE;          
    /* 若到最后则转到数组头部 */
    
    return OK;
}
```

循环队列的出队列操作代码如下：

```
/* 若队列不空，则删除Q中队头元素，用e返回其值 */
Status DeQueue(SqQueue *Q, QElemType *e)
{
    /* 队列空的判断 */
    if (Q->front == Q->rear)                
        return ERROR;
    /* 将队头元素赋值给e */
    *e = Q->data[Q->front];                 
    /* front指针向后移一位置， */
    Q->front = (Q->front + 1) % MAXSIZE;    
    /* 若到最后则转到数组头部 */
    return  OK;
}
```

到这里可以看出单是顺序存储，若不是循环队列，算法的时间性能是不高的，但循环队列又面临着数组可能会溢出的问题，所以我们还需要研究一下不需要担心队列长度的链式存储结构。


### 4.2.2.对列的链式存储结构及实现

链对列的结构：

```
/* QElemType类型根据实际情况而定，这里假设为int */
typedef int QElemType;       
/* 结点结构 */
typedef struct QNode         
{
    QElemType data;
    struct QNode *next;
} QNode, *QueuePtr;

/* 队列的链表结构 */
typedef struct               
{
    /* 队头、队尾指针 */
    QueuePtr front, rear;    
} LinkQueue;
```

入对列：

```
/* 插入元素e为Q的新的队尾元素 */
Status EnQueue(LinkQueue *Q, QElemType e)
{
    QueuePtr s = 
(QueuePtr)malloc(sizeof(QNode));
    /* 存储分配失败 */
    if (!s)               
        exit(OVERFLOW);
    s->data = e;
    s->next = NULL;
    /* 把拥有元素e新结点s赋值给原队尾结点的后继， */
    Q->rear->next = s;    
    /* 见上图中① */
    /* 把当前的s设置为队尾结点，rear指向s，见上图中② */
    Q->rear = s;  
            
    return OK;
}
```

出对列：

```
/* 若队列不空，删除Q的队头元素，用e返回其值，
并返回OK，否则返回ERROR */
Status DeQueue(LinkQueue *Q, QElemType *e)
{
    QueuePtr p;
    if (Q->front == Q->rear)
        return ERROR;
    /* 将欲删除的队头结点暂存给p，见上图中① */
    p = Q->front->next;          
    /* 将欲删除的队头结点的值赋值给e */
    *e = p->data;                
    /* 将原队头结点后继p->next赋值给头结点后继， */
    Q->front->next = p->next;    
    /* 见上图中② */
    /* 若队头是队尾，则删除后将rear指向头结点，见上图中③ */
    if (Q->rear == p)            
        Q->rear = Q->front;
    free(p);
    return OK;
}
```

对于循环队列与链队列的比较，可以从两方面来考虑，从时间上，其实它们的基本操作都是常数时间，即都为O(1)的，不过循环队列是事先申请好空间，使用期间不释放，而对于链队列，每次申请和释放结点也会存在一些时间开销，如果入队出队频繁，则两者还是有细微差异。对于空间上来说，循环队列必须有一个固定的长度，所以就有了存储元素个数和空间浪费的问题。而链队列不存在这个问题，尽管它需要一个指针域，会产生一些空间上的开销，但也可以接受。所以在空间上，链队列更加灵活。

总的来说，在可以确定队列长度最大值的情况下，可以用循环队列，如果无法预估队列的长度时，则用链队列。

----

# 5.总结
栈（stack）是限定仅在表尾进行插入和删除操作的线性表。

队列（queue）是只允许在一端进行插入操作，而在另一端进行删除操作的线性表。

它们均可以用线性表的顺序存储结构来实现，但都存在着顺序存储的一些弊端。因此它们各自有各自的技巧来解决这个问题。

对于栈来说，如果是两个相同数据类型的栈，则可以用数组的两端作栈底的方法来让两个栈共享数据，这就可以最大化地利用数组的空间。

对于队列来说，为了避免数组插入和删除时需要移动数据，于是就引入了循环队列，使得队头和队尾可以在数组中循环变化。解决了移动数据的时间损耗，使得本来插入和删除是O(n)的时间复杂度变成了O(1)。
