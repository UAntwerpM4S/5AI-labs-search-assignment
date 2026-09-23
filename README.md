# Lab 1: Search

<img src="https://inst.eecs.berkeley.edu/~cs188/archive/sp25/assets/projects/maze.png" height="300">

## Table of Contents

- [Introduction](#introduction)
- [Welcome to Pacman](#welcome-to-pacman)
- [PART I: Finding a Fixed Food Dot](#part-i-finding-a-fixed-food-dot)
    - [Exercise 1: Depth First Search](#exercise-1-depth-first-search)
    - [Exercise 2: Breadth First Search](#exercise-2-breadth-first-search)
    - [Exercise 3: Uniform-Cost Search](#exercise-3-uniform-cost-search)
    - [Exercise 4: A* search](#exercise-4-a-search)
- [PART II: Finding All the Corners](#part-ii-finding-all-the-corners)
    - [Exercise 5: Corners Problem](#exercise-5-corners-problem)
    - [Exercise 6: Corners Heuristic](#exercise-6-corners-heuristic)
- [Extra: Eating All The Dots](#extra-eating-all-the-dots)
    - [Exercise 7: Food Search Heuristic](#exercise-7-food-search-heuristic)
    - [Exercise 8: Suboptimal Search](#exercise-8-suboptimal-search)
- [Useful commands](#useful-commands)

## Introduction

In this project, your Pacman agent will find paths through his maze world, both to reach a particular location and to collect food efficiently. You will build general search algorithms and apply them to Pacman scenarios.

This repository includes the project code needed for the assignment. The code is split into several Python files, some of which you will need to read and understand to complete the assignment, while others contain support code that you can ignore unless you need to understand how the environment works.

### Files you’ll need to edit

- `search.py` — where all of your search algorithms will reside.
- `searchAgents.py` — where all of your search-based agents will reside.

### Files you might want to look at

- `pacman.py` — the main file that runs Pacman games.
    - This file describes a Pacman `GameState` type, which you use in this project.
- `game.py` — the logic behind how the Pacman world works.
    - This file describes several supporting types like `AgentState`, `Agent`, `Direction`, and `Grid`.
- `util.py` — useful data structures for implementing search algorithms.

### Supporting files you can ignore

- `graphicsDisplay.py` — graphics for Pacman
- `graphicsUtils.py` — support for Pacman graphics
- `textDisplay.py` — ASCII graphics for Pacman
- `ghostAgents.py` — agents to control ghosts
- `keyboardAgents.py` — keyboard interfaces to control Pacman
- `layout.py` — code for reading layout files and storing their contents

> **Important:** Please only change the parts of the files that are marked with "YOUR CODE HERE". Changing the name, scope, parameters of any of the provided functions or classes could introduce breaking changes in the code.

## Welcome to Pacman

After downloading the code, unzipping it, and changing to the directory, you should be able to play a game of Pacman by typing the following at the command line:

```bash
python pacman.py
```

Pacman lives in a shiny blue world of twisting corridors and tasty round treats. Navigating this world efficiently will be Pacman’s first step in mastering his domain.

The simplest agent in `searchAgents.py` is called `GoWestAgent`, which always goes West (a trivial reflex agent). This agent can occasionally win:

```bash
python pacman.py --layout testMaze --pacman GoWestAgent
```

But things get ugly for this agent when turning is required:

```bash
python pacman.py --layout tinyMaze --pacman GoWestAgent
```

If Pacman gets stuck, you can exit the game by typing `CTRL-c` into your terminal.

Soon, your agent will solve not only `tinyMaze`, but any maze you want.

Note that `pacman.py` supports a number of options that can each be expressed in a long way (for example, `--layout`) or a short way (for example, `-l`). You can see the list of all options and their default values via:

```bash
python pacman.py -h
```

A full list of the project’s common commands is included in the [Useful commands](#useful-commands) section below.

## PART I: Finding a Fixed Food Dot

In `searchAgents.py`, you’ll find a fully implemented `SearchAgent`, which plans out a path through Pacman’s world and then executes that path step-by-step. The search algorithms for formulating a plan are not implemented — that’s your job.

First, test that the `SearchAgent` is working correctly by running:

```bash
python pacman.py -l tinyMaze -p SearchAgent -a fn=tinyMazeSearch
```

The command above tells the `SearchAgent` to use `tinyMazeSearch` as its search algorithm, which is implemented in `search.py`. Pacman should navigate the maze successfully.

### Exercise 1: Depth First Search

Now it’s time to write full-fledged generic search functions to help Pacman plan routes! Pseudocode for the search algorithms you’ll write can be found in the lecture slides. Remember that a search node must contain not only a state but also the information necessary to reconstruct the path (plan) that gets to that state.

All of your search functions need to return a list of actions that will lead the agent from the start to the goal. These actions all have to be legal moves (valid directions, no moving through walls).

> **Hint:** Each algorithm is very similar. Algorithms for DFS, BFS, UCS, and A* differ only in the details of how the frontier is managed. So, concentrate on getting DFS right and the rest should be relatively straightforward. Indeed, one possible implementation requires only a single generic search method which is configured with an algorithm-specific queuing strategy.

> **Note:** Make sure to use the `Stack`, `Queue`, and `PriorityQueue` data structures provided to you in `util.py`! These data structure implementations have particular properties required for compatibility with the project.

Implement the depth-first search (DFS) algorithm in the `depthFirstSearch` function in `search.py`. To make your algorithm complete, write the graph search version of DFS, which avoids expanding any already visited states.

Your code should quickly find a solution for:

```bash
python pacman.py -l tinyMaze -p SearchAgent
python pacman.py -l mediumMaze -p SearchAgent
python pacman.py -l bigMaze -z .5 -p SearchAgent
```

The Pacman board will show an overlay of the states explored, and the order in which they were explored (brighter red means earlier exploration). Is the exploration order what you would have expected? Does Pacman actually go to all the explored squares on his way to the goal?

The solution found by your DFS algorithm for `mediumMaze` should have a length of 130, provided you consume successors in the order provided by `getSuccessors`. However, you might get 246 if you consume them in the reverse order. Is this a least-cost solution? If not, think about what depth-first search is doing wrong.

### Exercise 2: Breadth First Search

Implement the breadth-first search (BFS) algorithm in the `breadthFirstSearch` function in `search.py`. Again, write a graph search algorithm that avoids expanding any already visited states. Test your code the same way you did for depth-first search.

```bash
python pacman.py -l mediumMaze -p SearchAgent -a fn=bfs
python pacman.py -l bigMaze -p SearchAgent -a fn=bfs -z .5
```

Does BFS find a least-cost solution? If not, check your implementation.

If Pacman moves too slowly for you, try the option `--frameTime 0.01`.

### Exercise 3: Uniform-Cost Search

While BFS will find a fewest-actions path to the goal, we might want to find paths that are “best” in other senses. Consider `mediumDottedMaze` and `mediumScaryMaze`.

By changing the cost function, we can encourage Pacman to find different paths. For example, we can charge more for dangerous steps in ghost-ridden areas or less for steps in food-rich areas, and a rational Pacman agent should adjust its behavior in response.

Implement the uniform-cost graph search algorithm in the `uniformCostSearch` function in `search.py`. You should now observe successful behavior in all three of the following layouts, where the agents below are all UCS agents that differ only in the cost function they use:

```bash
python pacman.py -l mediumMaze -p SearchAgent -a fn=ucs
python pacman.py -l mediumDottedMaze -p StayEastSearchAgent
python pacman.py -l mediumScaryMaze -p StayWestSearchAgent
```

You should get very low and very high path costs for the `StayEastSearchAgent` and `StayWestSearchAgent`, respectively, due to their exponential cost functions (see `searchAgents.py` for details).

### Exercise 4: A* search

Implement A* search in the `aStarSearch` function in `search.py`. A* takes a heuristic function as an argument. Heuristics take two arguments: a state in the search problem (the main argument), and the problem itself (for reference information). The `nullHeuristic` heuristic function in `search.py` is a trivial example.

> **Hint:** In addition to keeping track of the visited state, you may also want to keep track of the best path cost to that state so far. Some nodes may need to be expanded more than once to find the optimal path.

You can test your A* implementation on the original problem of finding a path through a maze to a fixed position using the Manhattan distance heuristic, which is implemented already as `manhattanHeuristic` in `searchAgents.py`.

```bash
python pacman.py -l bigMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
```

You should see that A* finds the optimal solution slightly faster than uniform cost search. What happens on `openMaze` for the various search strategies?

## PART II: Finding All the Corners

The real power of A* will only be apparent with a more challenging search problem. Now it’s time to formulate a new problem and design a heuristic for it. In corner mazes, there are four dots, one in each corner. Our new search problem is to find the shortest path through the maze that touches all four corners (whether the maze actually has food there or not).

### Exercise 5: Corners Problem

Implement the `CornersProblem` search problem in `searchAgents.py`. You will need to choose a state representation that encodes all the information necessary to detect whether all four corners have been reached. Now, your search agent should solve:

```bash
python pacman.py -l tinyCorners -p SearchAgent -a fn=bfs,prob=CornersProblem
python pacman.py -l mediumCorners -p SearchAgent -a fn=bfs,prob=CornersProblem
```

To receive full credit, you need to define an abstract state representation that does not encode irrelevant information (like the position of ghosts, where extra food is, etc.). In particular, do not use a Pacman `GameState` as a search state. Your code will be very, very slow if you do (and also wrong).

An instance of the `CornersProblem` class represents an entire search problem, not a particular state. Particular states are returned by the functions you write, and your functions return a data structure of your choosing (for example: a tuple, set, or custom object) that represents a state.

Furthermore, while a program is running, remember that many states simultaneously exist, all on the queue of the search algorithm, and they should be independent of each other. In other words, you should not have only one state for the entire `CornersProblem` object; your class should be able to generate many different states to provide to the search algorithm.

Note that for some mazes like `tinyCorners`, the shortest path does not always go to the closest food first! For reference: The shortest path through `tinyCorners` takes 28 steps.

Our implementation of `breadthFirstSearch` expands just under 2000 search nodes on `mediumCorners`. However, heuristics (used with A* search) can reduce the amount of searching required.

### Exercise 6: Corners Heuristic

Implement a non-trivial, consistent heuristic for the `CornersProblem` in `cornersHeuristic`.

```bash
python pacman.py -l mediumCorners -p AStarCornersAgent -z 0.5
```

Note: `AStarCornersAgent` is a shortcut for:

```bash
python pacman.py -l mediumCorners -p SearchAgent -a fn=aStarSearch,prob=CornersProblem,heuristic=cornersHeuristic
```

#### Admissibility and consistency
Heuristics are functions that take search states and return numbers that estimate the cost to a nearest goal. More effective heuristics return values closer to the actual goal costs. To be admissible, the heuristic values must be lower bounds on the actual shortest path cost to the nearest goal and must be non-negative. To be consistent, it must additionally hold that if an action has cost c, then taking that action can only cause a drop in heuristic of at most c.

Remember that admissibility isn’t enough to guarantee correctness in graph search – you need the stronger condition of consistency. However, admissible heuristics are usually also consistent, especially if they are derived from problem relaxations. Therefore it is usually easiest to start out by brainstorming admissible heuristics. Once you have an admissible heuristic that works well, you can check whether it is indeed consistent, too. The only way to guarantee consistency is with a proof. However, inconsistency can often be detected by verifying that for each node you expand, its successor nodes are equal or higher in function value. Moreover, if UCS and A* ever return paths of different lengths, your heuristic is inconsistent. This stuff is tricky!

#### Non-trivial heuristics

The trivial heuristics are the ones that return zero everywhere (UCS) and the heuristic that computes the true completion cost. The former won’t save you any time, while the latter will be too expensive. You want a heuristic that reduces total compute time while staying admissible.

## Extra: Eating All The Dots

Now we’ll solve a hard search problem: eating all the Pacman food in as few steps as possible. For this, we’ll need a new search problem definition that formalizes the food-clearing problem: `FoodSearchProblem` in `searchAgents.py` (implemented for you). A solution is defined to be a path that collects all of the food in the Pacman world. For the present project, solutions do not take into account any ghosts or power pellets; solutions only depend on the placement of walls, regular food, and Pacman. If you have written your general search methods correctly, A* with a null heuristic (equivalent to uniform-cost search) should quickly find an optimal solution to `testSearch` with no code changes on your part (total cost of 7).

```bash
python pacman.py -l testSearch -p AStarFoodSearchAgent
```

Note: `AStarFoodSearchAgent` is a shortcut for:

```bash
python pacman.py -l testSearch -p SearchAgent -a fn=astar,prob=FoodSearchProblem,heuristic=foodHeuristic
```

You should find that UCS starts to slow down even for the seemingly simple `tinySearch`. As a reference, our implementation takes about 1 second to find a path of length 27 after expanding 5057 search nodes.

### Excercise 7: Food Search Heuristic

Fill in `foodHeuristic` in `searchAgents.py` with a heuristic for the `FoodSearchProblem`.

Try your A* search agent on the `trickySearch` board:

```bash
python pacman.py -l trickySearch -p AStarFoodSearchAgent
```

For reference, UCS finds the optimal solution in about 6.2 seconds, exploring over 16,000 nodes.

Make sure that your heuristic returns 0 at every goal state and never returns a negative value.

### Excercise 8: Suboptimal Search

Sometimes, even with A* and a good heuristic, finding the optimal path through all the dots is hard. In these cases, we’d still like to find a reasonably good path, quickly. In this section, you’ll write an agent that always greedily eats the closest dot. `ClosestDotSearchAgent` is implemented for you in `searchAgents.py`, but it’s missing a key function that finds a path to the closest dot.

Implement the function `findPathToClosestDot` in searchAgents.py. Our agent solves this maze (suboptimally!) in under a second with a path cost of 350:

```bash
python pacman.py -l bigSearch -p ClosestDotSearchAgent -z .5
```

> **Hint:** The quickest way to complete `findPathToClosestDot` is to fill in the `AnyFoodSearchProblem`, which is missing its goal test. Then, solve that problem with an appropriate search function. The solution should be very short!

Your `ClosestDotSearchAgent` won’t always find the shortest possible path through the maze. Make sure you understand why and try to come up with a small example where repeatedly going to the closest dot does not result in finding the shortest path for eating all the dots.


## Useful commands

```bash
python pacman.py
python pacman.py --layout testMaze --pacman GoWestAgent
python pacman.py --layout tinyMaze --pacman GoWestAgent
python pacman.py -h
python pacman.py -l tinyMaze -p SearchAgent -a fn=tinyMazeSearch
python pacman.py -l tinyMaze -p SearchAgent
python pacman.py -l mediumMaze -p SearchAgent
python pacman.py -l bigMaze -z .5 -p SearchAgent
python pacman.py -l mediumMaze -p SearchAgent -a fn=bfs
python pacman.py -l bigMaze -p SearchAgent -a fn=bfs -z .5
python pacman.py -l mediumMaze -p SearchAgent -a fn=ucs
python pacman.py -l mediumDottedMaze -p StayEastSearchAgent
python pacman.py -l mediumScaryMaze -p StayWestSearchAgent
python pacman.py -l bigMaze -z .5 -p SearchAgent -a fn=astar,heuristic=manhattanHeuristic
python pacman.py -l tinyCorners -p SearchAgent -a fn=bfs,prob=CornersProblem
python pacman.py -l mediumCorners -p SearchAgent -a fn=bfs,prob=CornersProblem
python pacman.py -l mediumCorners -p AStarCornersAgent -z 0.5
python pacman.py -l testSearch -p AStarFoodSearchAgent
python pacman.py -l trickySearch -p AStarFoodSearchAgent
python pacman.py -l bigSearch -p ClosestDotSearchAgent -z .5
```

---

The assignment and code in this project are adapted from the Pac-Man projects developed at [UC Berkeley](http://ai.berkeley.edu) for [CS 188](https://inst.eecs.berkeley.edu/~cs188/fa26/projects/).
