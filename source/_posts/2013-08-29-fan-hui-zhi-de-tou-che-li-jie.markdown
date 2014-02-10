---
layout: post
title: "返回值的透彻理解"
date: 2013-04-20 23:06
comments: true
categories: Programe
---
在读《C++编程关键路径》的时候，看到有一章讲的很好。于是分享开来：

```c
void swap(int a, int b) {
    int tmp;
    tmp = a;
    a = b;
    b = tmp;
}
int get_ini(int a) {
    int  i = i + a;
    return i;
}

char * get_memory0() {
    char * p = (char *) malloc(sizeof(char ) * 20);
    strcpy(p,"hello world");
    return p;
}

char * get_memory1() {
    char * p = "hello world";
    return p;
}

char * get_memory2() {
    char p[] = "hello world";
    return p;
}

int main(int argc,char * argv[]){
    int x = 4,y = 3;
    swap(x,y);
    int z = x - y;
    printf("z = %d\n",z);//问题1

    z = get_ini(z);
    printf("z = %d\n",z);//问题2

    char * c0 = get_memory0();//问题3
    printf("c0 = %s\n",co);

    char * c1 = get_memory1();//问题4
    printf("c1 = %s\n",c1);

    char * c2 = get_memory2();//问题5
    printf("c1 = %s\n",c2);
}
```

#### 问题1：
a,b没有发生交换，所有函数会在程序运行的时候程序栈上分配一块存储区，这块栈的函数存储区随函数开始而开始，随着函数结束而结束，函数结束后， 这块存储区自动释放，以供其他用途，栈的默认大小是1M,swap(int a, int b)是一个参数传值的函数，函数内部的a,b是参数int a 和 int b的局部拷贝，所以a，b实际可以看成局部变量，他们的值由int a ,int b实参复制而来，只在函数内部有效，当函数体内的a,b离开函数作用域时候，a,b变量就销毁了.函数实参值没有发生改变。

#### 问题2：
返回值i是一个局部变量，函数的返回参数怎么可能是局部变量？函数的返回值由传值和传址两种，传值会在返回处生成一个临时对象，
用于存放局部变量i的一份拷贝。临时对象没有名称。虽然i被销毁了，但是他的拷贝仍然存在。并在函数返回处赋值给z.c0 = “hello world”

#### 问题3：
程序运行期间堆的内存一直都在。返回的p是会被销毁，但是在返回处有p的拷贝对象，指向堆的地址。c1 = “hello world”

#### 问题4:
常量存储区：”hello world”在程序运行期间都在

#### 问题5：
p[]不是指针，是一个数组变量，函数返回时左值拷贝只想的是局部变量的p[12]的首地址，当局部数组p[12]离开作用域后会自动销毁，
这时，函数返回的左值，拷贝指向的是一个被销毁的局部变量地址。所以c2 = 未知

