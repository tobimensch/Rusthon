Embed CPython Script
-------------

The script below is marked with `@embed` which turns it into a static string in the final C++,
and is run after `cpython.initalize()` is called.


@embed
```python

class A():
	def __init__(self):
		self.value = 100
	def pymethod(self):
		print 'hello world'
	def add(self, a,b):
		return a+b

def foo():
	print 'foo'
	return A()

```
CPython CAPI
------------
The code below shows how to use the `cpython` module that wraps around the CPython C-API.
https://docs.python.org/2/c-api/object.html

Below `a.value as int` it is trival to know that `a` is a PyObject and that using the `.` operator
on it makes Rusthon generate the code that calls the CPython C-API.
Translation to C++:
```
auto v = static_cast<int>(PyInt_AS_LONG(PyObject_GetAttrString(a,"value")));
```

The syntax is used `->` to make it explicit that the object is a PyObject,
and to make Rusthon generate the required CPython C-API calls.  
For example `b->pymethod()` becomes this C++:
```
PyObject_Call(
	PyObject_GetAttrString(b,"pymethod"),
	Py_BuildValue("()"),
	NULL
);
```


Build Options
-------------
* @link:python2.7
* @include:/usr/include/python2.7
```rusthon
#backend:c++
import cpython

def main():
	state = cpython.initalize()
	with gil:
		a = cpython.foo()
		print 'addr of a:', a
		#print a.value as int
		#a.pymethod()
		b = a
		b->pymethod()
		print b->value as int
		v = b->value as int
		print v + 400
		c = b->pymethod()

		r = b->add(1, 2) as int
		print r
		#r = b->add(
		#	b->value, 
		#	b->value 
		#)
		#print r as int

	cpython.finalize(state)

```