# Project 3: Ants Vs. SomeBees



> Adapted from cs61a of UC Berkeley.



![splash](img/splash.png)

<center>Bees are coming! Create a better soldier with inherit-ants.</center>



## Introduction

In this project, you will create a tower defense game called *Ants Vs. SomeBees*. As the ant queen, you populate your colony with the bravest ants you can muster. Your ants must protect their queen from the evil bees that invade your territory. Irritate the bees enough by throwing leaves at them, and they will be vanquished. Fail to pester the airborne intruders adequately, and your queen will succumb to the bees' wrath. This game is inspired by PopCap Games' *Plants Vs. Zombies*.

This project combines functional and object-oriented programming paradigms, focusing on the material from [Chapter 2.5](http://composingprograms.com/pages/25-object-oriented-programming.html) of Composing Programs. The project also involves understanding, extending, and testing a large program.



## Final Product





## Download starter files

To get started, download all of the project code.

```shell
git clone https://github.com/JacyCui/sicp-proj03.git
cd sicp-proj03
unzip ants.zip
```

Below is a list of all the files you will see in the archive `ants.zip`. However, you only have to make changes to `ants.py`.

- `ants.py`: The game logic of Ants Vs. SomeBees
- `ants_gui.py`: The original GUI for Ants Vs. SomeBees
- `gui.py:` A new GUI for Ants Vs. SomeBees. Note that this doesn't work / is very buggy, but you can see the cute ants in motion here :)
- `graphics.py`: Utilities for displaying simple two-dimensional animations
- `utils.py`: Some functions to facilitate the game interface
- `ucb.py`: Utility functions for CS 61A
- `state.py`: Abstraction for gamestate for gui.py
- `assets`: A directory of images and files used by `gui.py`
- `img`: A directory of images used by `ants_gui.py`
- `ok`: The autograder
- `proj3.ok`: The `ok` configuration file
- `tests`: A directory of tests used by `ok`



## Logistics

You will turn in the following files:

- `ants.py`

You do not need to modify or turn in any other files to complete the project.

For the functions that we ask you to complete, there may be some initial code that we provide. If you would rather not use that code, feel free to delete it and start from scratch. You may also add new function definitions as you see fit.

However, please do **not** modify any other functions. Doing so may result in your code failing our autograder tests. Also, please do not change any function signatures (names, argument order, or number of arguments).

Throughout this project, you should be testing the correctness of your code. It is good practice to test often, so that it is easy to isolate any problems. However, you should not be testing *too* often, to allow yourself time to think through problems.

We have provided an **autograder** called `ok` to help you with testing your code and tracking your progress.

The primary purpose of `ok` is to test your implementations.

As you are not a student of UC Berkeley, You should always run `ok` like this:

```shell
python3 ok --local
```

If you want to test your code interactively, you can run

```shell
 python3 ok -q [question number] -i --local
```

with the appropriate question number (e.g. `01`) inserted. This will run the tests for that question until the first one you failed, then give you a chance to test the functions you wrote interactively.

You can also use the debug printing feature in OK by writing

```python
 print("DEBUG:", x)
```

which will produce an output in your terminal without causing OK tests to fail with extra output.



## The Game

A game of Ants Vs. SomeBees consists of a series of turns. In each turn, new bees may enter the ant colony. Then, new ants are placed to defend their colony. Finally, all insects (ants, then bees) take individual actions. Bees either try to move toward the end of the tunnel or sting ants in their way. Ants perform a different action depending on their type, such as collecting more food, or throwing leaves at the bees. The game ends either when a bee reaches the end of a tunnel (you lose), or the entire bee fleet has been vanquished (you win).

![gui_explanation](img/gui_explanation.png)

### Core concepts

**The Colony**. This is where the game takes place. The colony consists of several *places* that are chained together to form a tunnel where bees can travel through. The colony has some quantity of food that can be expended to deploy ant troops.

**Places**. A place links to another place to form a tunnel. The player can place a single ant into each place. However, there can be many bees in a single place.

**The Hive**. This is the place where bees originate. Bees exit the beehive to enter the ant colony.

**Ants**. Ants are the usable troops in the game that the player places into the colony. Each type of ant takes a different action and requires a different amount of food to place. The two most basic ant types are the `HarvesterAnt`, which adds one food to the colony during each turn, and the `ThrowerAnt`, which throws a leaf at a bee each turn. You will be implementing many more.

**Bees**. Bees are the antagonistic troops in the game that the player must defend the colony from. Each turn, a bee either advances to the next place in the tunnel if no ant is in its way, or it stings the ant in its way. Bees win when at least one bee reaches the end of a tunnel.

### Core classes

The concepts described above each have a corresponding class that encapsulates the logic for that concept. Here is a summary of the main classes involved in this game:

- `GameState`: Represents the colony and some state information about the game, including how much food is available, how much time has elapsed, where the `QueenAnt` resides, and all the `Place`s in the game.
- `Place`: Represents a single place that holds insects. At most one `Ant` can be in a single place, but there can be many `Bee`s in a single place. `Place` objects have an `exit` to the left and an `entrance` to the right which are also places. Bees travel through a tunnel by moving to a `Place`'s `exit`.
- `Hive`: Represents the place where `Bee`s start out (on the right of the tunnel).
- `AntHomeBase`: Represents the place `Ant`s are defending (on the left of the tunnel). If Bees get here, they win :(
- `Insect`: A superclass for `Ant` and `Bee`. All insects have an `armor` attribute, representing their remaining health, and a `place` attribute, representing the `Place` where they are currently located. Each turn, every active `Insect` in the game performs its `action`.
- `Ant`: Represents ants. Each `Ant` subclass has special attributes or a special `action` that distinguish it from other `Ant` types. For example, a `HarvesterAnt` gets food for the colony and a `ThrowerAnt` attacks `Bee`s. Each ant type also has a `food_cost` attribute that indicates how much it costs to deploy one unit of that type of ant.
- `Bee`: Represents bees. Each turn, a bee either moves to the `exit` of its current `Place` if no ant blocks its path, or stings an ant that blocks its path.

### Game Layout

Below is a visualization of a GameState. As you work through the unlocking tests and problems, we recommend drawing out similar diagrams to help your understanding.

![colony-drawing](img/colony-drawing.png)

### Playing the game

The game can be run in two modes: as a text-based game or using a graphical user interface (GUI). The game logic is the same in either case, but the GUI enforces a turn time limit that makes playing the game more exciting. The text-based interface is provided for debugging and development.

The files are separated according to these two modes. `ants.py` knows nothing of graphics or turn time limits.

To start a text-based game, run

```shell
python3 ants_text.py
```

To start a graphical game, run

```shell
python3 ants_gui.py
```

When you start the graphical version, a new browser window should appear. In the starter implementation, you have unlimited food and your ants can only throw leaves at bees in their current `Place`. Before you complete Problem 2, the GUI may crash since it doesn't have a full conception of what a Place is yet!

The game has several options that you will use throughout the project, which you can view with `python3 ants_text.py --help`.

```shell
usage: ants_text.py [-h] [-d DIFFICULTY] [-w] [--food FOOD]

Play Ants vs. SomeBees

optional arguments:
  -h, --help     show this help message and exit
  -d DIFFICULTY  sets difficulty of game (test/easy/medium/hard/extra-hard)
  -w, --water    loads a full layout with water
  --food FOOD    number of food to start with when testing
```



## Phase 1: Basic gameplay

In the first phase you will complete the implementation that will allow for basic gameplay with the two basic `Ant`s: the `HarvesterAnt` and the `ThrowerAnt`.

### Problem 0 (0 pt)

Answer the following questions after you have read the *entire* `ants.py` file.

To test your answers, run

```shell
python3 ok -q 00 -u --local
```

If you cannot answer these questions, read the file again, consult the core concepts/classes sections above, or make use of the comment section.

1. What is the significance of an insect's `armor` attribute? Does this value change? If so, how?
2. What are all of the attributes of the `Insect` class?
3. Is the `armor` attribute of the `Ant` class an instance attribute or class attribute? Why?
4. Is the `damage` attribute of an `Ant` subclass (such as ThrowerAnt) an instance attribute or class attribute? Why?
5. Which class do both `Ant` and `Bee` inherit from?
6. What do instances of `Ant` and instances of `Bee` have in common?
7. How many insects can be in a single `Place` at any given time (before Problem 9)?
8. What does a `Bee` do during its turn?
9. When does the game end?

Remember to run

```shell
python3 ok -q 00 -u --local
```















