---
title: CS61A-lab03-Q6_ChurchNumeral丘奇数
date: 2023-07-09 15:09:34
permalink: CS61A-lab03-ChurchNumeral/
tags: 
- CS61A
categories:
- Major
- CS61A
index_img: https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202307091514452.png
excerpt: CS61A-lab03-Q6-ChurchNumerals
---

## Q6: Church Numerals

> The logician `Alonzo Church` invented a system of representing non-negative integers entirely using functions. The purpose was to show that functions are sufficient to describe all of number theory: if we have functions, we do not need to assume that numbers exist, but instead we can invent them.
>
> Your goal in this problem is to rediscover this representation known as `Church numerals`. `Church numerals` are a way to represent non-negative integers via repeated function application. Specifically, church numerals ( such as `zero`, `one`, and `two` below ) are *functions* that take in a function `f` and return a new function which, when called, repeats `f` a number of times on some argument `x`. Here are the definitions of `zero`, as well as a `successor` function, which takes in a church numeral `n` as an argument and returns a function that represents the church numeral one higher than `n`:

```python
def zero(f):
    return lambda x: x

def successor(n):
    return lambda f: lambda x: f(n(f)(x))
```

> First, define functions `one` and `two` such that they have the same behavior as `successor(zero)` and `successsor(successor(zero))` respectively, but *do not call `successor` in your implementation*.
>
> Next, implement a function `church_to_int` that converts a church numeral argument to a regular Python integer.
>
> Finally, implement functions `add_church`, `mul_church`, and `pow_church` that perform addition, multiplication, and exponentiation on church numerals.

## Solution


```python
def one(f):
    """Church numeral 1: same as successor(zero)"""
    "*** YOUR CODE HERE ***"
    return lambda x: f(x)

def two(f):
    """Church numeral 2: same as successor(successor(zero))"""
    "*** YOUR CODE HERE ***"
    return lambda x:f(f(x))

three = successor(two)

def church_to_int(n):
    """Convert the Church numeral n to a Python integer.

    >>> church_to_int(zero)
    0
    >>> church_to_int(one)
    1
    >>> church_to_int(two)
    2
    >>> church_to_int(three)
    3
    """
    "*** YOUR CODE HERE ***"
    return n(lambda x: x+1)(0)

def add_church(m, n):
    """Return the Church numeral for m + n, for Church numerals m and n.

    >>> church_to_int(add_church(two, three))
    5
    """
    "*** YOUR CODE HERE ***"
    return lambda f: lambda x: m(f)(n(f)(x))


def mul_church(m, n):
    """Return the Church numeral for m * n, for Church numerals m and n.

    >>> four = successor(three)
    >>> church_to_int(mul_church(two, three))
    6
    >>> church_to_int(mul_church(three, four))
    12
    """
    "*** YOUR CODE HERE ***"
    return lambda f: m(n(f))

def pow_church(m, n):
    """Return the Church numeral m ** n, for Church numerals m and n.

    >>> church_to_int(pow_church(two, three))
    8
    >>> church_to_int(pow_church(three, two))
    9
    """
    "*** YOUR CODE HERE ***"
    return n(m)
```

## Analysis

根据我们对`zero`和`successor`两个函数的观察，`zero`就相当于自然数`0`的定义，而`successor`实际上相当于一个递增函数。

我们要做的，就是以这个`0`的定义为起点，定义出所有自然数以及他们的基础运算。

题目中提到，函数`one`的行为应该和`successor(zero)`的行为一致，也就是说：

```python
one = successor(zero)
two = successor(one) = successor(successor(zero))
```



观察`successor`，可以仿照其结构写出`one`函数和`two`函数：

```python
def one(f):
    return lambda x: f(zero(f)(x))

def two(f):
    return lambda x: f(one(f)(x))
```

其实，`zero(f)(x)`就是`x`，而`one(f)(x)`就是`f(x)`，所以我们也可以这样写：

```python
def one(f):
    return lambda x: f(x)

def two(f):
    return lambda x: f(f(x))
```

由`successor`的定义，我们可以发现自然数`N`对应的函数`n`的定义应当为：

> `n(f) = f((n - 1)(f))=f(f(f(f(f(x)))))`  
>
> #n个`f`

如果我们按照这样的定义方式，定义自然数`n`：

```python
def n(f):
    return lambda x: f((n-1)(f)(x))
```



### `church_to_int` 

 `church_to_int` 本质上就是计数嵌套的 `f`有多少个。每走一层就给计数变量加一。

