# Python coding style guide

## Goal of the document

This document gives guidance on writing Python code for MBL. It standardises different aspects of Python, beyond the syntax. The ultimate goal is to have code that is readable, maintainable, and conforms to Python upstream standards.

This is not a matter of personal preferences: the code is open-sourced, and upstream Python developers expect the code to be written in a certain and consistent way.

If you have any doubt on how to write good Python code, check how the standard library (on Linux it's under `/usr/lib/python3.5`) is written; you will find plenty of examples.

## Python 2 vs Python 3

You must use Python 3, unless there is a strict requirement to use Python 2. This because Python 2's days are numbered, and it will only be supported until Pycon 2020 with:

* Security fixes.
* Support for new OS versions and tool chains.
* (Rarely) bug fixes.

Check the countdown here: https://pythonclock.org/.

To ease the transition, the official porting guide has advice for running Python 2 code in Python 3.

## Mandatory PEPs

Three PEPs form the basis for writing any Python application: PEP20, PEP8 and PEP257.

### PEP20

Although it was born as joke, it has been the fundamental principles for Python developers for years.

#### Zen of Python

```
Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
>>
```

### PEP8

PEP8 is the official Python style guide. It covers every aspect of good formatting for Python code.

```
(pycodestyle) user1 at e10000 in ~$ pip install pycodestyle
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting pycodestyle
  Downloading http://pypi.arm.com/root/pypi/+f/cbc/619d09254895b/pycodestyle-2.4.0-py2.py3-none-any.whl (62kB)
    100% |████████████████████████████████| 71kB 15.7MB/s
Installing collected packages: pycodestyle
Successfully installed pycodestyle-2.4.0
(pycodestyle) user1 at e10000 in ~$ cat bad_pep8.py
variable = 100

import json

def test():
    print( "Hello world!" )

l = [1,2,3,4]

(pycodestyle) user1 at e10000 in ~$ pycodestyle bad_pep8.py
bad_pep8.py:3:1: E402 module level import not at top of file
bad_pep8.py:5:1: E302 expected 2 blank lines, found 1
bad_pep8.py:6:11: E201 whitespace after '('
bad_pep8.py:6:26: E202 whitespace before ')'
bad_pep8.py:8:1: E305 expected 2 blank lines after class or function definition, found 1
bad_pep8.py:8:1: E741 ambiguous variable name 'l'
bad_pep8.py:8:7: E231 missing whitespace after ','
bad_pep8.py:8:9: E231 missing whitespace after ','
bad_pep8.py:8:11: E231 missing whitespace after ','
bad_pep8.py:9:1: W391 blank line at end of file
(pycodestyle) user1 at e10000 in ~$

```

If you don't want to fix every line manually, use autopep8 to reformat the input file.

```
(pycodestyle) user1 at e10000 in ~$ pip install autopep8
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting autopep8
  Downloading http://pypi.arm.com/root/pypi/+f/228/4d4ae2052fedb/autopep8-1.3.5.tar.gz (109kB)
    100% |████████████████████████████████| 112kB 10.5MB/s
Requirement already satisfied: pycodestyle>=2.3 in /work/virtualenvs/pycodestyle/lib/python2.7/site-packages (from autopep8) (2.4.0)
Building wheels for collected packages: autopep8
  Running setup.py bdist_wheel for autopep8 ... done
  Stored in directory: /home/user1/.cache/pip/wheels/d1/3e/3e/7364d92cc3b432dc5120249d6c2cf3f9b0849a014c362e1d5e
Successfully built autopep8
Installing collected packages: autopep8
Successfully installed autopep8-1.3.5
(pycodestyle) user1 at e10000 in ~$ autopep8 bad_pep8.py
variable = 100

import json


def test():
    print("Hello world!")


l = [1, 2, 3, 4]
(pycodestyle) user1 at e10000 in ~$ autopep8 bad_pep8.py > good_pep8.py
(pycodestyle) user1 at e10000 in ~$ pycodestyle good_pep8.py
good_pep8.py:3:1: E402 module level import not at top of file
good_pep8.py:10:1: E741 ambiguous variable name 'l'
(pycodestyle) user1 at e10000 in ~$
```

A few notes from the code above:

* By default, autopep8 doesn't perform in-place changes (enable it with `-i`).
* It doesn't fix every error; it cannot generate code for you.

#### Black

Our sanity check pipelines runs a few checks on Python files, among them Black:

> By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.

> Black makes code review faster by producing the smallest diffs possible. Blackened code looks the same regardless of the project you’re reading. Formatting becomes transparent after a while and you can focus on the content instead.

Sanity checks run on all pull requests, and they perform the check but do not change any code. It is down to you to submit compliant code.

Black can be integrated [with many editors](http://black.readthedocs.io/en/stable/editor_integration.html).

### PEP257

PEP257 explains docstring conventions for Python.

Like PEP8, we have tools to check if we are missing docstrings:

```
(pep257) user1 at e10000 in ~$ pip install pep257
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting pep257
  Downloading http://pypi.arm.com/root/pypi/+f/d26/cd2a4f572e059/pep257-0.7.0-py2.py3-none-any.whl
Installing collected packages: pep257
Successfully installed pep257-0.7.0
(pep257) user1 at e10000 in ~$ pep257 bad_pep8.py
bad_pep8.py:1 at module level:
        D100: Missing docstring in public module
bad_pep8.py:5 in public function `test`:
        D103: Missing docstring in public function
(pep257) user1 at e10000 in ~$ vim good_pep8.py
"""Module docstring.                                                           

asd                                                                            
asd                                                                            
"""                                                                                                                                                                                                                                                                                                                          

import json                                                                    

variable = 100                                                                 


def test():                                                                    
    """Method docstring."""                                                    
    print("Hello world!")                                                      


l = [1, 2, 3, 4]

(pycodestylke) user1 at e10000 in ~$ pep257 good_pep8.py
(pycodestylke) user1 at e10000 in ~$
```

## Spaces vs Tabs

Use four-spaces indentation. This is [already covered by PEP8](https://www.python.org/dev/peps/pep-0008/#tabs-or-spaces).

For Vim users, add the following to `~/.vimrc`:

```
set expandtab       " tabs are converted to space                                                                                                                                                                                                                                                                             
set smarttab        " Be smart when using tabs ;)                              
set tabstop=4       " numbers of spaces of tab character                       
set shiftwidth=4    " numbers of spaces to (auto)indent                        
set softtabstop=4   " Number of spaces that a <Tab> counts for while performing editing  operations
```

## Virtual environments vs system packages

When running Python applications, you're like to have third party dependencies. There are several ways to install Python packages:

* Using the system package manager.
* using pip install `--user`.
* Using the Python3 built-in `venv` module.

We suggest using virtual environments, for the following reasons:

* They can be created and destroyed on the fly.
* You can control what you install (down to specific package versions).
* They don't pollute the system or other virtual environments with unwanted packages.

#### Virtual environments

Here's an virtualenv use example:

```
user01 at dev-machine in /tmp
$ python3 -m venv my_venv
user01 at dev-machine in /tmp
$ source ./my_venv/bin/activate
(my_venv) user01 at dev-machine in /tmp
$ pip list
Package       Version
------------- -------
pip           18.1
pkg-resources 0.0.0
setuptools    20.7.0
(my_venv) user01 at dev-machine in /tmp
$ pip install pytest
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting pytest
  Downloading https://files.pythonhosted.org/packages/81/27/d4302e4e00497448081120f65029696070806bc8e649b83f644de006d710/pytest-4.0.1-py2.py3-none-any.whl (217kB)
    100% |████████████████████████████████| 225kB 10.4MB/s
Collecting atomicwrites>=1.0 (from pytest)
  Using cached https://files.pythonhosted.org/packages/3a/9a/9d878f8d885706e2530402de6417141129a943802c084238914fa6798d97/atomicwrites-1.2.1-py2.py3-none-any.whl
Collecting attrs>=17.4.0 (from pytest)
  Using cached https://files.pythonhosted.org/packages/3a/e1/5f9023cc983f1a628a8c2fd051ad19e76ff7b142a0faf329336f9a62a514/attrs-18.2.0-py2.py3-none-any.whl
Collecting py>=1.5.0 (from pytest)
  Using cached https://files.pythonhosted.org/packages/3e/c7/3da685ef117d42ac8d71af525208759742dd235f8094221fdaafcd3dba8f/py-1.7.0-py2.py3-none-any.whl
Requirement already satisfied: setuptools in ./my_venv_test/lib/python3.5/site-packages (from pytest) (20.7.0)
Collecting pathlib2>=2.2.0; python_version < "3.6" (from pytest)
  Downloading https://files.pythonhosted.org/packages/2a/46/c696dcf1c7aad917b39b875acdc5451975e3a9b4890dca8329983201c97a/pathlib2-2.3.3-py2.py3-none-any.whl
Collecting six>=1.10.0 (from pytest)
  Using cached https://files.pythonhosted.org/packages/67/4b/141a581104b1f6397bfa78ac9d43d8ad29a7ca43ea90a2d863fe3056e86a/six-1.11.0-py2.py3-none-any.whl
Collecting pluggy>=0.7 (from pytest)
  Using cached https://files.pythonhosted.org/packages/1c/e7/017c262070af41fe251401cb0d0e1b7c38f656da634cd0c15604f1f30864/pluggy-0.8.0-py2.py3-none-any.whl
Collecting more-itertools>=4.0.0 (from pytest)
  Using cached https://files.pythonhosted.org/packages/79/b1/eace304ef66bd7d3d8b2f78cc374b73ca03bc53664d78151e9df3b3996cc/more_itertools-4.3.0-py3-none-any.whl
Installing collected packages: atomicwrites, attrs, py, six, pathlib2, pluggy, more-itertools, pytest
Successfully installed atomicwrites-1.2.1 attrs-18.2.0 more-itertools-4.3.0 pathlib2-2.3.3 pluggy-0.8.0 py-1.7.0 pytest-4.0.1 six-1.11.0
(my_venv) user01 at dev-machine in /tmp
$ pip list
Package        Version
-------------- -------
atomicwrites   1.2.1
attrs          18.2.0
more-itertools 4.3.0
pathlib2       2.3.3
pip            18.1
pkg-resources  0.0.0
pluggy         0.8.0
py             1.7.0
pytest         4.0.1
setuptools     20.7.0
six            1.11.0
(my_venv) user01 at dev-machine in /tmp
$ python -c "import pytest"
(my_venv) user01 at dev-machine in /tmp
$ echo $?
0
(my_venv) user01 at dev-machine in /tmp
$ deactivate
(my_venv) user01 at dev-machine in /tmp
$
```

Once the virtual environment has been created, you can use it without activating. You just need to call the binaries contained in the virtualenv itself:

```
(my_venv) user01 at dev-machine in /tmp
$ ./my_venv/bin/pip list
Package        Version
-------------- -------
atomicwrites   1.2.1
attrs          18.2.0
more-itertools 4.3.0
pathlib2       2.3.3
pip            18.1
pkg-resources  0.0.0
pluggy         0.8.0
py             1.7.0
pytest         4.0.1
setuptools     20.7.0
six            1.11.0
(my_venv) user01 at dev-machine in /tmp
$ ./my_venv/bin/python -c "import pytest"
(my_venv) user01 at dev-machine in /tmp
$ ./my_venv/bin/python
Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.


(my_venv) user01 at dev-machine in /tmp
$
```

## String formatting

Never use "+" for sting concatenation.

There are three ways to format strings:

* `%-formatting`. Please don't use this format.
* `str.format()`. This is the preferred MBL format.
* `f-Strings` can be used with Python 3.6 and higher; check the version available in your distribution.

## Returns, exit codes, error management and logging

Every Python application or script should have a clear workflow. This is driven by clearly returning and managing errors within the application.

### Return statement

Keep the number of return statements throughout the codebase to a minimum:

* One return per function, unless you have a very good reason.
* `__main__()` implementation can have multiple return statements only if it returns different codes.

### Error management

Errors are quite important for the lifecycle of the application. A good error management strategy depends on:

* Methods and functions may propagate the error upwards. The main implementation will catch it eventually.
* Methods and functions may catch the error, if they need to perform a corrective action.
* Methods and functions may catch the error, perform an action and repropagate the exception upwards.
* The last place you can catch errors is in `__main__()`.

### Exit codes

If the application performs as expected, it must return `0` (zero). Any other return code should be easily identifiable in the main implementation. Example:

```
import sys
import logging


def _main(args):                                                               
    """Main execution of the application."""                                   
    try:                                                                       
        print("Hello world")                                                                                                                                                                                                                                                                      
    except KeyboardInterrupt:                                                  
        logging.error("Ctrl-C detected. Stopping.")                            
        return 1                                                               
    except Exception as e:                                                     
        import traceback                                                       
        logging.error(e)                                                       
        traceback.print_exc()                                                  
        return 2                                                               
    return 0


if __name__ == "__main__":                                                     
    sys.exit(_main(sys.argv[1:]))
```

### Only `__main__()` can exit

Do no use any other `sys.exit()` call in the application.

```
import sys
...
...

if __name__ == "__main__":                                                     
    sys.exit(_main(sys.argv[1:]))
```

### print() vs logging

To choose between `print()` and logging:

* If you need to show anything that is not part of the application lifecycle, use logging. Examples:
    * The application connects to a database.
    * The application exits with an error. Use logging with the error level.
* If you need to show anything that is part of application lifecycle, use `print()`. Examples:
    * The application needs input from the user and must give context for the request.
    * The application returns a string to the user.
* There are cases where you might want to use both. Example:
    * The application needs to show lifecycle stages.

## Public vs Private vs Protected methods

Since Python has no truly protected or private attributes and anything can be publicly accessed, naming conventions play a big role in identifying method privacy levels:

* Public methods: the usual method for interacting with the object; has no specific naming convention as far as privacy.
* Protected methods: the method's name starts with a single underscore.
* Private methods: the method's name starts with a double underscore (and has no trailing double underscore).

Private methods' names are mangled to protect them from clashes when inherited. Subclasses can define their own `__private()` method, and it will not interfere with the same name on the parent class. Mangling is done by prepending a name with an extra underscore and the class name (regardless of how the name is used or if it exists), effectively giving the method a namespace. In the Parent class, any `__private` identifier is replaced by the name `_Parent__private`, while in the Child class the identifier is replaced by `_Child__private` (in the entire class definition).

Exmaple:

```
user1 at e10000 in /tmp$ cat c.py
class Class(object):

    def public_method(self):
        print("public method")

    def _protected_method(self):
        print("protected method")

    def __private_method(self):
        print("private method")


user1 at e10000 in /tmp$ python -i c.py
>> dir(Class)
['_Class__private_method', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_protected_method', 'public_method']
>> c = Class()
>> c.public_method()
public method
>> c._protected_method()
protected method
>> c._Class__private_method()
private method
>>
```

A few conventions for a readable and maintainable code:

* When you implement a class, the method order should be: 'special methods' (defined with double underscores, for example `__getiitem__`), public, protected, private. This puts at the top the methods that change the standard behaviour of the class, and only then the public interface of the class.
* Public methods are allowed to call any method within the class.
* Limit the use of private methods whenever possible. For example, use private methods to avoid clashes when inheriting classes.

## Testing

Test your code whenever is needed. Automation can play a big role and improve productivity. We recommend pytest.


## Pythonic code

Writing code in a Pythonic way is not only cool but it can bring more readability and better performance. This section is a collaborative one: users are invited to add their Pythonic code to generic solutions.

### Data structure constructors

The Pythonic way to initialize data structures is using the respective empty representation. Examples:

```
# Empty list                                                                   
data = []                                                                      

# Empty dictionary                                                             
data = {}                                                                      

# Empty string                                                                 
data = ""                                                                      

# Empty set                                                                    
data = set()                                                                   

# Empty tuple                                                                  
data = ()
```

The Python website has [a good tutorial about data structures](https://docs.python.org/3/tutorial/datastructures.html).

### class Class

Define the class without inheriting from an object..

```
# Python 3 only code
class Class:
    pass
```

There is a good discussion on [stackoverflow](https://stackoverflow.com/questions/4015417/python-class-inherits-object).

### pathlib vs os.path

`pathlib` is an evolution of the `os.path` library. The main benefits of using `pathlib` are:

* It offers classes representing filesystem paths with semantics appropriate for different operating systems.
* You want to make sure that your code only manipulates paths without actually accessing the OS.

### Type hints (PEP484)

Type hints are defined in PEP484. Although not mandatory, their usage is encouraged.


***

Copyright © 2020 Arm Limited (or its affiliates)
