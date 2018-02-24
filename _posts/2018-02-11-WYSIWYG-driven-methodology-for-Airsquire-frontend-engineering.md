---
layout: post
section-type: post
title: Visual Test-Driven driven for Airsquire frontend engineering
category: Tech
tags: [ 'Frontend', 'Airsquire' ]
---

### TL;DR

This article is divided into 3 parts 

In Airsquire, we are using Deterministic WYSIWYG driven methodology.

1. What -- What is Deterministic WYSIWYG driven methodology in frontend engineering
2. Why -- What is the befinit we can get from the methodology
2. How -- An opensourced project from Airsquire -- [AirProgressbar](https://www.npmjs.com/package/air-progressbar)

Basically in Airsquire the methodology is practiced mainly based on the following tech stack.

React -- Component-based user interface library 

Typescript -- Typed superset of Javscript

Jest -- Testing framework

Storybook -- UI driven development tool

Yarn -- Package manager

### What

So what is Deterministic WYSIWYG driven methodology? Apparently it contains 2 parts *Deterministic* and *WYSIWYG*. 

#### Deterministic

Here is the definition of deterministic for computer science from Wikipedia. In short, it means no randomness in program or the result is predictable for a given input.

> A deterministic model of computation, for example a deterministic Türing machine, is a model of computation such that the successive states of the machine and the operations to be performed are completely determined by the preceding state.

Actually for every piece of javascript you are running it is deterministic for computer in binary world. But in team collaboration, this is not always the truth. Especially when team are developing in [dynamicly typed language](https://www.computerhope.com/jargon/l/looslang.htm) like Javascript, Python, Ruby. Because in most of these languages is no compiler to do static type-checking, you may find yourself searching for a bug that is due to the interpreter misinterprets a variable.

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

![image](https://blog.testim.io/wp-content/uploads/2017/06/The_Present_02.jpg)

Frontend codebase is occupying a huge percentage in current technology world. This thinking has already widely adopted for lots of excellent companies' frontend team like [Facebook Flow](https://github.com/facebook/flow), [new Python 3.5 standard](https://docs.python.org/3/library/typing.html), [Microsoft Typescript](https://www.typescriptlang.org/). 


#### WYSIWYG

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


### Why

There are 3 reasons we are implementing this methodology

1. Visually-documented codebase
2. Fast iteration with high error tolerence
3. User experience based development

#### Visually-documented codebase

![image](https://youyue123.github.io/img/Visually-documented-codebase-1.png)
![image](https://youyue123.github.io/img/Visually-documented-codebase-2.png)

Compare the above example with this [3368 lines of code file](https://github.com/lonelyplanet/backpack-ui/blob/master/stories/index.jsx). They are presenting the same thing. The difference is obvious. Especially Airsquire team is developing a 3D solution and it is hard to imagine only by the code itself. This visually documented codebase really helps us to communicate with client directly and efficiently.

#### Fast iteration with high error tolerence

![image](https://youyue123.github.io/img/Fast-iteration-with-high-error-tolerence.png)

The static typing system helps to detected lots of low-level mistakes. Our team is delivering product weekly based. There is always a trade-off between test coverage and functional points. By using javascript we really befinit from its amazing opensource community, efficient syntax sugars, fast prototyping speed. But *there's no such thing as a free lunch*, team is facing unstability issues for using javascript in both frontend and backend. It really helps us to avoid weird debugging edge cases

Another reason is that our team’s developers are already accustom to statically-typed languages like Swift, C++, Java.

#### User experience based development

![image](https://youyue123.github.io/img/User-experience-based-development.jpg)

This is the following effect by our visual-documented codebase. Sometimes we will work in client side and asking their opnion about user experience. They are not computer science people and showing thousands line of code will just shock them. This document gives an interactive way of communication between team and customer directly. 



### How

To be Continue