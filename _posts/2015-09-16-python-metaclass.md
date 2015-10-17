---
layout: post
title: 理解python元类
date: 2015-09-16
meta: "python metaclass"
digest: 在Python中类用于创建对象（实例），而元类用来创建这些类。Python创建元类的方法有两种：通过type关键字或者在类定义中使用__metaclass__属性。这篇文章主要讲解metaclass的概念和如何使用的问题。
tags: [Python]
category: [Backend]
---

### 类也是对象
要理解元类这个概念，就必须理清python中对象与类的概念。在Python中，任何类都是一个对象，可对该对象赋予属性值和方法。为了创建类，可以有两种方法：

1. 通过class关键字声明和定义类；
2. 手动定义一个类。这就涉及到元类的相关概念了。为了手动创建类，有两种方法，一种是用type函数，另一种则是在类的定义中使用__metaclass__属性；

--------

### 什么是元类
如果类是能够制造对象（实例），那制造类的类即是元类(Metaclasses)。
元类就是用来创建这些类（对象）的，元类就是类的类，你可以这样理解 为：

    MyClass = MetaClass()
    MyObject = MyClass()


你已经看到了type可以让你像这样做：

    MyClass = type('MyClass', (), {})

**这是因为函数type实际上是一个元类。type就是Python在背后用来创建所有类的元类。**在Python中，所有东西都是对象，数字0， 字符串'abc'等，我们可以通过查看该对象的__class__属性知道其所属类：

    >>> age = 35
    >>> age.__class__
    <type 'int'>
    >>> name = 'bob'
    >>> name.__class__
    <type 'str'>
    >>> def foo(): pass
    >>>foo.__class__
    <type 'function'>
    >>> class Bar(object): pass
    >>> b = Bar()
    >>> b.__class__
    <class '__main__.Bar'>

    # 现在，对于任何一个__class__的__class__属性又是什么呢？
    >>> a.__class__.__class__
    <type 'type'>
    >>> age.__class__.__class__
    <type 'type'>
    >>> foo.__class__.__class__
    <type 'type'>
    >>> b.__class__.__class__
    <type 'type'>


由上可见，type即是Python中所有类的元类，可视做Python中的类工厂（区别于工厂类）。我们也可以通过__metaclass__自定义元类。


--------

### type手动创建自己的类
type的常规用法如下：

    >>> print type(1)
    <type 'int'>
    >>> print type("1")
    <type 'str'>
    >>> print type(ObjectCreator)
    <type 'type'>
    >>> print type(ObjectCreator())
    <class '__main__.ObjectCreator'>

当type函数接收一个参数时，检查对象类型。当其接收3个参数时，则创建一个类：

    type(类名, 父类的元组（针对继承的情况，可以为空），包含属性的字典（名称和值）)

如：

    >>> MyShinyClass = type('MyShinyClass', (), {})  # 可以自己添加类属性和方法
    # 返回一个类对象
    >>> print MyShinyClass
    <class '__main__.MyShinyClass'>
    >>> print MyShinyClass()  
    #  创建一个该类的实例
    <__main__.MyShinyClass object at 0x8997cec>

--------

### 定义\__metaclass__属性手动创建类
你可以在写一个类的时候为其添加\__metaclass__属性。

    class Foo(object):
        __metaclass__ = something…
    […]

如果你这么做了，Python就会用元类来创建类Foo。注意，这里面有些技巧。你首先写下class Foo(object)，但是类对象Foo还没有在内存中创建。Python会在类的定义中寻找\_\_metaclass\_\_属性，如果找到了，Python就会用它来创建类Foo，如果没有找到，就会用内建的type来创建这个类。把下面这段话反复读几次。当你写如下代码时 :

    class Foo(Bar):
        pass

Python做了如下的操作：

