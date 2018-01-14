---
layout: post
section-type: post
title: 2018 good found for javascript - Code formatting in Prettier
category: tech
tags: [ 'Javascript', 'Code formatting', 'Prettier', 'EsLint']
---

## Javascript Code lint really matters

Linting javascript code is always a debatable issue for either a large development team or a single developer. It matters because we want :
  1. Efficient team collaboration
  2. Sustainable codebase maintainability
  3. Less stupid mistake

And linting toolkit is even more important for javascript than Java or C++. As Javascript is too freedom because its dynamic and loosely-type feature. And it is one of the fastest envolving programming language in the world. Community and developers are even using its future feature(ES9 features for instance) by using (Babel)[https://babeljs.io/] or other transcompile tools. In the same time, Python community is still under discussing about (python 2 vs python 3)[https://www.activestate.com/blog/2017/01/python-3-vs-python-2-its-different-time]. Because of javascript is growing too fast, there are lots of ways to write your code like using semi-comma or not to use it. This can be a 1 day discussion for your team.

So it is really important to write your javascript code inside a limitation box. And it would be better if I am out the box someone can help me to move it back automatically.

## Comparison about javascript lint tool

There are lots of ways to help you to build the limitation box.

Here I list some of the major players today and compare pros and cons.

Name | Description | Pros | Cons
--- | --- | --- | --
[JSLint](http://jslint.com/)   | JSLint is a JavaScript program that looks for problems in JavaScript programs. It is a code quality tool.  | It comes with the out-of-box configuration   | 1.It is not ready for fully customizable configurations  2.It only supports 'the good parts of ES6' now 3.Undocumented features 4.No custom rule support
[JSHint](http://jshint.com/)   | JSHint is a community-driven tool that detects errors and potential problems in JavaScript code. | 1.Customizable configuration  2.Basic ES6 support 3.Contains Out-of-box features | 1.No JSX support 2.Hard to know which rule is failed 3.No custom rule support
[ESLint](https://eslint.org/) | The pluggable linting utility for JavaScript and JSX |  1.Completely configurable 2.Completely ES6 and JSX support 3.Good community to provide out-of-box configurations 4. Allow to create custom rules |  1.Bit hard to learn to use in the first time, because it has too many customizable points.

The comparison clearly shows that ESLint is the best tool for modern javascript now.

## Why EsLint is not enough

Because EsLint only help you to setup the limitation box, but when you step out of the box it will not help you to move back.

There are several choices for auto code formatting. Even Eslint itself has a feature for [automatically fixing](https://developer.ibm.com/node/2016/07/27/auto-fixing-formatting-your-javascript-with-eslint/). And in Atom, a package called [linter-eslint](https://github.com/AtomLinter/linter-eslint) will help to read your eslint configuration file and try to automatically fix for you

Actually personally I am facing issue while using the above tool, sometimes it just not help me to fix it. The best tool ever I have use is [Prettier](https://prettier.io/) and its Atom plugin - [PrettierAtom](https://github.com/prettier/prettier-atom). Actually this is still arguable
1. [If eslint can auto fix/format code why to use Prettier? #101](https://github.com/prettier/prettier-eslint/issues/101#issuecomment-335822167)
2. [Discussion in Boostrap team for adding Prettier to do code formatting](https://github.com/twbs/bootstrap/pull/24323)

## Recommended javascript lint setup

Use the default setting or default [prettier-standard](https://www.npmjs.com/package/prettier-standard) in Prettier as much as possible!!!

The best lint setup should not contains hundreds lines of code. It just need to be simple and useful. I quite love the desing principle in [Javascript Standard Style](https://standardjs.com/) -- "The beauty of JavaScript Standard Style is that it's simple. No one wants to maintain multiple hundred-line style configuration files for every module/project they work on. Enough of this madness!"

And adopting default standard style means ranking the importance of code clarity and community conventions higher than personal style. This might not make sense for 100% of projects and development cultures, however open source can be a hostile place for newbies. Setting up clear, automated contributor expectations makes a project healthier.

Anyway the goal of using this is to put more effort in real development instead of endless&useless style debating.

## Need to in the future

I would like to write a static type check for javascript in TypeScript vs Flow later. And in my company I already setup a hard rule to not use any so-called "freedom" of Javascript in production environment anymore. It's time to write professional javascript instead of being a "cowboy".
