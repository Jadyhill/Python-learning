# 20171104Pyhton函数学习
***
## 关于`iterble`和`iterator`
可直接用作于for循环的对象统称为可迭代对象

可以被`next（）`函数调用并不断返回下一个值的对象称为迭代器`iterator`
***
## 关于高阶函数
### 函数名也是变量名
函数名其实就是指向函数的变量
### 关于`map`和`reduce`函数
* `map()`函数只接收2个参数，一个是函数，一个是`Iterable`，`map`将传入的函数一次做那个用在序列的元素上，并把结果作为新的`iterator`返回
* `reduce`函数是把一个函数作用在一个序列[x1,x2,x3,···]上，这个函数必须接收2个参数，`reduce`把结果继续和序列的下一个元素做累积计算，代码如下：

```python
>>> reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2)x3)x4)
```
经典示例：把序列`[1,2,3,4]`变成`1234`的`reduce`函数代码：

```python
>>> from functools import reduce
>>> def fn(x,y):
···    	return x*10 + y
···
>>> reduce(fn, [1,2,3,4])
1234
```
### 关于`filter`函数
`filter()`也接收一个函数和一个序列。`filter`把传入的函数一次作用域每个元素，然后返回值是`Ture`还是`False`决定保留还是丢弃该元素。注意到`filter()`函数返回的其实是一个`Iterator`，也就是一个惰性序列，多以要强迫`filter()`完成计算结果，需要用一个`list()`函数获得结果。

通过Python，用[埃氏筛法](https://baike.baidu.com/item/%E5%9F%83%E6%8B%89%E6%89%98%E6%96%AF%E7%89%B9%E5%B0%BC%E7%AD%9B%E6%B3%95/374984?fr=aladdin)得到全部自然素数，代码如下：

```python
>>> def _odd_iter():
    	n = 1
        while True:
			n = n + 2
			yield n

#定义一个筛选器
>>> def _not_divisible(n):
		return lambda x: x % n > 0

#定义一个生成器
>>> def primes()"
	yield 2
	it = odd_iter() #初试序列
	while True:
		n = next(it) #返回序列的第一个数
		yield n
		it = filter(_not_divisible(n), it) #构造新序列

#打印1000以内的素数
>>> for n in primes():
		if n<1000:
			print(n)
		else:
			break
```

### 关于`sorted`函数
`Python`内置的`sorted`函数能对`list`进行排序。此外`sorted`函数还可以接收一个`key`函数来实现自定义排序。
默认情况下，对字符串的排序，是按照ASCII的大小进行比较的，由于`'Z'<'a'`，结果，大写的字母`Z`会排在小写字母`a``的前面。

**本质上来说`sorted`排序的关键在于实现一个映射函数**
***
## 返回函数
关于返回函数，最重要的是 **闭包** 的概念。
返回闭包需要牢记的一点就是： **返回函数不要引用任何循环变量，或者后续会发生变化的变量。**
***
## 匿名函数
匿名函数`lambda x: x*x`实际上就是：

```python
>>> def f(x):
		retun x*x
```
匿名函数有个限制：只能有1个表达式。不用写`return`，返回值就是该表达式的结果。


