---
layout: post
section-type: post
title: Flutter code coverage with Github actions
category: engineering
tags: []
---

![image](/img/flutter-test-coverage-header.png)

### TL;DR

This article will talked about the following items in sequence

1. Code coverage best practices
2. How to use flutter code coverage with VS Code.
3. My consideration on the Self maintained vs 3rd party CI/CD workflow integration for code coverage.
4. How to reflect the code coverage in pull request comments by using a simple github action

At the end I decided to build a self-maintained github action over 3rd party due to security and customizability considerations

### Code coverage best practices

![image](/img/code-coverage-matters.png)

Before talking about why code coverage is important I want to list down some important principles for code coverages from my experience

- High code coverage does not gurantee high quality of code
- Low code coverage dose gurantee most of the code is in a blackbox
- There is no fixed number for what is good or bad for code coverages
- Knowing what is tested and what is not is important, judge whether it is enough from human point view
- Make sure the critical user flow is covered
- Unit test code coverage is only a small fraction of the whole testing process, but it is the basic
- Do not obsesse on how to get from 90% code coverage to 95%

Apparentlly, code coverage is not a silver bullet to improve code quality, then why do we bother to do it? Here are the reasons

#### Logic transparency

We will know something we don't know via code coverage reporting.

![image](/img/dont-know.png)

And it is pretty hard to figure out how many logic branches in our code base for a human brain. But even 486 computer can point out the missing branches and statement in just a second. This will help engineers to rethink about logic design and implementation. Sometimes the code can just be much simplier.

![image](/img/logic-complexity.jpeg)

![image](/img/missing-branches.png)

#### Testable code

I am having the opinion that unit tests automatically help you write better code. If you write a giant method that is packed with tons of conditions and depended on complex parameters, environment viriables. I will assure you would suffer when comes to writing tests to have a reasonable coverage. Lowly coupled, highly cohesive code will be a natural result after refactoring with unit test and coverage

![image](/img/perfect-software-twitter.png)

#### Confidence of changing

Code coverage is a objective metric. The changing in code base is inevitable. Especially in the mordern software engineer we reply on a lot of external libraries.

Things can be tricky when upgrading them. But a good code coverage will at least help you to ensure the covered code is guranteed to work in the testing cases.

![image](/img/depend-hell.png)

Good article to share

[What is the defination of code Coverage](https://en.wikipedia.org/wiki/Code_coverage)

[Google Code Coverage Best Practices](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html)

[The importance of Code coverage](https://blog.cloudboost.io/the-importance-of-code-coverage-9b4d513f39b4)

### Tutorial (How to use flutter code coverage with VS Code)

### Self maintained vs 3rd party CI/CD integration

### Tutorial (How to build a github action to reflect code coverage in pull request comments)