Foo中有__metaclass__这个属性吗？如果是，Python会在内存中通过__metaclass__创建一个名字为Foo的类对象。如果Python没有找到__metaclass__，它会继续在Bar（父类）中寻找__metaclass__属性，并尝试做和前面同样的操作。如果Python在任何父类中都找不到__metaclass__，它就会在模块层次中去寻找__metaclass__，并尝试做同样的操作。如果还是找不到__metaclass__，Python就会用内置的type来创建这个类对象。

现在的问题就是，你可以在__metaclass__中放置些什么代码呢？答案就是：可以创建一个类的东西。那么什么可以用来创建一个类呢？type，或者任何使用到type或者子类化type的东东都可以。


--------

### 自定义元类
下面我们自定义元类，元类主要作用域在创建类之前改变类的定义，在写API时较常用到。

假想一个简单例子，你决定在你的模块里所有的类的属性都应该是大写形式。有好几种方法可以办到，但其中一种就是通过在模块级别设定__metaclass__。采用这种方法，这个模块中的所有类都会通过这个元类来创建，我们只需要告诉元类把所有的属性都改成大写形式就万事大吉了。

幸运的是，__metaclass__实际上可以被任意调用，它并不需要是一个正式的类（例如某些名字里带有‘class’的东西并不需要是一个class如classmethod）。所以，我们这里就先以一个简单的函数作为例子开始。

    # 元类会自动将你通常传给‘type’的参数作为自己的参数传入
    def upper_attr(future_class_name, future_class_parents, future_class_attr):
        
    '''返回一个类对象，将属性都转为大写形式'''
        
    #  选择所有不以'__'开头的属性
        attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
        
    # 将它们转为大写形式
        uppercase_attr = dict((name.upper(), value) for name, value in attrs)
       
    # 通过'type'来做类对象的创建
        return type(future_class_name, future_class_parents, uppercase_attr)
     
    __metaclass__ = upper_attr  
    #  这会作用到这个模块中的所有类
     
    class Foo(object):
        
    # 我们也可以只在这里定义__metaclass__，这样就只会作用于这个类中
        bar = 'bip'

    ### 输出结果:
    print hasattr(Foo, 'bar')
    # 输出: False
    print hasattr(Foo, 'BAR')
    # 输出:True
     
    f = Foo()
    print f.BAR
    # 输出:'bip'

现在让我们再做一次，这一次用一个真正的class来当做元类。

    # 请记住，'type'实际上是一个类，就像'str'和'int'一样
    # 所以，你可以从type继承
    class UpperAttrMetaClass(type):
        
    # __new__ 是在__init__之前被调用的特殊方法
        
    # __new__是用来创建对象并返回之的方法
        
    # 而__init__只是用来将传入的参数初始化给对象
        
    # 你很少用到__new__，除非你希望能够控制对象的创建
        
    # 这里，创建的对象是类，我们希望能够自定义它，所以我们这里改写__new__
        
    # 如果你希望的话，你也可以在__init__中做些事情
        
    # 还有一些高级的用法会涉及到改写__call__特殊方法，但是我们这里不用
        def __new__(upperattr_metaclass, future_class_name, future_class_parents, future_class_attr):
            attrs = ((name, value) for name, value in future_class_attr.items() if not name.startswith('__'))
            uppercase_attr = dict((name.upper(), value) for name, value in attrs)
     
            
    # 复用type.__new__方法
            
    # 这就是基本的OOP编程，没什么魔法
            return super(UpperAttrMetaClass, upperattr_metaclass).__new__(future_class_name, future_class_parents, uppercase_attr)     

确实，用元类来搞些“黑暗魔法”是特别有用的，因而会搞出些复杂的东西来。但就元类本身而言，它们其实是很简单的：

1)   拦截类的创建

2)   修改类

3)   返回修改之后的类

--------

### 为什么要用metaclass类而不是函数?

由于__metaclass__可以接受任何可调用的对象，那为何还要使用类呢，因为很显然使用类会更加复杂啊？这里有好几个原因：

1）  意图会更加清晰。当你读到UpperAttrMetaclass(type)时，你知道接下来要发生什么。

