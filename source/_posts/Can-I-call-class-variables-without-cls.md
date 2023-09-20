---
title: Can I call class variables without cls?
date: 2023-09-19 21:48:44
tags:
---
# Can I call class attributes without cls in Python ?

When we have a class variable but we find that we need call it in a non-class-method. Can we do operation like that? The answer is : **Yes** and **no**.

## Why the answer is yes?

For any class attributes , we can use **"self"** to call them. After the first time we to do so, we will create an instance attributes implicitly ( at the most of time). Let me start with an example. 

```python
class MyClass:
    class_var = 1

    def __init__(self, name):
        self.name = name

    # a method to call class variable
    def noclass_method(self):
        self.class_var += 1
        print(self.class_var)
        print(id(self.class_var))

        return

    # a method to show class variable
    @classmethod
    def show_class_var(cls):
        print(cls.class_var)
        print(id(cls.class_var))


if __name__ == '__main__':
    a = MyClass('a')
    a.noclass_method()
    print("-------no class method-------")
    MyClass.show_class_var()
    print("-------class variable-------")
    print(a.class_var)
    print("--------------")

    b = MyClass('b')
    b.noclass_method()
    print("-------no class method-------")
    MyClass.show_class_var()
    print("-------class variable-------")

# result :
# 2
# 4372670800
# -------no class method-------
# 1
# 4372670768
# -------class variable-------
# 2
# --------------
# 2
# 4372670800
# 3
# 4382206320
# -------no class method-------
# 1 
# 4372670768
# -------class variable-------
```

For the first instance "a" we can find that `self.class_var` has a different address with class variable and its value change to "2" ( 1+1 ).  When calling classmethod `show_class_var()` ,  result shows that we dont change class attribute which initialize as "1". And then , for instance "b" we call `noclass_method()`   twice and the `self.class_var`  equals to 3 (1+1+1) which means the two `slef.class_var` is instance variables actually. So for the first time an instance call it using key word **"self"** a instance variable will be created. And after that when we call a.class_var we will get instance variable. [The official document&#39;s example shows what will happend if class method and instance method has same name](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces:~:text=If%20the%20same%20attribute%20name%20occurs%20in%20both%20an%20instance%20and%20in%20a%20class%2C%20then%20attribute%20lookup%20prioritizes%20the%20instance%3A). 

## Why the answer is no.

If the class attribute is a mutable data type, call it by key word "self" will not create a new instance attribute. In other words, your method will change the shared class attribute. [Here is an example from official document.](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces:~:text=For%20example%2C%20the%20tricks%20list%20in%20the%20following%20code%20should%20not%20be%20used%20as%20a%20class%20variable%20because%20just%20a%20single%20list%20would%20be%20shared%20by%20all%20Dog%20instances%3A)

```python
class Dog:

    tricks = []             # mistaken use of a class variable

    def __init__(self, name):
        self.name = name

    def add_trick(self, trick):
        self.tricks.append(trick)

>>> d = Dog('Fido')
>>> e = Dog('Buddy')
>>> d.add_trick('roll over')
>>> e.add_trick('play dead')
>>> d.tricks                # unexpectedly shared by all dogs
['roll over', 'play dead']
```

## Summary

We can call class attributes by using **"self"** . If the attribute is a immutable data type, it will create an instance attribute implicitly. But if the attribute is a mutable data type, we can still call it but will modify the shared class attribute.

# Another way to modify class attributes in method.

We can call class attributes by using `self.__class__.your_attribute`

In my example, we can write a method like that:

```python
    def change_class_var(self):
        self.__class__.class_var += 1
```
