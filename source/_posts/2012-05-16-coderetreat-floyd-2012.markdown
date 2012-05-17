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

It's useful to step away from the code you work on day-to-day to focus on honing your skills merely for the sake of improving as a developer. Corey Haines' Coderetreat format facilitates this by strongly suggesting its participants delete their code immediately after each session. By the end of the day, the value created is not the code that is written, but the skills and insights learned from writing that code.

The Coderetreat consisted of five 45 minute pairing sessions building [Conway's Game of Life](http://en.wikipedia.org/wiki/Conway's_Game_of_Life). The 'zero-player' game is a form of cellular automata that consists of a grid of dead or alive 'cells' represented by the squares in the grid. Based on a set of four simple rules, the grid comes to life with complex patterns and interactions between the cells.

The rules are:

1. Any live cell with fewer than two live neighbours dies, as if caused by under-population.
2. Any live cell with two or three live neighbours lives on to the next generation.
3. Any live cell with more than three live neighbours dies, as if by overcrowding.
4. Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.

With just a simple initial configuration of cells, constructs like Gosper's Glider Gun can be made:

![Gosper's Glider Gun](/images/gospers_glider_gun.gif)

### Session One

For the first session, I worked with a pair to build up 'Cell' object using CoffeeScript and Jasmine. It went well, but about half way into the session, Corey Haines changed the requirements of the game to include zombie cells that come to life based on its own set of rules. We were asked about the impact of introducing these new requirements into our code. Now, we weren't hugely affected by the change due to not being very far into the implementation, but it was obvious to see how we could have been bitten by the change. 

We chose to deal with alive and dead cells through a boolean attribute on the Cell class. Had we fleshed out the game more, our logic and objects would be tightly bound to this concept and introducing a third state would require considerable time and effort.
