---
title: How to redirect output of print to log in python
date: 2023-10-11 20:47:32
tags:
---
# How "print" work in python?

In python print point to sys.stdout.write. In other words, when calling print, python will call **sys.stdout.write(obj+'\n') .**

Let me give an example to prof that.

First I define a function named fun1 which write parameter to a file test.txt. Then make sys.stdout.write point to fun1. And try to print something.

```python
import sys

def fun1(temp):
    with open("test.txt","a+") as file:
        file.write(temp)

sys.stdout.write=fun1
print(1)
print(2)
print(3)

```

Then we will find we have a new file where it's content is 1 2 3.

# How to redirect it to logging?

So we have known how print work . Then we need redirct it.

We should make stdout point to another class which **overwrite write and flush** **method**. Like that:

```python
class LoggerWriter:
    def __init__(self, logger):
        self.logger = logger

    def write(self, message):
        self.logger.info(message)

    def flush(self):
        pass

```

Besides, we should save a pointer to origin stdout. 

The complete code to do it  ( using decorator ):

```python
import logging
import sys
import os
import datetime

def log_prints(func):
    logger = logging.getLogger(__name__)
    if logger.hasHandlers():
        return logger

    log_file = os.path.join("./log"
                            , os.path.basename(sys.argv[0]).replace(".py", "_%s.log" \
                                                                    % datetime.datetime.now().strftime("%Y%m%d")))
    handler = logging.FileHandler(log_file)
    formatter = logging.Formatter('%(asctime)s %(funcName)-12s %(levelname)-8s %(message)s')
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.addHandler(logging.StreamHandler())
    logger.setLevel(logging.DEBUG)
    logger.info("%s started by %s" % (os.path.basename(sys.argv[0]), os.getlogin()))

    def wrapper(*args, **kwargs):
        import sys
        original_stdout = sys.stdout
        sys.stdout = LoggerWriter(logger)

        result = func(*args, **kwargs)

        sys.stdout = original_stdout

        return result

    return wrapper

class LoggerWriter:
    def __init__(self, logger):
        self.logger = logger

    def write(self, message):
        self.logger.info(message)

    def flush(self):
        pass


```
