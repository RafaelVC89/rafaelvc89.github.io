---
layout: page
title: Puzzle 1
description: Multiple solutions to this numbers puzzle!
img: assets/img/project1_puzzle.png
importance: 1
category: Algorithms
---
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/project1_puzzle_small.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<br />
<br />

## Introduction
This puzzle has been around for years, it lets the user move a single number piece to the empty slot and the objective is to arrange all the pieces in order.

You can also find the same puzzle with letters instead of numbers, or even images, in any case, the solution is the same.


The solution will be created using C++.
<br />
<br />

## Initial Setup
Let's start by creating the puzzle and making random movements:

{% raw %}
```cpp
char PUZZLE[3][3] = 
    {1, 2, 3,
     4, 5, 6,
     7, 8, 0};

void makeRandomMovement(int &row, int &column)
{
    int movement = rand() % 4;
    switch (movement)
    {
        case 0://Left
            if(column > 0)
            {
                PUZZLE[row][column] = PUZZLE[row][column - 1];
                column--;
            }
            break;
        case 1://Up
            if(row > 0)
            {
                PUZZLE[row][column] = PUZZLE[row - 1][column];
                row--;
            }
            break;
        case 2://Right
            if(column < 2)
            {
                PUZZLE[row][column] = PUZZLE[row][column + 1];
                column++;
            }
            break;
        case 3://Down
            if(row < 2)
            {
                PUZZLE[row][column] = PUZZLE[row + 1][column];
                row++;
            }
            break;
        default:
            cout << "Error, default should not be reached." << endl;
            break;
    }
    PUZZLE[row][column] = 0;
}

int main()
{
    //Initialize random seed with a static value to get the same random number each time
    srand(4);
    int row = 2;
    int column = 2;
    for (int i = 0; i < 1000; ++i)
    {
        makeRandomMovement(row, column);
    }
    return 0;
}
```
{% endraw %}


To measure time performance, let's count time elapsed for the solution: 
{% raw %}
```cpp
int main()
{
    //Initialize random seed with a static value to get the same random number each time
    srand(4);
    int row = 2;
    int column = 2;
    for (int i = 0; i < 1000; ++i)
    {
        makeRandomMovement(row, column);
    }
    auto start = steady_clock::now();
    solve(PUZZLE);
    auto end = steady_clock::now();
    cout << "Solution took " << duration_cast<milliseconds>(end - start).count() << " milliseconds." << endl;
    return 0;
}
```
{% endraw %}
<br />
<br />

## Path Finding: Depth First
Let’s try to solve the problem using a path-finding solution with a depth-first approach, we will also use a memoization technique to prevent the same state to be visited twice thus preventing infinite recursion:

{% raw %}
```cpp
void solve(char puzzle[3][3])
{
    cout << "Solution starting." << endl;
    initializeGlobalVariables();
    clearVisited();
    findEmptyPiece(puzzle);
    depthFirst(puzzle, NONE);
    cout << "Solution found!" << endl;
}

//Depth first solution uses the previous movement information to reduce unnecessary movements
void depthFirst(char puzzle[3][3], direction_enum previousMovement)
{
    if (solved(puzzle))
    {
        SOLVED = true;
        return;
    }
    if (visited(puzzle))
    {
        return;
    }
    mark_visit(puzzle);
    //LEFT
    if (previousMovement != RIGHT && EMPTY_COL > 0)
    {
        makeMovement(puzzle, LEFT);
        depthFirst(puzzle, LEFT);
        if (SOLVED)
        {
            return;
        }
        makeMovement(puzzle, RIGHT);
    }
    //UP
    if (previousMovement != DOWN && EMPTY_ROW > 0)
    {
        makeMovement(puzzle, UP);
        depthFirst(puzzle, UP);
        if (SOLVED)
        {
            return;
        }
        makeMovement(puzzle, DOWN);
    }
    //RIGHT
    if (previousMovement != LEFT && EMPTY_COL < 2)
    {
        makeMovement(puzzle, RIGHT);
        depthFirst(puzzle, RIGHT);
        if (SOLVED)
        {
            return;
        }
        makeMovement(puzzle, LEFT);
    }
    //DOWN
    if (previousMovement != UP && EMPTY_ROW < 2)
    {
        makeMovement(puzzle, DOWN);
        depthFirst(puzzle, DOWN);
        if (SOLVED)
        {
            return;
        }
        makeMovement(puzzle, UP);
    }
}

void makeMovement(char puzzle[3][3], direction_enum direction)
{
    switch (direction)
    {
    case LEFT:
        puzzle[EMPTY_ROW][EMPTY_COL] = puzzle[EMPTY_ROW][EMPTY_COL - 1];
        EMPTY_COL--;
        break;
    case UP:
        puzzle[EMPTY_ROW][EMPTY_COL] = puzzle[EMPTY_ROW - 1][EMPTY_COL];
        EMPTY_ROW--;
        break;
    case RIGHT:
        puzzle[EMPTY_ROW][EMPTY_COL] = puzzle[EMPTY_ROW][EMPTY_COL + 1];
        EMPTY_COL++;
        break;
    case DOWN:
        puzzle[EMPTY_ROW][EMPTY_COL] = puzzle[EMPTY_ROW + 1][EMPTY_COL];
        EMPTY_ROW++;
        break;
    default:
        break;
    }
    puzzle[EMPTY_ROW][EMPTY_COL] = 0;
    printCurrentPuzzle(puzzle);
}
```
{% endraw %}
As expected, this direct approach does not reach a solution, as it generates a **Segmentation Fault Error**, this is caused due to the excessive recursive functions that the program calls as this puzzle has thousands of combinations.
For our next step, let’s try a Breath First approach:


Working on the solution, feel free to check the project repository: [Github Repository](https://github.com/RafaelVC89/puzzle1_numbers)
