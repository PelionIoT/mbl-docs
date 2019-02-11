## Design and coding style

Be consistent with your changes. Some of the main principles are:

* OpenEmbedded layers and recipes: [https://www.openembedded.org/wiki/Styleguide](https://www.openembedded.org/wiki/Styleguide).
* Python code:
  * Mandatory PEPs (Python Enhancement Proposals): PEP20, PEP8, PEP257.
  * Automated check scripts in our `mbl-tools` repository: [https://github.com/ARMmbed/mbl-tools/tree/mbl-os-0.5/ci/sanity-check](https://github.com/ARMmbed/mbl-tools/tree/mbl-os-0.5/ci/sanity-check).

## C++ coding style guide

### Names

#### Global variables

* Global variables have names with a `g_` prefix (e.g. `g_time_prefix_format`).

#### Classes and Structs

* Names of classes, structs and aliases for them are in `UpperCamelCase`. When the name includes an initialism, only the first letter of the initialism is capitalized (e.g. `MblCloudClient`).

* Use `class` rather than `struct` whenever it contains anything but public data members.

* Names of data members are in `lower_snake_case`.

* Static data member names have an `s_` prefix (e.g. `s_instance`).

#### 2. Enums

* Names of enum types and aliases for them are in `UpperCamelCase`. When the name includes an initialism, only the first letter of the initialism is capitalized (e.g.  `MblError`).

* In C++98 code, enumerator names are usually prefixed with the name of the enum type and an underscore (e.g. `MblError_None`) to mitigate the lack of scoping for enumerator names. In C++11 code that doesn't touch C/C++98 interfaces (and isn't likely to in the future) then scoped enums should be used instead.

#### 3. Methods and Free Functions

* Names of methods and free functions are in `lower_snake_case` except when the name includes a type, when the type name is used verbatim (e.g. `MblError_to_str`).


#### Namespaces

* Namespace names are in `lower_snake_case`.

* Closing braces for namespace blocks are commented with the name of the namespace. For example:

  ```
  namespace mbl {

  /**
   * Initialize the log and trace mechanisms. After calling this the mbed-trace
   * library can be used for logging.
   */
  MblError log_init();

  } // namespace mbl
  ```

#### Files

* Files containing code for a single class are named after that class (e.g. `MblCloudClient.h` and `MblCloudClient.cpp`).

* Files containing code for a single function are named after that function.

* Files containing a number of functions are given a descriptive name in `lower_snake_case` such as `update_handlers.h` and `update_handlers.cpp`

### Whitespace

* Indent 4 spaces at a time (no tabs).

* Function/method definitions look like this:

    ```C++
    ReturnType ClassName::method_name(ParamType1 param1, ParamType2 param2)
    {
        // Stuff
    }
    ```

    or if that gets too long:

    ```C++
    ReturnType ClassName::method_name(
        ParamType1 param1,
        ParamType2 param2,
        ParamType3 param3,
        ParamType4 param4)
    {
        // Stuff
    }
    ```

    or if there's a constructor's initialization list:

    ```C++
    ClassName::ClassName(ParamType1 param1, ParamType2 param2)
        : member1_(param1)
        , member2_(param2)
    {
        // Stuff
    }
    ```

    or possibly:

    ```C++
    ClassName::ClassName(
        ParamType1 param1,
        ParamType2 param2,
        ParamType3 param3,
        ParamType4 param4
    )
        : member1_(param1)
        , member2_(param2)
    {
        // Stuff
    }
    ```

* C++98 enum definitions look like this:

    ```C++
    enum EnumType {
        EnumType_Value1,
        EnumType_Value2,
        EnumType_Value3,
    };
    ```

* Namespace blocks are not indented:

    ```C++
    namespace mbl {

    class MblCloudClient
    {
        // stuff
    };

    } // namespace mbl
    ```

* `if`s look like:

    ```C++
    if (condition) {
        // Stuff
    }
    else {
        // Other stuff
    }
    ```

* `for`s look like:

    ```C++
    for (i = 0; i < n; ++i) {
        // Stuff
    }
    ```

* `while`s look like:

    ```C++
    while (condition) {
        // Stuff
    }
    ```

* `switch`es look like:

    ```C++
    switch (value) {
        case EnumValue1: return stuff;
        case EnumValue2: return other_stuff;
    }
    ```

    or

    ```C++
    switch (value) {
        case EnumValue1:
            DoStuff();
            break;
        case EnumValue2:
            DoOtherStuff();
            break;
    }
    ```

### Other guidelines

mbl-cloud-client contains a mixture of C++98 and C++11, and in certain places must provide C-style APIs. Not all of the following guidelines will apply to both C++98 and C++11 and C-style APIs.

The list of C++ Core Guidelines below is not a complete list of the C++ Core Guidelines that should be followed in this project, it is merely the highlights.

* Don't use exceptions.
* Switch statements over an enum value should not contain a `default` case so that the compiler can catch missing cases.
* From the C++ Core Guidelines:
    * [I.2: Avoid non-`const` global variables](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#i2-avoid-non-const-global-variables)
    * [I.3: Avoid singletons](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#i3-avoid-singletons)
    * [I.22: Avoid complex initialization of global objects](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#i22-avoid-complex-initialization-of-global-objects)
    * [C.4: Make a function a member only if it needs direct access to the representation of a class](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-member)
    * [C.5: Place helper functions in the same namespace as the class they support](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-helper)
    * [C.7: Don't define a class or enum and declare a variable of its type in the same statement](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c7-dont-define-a-class-or-enum-and-declare-a-variable-of-its-type-in-the-same-statement)
    * [C.9: Minimize exposure of members](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c9-minimize-exposure-of-members)
    * [C.10: Prefer concrete types over class hierarchies](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c10-prefer-concrete-types-over-class-hierarchies)
    * [C.20: If you can avoid defining default operations, do](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c20-if-you-can-avoid-defining-default-operations-do)
    * [C.21: If you define or =delete any default operation, define or =delete them all](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c21-if-you-define-or-delete-any-default-operation-define-or-delete-them-all)
    * [C.22: Make default operations consistent](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-zero)
    * [C.35: A base class destructor should be either public and virtual, or protected and nonvirtual](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#Rc-zero)
    * [C.46: By default, declare single-argument constructors explicit](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c46-by-default-declare-single-argument-constructors-explicit)
    * [C.47: Define and initialize member variables in the order of member declaration](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c47-define-and-initialize-member-variables-in-the-order-of-member-declaration)
    * [C.49: Prefer initialization to assignment in constructors](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c49-prefer-initialization-to-assignment-in-constructors)
    * [C.128: Virtual functions should specify exactly one of `virtual`, `override`, or `final`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c128-virtual-functions-should-specify-exactly-one-of-virtual-override-or-final)
    * [C.129: When designing a class hierarchy, distinguish between implementation inheritance and interface inheritance](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c129-when-designing-a-class-hierarchy-distinguish-between-implementation-inheritance-and-interface-inheritance)
    * [C.132: Don't make a function virtual without reason](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c132-dont-make-a-function-virtual-without-reason)
    * [C.133: Avoid protected data](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c133-avoid-protected-data)
    * [C.149: Use `unique_ptr` or `shared_ptr` to avoid forgetting to delete objects created using new](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c149-use-unique_ptr-or-shared_ptr-to-avoid-forgetting-to-delete-objects-created-using-new)
    * [C.164: Avoid conversion operators](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#c164-avoid-conversion-operators)
    * [R.1: Manage resources automatically using resource handles and RAII (Resource Acquisition Is Initialization)](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r1-manage-resources-automatically-using-resource-handles-and-raii-resource-acquisition-is-initialization)
    * [R.3: A raw pointer (a `T*`) is non-owning](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r3-a-raw-pointer-a-t-is-non-owning)
    * [R.4: A raw reference (a `T&`) is non-owning](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r4-a-raw-reference-a-t-is-non-owning)
    * [R.10: Avoid `malloc()` and `free()`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r10-avoid-malloc-and-free)
    * [R.12: Immediately give the result of an explicit resource allocation to a manager object](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r12-immediately-give-the-result-of-an-explicit-resource-allocation-to-a-manager-object)
    * [R.20: Use `unique_ptr` or `shared_ptr` to represent ownership](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#rsmart-smart-pointers)
    * [R.30: Take smart pointers as parameters only to explicitly express lifetime semantics](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#r30-take-smart-pointers-as-parameters-only-to-explicitly-express-lifetime-semantics)
    * [ES.1: Prefer the standard library to other libraries and to "handcrafted code"](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es1-prefer-the-standard-library-to-other-libraries-and-to-handcrafted-code)
    * [ES.5: Keep scopes small](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es5-keep-scopes-small)
    * [ES.10: Declare one name (only) per declaration](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es10-declare-one-name-only-per-declaration)
    * [ES.12: Do not reuse names in nested scopes](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es12-do-not-reuse-names-in-nested-scopes)
    * [ES.21: Don't introduce a variable (or constant) before you need to use it](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es21-dont-introduce-a-variable-or-constant-before-you-need-to-use-it)
    * [ES.25: Declare an object `const` or `constexpr` unless you want to modify its value later on](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es25-declare-an-object-const-or-constexpr-unless-you-want-to-modify-its-value-later-on)
    * [ES.26: Don't use a variable for two unrelated purposes](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es26-dont-use-a-variable-for-two-unrelated-purposes)
    * [ES.47: Use `nullptr` rather than `0` or `NULL`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es47-use-nullptr-rather-than-0-or-null)
    * [ES.49: If you must use a cast, use a named cast](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es49-if-you-must-use-a-cast-use-a-named-cast)
    * [ES.71: Prefer a range-for-statement to a for-statement when there is a choice](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es71-prefer-a-range-for-statement-to-a-for-statement-when-there-is-a-choice)
    * [ES.75: Avoid do-statements](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#es75-avoid-do-statements)
    * [CP.20: Use RAII, never plain `lock()`/`unlock()`](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#cp20-use-raii-never-plain-lockunlock)
    * [CP.22: Never call unknown code while holding a lock (e.g., a callback)](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#cp22-never-call-unknown-code-while-holding-a-lock-eg-a-callback)
    * [NL.18: Use C++-style declarator layout](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl18-use-c-style-declarator-layout)
    * [NL.20: Don't place two statements on the same line](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl20-dont-place-two-statements-on-the-same-line)
    * [NL.25: Don't use `void` as an argument type](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl25-dont-use-void-as-an-argument-type)
    * [NL.26: Use conventional `const` notation](https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md#nl26-use-conventional-const-notation)

## Python coding style guide

### Goal of the document
The intent of this document is to give guidance on how to write Python code within the Mbed Linux team.

The document wants to standardise different aspects of Python, beyond the syntax. The ultimate goal is to have code which is readable and maintainable and conforms to Python upstream standard.

This is not a matter of personal preferences: the code will be opensource and upstream python developers expect the code to be written in a certain and consistent way.

If you have any doubt on how to write good Python code, check how the standard library (on Linux is under /usr/lib/python3.5) is written: you will find plenty of examples.

### Python 2 vs Python 3

It is mandatory to use Python 3 unless there is a strict requirement to use Python 2. This because Python 2 has its days numbered, and it will only be supported until Pycon 2020 with:

* security fixes
* support for new OS versions / tool chains
* rarely bug fixes

Check the countdown here: https://pythonclock.org/.

To ease the transition, the official porting guide has advice for running Python 2 code in Python 3.

### Mandatory PEPs
If you don't know what a PEP is, I suggest you to read a post I wrote time ago on my personal website. There are basically three PEPs which are the fundamental basics for writing any python application:

#### PEP20

This is the PEP you need to keep in mind whenever you develop (not only) with Python. Although it was born as joke, it has been the fundamental principles for Python developers for years.

##### Zen of Python

```
Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import this
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
>>>
```

#### PEP8
PEP8 is the PEP which defines style guide for python. It covers every aspect of good formatting Python code. You don't need to to learn now every rule in the PEP8 but it will come with time. Fortunately there are tools which help you to have a PEP8 clean code. Here an example:

#####PEP8

```
(pycodestyle) dierus01 at e108032-lin in ~$ pip install pycodestyle
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting pycodestyle
  Downloading http://pypi.arm.com/root/pypi/+f/cbc/619d09254895b/pycodestyle-2.4.0-py2.py3-none-any.whl (62kB)
    100% |████████████████████████████████| 71kB 15.7MB/s
Installing collected packages: pycodestyle
Successfully installed pycodestyle-2.4.0
(pycodestyle) dierus01 at e108032-lin in ~$ cat bad_pep8.py
variable = 100

import json

def test():
    print( "Hello world!" )

l = [1,2,3,4]

(pycodestyle) dierus01 at e108032-lin in ~$ pycodestyle bad_pep8.py
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
(pycodestyle) dierus01 at e108032-lin in ~$

```

If you don't want to fix every line manually, there is autopep8 which reformat the input file in order to comply as much as possible to PEP8.

##### autopep8

```
(pycodestyle) dierus01 at e108032-lin in ~$ pip install autopep8
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting autopep8
  Downloading http://pypi.arm.com/root/pypi/+f/228/4d4ae2052fedb/autopep8-1.3.5.tar.gz (109kB)
    100% |████████████████████████████████| 112kB 10.5MB/s
Requirement already satisfied: pycodestyle>=2.3 in /work/virtualenvs/pycodestyle/lib/python2.7/site-packages (from autopep8) (2.4.0)
Building wheels for collected packages: autopep8
  Running setup.py bdist_wheel for autopep8 ... done
  Stored in directory: /home/dierus01/.cache/pip/wheels/d1/3e/3e/7364d92cc3b432dc5120249d6c2cf3f9b0849a014c362e1d5e
Successfully built autopep8
Installing collected packages: autopep8
Successfully installed autopep8-1.3.5
(pycodestyle) dierus01 at e108032-lin in ~$ autopep8 bad_pep8.py
variable = 100

import json


def test():
    print("Hello world!")


l = [1, 2, 3, 4]
(pycodestyle) dierus01 at e108032-lin in ~$ autopep8 bad_pep8.py > good_pep8.py
(pycodestyle) dierus01 at e108032-lin in ~$ pycodestyle good_pep8.py
good_pep8.py:3:1: E402 module level import not at top of file
good_pep8.py:10:1: E741 ambiguous variable name 'l'
(pycodestyle) dierus01 at e108032-lin in ~$
```

Few things from the code above:

* autopep8 by default doesn't perform in place changes (enable it with -i)
* it doesn't fix every error: it cannot generate code for you, don't be lazy

Pystylecode is never wrong, you are (smile) Let's try to stick as much as possible to PEP8 without introducing any exception.

##### Black

Our sanity check pipelines will run few checks on python files. Among those checks, black is run. From their website:

> By using Black, you agree to cede control over minutiae of hand-formatting. In return, Black gives you speed, determinism, and freedom from pycodestyle nagging about formatting. You will save time and mental energy for more important matters.

> Black makes code review faster by producing the smallest diffs possible. Blackened code looks the same regardless of the project you’re reading. Formatting becomes transparent after a while and you can focus on the content instead.

Sanity checks will be running on a submitted Pull request hence they will perform only a checks without changing any code. it is down to the user submitting compliant code.

Black can be integrated on many editors: http://black.readthedocs.io/en/stable/editor_integration.html

#### PEP257

Code is good when there is documentation attached. Luckily we have PEP257 which explains docstring convention for Python. Those are conventions, but please let's stick as much as possible to those.

Like pep8, we have tools to check if we are missing docstrings.

##### pep257

```
(pep257) dierus01 at e108032-lin in ~$ pip install pep257
Looking in indexes: http://pypi.arm.com/dsggnu/dev/simple
Collecting pep257
  Downloading http://pypi.arm.com/root/pypi/+f/d26/cd2a4f572e059/pep257-0.7.0-py2.py3-none-any.whl
Installing collected packages: pep257
Successfully installed pep257-0.7.0
(pep257) dierus01 at e108032-lin in ~$ pep257 bad_pep8.py
bad_pep8.py:1 at module level:
        D100: Missing docstring in public module
bad_pep8.py:5 in public function `test`:
        D103: Missing docstring in public function
(pep257) dierus01 at e108032-lin in ~$ vim good_pep8.py
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

(pycodestylke) dierus01 at e108032-lin in ~$ pep257 good_pep8.py
(pycodestylke) dierus01 at e108032-lin in ~$
```

### Spaces vs Tabs

TL;DR: use 4 spaces indentation

This is already covered by PEP8: https://www.python.org/dev/peps/pep-0008/#tabs-or-spaces

Read it few times and convince yourself that spaces are the way to go. We don't want to end up like this.

For Vim users, stick the following in your ~/.vimrc and your are good to go

```
set expandtab       " tabs are converted to space                                                                                                                                                                                                                                                                             
set smarttab        " Be smart when using tabs ;)                              
set tabstop=4       " numbers of spaces of tab character                       
set shiftwidth=4    " numbers of spaces to (auto)indent                        
set softtabstop=4   " Number of spaces that a <Tab> counts for while performing editing  operations
```

### Virtual environments vs system packages

When running python applications, it is likely you need third party dependencies. There are several ways to install python packages:

* using the system package manager
* using pip install `--user`
* using the Python3 built-in venv module

The suggested method is using virtual environments for the following reasons:

* it can be created/destroyed on the fly
* we can control what we install (down to specific package version)
* it doesn't pollute the system or other virtual environments with unwanted packages

##### Virtual environments

Here an example on how to use virtualenvs.

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

Once the virtual environment has been created, you can use it without activating. You just need to call the binaries which are contained in the virtualenv itself.

##### Virtual environment without activation

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
>>>
(my_venv) user01 at dev-machine in /tmp
$
```

### String formatting

First things first. The basic rule is: never use "+" for sting concatenation. Just don't.

There are 3 ways to format strings. These are:

* The old way: %-formatting
* The current way: str.format()
* The future: f-Strings

Let's forget about the old way and let's use the current way: str.format()

f-Strings are introduced in Python 3.6. The current version of Python on Mbed Linux is 3.5.5 so at the moment we are not able to use f-Strings.

### Returns, exit codes, error management and logging

Every Python application/script should have a clear workflow. This is driven by having a good story in returning and managing errors withing the application.

#### Return statement

Number of return statements should be minimal throughout the codebase:

* one return per function, unless you have a very good reason
* __main__() implementation could have multiple return statements only if it is returning different return codes

#### Error management

Errors are quite important for the life cycle of the application. Those are the rules for having a good strategy in error management:

* methods/functions might propagate the error upwards. Main implementation will catch it eventually.
* methods/functions might catch errors only if they need to perform a corrective action
* methods/functions might catch errors, perform an action and re-propagate upwards the exception
* __main__() is the last resort to catch errors

#### Exit codes

If the application performs as expected, it must return 0 (zero). Any other return code should be easily identifiable in the main implementation. Example:

##### Exit codes

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

#### Only __main__() can exit

Any other sys.exit() call in the application is just wrong. Don't do it unless you have a very good reason (I doubt you have)

##### sys.exit

```
import sys
...
...

if __name__ == "__main__":                                                     
    sys.exit(_main(sys.argv[1:]))
```

#### print() vs logging

For knowing when to use print of logging, just follow these rules:

* if you need to show anything which is not part of the application life cycle, use logging
* if you need to show anything which is part of application life cycle, use print()

There are cases where you might want to use both.

Examples:

* an application needs input from the user and it needs to give context on the request. Use print()
* an application connects to a database. Use logging()
* the application behaviour is to return a string to the user. Use print()
* an application exits with an error. Use logging (with the error level)
* an application needs to show life cycle stages. Use logging and print()

### Public vs Private vs Protected methods

Python doesn't have a proper privacy model and there are no access modifiers like in C++ or Java. There are no truly protected or private attributes and anything can be publicly accessed. Conventions play a big role here and depending how you name methods, those can be either:

* public methods: they usually are the method for interacting with the object
* protected methods: method's name starts with a single underscore
* private methods: method's name starts with a double underscore (and no trailing double underscore)

Private methods' names are mangled to protect them from the clashes when inherited. Subclasses can define their own `__private()` method and these will not interfere with the same name on the parent class.

Mangling is done by prepending any such name with an extra underscore and the class name (regardless of how the name is used or if it exists), effectively giving them a namespace. In the Parent class, any `__private` identifier is replaced by the name `_Parent__private`, while in the Child class the identifier is replaced by `_Child__private`, everywhere in the class definition.

Here an example:

```
dierus01 at e108032-lin in /tmp$ cat c.py
class Class(object):

    def public_method(self):
        print("public method")

    def _protected_method(self):
        print("protected method")

    def __private_method(self):
        print("private method")


dierus01 at e108032-lin in /tmp$ python -i c.py
>>> dir(Class)
['_Class__private_method', '__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_protected_method', 'public_method']
>>> c = Class()
>>> c.public_method()
public method
>>> c._protected_method()
protected method
>>> c._Class__private_method()
private method
>>>
```

Few conventions for having a readable and maintainable code:

* when you implement a class, the methods order should be: 'special methods' (defined with double underscores, for example __getiitem__), public, protected and private. In this way we have at the top the methods which change the standard behaviour of the class, and then the public interface of the class.
* public methods are allowed to call any method within the class
* let's limit the use of private methods unless we need to (E.g.: avoid clashes when inheriting classes)

### Testing

Test your code whenever is needed and it makes sense to test it. Automation could play a big role and can improve productivity.

pytest is the framework of choice for writing tests. This section is well covered on: Pytest MBL guidelines

### Pythonic code
Writing code in a pythonic way is not only cool but it can bring more readability and better performance. This section is a collaborative one: users are invited to add their pythonic code to generic solutions.

#### Data structure constructors

The pythonic way to initialize data structures is using the respective empty representation. Examples:

##### Data structures

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

A good read about data structure can be found on python website: https://docs.python.org/3/tutorial/datastructures.html

#### class Class

TL;DR: define the class without inheriting from object

##### class definition

```
# Python 3 only code
class Class:
    pass
```

If you need the reasoning behind it, read here: https://stackoverflow.com/questions/4015417/python-class-inherits-object

#### pathlib vs os.path

pathlib is an evolution of os.path library. The main benefits of using pathlib are:

* This module offers classes representing filesystem paths with semantics appropriate for different operating systems
* You want to make sure that your code only manipulates paths without actually accessing the OS

#### Type hints (PEP484)

Type hints are defined in PEP484. Although not mandatory their usage is encouraged.

#### setup.py

Setup.py files should include licensing information. The owner should be "Arm Ltd."

Do not include personal contact details such as email. The license we are currently using is BSD-3-Clause.
