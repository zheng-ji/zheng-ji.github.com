---
layout: post
title: "C++与Python的混合编程"
date: 2013-06-08 23:42
comments: true
categories: Programe
---
#### python拓展编写
C++编写python的拓展。提高程序运行效率
我理解的一般流程:
* 编写自己的业务逻辑代码本例子如

```
string add(int a,int b)
```

* 包装为python函数，用于解析python传进来的参数

```c
PyObject* wrap_add(PyObject* self,PyObject* args);
//解析参数
PyArg_ParseTuple(args,"i|i",&a,&b);
//构建返回值
Py_BuildValue("s",add(a,b).c_str());
```

* 编写映射函数

```c
static PyMethodDef bintMethods[] =
{
        {"add", wrap_add, METH_VARARGS, "For add"},
            {NULL, NULL,0,NULL}
}
```

*.模块初始化函数

```c
void initbint() {
    PyObject* m;
    m = Py_InitModule("bint", bintMethods);
}

```

如下

```c    
#include "BigNum.h"
#include <python2.7/Python.h>
#include <iostream>

string add(int a,int b) {
    Bint num1(a);
    Bint num2(b);
    Bint ans = num1 + num2;
    return Bint::write(ans);
}

string multiple(int a,int b) {
    Bint num1(a);
    Bint num2(b);
    Bint ans = num1 * num2;
    return Bint::write(ans);
}

PyObject* wrap_add(PyObject* self,PyObject* args) {
    int a,b;
    if (!PyArg_ParseTuple(args,"i|i",&a,&b))
        return NULL;
    return Py_BuildValue("s",add(a,b).c_str());
}

PyObject* wrap_mul(PyObject* self,PyObject* args) {
    int a,b;
    if (!PyArg_ParseTuple(args,"i|i",&a,&b))
        return NULL;
    return Py_BuildValue("s",multiple(a,b).c_str());
}

static PyMethodDef bintMethods[] =
{
    {"add", wrap_add, METH_VARARGS, "For add"},
    {"mul", wrap_mul, METH_VARARGS, "For mul"},
    {NULL, NULL,0,NULL}
};

extern "C"              //不加会导致找不到initbint
void initbint() {
    PyObject* m;
    m = Py_InitModule("bint", bintMethods);
}
```

编译成动态链接库

```
all:
    g++ -fPIC -shared BigNum.cpp -o Bint.so
clean:
    rm -rf bint.so
```

```python           
import bint
bint.mul(4000000,5000000)
```

GitHub上有[源码](http://innerbrilliant.sinaapp.com/?p=515)

### C++调用Python

python的开发效率之高是毋庸置疑的，C++/C的语言性能之快也是让人羡慕的。这一次，鱼和熊掌是可以兼得的 ：），混合编程，使得我们可以取之所长，游走在C与python之间。很多游戏开发中使用python来实现战斗脚本。
* 初始化调用
Py_Initialize();

* PyObject* PyImport_ImportModule (char *name)
一般都是通过(pmod = PyImport_ImportModule ("zhengji.app_context")先来
加载一个模块（py脚本),得到一个PyObject *pmod对象,失败返回NULL类型

* 获取某个方法或者类，PyObject * o是pmod
PyObject* PyObject_GetAttrString (PyObject *o, char *attr_name)
 
*调用该方法 callable_object是第二步返回的指针
PyObject* PyObject_CallFunction (PyObject *callable_object, char *format, ...)
 
* 将PyObject* 返回char*
char* PyString_AsString (PyObject *string)
 
* 结束初始化
Py_Finalize();

下面是script.py的内容

```python
#!/usr/bin/python
# Filename: script.py
class Student:
    def SetName(self,name):
        self._name = name
    def PrintName(self):
        print self._name
    def hello():
        print "Hello World\n"
    def world(name):
        print "name" 
```

#### C++调用Script.py

```c
#include <python2.7/Python.h>
#include <iostream>
#include <string>

int main () {

    //使用python之前，要调用Py_Initialize();这个函数进行初始化
    Py_Initialize();

    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('./')");

    PyObject * pModule = NULL;
    PyObject * pFunc = NULL;
    PyObject * pClass = NULL;
    PyObject * pInstance = NULL;

    //这里是要调用的文件名
    pModule = PyImport_ImportModule("script");
    //这里是要调用的函数名
    pFunc= PyObject_GetAttrString(pModule, "hello");
    //调用函数
    PyEval_CallObject(pFunc, NULL);
    Py_DECREF(pFunc); 

    pFunc = PyObject_GetAttrString(pModule, "world");
    PyObject_CallFunction(pFunc, "s", "zhengji");
    Py_DECREF(pFunc); 

    //测试调用python的类
    pClass = PyObject_GetAttrString(pModule, "Student");
    if (!pClass) {
        printf("Can't find Student class.\n");
        return -1;
    }
    pInstance = PyInstance_New(pClass, NULL, NULL);
    if (!pInstance) {
        printf("Can't create Student instance.\n");
        return -1;
    }
    PyObject_CallMethod(pInstance, "SetName", "s","my family");
    PyObject_CallMethod(pInstance, "PrintName",NULL,NULL);
    //调用Py_Finalize，这个根Py_Initialize相对应的。
    Py_Finalize();
    return 0;
}
```

编译C++代码

```
g++ zj.cpp -o zj -lpython2.7
```

输出结果

```
zj@hp:~/tmp/CcalPy$ ./zj
Hello World
name
my family
```

