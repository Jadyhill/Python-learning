# 关于Python的迭代对象、迭代器和生成器
在了解Python的数据结构时，容器（`container`）、可迭代对象（`iterable`）,迭代器（`iterator`）、生成器（`generator`）、列表（`list`）、集合（`set`）和字典推导式（`dict comprehension`）众多概念餐咋在一起的时候，很难分清楚其中的概念。那么这篇文章，就是问了分清这些概念而写的。

![relationships](http://oyrqaw6tr.bkt.clouddn.com/relationships.png)
***
## 容器（container）
容器是一个是个把多种数据组织在一起的数据结构，容器中的数据可以逐个迭代获取，可以用`in`, `not in` 关键字判断元素是否包含在容器中。通常来说，这类数据结构把所有的元素存储在内存中（也有一些特例，并不是所有的元素都放在内存中，比如迭代器和发生器对象）。在Python中，常见的容器对象有：

* list, deque, ···
* set, forzensets, ···
* dict, defaultdict, OrderedDict, Counter, ···
* tuple, namedtuple, ···
* str

容器比较容易理解，从技术角度开看，当它可以用来询问某个元素是否包含在其中时，那么这个对象就可以认为是一个容器，比如 `list`, `set` 和 `tuples` 都是容器对象：

```python
>>> assert 1 in [1,2,3] # lists
>>> assert 4 not in [1,2,3]
>>> assert 1 in {1,2,3} # sets
>>> assert 4 not in {1,2,3}
>>> assert 1 in (1,2,3) # tuples
>>> assert 4 not in (1,2,3)
```
询问某元素是否在`dict`中，可以用`dict`中的`key`：

```python
>>> d = {1:'food', 2:'bar', 3:'qux'}
>>> assert 1 in d
>>> assert 'food' not in d # 'food' 并不是dic中的元素
```
询问`substring`是否在`string`中：

```python
>>> s = 'football'
>>> assert 'b' in s
>>> assert 'x' not in s
>>> assert 'foo' in s
```
尽管绝大多数容器提供了某种方式来获取其中的每一个元素，但这并不是容器本身提供的能力，而是**可迭代对象**赋予了容器这种能力，当然并不是所有的容器都是可以迭代的，比如：`Bloom filter`，虽然 Bloom filter 可以用来检测某个元素是否包含在容器中，当并不能从容其中获取其中的每一个值，因为 Bloom filter 压根儿就没把元素放在容器中，而是通过散列函数映射成一个值，保存在数组里。
***
## 可迭代对象（iterable）
很多容器都是可迭代对象。但凡是可以返回一个**迭代器**的对象都可以被称之为迭代对象。

```python
>>> x = [1,2,3]
>>> y = iter(x)
>>> z = iter(x)
>>> next(y)
1
>>> next(z)
1
>>> type(x)
>>> <class 'list'>
>>> type(y)
>>> <class 'list_iterator'>
```
这里`x`是一个可迭代对象，可迭代对象和容器一样是一种通俗的做法，并不是指某种具体的数据类型，`list`是可迭代对象，`dict`是可迭代对象，`set`也是。`y`和`z`是两个独立的迭代器，迭代器内部持有一个状态，该状态用于记录当前迭代所在的位置，以便下次迭代的时候获取正确的元素。迭代器有一种具体的迭代器类型，比如`list_iterator`，`set_iterator`。可迭代对象实现了`iter`方法，该方法返回一个迭代器对象。

```python
x = [1,2,3]
for elem in x:
    ···
```
实际执行情况是：
![iterator-VS-iterable](http://oyrqaw6tr.bkt.clouddn.com/iterable-vs-iterator.png)
反编译上面的代码，你可以看到解释器显示地调用`GET_ITER`指令，相当于调用`iter(x)`，`FOR_ITER`指令就是调用了`next()`方法，不断地获取迭代器中的下一个元素，但是你没法直接从指令中看出来，因为它被解释器优化了。

```python
>>> import dis
>>> x = [1,2,3]
>>> dis.dis('for_in x: pass')
  1         0 SETUP_LOOP        14 (to 17)
            3 LOAD_NAME         0 (x)
            6 GET-ITER          
       >>   7 FOR_ITER          6 (to 16)
           10 STORE_NAME        1 (_)
           13 JUMP_ABSOLUTE     7
       >>  16 POP_BLOCK
       >>  17 LOAD_CONST        0 (None)
           20 RETURN_VALUE
```
***
## 迭代器（iterator）
迭代器是一种带状态的对象，它能在调用`next()` 方法的时候返回容器中的下一个值，任何实现了`iter`和`next()`方法的对象都是迭代器，`iter`返回迭代器自身，`next`返回容器的下一个值，如果容器中没有更多元素了，则会抛出`StopIteration`异常。

所以，迭代器就是实现了工厂模式的对象，它在你每次询问要下一个值的时候返回。例如`itertools`函数返回的都是迭代器对象。

```python
>>> from itertools import count
>>> counter = count(start = 13)
>>> next(counter) 
13
>>>next(counter) 
14
```
从一个有限序列中生成无线序列：

```python
>>> from itertools import cycle
>>> colors = cycle(['red',['white',['blue'])
>>> next(colors)
'red'
>>> next(colors)
'white'
>>> next(colors)
'blue'
>>> next(colors)
'red'
```
从无线序列中生成有限序列：

```python
>>> from itertools import islice
>>> colors = cycle(['red','white','blue']) # infinite
>>> limited = islice(colors,0,4)
>>> for x in limited:
···         print(x)
red
white
blue
red
```
自定义迭代器，以斐波那契数为例：

```python
class Fib:
    def__init__(self)
        self.prve = 0
        self.curr = 1
    
    def__iter__(self)
        return self
    
    def__next__(self)
        value = self.curr
        self.curr += self.prev
        return value

>>>f= Fib()
>>>list(islice(f,0,10))
[1,1,2,3,5,8,13,21,34,55]
```
`Fib`既是一个可迭代对象（它实现了`iter`方法），又是一个迭代器（因为实现了`next`方法）。实现变量`prev`和`curr`用户维护迭代器内部的状态。每次调用`next()`方法的时候完成两件事：

1. 为下一次调用`next()`修改状态
2. 为当前这次调用生成放回结果

迭代器就像一个懒加载的工厂，等到有人需要的时候才给他返回值，没有调用的时候就处于休眠状态，等待下次调用。
***
## 生成器（generator）
生成器其实就是一种特殊的迭代器。它不需要想上面的类一样写`iter()`和`next()`方法了，只需要一个关键字`yiled`。生成器一定是迭代器（反之不成立），因为任何生成器也是一种懒加载的模式生成值。用生成器实现斐波切数列的列子如下：

```python
def fib():
    prev, curr = 0, 1
    while True:
        yield curr
        prev, curr = curr, curr + prev
    
>>> f = fib()
>>> list(islice(f,0,10))
[1,1,2,3,5,8,13,21,34,55] 
```
`fib`就是一个普通的 python 函数，它的特殊性在于函数中没有关键字`return`，函数的返回值是一个生成器对象。当执行`f=fib()`返回的是一个生成器对象，此函数体中的代码并不会执行，只有显示或者隐示地调用`next`的时候才会真正执行里面的代码。

生成器在Python中是一个非常强大的编程结构，可以用更少的中间变量写流式代码。此外，相对于其他容器对象，它更节省内存个CPU，当然它可以用更少的代码来实现相似的功能。比如：

```python
def something():
    result = []
    for ··· in ··· :
        result.append(x)
        return result
```
都可以用生成器函数来替换：

```python
def iter_someting():
    for ··· in ··· :
        yield x
```
***
## 生成器表达式（generator expression）
生成器表达式是列表推导式的生成器版本，看起来像列表推导式，但是它返回的是一个生成器对象而不是列表对象。

```python
>>> a = (x*x for x in range(10))
>>> a
<generator object <genexpr> at 0x401f08>
>>> sum(a)
258
```
***
## 总结
* 容器是一系列元素的集合，`str`, `list`, `set`, `dict`, `file` 对象都可以看做容器，容器都可以被迭代（用在`for`和`while`等语句中），因此他们被称为可迭代对象；
* 可迭代对象实现了`iter`方法，该方法返回一个迭代对象；
* 迭代器持有一个内部状态的字段，同于记录下一次迭代返回值，他事先了`next`和`iter`方法，迭代器不会一次性把所有元素加载到内存中，而是需要的时候才会返回结果；
* 生成器是一种特殊的迭代器，他的返回值不是通过`return`而是`yield`。

***

**参考文章** [Iterator Types](https://docs.python.org/2/library/stdtypes.html#iterator-types)