考虑将` f` 设置为自增函数 `increment` 。这样，传入的参数值就是计数变量的初值，函数的返回值就是终值了。

```python
def two(f):
    return lambda x: f(f(x))

increment = lambda x: x + 1

def church_to_int(n):
    return n(increment)(0)

'''
e.g.
	church_to_int(two)
	↓
	two(lambda x: x + 1)(0)
	↓
    ↓ # 构造two(f)
	↓
	(lambda x: x + 1)((lambda x: x + 1)(0))
	↓
	2
'''
```



事实上，有了`n(f)(x)`就是`n`的思想以后，用平常思维就可以写出答案正确的解：

```python
def add_church(m, n):
    return lambda f: lambda x: n(f)(x) + m(f)(x)

def mul_church(m, n):
    return lambda f: lambda x: m(f)(x) * n(f)(x)

def pow_church(m, n):
    return lambda f: lambda x: m(f)(x) ** n(f)(x)
```



**显然，这不会也不应该是最佳答案**



### `add_church`

我们把`m`和`n`各自看成整体。`m`是`m`个`f`函数嵌套的整体，可以看成是关于函数`f`的函数，`n`是`n`个`f`函数嵌套的整体.

当`m`和`n`传入的参数`f`相同时，我们要求`m+n`，即想办法获得一个`m+n`层的高阶函数.

用数学表达式为：`f(f(f(.....[f(f(f(x)))].....)))`，内部的括号为`n`层f嵌套，外层为`m`层括号嵌套。

考虑计算 `m+n`，也就是把 `m+n`个` f` 套在一起。而我们知道`m(f)`可以实现 `m`次嵌套,`n(f)`可以实现 `n` 次嵌套。

那么，我们在`n(f)`外面套一个`m(f)`即可：



```python
def add_church(m, n):
	return lambda f: lambda x: m(f)(n(f)(x))
```



### `mul_church` 

计算 `n×m`也就是 `m` 个` n` 相加，我们需要先把函数`f`嵌套`n`次，再把嵌套之后得到的`n`函数值嵌套`m`次。

这一次对于`m`函数而言，它的参数不再是函数`f`，而是嵌套之后的函数值`n`。

换言之，即 `n(f)` 自己套自己套上 `m`次：



```python
def mul_church(m, n):
	return lambda f: m(n(f))
	'''or
    return lambda f: lambda x: m(n(f))(x)
    '''
```



### `pow_church` 

`m(f)` 的功用是将 `m`个` f` 嵌套起来，那么 `n(m(f))`的功用也就是将 `n`个`m(f)`嵌套起来，即

<p align='center'><img src="https://blog-pic-storage.oss-cn-shanghai.aliyuncs.com/img/202307281702055.png" style="zoom:50%;" /></p>

```python
def pow_church(m, n):
    return n(m)
    '''or
    return lambda f: n(m)(f)
    '''
```



### FAQ

`m x n`写成：`lambda f: m(n(f))`， `m^n`写成：`lambda f: n(m)(f)`。这两者看起来非常非常相似，为什么结果相差这么大呢？



> 最简单的解释就是**作用对象不同**。
>
> 先来看第一条：`lambda f: m(n(f))`。把`m`和`n`分别看成是一个整体，在这行代码当中，`n`接收了一个参数`f`。而`m`接收的参数是`n(f)`。我们可以做一个换元，我们假设`g = n(f)`，那么`m`函数的入参就是`g`。
>
> 再来看第二条代码：`lambda f: n(m)(f)`，这里`m`函数没有入参，而`n`函数的入参是`m`。
>
> 表面上一个入参是`n(f)`一个是`m`，好像差不多。其实相差很大。
>
> `n(f)`是`n`函数作用于`f`上的结果，这个结果不再具备将函数嵌套`n`层的能力。所以外层再套上`m`函数，得到的也只是这个结果本身嵌套`m`次。
>
> 而`n(m)`不同，传入的参数是`m`本身，而非`m`作用之后的结果，`m`函数的功能就是将输入的函数嵌套`m`次，所以当它经过`n`函数作用于自身的时候，总层数会不停地乘上`m`。

## Reference

[^1]:[日拱一卒，伯克利CS61A，作业3](https://zhuanlan.zhihu.com/p/505962536)
[^2]:[CS61A Homework: Church Numerals](https://www.cnblogs.com/Shimarin/p/13823520.html)
[^3]:[记录一道颇有意思的CS61A python作业题Church numerals](https://zhuanlan.zhihu.com/p/267917164)
