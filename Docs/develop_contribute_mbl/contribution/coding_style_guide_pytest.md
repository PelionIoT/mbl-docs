# pytest in MBL

Mbed Linux OS (MBL) uses [pytest](https://docs.pytest.org/en/latest/) as its test automation framework for its flexibility, extensibility and ease of use. You can write almost any kind of test with it, from unit tests all the way to full system tests. Because the MBL test codebase is Python-based, it needs to comply with [Python MBL guidelines](../develop-mbl/python-coding-style-guide.html). This page focuses on MBL, and is a subset of the [extensive documentation](https://docs.pytest.org/en/latest/contents.html) on the pytest site.

## Test environment

We recommend using [a virtual environment](https://docs.pytest.org/en/latest/goodpractices.html) for testing.

## Where are our tests?

Our tests are in [the mbl-core repository](https://github.com/ARMmbed/mbl-core/) and are identified by the component they're testing. For example:

* Tests for the RNG: [https://github.com/ARMmbed/mbl-core/blob/master/mbl-core/tests/devices/random/test_random_number_generator.py](https://github.com/ARMmbed/mbl-core/blob/master/mbl-core/tests/devices/random/test_random_number_generator.py)
* Tests for the mbl-app-manager: [https://github.com/ARMmbed/mbl-core/tree/master/firmware-management/mbl-app-manager/tests](https://github.com/ARMmbed/mbl-core/tree/master/firmware-management/mbl-app-manager/tests). They are divided into *native* (executed off target) and *target* (executed on target) directories.

## How do we write tests?

pytest discovers all tests following its [conventions for Python test auto discovery](https://docs.pytest.org/en/latest/goodpractices.html#test-discovery); read those to understand how you can name discoverable tests.

The way you write the test itself depends mainly on its complexity. A good starting point is the [Getting started guide](https://docs.pytest.org/en/latest/getting-started.htm) and [examples](https://docs.pytest.org/en/latest/example/simple.html) on the pytest site. The key point of a test: it should assert something. If the assertion is false, the test fails. pytest provides a good range of facilities for asserting return codes, strings, values, exceptions, exits, and many others.

Please make sure the test is easy to understand and readable. Always provide Docstring, and consider grouping your tests into an object orientated class type in Python.

## How do we run tests?

We run tests on the target itself - WaRP7, Raspberry PI3 and so on - so the test code has to be present on the target, along with anything else we need: dependencies and libraries, for example.

The invocation happens on the target, because tests might need access to local resources (which are present only on the target).

## Invocation

You can invoke testing through the Python interpreter from the command line:

```
python -m pytest [...]
```

<span class="notes">**Note**: This requires that `pytest` library be present on the target at the moment of invocation.</span>

This is almost equivalent to invoking the command line script pytest [...] directly, except that calling with Python will also add the current directory to `sys.path`.

The full documentation of the usage and invocation can be [found on the pytest site](https://docs.pytest.org/en/latest/usage.html).

## Exit codes

Running pytest can result in six different exit codes:

Exit code 0:	All tests were collected and passed successfully

Exit code 1:	Tests were collected and run but some of the tests failed

Exit code 2:	Test execution was interrupted by the user

Exit code 3:	Internal error happened while executing tests

Exit code 4:	pytest command line usage error

Exit code 5:	No tests were collected


***

Copyright © 2020 Arm Limited (or its affiliates)
