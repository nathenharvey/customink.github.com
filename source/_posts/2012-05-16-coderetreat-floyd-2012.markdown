---
layout: post
title: "Coderetreat Floyd 2012"
date: 2012-05-16 15:27
author: tien-nguyen
comments: true
published: true
categories:
  - coderetreat
  - craftsmanship
  - object oriented design
  - test driven development

---

## One Saturday dedicated to honing your craft

It's easy to get caught up in the whirlwind of projects and deadlines and forget that software development is very much a craft that needs to be practiced. The complex requirements and external pressures of a production application often cloud our judgement, forcing us to forgo the fundamental concepts that create great software. 

It's useful to step away from the code you work on day-to-day to focus on honing your skills merely for the sake of improving as a developer. Corey Haines' [Coderetreat](http://coderetreat.org) format facilitates this by strongly suggesting its participants delete their code immediately after each session. By the end of the day, the value created is not the code that is written, but the skills and insights learned from writing that code.

<!-- more -->

The Coderetreat consisted of five 45 minute pairing sessions building [Conway's Game of Life](http://en.wikipedia.org/wiki/Conway's_Game_of_Life). The 'zero-player' game is a form of cellular automata that consists of a grid of dead or alive 'cells' represented by the squares in the grid. Based on a set of four simple rules, the grid comes to life with complex patterns and interactions between the cells.

The rules are:

1. Any live cell with fewer than two live neighbours dies, as if caused by under-population.
2. Any live cell with two or three live neighbours lives on to the next generation.
3. Any live cell with more than three live neighbours dies, as if by overcrowding.
4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

With just a simple initial configuration of cells, patterns like Gosper's Glider Gun can be made:

![Gosper's Glider Gun](/images/gospers_glider_gun.gif)

Rules and image taken from [Conway's Game of Life Wikipedia Article](http://en.wikipedia.org/wiki/Conway's_Game_of_Life)

### Session One

For the first session, I worked with another developer to build up 'Cell' object using CoffeeScript and Jasmine. It went well, but about half way into the session, the requirements of the game were changed to include zombie cells that come to life based on its own set of rules. We were asked about the impact of introducing these new requirements into our code. We were largely unaffected by the change due to not being very far into the implementation, but it was obvious to see how we could have been bitten by the change. 

We chose to deal with alive and dead cells through a boolean attribute on the Cell class. Had we fleshed out the game more, our logic and objects would be tightly bound to this concept and introducing a third state would require considerable time and effort.

Thus, the stage was set for the remaining sessions. We would learn how to better build software that can bend and mold to the often changing demands of the real world.

### Session Two

In Session Two, we were introduced to a set of constraints:

- No primitives across method boundaries
- Less than 3 lines of code per method
- No if statements
- No explicit loops

Immediately we were able to notice changes in our design:

Without if statements, instead of a Cell class with a boolean state, we pushed the state logic into AliveCell and DeadCell classes. Our CellCollection class only needed to know about its Cell-like objects, specific behavior was encapsulated within each AliveCell or DeadCell.

Having less than 3 lines of code per method encouraged a top-down approach to designing our game. With only two lines available to us per method, we were forced to continuously delegate behavior to new objects. The result was a healthy layer of abstractions above very specific implementing objects.

### Session Three

Session Three lifted the constraints from Session Two, but introduced the mute session:

Each pair would have one test writer and one code writer. Adhering to TDD, the test writer would write a test that defined the desired behavior of an object. The code writer would write the code to match that test with two caveats:

1. Neither the test writer nor the code writer could speak to each other
2. The code writer was instructed to be an 'Evil Coder' by only writing enough (likely wrong) code to get the corresponding test to pass

We found that the top-down abstraction-building method learned in the previous session was helpful in avoiding the obstacle that was the Evil Coder. It was easy to write specs of objects merely calling methods on other objects. By not testing the result state of some combination of logic, the Evil Coder was forced to implement exactly the behaviors defined by the test writer.

At the detail layer, proper use of stubs and mocks prevented the Evil Coder from purposefully implementing the wrong code.

### Session Four

Session Four again dropped all previous constraints but required a 2-minute green-to-green cycle:

At the start of writing a test, we would start a timer for 2 minutes. If by the end of those 2 minutes we did not have that test passing, we would delete our changes for that cycle and start over. Between each cycle, we would have 2 minutes to refactor what we had.

After several minutes the entire room rang with alarms. We found ourselves trying to test too much at once. In order to fit within the 2 minutes cycles, it was often helpful to initially use a naïve approach to get our test passing and refactor later. 

By taking the next simplest step in every cycle, we ere able to have completely working code every 2 minutes.

### Session Five

Session Five removed all constraints and allowed us to use what we learned in the previous sessions freely. 

Interestingly enough, although Session Five had the same lack of constraints as Session One, the code we wrote was remarkably different and incorporated many of the ideas from the previous sessions.

### Wrapping Up

At the end of the day, all of the participants grouped up and discussed what they had discovered in during the retreat.

Session Two showed us how encapsulating behaviors into objects can dramatically reduce the impact of changes to your system. Session Three reinforced the idea of Tests as Documentation and revealed an approach that allows for a high degree of unit test coverage. Session Four encouraged us to work in smaller batches, allowing constant verification of working code and reducing the need to debug larger chunks of code at infrequent intervals.

These concepts are not new. They are fundamental aspects of object oriented design and test driven development that have been practiced successfully for years. The Coderetreat is a great way to expose these concepts to those who have never heard of them, and to reinforce the concepts to those who have merely forgotten. 

It's also a great way of keeping in touch with the developer community, with more relaxed gathering and discussion before and after the actual event. I had a great weekend in Floyd and I would recommend a Coderetreat for anyone interested a weekend of practice and reflection.

I hope to see you at one some day!
