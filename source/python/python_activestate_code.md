---
date:       2016-03-25
title:      ActiveState上一些Python优秀代码
slug:       python-activestate-code
author:     ryan
tags:       [python, gist, note]
weight:     660325
---

## 重定向对象属性

原始链接 [Rebind](http://code.activestate.com/recipes/546510-rebind/)


```python
# -*- coding: utf-8 -*-
# redirect.py
"""
Temporarily rebind an attribute using contextmanager
"""

from __future__ import with_statement
from contextlib import contextmanager


@contextmanager
def redirect(object_, attr, value):
    orig = getattr(object_, attr)
    setattr(object_, attr, value)
    yield
    setattr(object_, attr, orig)

if __name__ == "__main__":
    import sys
    with redirect(sys, 'stdout', open('stdout', 'w')):
        print "hello"

    print "we're back"
```


## 面向事件编程信号/槽的实现

原始链接 [Improved Signals/Slots implementation in Python](http://code.activestate.com/recipes/577980-improved-signalsslots-implementation-in-python/)


```python
# -*- coding: utf-8 -*-
# signal.py
"""
A signal/slot implementation

File:    signal.py
Author:  Thiago Marcos P. Santos
Author:  Christopher S. Case
Author:  David H. Bronke
Created: August 28, 2008
Updated: December 12, 2011
License: MIT

"""
from __future__ import print_function
import inspect
from weakref import WeakSet, WeakKeyDictionary


class Signal(object):
    def __init__(self):
        self._functions = WeakSet()
        self._methods = WeakKeyDictionary()

    def __call__(self, *args, **kargs):
        # Call handler functions
        for func in self._functions:
            func(*args, **kargs)

        # Call handler methods
        for obj, funcs in self._methods.items():
            for func in funcs:
                func(obj, *args, **kargs)

    def connect(self, slot):
        if inspect.ismethod(slot):
            if slot.__self__ not in self._methods:
                self._methods[slot.__self__] = set()

            self._methods[slot.__self__].add(slot.__func__)

        else:
            self._functions.add(slot)

    def disconnect(self, slot):
        if inspect.ismethod(slot):
            if slot.__self__ in self._methods:
                self._methods[slot.__self__].remove(slot.__func__)
        else:
            if slot in self._functions:
                self._functions.remove(slot)

    def clear(self):
        self._functions.clear()
        self._methods.clear()


# Sample usage:
if __name__ == '__main__':
    class Model(object):
        def __init__(self, value):
            self.__value = value
            self.changed = Signal()

        def set_value(self, value):
            self.__value = value
            self.changed()  # Emit signal

        def get_value(self):
            return self.__value

    class View(object):
        def __init__(self, model):
            self.model = model
            model.changed.connect(self.model_changed)

        def model_changed(self):
            print("   New value:", self.model.get_value())

    print("Beginning Tests:")
    model = Model(10)
    view1 = View(model)
    view2 = View(model)
    view3 = View(model)

    print("Setting value to 20...")
    model.set_value(20)

    print("Deleting a view, and setting value to 30...")
    del view1
    model.set_value(30)

    print("Clearing all listeners, and setting value to 40...")
    model.changed.clear()
    model.set_value(40)

    print("Testing non-member function...")

    def bar():
        print("   Calling Non Class Function!")

    model.changed.connect(bar)
    model.set_value(50)
```