2） 你可以使用OOP编程。元类可以从元类中继承而来，改写父类的方法。元类甚至还可以使用元类。

3）  你可以把代码组织的更好。当你使用元类的时候肯定不会是像我上面举的这种简单场景，通常都是针对比较复杂的问题。将多个方法归总到一个类中会很有帮助，也会使得代码更容易阅读。

4） 你可以使用__new__, __init__以及__call__这样的特殊方法。它们能帮你处理不同的任务。就算通常你可以把所有的东西都在__new__里处理掉，有些人还是觉得用__init__更舒服些。

5） 哇哦，这东西的名字是metaclass，肯定非善类，我要小心！


--------

### 类和元类中魔法函数的调用关系
 上面说到\_\_new\_\_、\_\_init\_\_、\_\_call\_\_，下面看下这些函数在类的定义和创建实例时的作用，顺便把装饰器也一并捎上。
 
    class my_metaclass(type):
        def __new__(cls, class_name, parents, attributes):
            print "- my_metaclass.__new__ - Creating class instance of type", cls
            return super(my_metaclass, cls).__new__(cls,
                                                   class_name,
                                                   parents,
                                                   attributes)

        def __init__(self, class_name, parents, attributes):
            print "- my_metaclass.__init__ - Initializing the class instance", self
            super(my_metaclass, self).__init__(self)

        def __call__(self, *args, **kwargs):
            print "- my_metaclass.__call__ - Creating object of type ", self
            return super(my_metaclass, self).__call__(*args, **kwargs)

    def my_class_decorator(cls):
        print "- my_class_decorator - Chance to modify the class", cls
        return cls

    @my_class_decorator 
    class C(object):
        __metaclass__ = my_metaclass

        def __new__(cls):
            print "- C.__new__ - Creating object."
            return super(C, cls).__new__(cls)
     
        def __init__(self):
            print "- C.__init__ - Initializing object."


    print '*******************'
    c = C()
    print '-------------------'
    print "Object c =", c

结果为：

    - my_metaclass.__new__ - Creating class instance of type <class '__main__.my_metaclass'>
    - my_metaclass.__init__ - Initializing the class instance <class '__main__.C'>
    - my_class_decorator - Chance to modify the class <class '__main__.C'>
    *******************
    - my_metaclass.__call__ - Creating object of type  <class '__main__.C'>
    - C.__new__ - Creating object.
    - C.__init__ - Initializing object.
    -------------------
    Object c = <__main__.C object at 0x7f58522188d0>

分析:

经过`@my_class_decorator`修饰后的类C实际上为:

    C = my_class_decorator(C)

而`my_class_decorator(C)`中的C类表示的对象此时还未创建，因此会调用C定义时声明的元类my_metaclass来创建C类。为了创建C类，需要先对元类my_metaclass进行构造和初始化，即

1.调用了my_metaclass的\__new__和\__init__方法。这样便创建了my_metaclass的实例，可以用该实例进行类的构造了。
2.紧接着调用装饰器函数，返回经过元类实例化后的类，暂记为C_tmp，所以会打印my_class_decorator。
3.在`c = C()`中，现在C为C_tmp，即元类的实例，所以是对实例的函数调用——调用my_metaclass的\__call__函数，因此打印my_metaclass.\__call__。在该函数内部调用`super(my_metaclass, self).__call__(*args, **kwargs)`即type.\__call__。
4.上述函数该函数调用现在类C的\__new__构造C的实例。
5.再调用C的\__init__函数进行初始化。

我们可以将4中的type.\__call__函数屏蔽，则5和6两步中的两函数不会被调用。

**【注意】** 

1.在上述过程中我们看到装饰器调用发生在被装饰对象定义时：

    @my_decorator
    class C(object):
        pass

而不是被装饰对象调用时，如`c = C()`。

2.在元类中如果定义了__call__函数，则必须返回元类的父类（一般为type，因其时Python内部的元类）的__call__，使其对类（元类构造的类）进行构造和初始化。

    In [54]: a = type.__call__(int, 1)

    In [55]: a

    Out[55]: 1


