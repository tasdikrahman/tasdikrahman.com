---
title: "Test-driven development as a school of thought"
description: "Test-driven development as a school of thought"
tags: [tdd, testing]
comments: true
share: true
cover_image: '/content/images/2019/02/tdd.jpeg'
---

[Software is eating the world](https://www.wsj.com/articles/SB10001424053111903480904576512250915629460), and so is the world of software development constantly changing. 

One way of developing a project would involve
- analysts figuring out the business requirements and sit for a few weeks if not months
- these requirements would be given out to the architects who would in case break the problem down into manageable chunks
- the chunks themselves would be then given out to the teams which would be the delivering the specific modules.

Nothing, but a typical [waterfall model](https://en.wikipedia.org/wiki/Waterfall_model) scenario, coming with it's obvious pros and cons. 

TDD or Test driven development follows a similar approach to this, which would be gathering the requirements, doing the analysis, modelling the desired system before writing any code for it.

To write a test, you will be gathering what the input should be and what the output should be(on a very high level of thought). 

Once you have some knowledge about the what needs to be done, you will be able to write those fine grained requirements in the form of tests. 

When you are writing those tests, you will figure out the gaps and misunderstandings in the requirements before you've committed those gaps and misunderstandings to the project in the form of executable code. 

What you would gain immediately after adopting TDD
- you get a better understanding of the project that you're gonna write.

This is because of the fact that, you are only implementing the features that are required immediately in the first iteration.

- you will have more confidence while refactoring your code

The problem comes when you would want to refactor certain areas of your code base, and this is where TDD shines if you ask me. You would be confident about the functionality as you would only be testing the end behaviour of your program and not the implementation details of how is it being achieved in unit tests.

This way, you can easily go with the [Red Green refactor](https://blog.cleancoder.com/uncle-bob/2014/12/17/TheCyclesOfTDD.html) cycle. 

- immediate feedback on whether what you're working is fine or not, hence fewer bugs

If you're covering all the edge cases as part of writing the tests, you are already sorted in having a system which has minimal bugs in terms of functionality. 

- loosely coupled, highly modular code. 

Something which I have been trying to follow lately is to not check in any production code, before writing the tests for it. 

Write the specs, run the spec, let it fail (red). Implement the interface that you're testing (make the code green), and then refactor. 

Some of the things which I noticed worked well while writing tests were

- Avoid writing procedural style tests
- Use test doubles while testing, and make sure you are running single units of code in isolation.
- Follow the [given when then](https://martinfowler.com/bliki/GivenWhenThen.html) style of formatting your tests
- Unit tests need true isolation, and they shouldn't be hitting databases or opening sockets when you are testing something. 
- Use only one assertion per test. 

Testing is like security, you can never be 100% per cent sure whether you've got it, but it surely adds to the confidence on what you've built.

So let's say if you have written some 20 tests, and all of them pass, you wouldn't be sure if what you have written is correct or not. But let's say you started with all them being in the red state, you would have much more confidence in the system that you have built.

This back and forth does take time. But it's a process which tries to address the concern of having doubts about whether what you built is a resilient system or not.

## Resources 

- http://misko.hevery.com/2008/08/14/procedural-language-eliminated-gotos-oo-eliminated-ifs/
- https://martinfowler.com/bliki/GivenWhenThen.html
- https://stackoverflow.com/questions/920992/unit-test-adoption
- http://www.natpryce.com/articles/000714.html
- https://javaranch.com/journal/200603/EvilUnitTests.html