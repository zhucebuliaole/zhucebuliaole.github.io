---
title: Write a singleton in python by import module
date: 2023-10-11 21:18:51
tags:
---
# What is singleton?

**Singleton** is a creational design pattern that lets you ensure that a class has only one instance, while providing a global access point to this instance.

# How to implement singleton using module?

In python it is easy to implement singleton using module. Init a instance is all you need.

```python
class xxx:
# implement your class

# only one instance
my_instance = xxx()
```

Then if you import my_instance on other files you can have a singleton instance. However, if you implement singleton by this way you should know the instance will init at the begining. In other words, **there is no lazy loading**. 

# Why this way can work?

[It involves the import system in python](https://docs.python.org/3/reference/import.html#:~:text=5.3.1.%20The,corresponding%20module%20object.). If a module has been imported , python will not import it again but will check if it be binded to another name. 

>
> The first place checked during import search is [`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules"). This mapping serves as a cache of all modules that have been previously imported, including the intermediate paths. So if `foo.bar.baz` was previously imported, [`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules") will contain entries for `foo`, `foo.bar`, and `foo.bar.baz`. Each key will have as its value the corresponding module object.
>
> During import, the module name is looked up in [`sys.modules`](https://docs.python.org/3/library/sys.html#sys.modules "sys.modules") and if present, the associated value is the module satisfying the import, and the process completes. However, if the value is `None`, then a [`ModuleNotFoundError`](https://docs.python.org/3/library/exceptions.html#ModuleNotFoundError "ModuleNotFoundError") is raised. If the module name is missing, Python will continue searching for the module.

So when you import your module, the init method will be invok only one time. Then you can say that it is a singleton here.
