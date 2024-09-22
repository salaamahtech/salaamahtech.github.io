# Overview
	- Semi-compiled to byte-code, executed by interpreter
	- Type is associated with objects, not variables - dynamically typed?
	- Python runtimes:Cython, Jython, IronPython, PyPy
	- PEP 8: Python Enhancement Proposal No. 8
- # Data types
	- ## String
		- In Python 2, a sequence of characters can be represented as
			- `str` - raw 8-bit values
			- `unicode` - unicode characters
		- In Python 3, it is
			- `bytes` - raw 8-bit values
			- `str` - unicode characters
	- ## List
		- ### List Comprehension
			- Python provides a compact syntax for deriving one list from another called *list comprehension*
			  
			  ```python
			  a =[1, 2, 3, 4]
			  squares = [x**2 for x in a]
			  print squares # [1, 4, 9, 16]
			  ```
			- Cons
				- More than 2 expressions in list comprehension becomes unreadable
				- Not scalable.
		- ### Generator Expression
			- For example, say you want to read a file and return the number of characters on each line. Doing this with list comprehension would require holding the length of every line of the file in memory.
			- To solve this, Python provides ***generator expressions***, a generalization of list comprehensions and generators. Generator expressions don't materialize the whole output sequence when they're run. Instead, generator expressions evaluate to an iterator that yields one item at a time from the expression.
	- ## Dict
		- ### Dict Comprehension
	- ## Set
		- ### Set Comprehension
- # Features
	- ## Slicing
		- ```python
		  a = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
		  a[:]      # ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
		  a[:5]     # ['a', 'b', 'c', 'd', 'e']
		  a[:-1]    # ['a', 'b', 'c', 'd', 'e', 'f', 'g']
		  a[4:]     # ['e', 'f', 'g', 'h']
		  a[-3:]    # ['f', 'g', 'h']
		  a[2:5]    # ['c', 'd', 'e']
		  a[2:-1]   # ['c', 'd', 'e', 'f', 'g']
		  a[-3:-1]  # ['f', 'g']
		  ```
	- ## Striding
		- Python has special syntax for the stride of a slice in the form `somelist[start:end:stride]`. This lets you take every nth item when slicing a sequence.
		- ```python
		  a = ['red', 'orange', 'yellow', 'green', 'blue', 'purple']
		  odds = a[::2]
		  evens = a[1::2]
		  print(odds) # ['red', 'yellow', 'blue']
		  print(evens) # ['orange', 'green', 'purple']
		  ```
		- The problem is that the stride syntax often causes unexpected behavior that can introduce bugs. For example, a common Python trick for reversing a byte string is to slice the string with a stride of -1.
	- ## Zip
		-
		- ```python
		  names = ['Alice', 'Bob', 'Charlie']
		  scores = [85, 90, 95]
		  combined = zip(names, scores)
		  print(list(combined))
		  ```
		- Output: 
		  ```
		  [('Alice', 85), ('Bob', 90), ('Charlie', 95)]
		  ```
		- Unzipping
			- ```python
			  zipped = [('Alice', 85), ('Bob', 90), ('Charlie', 95)]
			  names, scores = zip(*zipped)
			  print(names)   # Output: ('Alice', 'Bob', 'Charlie')
			  print(scores)  # Output: (85, 90, 95)
			  ```
	- ## Picking
		- TBD
	- ## Monkey Patching
		- TBD
	- ## Decorator Functions
		- **Python decorators** are a powerful feature that allows you to modify or enhance the behavior of a function or method without changing its code. They are essentially functions that wrap other functions, adding additional functionality to them.
		- ### How Decorators Work:
			- A decorator is a function that takes another function as an argument and returns a new function.
			- This allows you to "decorate" or "wrap" the original function with extra logic.
		- ### Example
			- Let’s say we want to create a decorator that prints a message before and after the execution of a function.
			- #### Step 1: Define the Decorator
			  
			  A decorator is just a function that takes another function as an argument:
			  
			  ```python
			  def my_decorator(func):
			    def wrapper():
			        print("Before the function call")
			        func()  # Call the original function
			        print("After the function call")
			    return wrapper
			  ```
			- #### Step 2: Use the Decorator
			  
			  We can now use this decorator with any function by using the `@` syntax.
			  
			  ```python
			  @my_decorator
			  def say_hello():
			    print("Hello!")
			  
			  # Calling the decorated function
			  say_hello()
			  ```
		- #### Output:
		  
		  ```r
		  Before the function call
		  Hello!
		  After the function call
		  ```
		- ### How It Works:
			- The `my_decorator` function takes `say_hello` as an argument and returns a new function `wrapper`.
			- The `wrapper` function adds logic (printing before and after) around the call to `func()`, which is the original `say_hello` function.
			- When you call `say_hello()`, it actually calls the `wrapper` function created by the decorator.
			  
			  This allows you to reuse the decorator for multiple functions and enhance their behavior in a consistent way.
	- ## Generator Functions
		- A generator function is defined like a normal function, but whenever it needs to generate a value, it does so with the `yield` keyword rather than `return`.
		- If the body of a function contains `yield`, the function automatically becomes a generator function (even if it also contains a `return` statement)
		- Generator functions create **generator iterators** or **generators** - a special type of iterator.
		- To get the next value from a generator, we use the same built-in function as for iterators: `next()` which calls the generator's `__next__()` method. When `next()` is called on a generator, it calls `yield`.
		- `yield` is just `return` (plus a little magic explained below) for generator functions.
		- When a generator function calls `yield`, the "state" of the generator function is frozen; the values of all variables are saved and the next line of code to be executed is recorded until `next()` is called again. When it is called again, the generator function simply resumes where it left off. If `next()` is never called again, the state recorded during the `yield` call is (eventually) discarded.
		- If a generator function calls `return` or reaches the end its definition, a `StopIteration` exception is raised. This signals to whoever was calling `next()` that the generator is exhausted (this is normal `iterator` behavior).
		  
		  ```python
		  def simple_generator_function():
		  	yield 1
		  	yield 2
		  	yield 3
		  
		  # using the generator function
		  for value in simple_generator_function():
		  	print(value)
		  
		  # another way to use the generator function
		  our_generator = simple_generator_function()
		  next(our_generator)
		  next(our_generator)
		  next(our_generator)
		  ```
		- Generators have the power
			- to yield a value,
			- receive a value (via `send()` method), or
			- both yield a value and receive a (possibly different) value in a single statement.
		- A statement of the form `other = yield foo` means, "*`yield` the value 'foo' and, when a value is sent to me, set `other` to that value*." You can "send" values to a generator using the generator's `send` method. This way you can set a variable inside a generator to a different value passed in by the user.
	- ## Global Interpreter Lock
		- Global Interpreter Lock (GIL) is a type of process lock that is used by Python whenever it deals with processes. Generally, Python only uses only one thread to execute the set of written statements. The performance of the single-threaded process and the multi-threaded process will be the same in Python and this is because of GIL in Python. We can not achieve multithreading in Python because we have a global interpreter lock that restricts the threads and works as a single thread.
	- ## Async IO
		- https://realpython.com/async-io-python/
		- https://fastapi.tiangolo.com/async/
