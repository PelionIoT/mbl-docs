# pytest in MBL

## Introduction

Mbed Linux OS (MBL) uses [pytest](https://docs.pytest.org/en/latest/) as its test automation framework for its flexibility, extensibility and ease of use. You can write almost any kind of test with it, from unit tests all the way to full system tests. Because the MBL test codebase is Python-based, it needs to comply with [Python MBL guidelines](../develop-mbl/python-coding-style-guide.html). This page focuses on MBL, and is a subset of the [extensive documentation](https://docs.pytest.org/en/latest/contents.html) on the pytest site.

## MBL testing

### Where are our tests?

Our tests are in [the mbl-core repository](https://github.com/ARMmbed/mbl-core/) and are identified by the component they're testing. For example:

* Tests for the RNG: [https://github.com/ARMmbed/mbl-core/blob/master/mbl-core/tests/devices/random/test_random_number_generator.py](https://github.com/ARMmbed/mbl-core/blob/master/mbl-core/tests/devices/random/test_random_number_generator.py)
* Tests for the mbl-app-manager: [https://github.com/ARMmbed/mbl-core/tree/master/firmware-management/mbl-app-manager/tests](https://github.com/ARMmbed/mbl-core/tree/master/firmware-management/mbl-app-manager/tests). They are divided into *native* (executed off target) and *target* (executed on target) directories.

### How do we write tests?

pytest discovers all tests following its [conventions for Python test auto discovery](https://docs.pytest.org/en/latest/goodpractices.html#test-discovery); read those to understand how you can name discoverable tests.

The way you write the test itself depends mainly on its complexity. A good starting point is the [Getting started guide](https://docs.pytest.org/en/latest/getting-started.htm) and [examples](https://docs.pytest.org/en/latest/example/simple.html) on the pytest site. The key point of a test: it should assert something. If the assertion is false, the test fails. pytest provides a good range of facilities for asserting return codes, strings, values, exceptions, exits, and many others.

Please make sure the test is easy to understand and readable. Always provide Docstring, and consider grouping your tests within<!--"within" - as part of classes of other things - or "in" - a class of tests--> Python classes.

<!--I removed the bit about configuring an environment. It makes sense internally, but not to external readers-->
<!--for the same reason, I removed a few other lines from the doc. They would have seemed patronising at best.-->

### How do we run tests?

We run tests on the target itself - WaRP7, Raspberry PI3 and so on - so the test code has to be present on the target, along with anything else we need: dependencies and libraries, for example.

The invocation happens on the target, because tests might need access to local resources (which are present only on the target).

## Main features

### Assert statement

pytest uses the standard Python `assert` for verifying expectations and values in Python tests. You can assert expected exceptions, expected warnings and defying your own assertion. More documentation [can be found on the pytest site](https://docs.pytest.org/en/latest/assert.html#assert).
<!--we're teaching people how to use a third-party tool; is that useful? Feels like it should be dropped now that this is going out of Confluence-->

### Auto discovery

Auto discovery has been already covered in the How do we write tests? section. It doesn't matter where you put your tests - pytest will discover them if they respect the conventions.

### Fixtures

> A test fixture sets up the system for the testing process by providing it with all the necessary code to initialize it, thereby satisfying whatever preconditions there may be. An example could be loading up a database with known parameters from a customer site before running your test.

> [...]

> This allows for tests to be repeatable, which is one of the key features of an effective test framework.

> https://en.wikipedia.org/wiki/Test_fixture


<!--this, again, feels like teaching people pytest rather than how MBL uses it; it should probably be removed-->

[Pytest does use fixtures extensively](https://docs.pytest.org/en/latest/fixture.html), and they offer a great improvement over the the classic setup/teardown functions:

* Fixtures have explicit names and are activated by declaring their use from test functions, modules, classes or whole projects.
* Fixtures are implemented in a modular manner, as each fixture name triggers a fixture function that can itself use other fixtures.
* Fixture management scales from simple unit to complex functional testing, allowing to parametrise fixtures and tests according to configuration and component options, or to reuse fixtures across function, class, module or whole test session scopes.

Fixtures can be shared across multiple test files [using `conftest.py`](https://docs.pytest.org/en/latest/fixture.html#conftest-py-sharing-fixture-functions).

### Monkeypatching

In pytest, monkeypatch is a fixture that helps you safely set and delete an attribute, dictionary item or environment variable or to modify `sys.path` for importing.

Read more:

* [Full documentation](https://docs.pytest.org/en/latest/monkeypatch.html).
* [And a useful blog post from holgerkrekel.net](https://holgerkrekel.net/2009/03/03/monkeypatching-in-unit-tests-done-right/).

### Skipping tests

pytest allows skipping tests for different reasons:

* `skip`: when the test is unsuitable for a specific platform. For example, skip camera tests on the RPi3, which doesn't have a camera.
* `xfail`: we expect a test to fail, and we know why. If it passes, it's an `xpass` and will be reported.

Full documentation and examples can be found [on the pytest site](https://docs.pytest.org/en/latest/skipping.html).

## Usage and invocations

### Invocation

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
