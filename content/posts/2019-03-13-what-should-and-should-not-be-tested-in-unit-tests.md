---
title: "What should and should not be tested in unit tests?"
description: "What should and should not be tested in unit tests?"
tags: [tdd, testing]
comments: true
share: true
cover_image: '/content/images/2019/03/pillar-capitals-2135682_1920.jpg'
---

> I have written about [F.I.R.S.T principles](https://www.tasdikrahman.com/2019/03/13/f-i-r-s-t-principles-of-testing/) of testing and [TDD as a school of thought](https://www.tasdikrahman.com/2019/02/08/why-should-I-follow-test-driven-development/)

Probably an extreme opinion, but this is how [Jeff Atwood](https://blog.codinghorror.com/i-pity-the-fool-who-doesnt-write-unit-tests/) puts it

> I Pity The Fool Who Doesn’t Write Unit Tests

### But what should you test?

This I generally try to follow

- Test the common case of everything you can. This will tell you when that code breaks after you make some change (which is, in my opinion, the single greatest benefit of automated unit testing).
- Test the edge cases of a few unusually complex code that you think will probably have errors.
- Whenever you find a bug, write a test case to cover it before fixing it
- Add edge-case tests to less critical code whenever someone has time to kill.

This will not only help you deliver and release faster, but will also make you more confident about your own codebase.

> Writing tests and having 100% code coverage does not necessarily mean that your code is bug free, but I feel it’s certainly better than having no tests at all.

### What should you not test?

- The code is trivial. A getter that returns 0 doesn’t need to be tested, and changes will be covered by tests for its consumers.
- The code simply passes through into a stable API. I’ll assume that the standard library works properly.
- The code needs to interact with other deployed systems; then an integration test is called for.
- If the test of success/fail is something that is so difficult to quantify as to not be reliably measurable, such as steganography being unnoticeable to humans.
- If the test itself is an order of magnitude more difficult to write than the code.
- If the code is throw-away or placeholder code. If there’s any doubt, test.

### References

- https://softwareengineering.stackexchange.com/a/147075/169827
- https://softwareengineering.stackexchange.com/a/754/169827
- https://blog.codinghorror.com/i-pity-the-fool-who-doesnt-write-unit-tests/
