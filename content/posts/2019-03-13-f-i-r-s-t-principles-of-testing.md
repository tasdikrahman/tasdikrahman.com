---
title: "F.I.R.S.T principles of testing"
description: "F.I.R.S.T principles of testing"
tags: [tdd, testing]
comments: true
share: true
cover_image: '/content/images/2019/03/painter.jpeg'
---

# First principles of testing stand for

- Fast
- Isolated/Independent
- Repeatable
- Self-validating
- thorough

> Bugs are introduced in the parts of code, which we usually don’t pay attention to, or places which are too hard to understand.

### Fast

The developer shouldn’t hesitate to run the run the unit tests at any point of their development cycle, even if there are thousands of unit tests. They should run and show you the desired output in a matter of seconds

### Isolated

For any given unit test, for its environment variables or for its setup. It should be independent of everything else should so that it results is not influenced by any other factor.

Should follow the [3 A’s of testing: Arrange, Act, Assert](https://xp123.com/articles/3a-arrange-act-assert/)

In some literature, it’s also called as [Given, when, then](https://martinfowler.com/bliki/GivenWhenThen.html).

### Arrange

All the data should be provided to the test when you’re about to run the test and it should not depend on the environment you are running the tests

### Act

Invoke the actual method under test

### Assert

At any given point, a unit test should only assert one logical outcome, multiple physical asserts can be part of this physical assert, as long as they all act on the state of the same object.

Preferably, don’t do any actions after the assert call

### Repeatable

Tests should be repeatable and deterministic, their values shouldn’t change based on being run on different environments.
Each test should set up its own data and should not depend on any external factors to run its test

### Self-validating

You shouldn’t need to check manually, whether the test passed or not.

### Thorough

- should cover all the happy paths
- try covering all the edge cases, where the author would feel the function would fail.
- test for illegal arguments and variables.
- test for security and other issues
- test for large values, what would a large input do their program.
- should try to cover every use case scenario and not just aim for 100% code coverage.

## References:

- https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing
- https://martinfowler.com/bliki/GivenWhenThen.html
- https://xp123.com/articles/3a-arrange-act-assert/
