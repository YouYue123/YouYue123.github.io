---
layout: post
section-type: post
title: Flutter code coverage with Github actions
category: engineering
tags: []
---

### TL;DR

This article will talk about the following items in sequence

1. Why code coverage is important
2. How to use flutter code coverage with VS Code.
3. CI/CD integration
4. How to reflect the code coverage in pull request comments by using github action

### Why code coverage is important

![image](/img/code-coverage-matters.png)

Before talking about why code coverage is important I want to list down some important principles for code coverages from my experience

- High code coverage does not gurantee high quality of code
- Low code coverage dose gurantee most of the code is in a blackbox
- There is no fixed % number for what is good or bad for code coverages
- Knowing what is tested and what is not is important, make judgement for sufficiency from human point view
- Unit test code coverage is only a small fraction of the whole testing process, but it is the basic
- Do not obsesse on how to get from 90% code coverage to 95%

Apparentlly, code coverage is not a silver bullet to improve code quality, why do we bother to do it? Here are the reasons

#### Logic transparency

We will know things we don't know

![image](/img/dont-know.png)

It is pretty hard to figure out how many logic branches in our codebase for a human brain. But even an intel Celeron PC can point out the missing branches and statement in just a second.

![image](/img/logic-complexity.jpeg)

This will help engineers to rethink carefully about logic branches. Sometimes the logic can just be much simplier to achieve the same goal.

![image](/img/missing-branches.png)

#### Testable code

I am having the opinion that unit tests automatically help you write better code.

If you write a giant method that is packed with tons of conditions and depended on complex parameters, environment viriables. I will assure you would suffer when it comes to writing tests to have a reasonable coverage.

Lowly coupled, highly cohesive code will be a natural result with good unit test and coverage

#### Confidence of changing

The changing in codebase is inevitable. Especially in the mordern software engineering world, we reply on a lot of external dependencies.

Things can be tricky when upgrading them. But a good code coverage will at least help you to ensure a bottom line of quality

![image](/img/depend-hell.png)

Good articles to share

[What is the defination of code Coverage](https://en.wikipedia.org/wiki/Code_coverage)

[Google Code Coverage Best Practices](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html)

[The importance of Code coverage](https://blog.cloudboost.io/the-importance-of-code-coverage-9b4d513f39b4)

### Tutorial (How to use flutter code coverage with VS Code)

Flutter provides an efficient and out-of-box testkit. People don't need to fight for their favourite framework like the javascript world(Jest vs Jasmine).

By running the following command in a flutter project, it will generate a coverage folder which contains an [lcov](https://github.com/linux-test-project/lcov) info file.

```bash
flutter test --coverage
```

In order to see the coverage in VS code nicely, I would recommend 2 extensions

- [Flutter Coverage](https://marketplace.visualstudio.com/items?itemName=Flutterando.flutter-coverage) for overview

But if the coverage folder is not on the root directory of project, you might need to configure it something as below

```
     "flutter-coverage.coverageFilePaths": [
        "client/user/coverage"
    ]
```

- [Coverage Gutters](https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters) for line by line information

With these 2 plugins you will get something like the following shows

![image](/img/flutter-test-coverage-header.png)

Another way of viewing it is to convert the lcov.info into a html report.

By running the following command, a html report will be generated. You might need to [install lcov](https://formulae.brew.sh/formula/lcov) for it

```bash
genhtml coverage/lcov.info -o ./coverage/html
```

![image](/img/lcov.png)

#### Caveat

The Functions and Branches test coverage will have no data from the out-of-box flutter test.

But there is a dart library from Dart team called package:coverage which supports it. But making them to work together is another long topic

Related discsussion

https://github.com/flutter/flutter/issues/108313

https://github.com/dart-lang/coverage/issues/141

### CI/CD integration

[Codecov security breach](https://blog.gitguardian.com/codecov-supply-chain-breach/)

[Sonarqube security breach](https://www.bleepingcomputer.com/news/security/fbi-hackers-stole-government-source-code-via-sonarqube-instances/)

| Platform          | [Sonarqube](https://en.wikipedia.org/wiki/SonarQube) (Self-hosted)                          | [CodeCov](https://about.codecov.io/)       | [Github Actions](https://github.com/features/actions)          |
| ----------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------ | -------------------------------------------------------------- |
| Description       | Opensourced Platform for continuous inspection of code quality to perform automatic reviews | Platform dedicated for code coverage       | Toolkit to automate your build, test, and deployment pipeline. |
| Hard to customize | <span style="color: green;">Low</span>                                                      | <span style="color: red;">High</span>      | <span style="color: green;">Low</span>                         |
| Complexity        | <span style="color: red;">High</span>                                                       | <span style="color: green;">Low</span>     | <span style="color: green;">Low</span>                         |
| Security concerns | <span style="color: red;">High</span>                                                       | <span style="color: red;">High</span>      | <span style="color: green;">Low</span>                         |
| Cost              | <span style="color: red;">High</span>                                                       | <span style="color: yellow;">Medium</span> | <span style="color: green;">Low</span>                         |
| Maintainance      | <span style="color: red;">High</span>                                                       | <span style="color: green;">Low</span>     | <span style="color: yellow;">Medium</span>                     |

From the above analysis, I decided to go for Github acions mainly for security, customization and synthesis to our existing CI/CD workflows in Github.

### Tutorial (How to build a github action to reflect code coverage in pull request comments)

Then let's talk about how to make it working.