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
For our next step, let’s try a Breadth First approach:
<br />
<br />

## Path Finding: Breadth First
Another path finding solution called Breadth First uses a Stack or a Queue to store the pending states instead of using recursion, the states are analyzed and new states are added to the Stack as they arise:

{% raw %}
```cpp
struct State
{
    char puzzle[3][3];
    direction_enum previousMovement;
    int emptyRow;
    int emptyCol;
};

class StatesStack
{
public:
    StatesStack():mStates(new State[400000]), mSize(0) {}
    ~StatesStack() { delete[] mStates; }
    bool empty() { return mSize == 0;} 
    void push(State state)
    {
        mStates[mSize] = state;
        mSize++;
    }
    State pop()
    {
        mSize--;
        return mStates[mSize];
    }
private:
    State* mStates;
    int mSize;
};

void solve(char puzzle[3][3])
{
    cout << "Solution starting." << endl;
    unordered_set<int> visitedSet;

    State newState;
    newState.previousMovement = NONE;
    copyPuzzle(newState.puzzle, puzzle);
    findEmptyPiece(newState);

    StatesStack statesStack;
    statesStack.push(newState);

    direction_enum previousMovement = NONE;
    bool solutionFound = false;
    int statesAnalyzed = 0;
    while(!solutionFound && !statesStack.empty())
    {
        statesAnalyzed++;
        State currentState = statesStack.pop();
        previousMovement = currentState.previousMovement;
        if (solved(currentState.puzzle))
        {
            solutionFound = true;
            copyPuzzle(puzzle, currentState.puzzle);
            continue;
        }
        
        //LEFT
        if (previousMovement != RIGHT && currentState.emptyCol > 0)
        {
            makeMovement(currentState, LEFT);
            if (!visited(visitedSet, currentState.puzzle))
            {
                statesStack.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, RIGHT);
        }
        //UP
        if (previousMovement != DOWN && currentState.emptyRow > 0)
        {
            makeMovement(currentState, UP);
            if (!visited(visitedSet, currentState.puzzle))
            {
                statesStack.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, DOWN);
        }
        //RIGHT
        if (previousMovement != LEFT && currentState.emptyCol < 2)
        {
            makeMovement(currentState, RIGHT);
            if (!visited(visitedSet, currentState.puzzle))
            {
                statesStack.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, LEFT);
        }
        //DOWN
        if (previousMovement != UP && currentState.emptyRow < 2)
        {
            makeMovement(currentState, DOWN);
            if (!visited(visitedSet, currentState.puzzle))
            {
                statesStack.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, UP);
        }
    }
    cout << "Solution found! " << statesAnalyzed << " states were analyzed." << endl << endl;
}
```
{% endraw %}

<br />
This implementation reaches a solution!:
{% raw %}
```cpp
Puzzle to solve: 
4 1 2
5 8 7
0 6 3

Solution starting.
Solution found! 80229 states were analyzed.

Puzzle after solution: 
1 2 3
4 5 6
7 8 0

Solution took 887 milliseconds.
```
{% endraw %}
The catch is that it uses a large amount of memory to store the pending states and it analyzed more than 80 thousand states to reach a solution. And it takes more than 800 milliseconds, this can be improved using A star search algorithm.

<br />
<br />
## Path Finding: A*
A* is the most optimized solution, as it sets a higher priority to the states that are closer to solving the puzzle.
This solution uses a priority queue (min-heap) structure to store the states that are still pending to be analyzed and it provides the one with higher priority each time you call the pop function:

