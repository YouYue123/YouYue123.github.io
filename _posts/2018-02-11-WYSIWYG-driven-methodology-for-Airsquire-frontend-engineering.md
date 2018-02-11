---
layout: post
section-type: post
title: Deterministic WYSIWYG driven methodology for Airsquire frontend engineering
category: Tech
tags: [ 'frontend', 'TDD' ]
---

## TL;DR

This article is divided into 3 parts 

1. What -- What is Deterministic WYSIWYG driven methodology in frontend engineering
2. Why -- What is the befinit we can get from the methodology
2. How -- An opensourced project from Airsquire -- [AirProgressbar](https://www.npmjs.com/package/air-progressbar)

Basically in Airsquire the methodology is practiced mainly based on the following tech stack.

Tech stack item | Explaination
------ | -------
React  | Component-based user interface library 
Typescript | Typed superset of Javscript
Jest | Testing framework
Storybook | UI driven development tool
Yarn | Package manager

### What

So what is Deterministic WYSIWYG driven methodology? Apparently it contains 2 parts *Deterministic* and *WYSIWYG*. 

#### Deterministic

Here is the definition of deterministic for computer science from Wikipedia. In short, it means no randomness in program or the result is predictable for a given input.

> A deterministic model of computation, for example a deterministic Türing machine, is a model of computation such that the successive states of the machine and the operations to be performed are completely determined by the preceding state.

Actually for every piece of javascript you are running it is deterministic for computer in binary world. But in team collaboration, this is not always the truth. Especially when team are developing in [dynamicly typed language](https://www.computerhope.com/jargon/l/looslang.htm) like Javascript, Python, Ruby. Because in most of these languages is no compiler to do static type-checking, you may find yourself searching for a bug that is due to the interpreter misinterpreting the type of a variable.

Here is the definition of deterministic for a development team.

> Codebase is understandable and behavior is predictable for even new team members without knowing those special programming tricks.

This is a sample of "non-deterministic" situation in Javascript

```javascript 
var a = 10
a += '0'
if (a == 10) {
    console.log('I add a zero so it should be still 10')
} else {
    console.log('WTF is happending. Is my math screwed up?')
}
```

This example above will output *WTF is happending. Is my math screwed up?* Because javascript has a rule to change your number into a string explicitly when you want to use '+' between a string and a number. This is already kind-of non-deterministic for a human, because there are so many hidden rules. Imagine you have 100k lines of code which is shared between a team of 10. Every day there are 100 commits. In order to maintain the rubust the only way is to write more test case to cover these parts which can easily checked by a static type-checking system.

The thinking has already widely adopted for lots of excellent companies like [Facebook Flow](https://github.com/facebook/flow), [new Python 3.5 standard](https://docs.python.org/3/library/typing.html), [Microsoft Typescript](https://www.typescriptlang.org/). 


### WYSIWYG

[WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) is an acronym for "what you see is what you get". WYSIWYG implies a user interface that allows the user to view something very similar to the end result

In current frontend engineering world, most people are practicing in component-based approach to seperate outer and inner state management. And peoples are building amazing tools to mock component lifecycle for testing every component individually like [Enzyme](https://github.com/airbnb/enzyme) from Airbnb. Here is the example

```javascript
import React from 'react';
import { expect } from 'chai';
import { render } from 'enzyme';

import Foo from './Foo';

describe('<Foo />', () => {
  it('renders three `.foo-bar`s', () => {
    const wrapper = render(<Foo />);
    expect(wrapper.find('.foo-bar').length).to.equal(3);
  });

  it('renders the title', () => {
    const wrapper = render(<Foo title="unique" />);
    expect(wrapper.text()).to.contain('unique');
  });
});
```

Ther above example is good but not enough. Because frontend is a visual interface. Computer is very good to understand code level test case but we are not. Human are basically visual sensitive and easily detect error for even [1 pixel level problem](http://cdn.ustwo.com/PPP/PP3.pdf). And another problem is that sometimes frontend problem is subjective. Before you see it you will never know whether you will like it or not. Give you 1 minites. Can you imagine a user interface based on the above example? Here is one of the possible [result](https://www.youtube.com/watch?v=fpxDuTNzEqk). 



#### Why

There are 3 reasons we are implementing this methodology

1. Visual-documented codebase
2. Fast iteration with high error tolerence
3. 


### How



