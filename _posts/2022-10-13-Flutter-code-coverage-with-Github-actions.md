---
layout: post
section-type: post
title: Flutter code coverage with Github actions
category: engineering
tags: []
---

### Summary

This article will talk about the following items in sequence

1. Why code coverage is important
2. Flutter code coverage with VS Code
3. CI/CD integration
4. Code coverage with github action

### Why code coverage is important

![image](/img/code-coverage-matters.png)

Before talking about why code coverage is important I want to list down some important principles for code coverage from my experience

- High code coverage does not gurantee high quality of code
- Low code coverage dose gurantee most of the code is in a blackbox
- There is no fixed % number to determine good and bad code coverage
- Knowing what is tested and what is not is important
- Make judgement for sufficient code coverage from human point view
- Unit test code coverage is only a small fraction of the whole testing process, but it is the basic
- Do not obsesse on how to get from 90% code coverage to 95%
- Do make effort to get from 15% code coverage to 50%

Apparentlly, code coverage is not a silver bullet to improve code quality, why do we bother to do it? Here are the reasons

#### Logic transparency

We will know things we don't know

![image](/img/dont-know.png)

It is pretty hard to figure out how many logic branches in our codebase for a human brain. But even an intel Celeron PC can point out the missing branches and statement in just a second

This will help engineers to rethink carefully about logic branches. Sometimes the logic can just be much simplier to achieve the same goal

![image](/img/logic-complexity.jpeg)

#### Testable code

I am having the opinion that unit tests automatically help you write better code

If you write a giant method that is packed with tons of conditions. In the same time, it also depended on complex inputs

I will assure you would suffer when it comes to write tests with a reasonable coverage

Lowly coupled, highly cohesive code will be a natural result with good unit test and coverage

![image](/img/missing-branches.png)

#### Confidence of changing

The changing in codebase is inevitable. Especially in the mordern software engineering world, we reply on a lot of dependencies

Things can be tricky when changing them.

A good code coverage will at least help you to ensure the bottom line of quality

![image](/img/depend-hell.png)

Good articles to share

[What is the defination of code Coverage](https://en.wikipedia.org/wiki/Code_coverage)

[Google Code Coverage Best Practices](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html)

[The importance of Code coverage](https://blog.cloudboost.io/the-importance-of-code-coverage-9b4d513f39b4)

### Flutter code coverage with VS Code

Flutter provides an efficient and out-of-box testkit. People don't need to fight for their favourite framework like the javascript world(Jest vs Jasmine)

By running the following command in a flutter project, it will generate a coverage folder which contains an [lcov.info](https://github.com/linux-test-project/lcov)

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

With these 2 plugins you will get something like this:

![image](/img/flutter-test-coverage-header.png)

Another way of viewing it is to convert the lcov.info into a html report.

By running the following command, a html report will be generated. [install lcov](https://formulae.brew.sh/formula/lcov) is a prerequisite

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

| Platform          | [Sonarqube](https://en.wikipedia.org/wiki/SonarQube) (Self-hosted)                          | [CodeCov](https://about.codecov.io/)       | [Github Actions](https://github.com/features/actions)          |
| ----------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------ | -------------------------------------------------------------- |
| Description       | Opensourced Platform for continuous inspection of code quality to perform automatic reviews | Platform dedicated for code coverage       | Toolkit to automate your build, test, and deployment pipeline. |
| Hard to customize | <span style="color: green;">Low</span>                                                      | <span style="color: red;">High</span>      | <span style="color: green;">Low</span>                         |
| Complexity        | <span style="color: red;">High</span>                                                       | <span style="color: green;">Low</span>     | <span style="color: green;">Low</span>                         |
| Security concerns | <span style="color: red;">High</span>                                                       | <span style="color: red;">High</span>      | <span style="color: green;">Low</span>                         |
| Cost              | <span style="color: red;">High</span>                                                       | <span style="color: yellow;">Medium</span> | <span style="color: green;">Low</span>                         |
| Maintainance      | <span style="color: red;">High</span>                                                       | <span style="color: green;">Low</span>     | <span style="color: yellow;">Medium</span>                     |

From the above analysis, I decided to go for Github acions mainly for

-- Security

I don't really like the idea to send source code out to a 3rd party platform.

Especially there are some security breaches happened for both Sonaqube and CodeCov recently.

[Codecov security breach](https://blog.gitguardian.com/codecov-supply-chain-breach/)

[Sonarqube security breach](https://www.bleepingcomputer.com/news/security/fbi-hackers-stole-government-source-code-via-sonarqube-instances/)

-- Simplicity

I just want something simple to create good enough code coverage info during the pull request process

A self-hosted code quality platform like Sonarcube is an overkill for me

Also I don't want to introduce another platform in our process just for this, that's why I passed codecov on this too

-- Synthesis

Due to all CI/CD workflows are running in Github, a github action can easily access all resources and reuse the logic from others.

### Code coverage with github action

As my personal preference I choosed [Typescript action](https://github.com/actions/typescript-action) as base.

But there is a special thing for typescript action, it will take two steps to be compiled to the final deliverables.

```bash
typescript --> javascript --> bundled javascript
          (tsc)          (ncc)
```

#### Goal

- Overall coverage as summary
- Changed file coverage as detail
- Create comment on pull request
- Update comment when update applies to pull request

#### Implementation key points

The actual implementation is straightforward. It is mostly based on the native github libraries.

- [@actions/core](https://www.npmjs.com/package/@actions/core)

  1. To get input from environment
  2. To output logs

- [@actions/exec](https://www.npmjs.com/package/@actions/exec)

  1. To install lcov
  2. To run lcov command to generate summary and detail

- [@actions/github](https://www.npmjs.com/package/@actions/github)
  1. To verify github event type
  2. To get the changed file list for this pull request
  3. To create comment when there is not coverage comment yet
  4. To update comment when there is existing coverage comment

#### Caveat

The shown implementation will be only working for Debian GNU/Linux due to the usage of apt-get.