{% raw %}
```cpp
void setStatePriority(State &state)
{
    state.priority = 0;
    if (state.puzzle[0][0] == 1) state.priority+=2;
    if (state.puzzle[0][1] == 2) state.priority++;
    if (state.puzzle[0][2] == 3) state.priority++;
    if (state.puzzle[1][0] == 4) state.priority++;
    if (state.puzzle[1][1] == 5) state.priority++;
    if (state.puzzle[1][2] == 6) state.priority++;
    if (state.puzzle[2][0] == 7) state.priority++;
    if (state.puzzle[2][1] == 8) state.priority++;
    if (state.puzzle[2][2] == 0) state.priority++;
    if (state.puzzle[0][0] == 1 && state.puzzle[0][1] == 2 && state.puzzle[0][2] == 3) state.priority+=6; // extra priority if the top numbers are already in the right position
}

class PriorityQueue
{
public:
    PriorityQueue():mStates(new State[400000]), mSize(0) {}
    ~PriorityQueue() { delete[] mStates; }
    bool empty() { return mSize == 0;} 
    void push(State state);
    State pop();

private:
    int getParentIndex(int childIndex) { return (childIndex - 1) / 2; }
    int getLeftChildIndex(int parentIndex) { return (parentIndex * 2) + 1; }
    int getRightChildIndex(int parentIndex) { return (parentIndex * 2) + 2; }
    void siftUp(int childIndex);
    void siftDown(int parentIndex);

    State* mStates;
    int mSize;
};

void PriorityQueue::push(State state)
{
    mStates[mSize] = state;
    mSize++;
    siftUp(mSize - 1);
}

State PriorityQueue::pop()
{
    State returnState = mStates[0];
    mSize--;
    mStates[0] = mStates[mSize];
    siftDown(0);
    return returnState;
}

void PriorityQueue::siftUp(int childIndex)
{
    if(childIndex == 0)
    {
        return;
    }
    int parentIndex = getParentIndex(childIndex);
    if(mStates[childIndex].priority > mStates[parentIndex].priority)
    {
        State temporalState = mStates[childIndex];
        mStates[childIndex] = mStates[parentIndex];
        mStates[parentIndex] = temporalState;
        siftUp(parentIndex);
    }
}
void PriorityQueue::siftDown(int parentIndex)
{
    int highestPriorityChildIndex = getLeftChildIndex(parentIndex);
    if (highestPriorityChildIndex < mSize)
    {
        int rightChildIndex = getRightChildIndex(parentIndex);
        if(rightChildIndex < mSize && mStates[rightChildIndex].priority > mStates[highestPriorityChildIndex].priority)
        {
            highestPriorityChildIndex = rightChildIndex;
        }
        if (mStates[highestPriorityChildIndex].priority > mStates[parentIndex].priority)
        {
            State temporalState = mStates[highestPriorityChildIndex];
            mStates[highestPriorityChildIndex] = mStates[parentIndex];
            mStates[parentIndex] = temporalState;
            siftDown(highestPriorityChildIndex);
        }
    }
}

void solve(char puzzle[3][3])
{
    cout << "Solution starting." << endl;
    unordered_set<int> visitedSet;

    State newState;
    newState.previousMovement = NONE;
    copyPuzzle(newState.puzzle, puzzle);
    findEmptyPiece(newState);
    setStatePriority(newState);

    PriorityQueue priorityQueue;
    priorityQueue.push(newState);

    direction_enum previousMovement = NONE;
    bool solutionFound = false;
    int statesAnalyzed = 0;
    while(!solutionFound && !priorityQueue.empty())
    {
        statesAnalyzed++;
        State currentState = priorityQueue.pop();
        
        previousMovement = currentState.previousMovement;
        if (solved(currentState.puzzle))
        {
            solutionFound = true;
            copyPuzzle(puzzle, currentState.puzzle);
            continue;
        }
        
        //LEFT
        if (previousMovement != RIGHT && currentState.emptyCol > 0)
        {
            makeMovement(currentState, LEFT);
            if (!visited(visitedSet, currentState.puzzle))
            {
                setStatePriority(currentState);
                priorityQueue.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, RIGHT);
        }
        //UP
        if (previousMovement != DOWN && currentState.emptyRow > 0)
        {
            makeMovement(currentState, UP);
            if (!visited(visitedSet, currentState.puzzle))
            {
                setStatePriority(currentState);
                priorityQueue.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, DOWN);
        }
        //RIGHT
        if (previousMovement != LEFT && currentState.emptyCol < 2)
        {
            makeMovement(currentState, RIGHT);
            if (!visited(visitedSet, currentState.puzzle))
            {
                setStatePriority(currentState);
                priorityQueue.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, LEFT);
        }
        //DOWN
        if (previousMovement != UP && currentState.emptyRow < 2)
        {
            makeMovement(currentState, DOWN);
            if (!visited(visitedSet, currentState.puzzle))
            {
                setStatePriority(currentState);
                priorityQueue.push(currentState);
                mark_visit(visitedSet, currentState.puzzle);
            }
            makeMovement(currentState, UP);
        }
    }
    cout << "Solution found! " << statesAnalyzed << " states were analyzed." << endl << endl;
}
```
{% endraw %}

<br />
As a result, the solution is found with less than 1% of the states analyzed compared to previous solutions:
{% raw %}
```cpp
Puzzle to solve: 
4 1 2
5 8 7
0 6 3

Solution starting.
Solution found! 470 states were analyzed.

Puzzle after solution:
1 2 3
4 5 6
7 8 0

Solution took 2 milliseconds.
```
{% endraw %}

<br />
Feel free to check the full code of all the implementations in the project repository: [Github Repository](https://github.com/RafaelVC89/puzzle1_numbers)
<br />
**Happy puzzling!**