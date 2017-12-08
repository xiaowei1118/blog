title: python学习笔记一
date: 2015-12-11 16:49:46
tags: [python]
categories: coding
---

## 目的
这里记录一些在学习python过程中以前在别的语言中没有见过的语法，感谢[廖雪峰老师的课程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)，对新手受益匪浅。

## 记录
### python的函数参数

#### 参数默认值
python函数的参数默认值一定不要指向一个可变对象。
```
def add_end(L=[]):
    L.append('END')
    return L


>>add_end()  //当你使用默认参数调用时，一开始结果也是对的：
['END']

//但是，再次调用add_end()时，结果就不对了：    
>>> add_end()
['END', 'END']
>>> add_end()
['END', 'END', 'END']
```
以上错误是因为默认参数值指向的变量是可变的。

#### 函数可变参数
通过在参数值前面添加*号的形式,可以将任意个数的参数传递过来，实际上，这个numbers
默认会被封装成为tuple或者list,如果要传list，则可以通过`calc(*list)`形式来接收
```
def calc(*numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
```
这样，calc函数的参数个数就不固定了

#### 关键字参数
可变参数允许你传入0个或任意个参数，这些可变参数在函数调用时自动组装为一个tuple。而关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict
```
def person(name, age, **kw):
    print('name:', name, 'age:', age, 'other:', kw)
//函数person除了必选参数name和age外，还接受关键字参数kw。在调用该函数时，可以只传入必选参数：

>>> person('Michael', 30)
name: Michael age: 30 other: {}

>>> person('Bob', 35, city='Beijing')
name: Bob age: 35 other: {'city': 'Beijing'}
>>> person('Adam', 45, gender='M', job='Engineer')
name: Adam age: 45 other: {'gender': 'M', 'job': 'Engineer'}
```

#### 命名关键字参数
命名关键字可以约束传入制定的可变数量的参数
```
def person(name, age, *, city, job):
    print(name, age, city, job)

>>> person('Jack', 24, city='Beijing', job='Engineer')
Jack 24 Beijing Engineer

>>> person('Jack', 24, 'Beijing', 'Engineer')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: person() takes 2 positional arguments but 4 were given
//由于调用时缺少参数名city和job，Python解释器把这4个参数均视为位置参数，但person()函数仅接受2个位置参数

def person(name, age, *, city='Beijing', job):
    print(name, age, city, job)
//命名关键字参数也可以有缺省值(设置默认值)
```

#### 参数组合
在Python中定义函数，可以用必选参数、默认参数、可变参数、关键字参数和命名关键字参数，这5种参数都可以组合使用，除了可变参数无法和命名关键字参数混合。但是请注意，参数定义的顺序必须是：必选参数、默认参数、可变参数/命名关键字参数和关键字参数。

比如定义一个函数，包含上述若干种参数：
```
def f1(a, b, c=0, *args, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'args =', args, 'kw =', kw)

def f2(a, b, c=0, *, d, **kw):
    print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)
```

### 切片
对这种经常取指定索引范围的操作，用循环十分繁琐，因此，Python提供了切片（Slice）操作符，能大大简化这种操作。
```
>>> L = ['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
>>> L[0:3]
['Michael', 'Sarah', 'Tracy']

>>> L[:3]        //索引默认从0开始
['Michael', 'Sarah', 'Tracy']

>>> L[-2:]             //支持倒数
['Bob', 'Jack']
>>> L[-2:-1]
['Bob']

>>> L[:]       //复制数组
['Michael', 'Sarah', 'Tracy', 'Bob', 'Jack']
```
### enumerate函数
Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身
```
>>> for i, value in enumerate(['A', 'B', 'C']):
...     print(i, value)
...
0 A
1 B
2 C
```

### 列表生成式
列表生成式即ListComprehensions，是Python内置的非常简单却强大的可以用来创建list的生成式。
```
[x * x for x in range(1, 11) if x % 2 == 0]

[4, 16, 36, 64, 100]
```

### 生成器
如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间。在Python中，这种一边循环一边计算的机制，称为生成器：generator。
#### 生成器方法一
```
>>> g = (x * x for x in range(10))
>>> g
<generator object <genexpr> at 0x1022ef630>
next(g)         //通过next函数获取生成器里面的内容
for n in g:     //也可以通过循环迭代
   print(n)
```
我们讲过，generator保存的是算法，每次调用next(g)，就计算出g的下一个元素的值，直到计算到最后一个元素，没有更多的元素时，抛出StopIteration的错误

#### 生成器方法二
如果一个函数定义中包含yield关键字，那么这个函数就不再是一个普通函数，而是一个generator：
```
def fib(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n = n + 1
    return 'done'
```
变成generator的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行时从上次返回的yield语句处继续执行。

for循环调用generator时，发现拿不到generator的return语句的返回值。如果想要拿到返回值，必须捕获StopIteration错误，返回值包含在StopIteration的value中.
```
>>> g = fib(6)
>>> while True:
...     try:
...         x = next(g)
...         print('g:', x)
...     except StopIteration as e:
...         print('Generator return value:', e.value)
...         break
...
g: 1
g: 1
g: 2
g: 3
g: 5
g: 8
Generator return value: done
```
