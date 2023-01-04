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


This puzzle has been around for years, it lets the user move a single number piece to the empty slot and the objective is to arrange all the pieces in order.

You can also find the same puzzle with letters instead of numbers, or even images, in any case, the solution is the same.


The solution will be created using C++.

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

Working on the solution, feel free to check the project repository: [Github Repository](https://github.com/RafaelVC89/puzzle1_numbers)
