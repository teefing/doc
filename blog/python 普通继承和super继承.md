---
title: python 普通继承和super继承
date: 2017-11-18
tags:
  - Python
category: 专业知识
---

# 普通继承

    class Parent(object):
        def __init__(self, name):
            self.name = name

        def say(self):
            print('this is %s' % self.name)


    class Child(Parent):
        def __init__(self, name, age):
            Parent.__init__(self, name)
            self.age = age
        def say(self):
            print('this is %s, his age is %s' % (self.name, self.age))

    child = Child('Bob', 1)
    child.say()

运行结果

    this is Bob, his age is 1

# super 继承

修改 Child 类
  
 class Child(Parent):
def **init**(self, name, age):
super(Child, self).**init**(name)
self.age = age
  
 def say(self):
print('this is %s, his age is %s' % (self.name, self.age))

第三行把 Parent 改成了 super(Child, self),并且去掉了后面`__init__`中的 self,

运行结果与上面是一样的

这里使用了 super， self 会隐式传入 init 中，可是感觉这样写其实有点多余，比较 super 中的 Child 和 self 都是固定的不是吗，每次写都很麻烦

于是，找到了一颗语法糖

直接写 super(), 删除 Child 和 self 也是可行的

# 普通继承和 super 的不同点

表面上看 普通继承和 super 继承的结果是一致的，实际上这两种方法的内部处理机制大大不同，当涉及多继承情况时，就会表现出明显的差异来，直接给例子：

代码一：

    class A:
        def __init__(self):
            print("Enter A")
            print("Leave A")

    class B(A):
        def __init__(self):
            print("Enter B")
            A.__init__(self)
            print("Leave B")

    class C(A):
        def __init__(self):
            print("Enter C")
            A.__init__(self)
            print("Leave C")

    class D(A):
        def __init__(self):
            print("Enter D")
            A.__init__(self)
            print("Leave D")

    class E(B, C, D):
        def __init__(self):
            print("Enter E")
            B.__init__(self)
            C.__init__(self)
            D.__init__(self)
            print("Leave E")

    E()

结果：

    Enter E
    Enter B
    Enter A
    Leave A
    Leave B
    Enter C
    Enter A
    Leave A
    Leave C
    Enter D
    Enter A
    Leave A
    Leave D
    Leave E

执行顺序很好理解，唯一需要注意的是公共父类 A 被执行了多次。

代码二：

    class A:
        def __init__(self):
            print("Enter A")
            print("Leave A")

    class B(A):
        def __init__(self):
            print("Enter B")
            super(B, self).__init__()
            print("Leave B")

    class C(A):
        def __init__(self):
            print("Enter C")
            super(C, self).__init__()
            print("Leave C")

    class D(A):
        def __init__(self):
            print("Enter D")
            super(D, self).__init__()
            print("Leave D")

    class E(B, C, D):
        def __init__(self):
            print("Enter E")
            super(E, self).__init__()
            print("Leave E")

    E()

结果：

    Enter E
    Enter B
    Enter C
    Enter D
    Enter A
    Leave A
    Leave D
    Leave C
    Leave B
    Leave E

在 super 机制里可以保证公共父类仅被执行一次，至于执行的顺序，是按照 mro 进行的`（E.__mro__）`。
