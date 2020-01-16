---
title: " python函数式编程\t\t"
tags:
  - python
url: 286.html
id: 286
comments: false
categories:
  - python
date: 2018-10-29 11:59:50
---

参考引用：[函数式编程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/0014317848428125ae6aa24068b4c50a7e71501ab275d52000)

高阶函数
----

### map/reduce

#### map

map接收两个参数，一个是函数，另一个是Iterable,map将传入的函数依次作用到序列的每个元素，并把结果作为新的Iterator返回 map()传入的第一个参数是f，即函数对象本身。由于结果r是一个Iterator，Iterator是惰性序列，因此通过list()函数让它把整个序列都计算出来并返回一个list。 eg:

    def add_one(x):
        return x + 1
    
    
    print(list(map(add_one, [1, 2, 3, 4, 5, 6, 7])))
    --------------
    结果：
    [2, 3, 4, 5, 6, 7, 8]
    

#### reduce

reduce 也接收两个参数，一个是有`两个参数的函数`，一个是Iterable,他会把Iterable中的两个数传入函数，并将结果和第三个数再传入函数，以此循环，直到Iterable中没有数 eg:

    # 求和的例子
    # 导入reduce
    from functools import reduce
    def total(x, y):
        return x + y
    
    # rang左闭右开
    print(reduce(total, range(1, 10)))
    -------
    结果：
    45
    

### filter

如字面理解的那样，过滤 接收两个参数，一个是函数，一个是Iterable 通过函数返回的布尔类型来决定是否保留或者丢弃元素 True:保留，False:丢弃 eg:

    def even_num(x):
        return x % 2 == 0
    
    
    print(list(filter(even_num, range(1, 100))))
    ----
    结果：
    [2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30, 32, 34, 36, 38, 40, 42, 44, 46, 48]
    

### sorted

使用sorted可以对dict,list等进行排序，接收一个key参数，如果传了key参数，那么，会先按照key参数进行排序，然后再把原本的参数按照这个排序来排列 eg:

    # list
    li = [1,5,8,3,26,7]
    
    # dict
    L = [('Bob', 75), ('Adam', 92), ('Bart', 66), ('Lisa', 88)]
    
    
    def by_name(t):
        return t[0]
    
    
    def by_score(t):
        return -t[1]
    
    
    print(sorted(L, key=by_name))
    print(sorted(L, key=by_score))
    print(sorted(li))
    ---
    结果：
    [('Adam', 92), ('Bart', 66), ('Bob', 75), ('Lisa', 88)]
    [('Adam', 92), ('Lisa', 88), ('Bob', 75), ('Bart', 66)]
    [1, 3, 5, 7, 8, 26]
    

* * *

返回函数
----

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果值返回。 注：返回函数是惰性调用，返回的只是一个函数，只有调用这个函数的时候才会真正计算值 而且调用创建函数的时候返回的函数都是一个新的函数，即使传入的值是一样的

    def lazy_sum(*args):
        def sum():
            ax = 0
            for n in args:
                ax = ax + n
            return ax
        return sum
    
    f1 = lazy_sum(1, 3, 5, 7, 9)
    f2 = lazy_sum(1, 3, 5, 7, 9)
    print(f1==f2)
    -----
    结果：False
    

### 闭包

函数中写函数，内层函数可以调用外层函数的参数 eg:

    def lazy_sum(*args):
        def sum():
            ax = 0
            for n in args:
                ax = ax + n
            return ax
        return sum
    

函数的作用域可参考:[https://www.cnblogs.com/saintdingspage/p/7788958.html](https://www.cnblogs.com/saintdingspage/p/7788958.html)

匿名函数
----

没有名字的函数，可以简化代码 用法:lambda \[参数名称\]\[:\]\[表达式\]

    # 如上面map的例子可以简化为
    print(list(map(lambda x: x+1, [1, 2, 3, 4, 5, 6, 7])))
    # 可以通过赋值给变量调用
    f = lambda x: x+1
    print(list(map(f, [1, 2, 3, 4, 5, 6, 7])))
    

注：匿名函数只是一个表达式，后面只能包含`一个`语句

装饰器
---

可参考:[作为程序员，起码要知道的 Python 修饰器！](https://mp.weixin.qq.com/s?__biz=MjM5MjAwODM4MA==&mid=2650705784&idx=2&sn=bfbc2f5a5d4cfd6a079a8efe13c00936&chksm=bea6feab89d177bdc46685c1466c6331b9437d5d4db91c0e0eb576cbcafe316893cbfee08d21&mpshare=1&scene=23&srcid=1101xtaCKdJmxLSnoYIsmfws#rd) 个人感觉类似于Java中spring的aop+注解，当不想修改一个函数，又想在调用这个函数之前或者之后加入其它代码的时候可以用 eg:

    def log(text):
        def decorator(func):
            def wapper(*args, **kw):
                print('%s %s():' % (text, func.__name__))
                return func(*args, **kw)
    
            return wapper
    
        return decorator
    
    
    @log('execute')
    def now():
        print('2015-3-25')
    ---
    结果：
    execute now():
    2015-3-25
    

相当于增强原函数的功能，加了装饰器之后，实际调用的是装饰器中返回的内容，具体可参考上面的url

### 偏函数

functools.partial的作用就是，把一个函数的某些参数给固定住（也就是设置默认值），返回一个新的函数，调用这个新函数会更简单。 eg:

    from functools import partial
    int2 = partial(int, base=2)
    
    print(int2('1010'))
    

这里相当于将int()函数的第二个参数base默认为2，变为一个新的函数int2()，调用起来更方便

    # 上面的方法等同于
    def int2(x, base=2):
        return int(x, base)