- # Modules
	- ## itertools
	- ## collections
- # Object-oriented Programming
	- Each Python class definition has an implicit superclass: `object`
	- The rules for the visibility of Python names are as follows:
		- Most names are public.
		- Names that start with `_` are somewhat less public. Use them for implementation details that are truly subject to change.
		- Names that begin and end with `__` are internal to Python.
	- ## How Python class instantiation work?
		- Let’s say you have a class `Foo`:
		- ```python
		  class Foo(object):
		    def __init__(self, x, y=0):
		        self.x = x
		        self.y = y
		  ```
		- When `Foo(1, y=2)` is called, how is it that a new instance of `Foo` is returned with `x=1, y=2` initialized:
			- `Foo(*args, **kwargs)` is equivalent to `Foo.__call__(*args, **kwargs)`.
			- Since `Foo` is an instance of `type`, `Foo.__call__(*args, **kwargs)` calls `type.__call__(Foo, *args, **kwargs)`.
			- `type.__call__(Foo, *args, **kwargs)` calls `type.__new__(Foo, *args, **kwargs)` which returns `obj`.
				- `__new__` method is responsible for allocating memory and actual object creation. It should be noted that while `__new__` is a static method, you don’t need to declare it with `@staticmethod` - it is special-cased by the Python interpreter.
			- `obj` is then initialized by calling `obj.__init__(*args, **kwargs)`.
			- `obj` is returned.
			- So it goes like
			  `Foo(*args, **kwargs)` -> `Foo.__call__(*args, **kwargs)` -> `type.__call__(Foo, *args, **kwargs)` -> `type.__new__(Foo, *args, **kwargs)` -> `obj.__init__(*args, **kwargs)`
		- ### Singleton pattern
			- ```python
			  class Singleton(object):
			    _instance = None
			    def __new__(cls, *args, **kwargs):
			      if cls._instance is None:
			          cls._instance = super().__new__(cls, *args, **kwargs)
			      return cls._instance
			  ```
		- ### Borg pattern (nonpattern)
			- Unlike singleton, this creates multiple distinct instances, but the state is shared. There is no real use-case for this pattern.
			- ```python
			  class Borg(object):
			  	_dict = None
			  
			  	def __new__(cls, *args, **kwargs):
			  		obj = super().__new__(cls, *args, **kwargs)
			  		if cls._dict is None:
			  			cls._dict = obj.__dict__
			  		else:
			  			obj.__dict__ = cls._dict
			  		return obj
			  ```
	- ## Naming Notations
		- **Notations**
			- **Postfix notation** (object-oriented): `object.method()` or `object.attribute`.
			- **Prefix notation** (procedural programming): `function(object)`. The prefix notation `str(x)` is implemented as `x.__str__()` under the hood.
			- **Infix notation**: `object+other`. A construct such as `a+b` may be implemented as `a.__add__(b)` or `b.__radd__(a)` depending on the type of compatibility rules that were built into the class definitions for objects `a` and `b`.
	- ## Categories
		- **Attribute Access**: E.g., `object.attribute = value` or `del object.attribute`.
		- **Callables**: This special method implements what we see as a function that is applied to arguments, much like the built-in `len()` function.
		- **Collections**: These special methods implement the numerous features of collections. This involves methods such as `sequence[index]`, `mapping[key]`, and `set1|set2`.
		- **Numbers**: These special methods provide arithmetic operators and comparison operators. We can use these methods to expand the domain of numbers that Python works with.
		- **Contexts**: There are two special methods we'll use to implement a context manager that works with the `with` statement.
		- **Iterators**: There are special methods that define an iterator. This isn't essential since generator functions handle this feature so elegantly.
	- ## Special methods
		- `__str__()` - The `print()` function will use `str()` to prepare an object for printing.
		- `__repr__()`- The `format()` method of a string can also access these methods. When we use `%r` or `%s` formatting, we're requesting `__repr__()` or `__str__()`, respectively.
		- `__bool__()` -
		- `__bytes__()` -
		- `__del__()` -
- # Notes
	- Docstring. RST (ReStructured Text) markup.
- # References
	- Books
		- Learning Python the hard way
		- Think Python
		- Mastering object-oriented Python