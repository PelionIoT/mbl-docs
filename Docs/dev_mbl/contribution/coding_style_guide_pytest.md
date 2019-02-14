## pytest in MBL

## Introduction

[Pytest](https://docs.pytest.org/en/latest/) is a framework for testing automation. You can write almost any kind of test with it: from unit tests to more complex and complicated tests for application, libraries and system tests.

It has been chosen as default testing framework for Mbed Linux due to its flexibility, extensibility and easiness to use.

Pytest needs Python to be run. All our management scripts we have on Mbed Linux are written in Python, so choosing a framework Python based was a natural choice.

As tests codebase is Python based, it will need to comply to Python MBL guidelines.

## Pytest documentation
Pytest website contains an [extensive documentation](https://docs.pytest.org/en/latest/contents.html) which we should always refer to. In this page we will report specific cases for MBL project.

## MBL testing

### Where are our tests?

Tests are checked in on [mbl-core repository](https://github.com/ARMmbed/mbl-core/) and they go along with the component to test.

As examples:

* Tests about the RNG are located here: https://github.com/ARMmbed/mbl-core/blob/master/mbl-core/tests/devices/random/test_random_number_generator.py
* Tests for the mbl-app-manager are located here: https://github.com/ARMmbed/mbl-core/tree/master/firmware-management/mbl-app-manager/tests
    * native directory contains code which will be executed off target
    * target directory contains code which will be executed on target
Whenever a new test needs to be created, please discuss with the team where to place it.

### How do we write tests?

Pytest discovers all tests following its [Conventions for Python test discovery](https://docs.pytest.org/en/latest/goodpractices.html#test-discovery): have a read to these conventions to understand how you can name tests.

The way you write the test itself depends mainly on its complexity. A good starting point is the [Getting started guide](https://docs.pytest.org/en/latest/getting-started.htm) on pytest website. Other examples can be found here: https://docs.pytest.org/en/latest/example/simple.html

No boilerplate code is needed to set up the framework: just start writing tests.

The key point of a test that it should assert something. If the assertion is false, the test fails. Pytest provides a good range of facilities for asserting return codes, strings, values, exceptions, exits, etc...

Make sure the test is easy to understand and readable. Docstring should be always provided and tests can be groups within Python classes too.

### How do we configure the environment?

Before running tests a virtual environment needs to be created and activated. As we are using Python3, venv is now part of the standard library. However, since ensurepip is missing, virtual environments have to be created by giving them access to the system site package to use pip (see  IOTMBL-935 - Use pip from within virtual environment rather than system pip OPEN  ).


```
root@mbed-linux-os-1234:/scratch# python3 -m venv test_venv --without-pip --system-site-packages && cd test_venv
root@mbed-linux-os-1234:/scratch/test_venv# source bin/activate
```

Third party python packages need to be installed with pip in a virtual environment (as suggested by https://docs.pytest.org/en/latest/goodpractices.html). Among those packages, pytest and its dependencies need to be installed

Pip can be run as a python module using the -m option as shown below or it can be run with the command "pip3".

```
(test_venv) mbed-linux-os-1234:/scratch/test_venv# python3 -m pip install -U pytest
```


A list of items to install with pip can be provided with a "requirement file"; usually named requirements.txt. If such requirement file is provided, run the following command to install all the items in the list.

```
(test_venv) mbed-linux-os-1234:/scratch/test_venv# python3 -m pip install -r requirements.txt
```


If a full python package is not available, pip is able to install it anyway provided there is a minimal setup.py with the package itself.

All the logic above should be encapsulated in a script which automate all of it:  IOTMBL-852 - Create tooling for setting up/running mbl testing on target OPEN

### How do we run tests?

We run tests on the target (WaRP7, Raspberry PI3, etc...) hence tests code has to be present on the target itself.

The assumption is that we have everything we need (tests code, dependencies, libraries, etc...) on the target. If we need to compile or download dependencies/libraries, this will be executed off target and the output of it will be copied onto the target.

The invocation happens on the target because tests might need access to local resources which a present only on the target.

## Main features

### Assert statement

Pytest uses the standard Python assert for verifying expectations and values in Python tests.

You can assert about expected exceptions, expected warnings and defying your own assertion. More documentation can be found here: https://docs.pytest.org/en/latest/assert.html#assert

### Auto discovery

Auto discovery is a great feature of pytest and this has been already covered in the How do I write my tests? section. It doesn't matter where you put your tests, pytest will discover it if they respect the conventions.

### Fixtures

> A test fixture sets up the system for the testing process by providing it with all the necessary code to initialize it, thereby satisfying whatever preconditions there may be.[1] An example could be loading up a database with known parameters from a customer site before running your test.

> [...]

> This allows for tests to be repeatable, which is one of the key features of an effective test framework.

Pytest does use fixture extensively and they offer a great improvement over the the classic setup/teardown functions:

* fixtures have explicit names and are activated by declaring their use from test functions, modules, classes or whole projects.
* fixtures are implemented in a modular manner, as each fixture name triggers a fixture function which can itself use other fixtures.
* fixture management scales from simple unit to complex functional testing, allowing to parametrize fixtures and tests according to configuration and component options, or to re-use fixtures across function, class, module or whole test session scopes.

Fixtures can be shared across multiple test files via conftest.py: https://docs.pytest.org/en/latest/fixture.html#conftest-py-sharing-fixture-functions

It's pointless to report here the full set of functionality of fixtures. Please refer to the full documentation here: https://docs.pytest.org/en/latest/fixture.html

### Monkeypatching

In pytest, monkeypatch is a fixture that helps you to safely set/delete an attribute, dictionary item or environment variable of to modify sys.path for importing. Full documentation can be found here: https://docs.pytest.org/en/latest/monkeypatch.html

Here a blog post about monkeypatching done right: https://holgerkrekel.net/2009/03/03/monkeypatching-in-unit-tests-done-right/

### Skipping tests

Pytest allows us to skip tests for different reasons:

* skip: not suitable for a specific platform (Eg. Raspberry PI3 doesn't have a camera, so camera tests will be skipped)
* xfail: we expect a test is failing for a known reason. If it passes, it's a xpass and it will be reported

Full documentation and examples can be found here: https://docs.pytest.org/en/latest/skipping.html

## Usage and invocations

### Invocation

You can invoke testing through the Python interpreter from the command line:

```
python -m pytest [...]
```

This is almost equivalent to invoking the command line script pytest [...] directly, except that calling via python will also add the current directory to sys.path.

The full documentation of the usage and invocation can be found here: https://docs.pytest.org/en/latest/usage.html

NOTE: This requires that pytest library is present on the target at the moment of the invocation

## Exit codes

Running pytest can result in six different exit codes:

Exit code 0:	All tests were collected and passed successfully
Exit code 1:	Tests were collected and run but some of the tests failed
Exit code 2:	Test execution was interrupted by the user
Exit code 3:	Internal error happened while executing tests
Exit code 4:	pytest command line usage error
Exit code 5:	No tests were collected