**元类与装饰器的区别**: 事实上，任何能够用类装饰器完成的功能都能够用元类来实现。类装饰器有着很简单的语法结构易于阅读，所以提倡使用。但就元类而言，它能够做的更多，因为它在类被创建之前就运行了，而类装饰器则是在类创建之后才运行的。记住这点，让我们来同时运行一下两者，请注意运行的先后顺序：

    def my_metaclass(class_name, parents, attributes):
        print "In metaclass, creating the class."
        return type(class_name, parents, attributes)
     
    def my_class_decorator(class_):
        print "In decorator, chance to modify the class."
        return class_
     
    @my_class_decorator
    class C(object):
        __metaclass__ = my_metaclass
     
        def __init__(self):
            print "Creating object."
     
     print '***********'

输出为：

    In metaclass, creating the class.
    In decorator, chance to modify the class.
    ****************  
    Creating object.

结果表明

1.在@my_class_decorator时会调用C = my_class_decorator(C)，以后在对C进行调用都是my_class_decorator(C)的返回值进行操作。

2.类装饰器与元类有着很多共同点。事实上，任何能够用类装饰器完成的功能都能够用元类来实现。类装饰器有着很简单的语法结构易于阅读，所以提倡使用。但就元类而言，它能够做的更多，因为它在类被创建之前就运行了，而类装饰器则是在类创建之后才运行的。

--------

### 究竟为什么要使用元类？

现在回到我们的大主题上来，究竟是为什么你会去使用这样一种容易出错且晦涩的特性？好吧，一般来说，你根本就用不上它：
“元类就是深度的魔法，99%的用户应该根本不必为此操心。如果你想搞清楚究竟是否需要用到元类，那么你就不需要它。那些实际用到元类的人都非常清楚地知道他们需要做什么，而且根本不需要解释为什么要用元类。”  —— Python界的领袖 Tim Peters
元类的主要用途是创建API。一个典型的例子是Django ORM。它允许你像这样定义：

    class Person(models.Model):
        name = models.CharField(max_length=30)
        age = models.IntegerField()

但是如果你像这样做的话：

    guy  = Person(name='bob', age='35')
    print guy.age

这并不会返回一个IntegerField对象，而是会返回一个int，甚至可以直接从数据库中取出数据。这是有可能的，因为models.Model定义了__metaclass__， 并且使用了一些魔法能够将你刚刚定义的简单的Person类转变成对数据库的一个复杂hook。Django框架将这些看起来很复杂的东西通过暴露出一个简单的使用元类的API将其化简，通过这个API重新创建代码，在背后完成真正的工作。
 

--------

###结语

首先，你知道了类其实是能够创建出类实例的对象。好吧，事实上，类本身也是实例，当然，它们是元类的实例。

    >>>class Foo(object): pass
    >>> id(Foo)
    142630324

Python中的一切都是对象，它们要么是类的实例，要么是元类的实例，除了type。type实际上是它自己的元类，在纯Python环境中这可不是你能够做到的，这是通过在实现层面耍一些小手段做到的。其次，元类是很复杂的。对于非常简单的类，你可能不希望通过使用元类来对类做修改。你可以通过其他两种技术来修改类：

1. [Monkey patching](http://en.wikipedia.org/wiki/Monkey_patch)

2. class decorators

当你需要动态修改类时，99%的时间里你最好使用上面这两种技术。当然了，其实在99%的时间里你根本就不需要动态修改类。


--------


## 待续 ##
    1. metaclass在实际使用中的例子
    2. 装饰器在类定义后修改类与metaclass定义类前修改类的区别。注意本次实现中的装饰器调用的时机


--------

### 参考资料

[深刻理解Python中的元类(metaclass)](http://blog.jobbole.com/21351/)

[Python高级特性（3）: Classes和Metaclasses](http://blog.jobbole.com/67748/)

------------------
