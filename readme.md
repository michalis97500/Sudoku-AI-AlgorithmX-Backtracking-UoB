## Introduction

This project was an assignemnt during my Masters course at The University of Bath on February 2022.
Course: MSc Computer Science Module: Artificial Intelligence

The aim of the assignment is to construct a sudoku solver for 9x9 grid sudokus. The choice of algorithm was completely up to the developer.
Marks were awarded for the software's ability to solve different levels of hardness of sudokus as well as the speed of producing solutions

## Background

There are a number of different ways to approach this problem, the most common being backtracking and constraint propagation. I chose to use Donald Knuth's Algorithm X to solve the problem. This is a very efficient algorithm that uses a technique called dancing links to solve the problem. The algorithm is described in detail in the paper [Dancing Links](http://www-cs-faculty.stanford.edu/~uno/papers/dancing-color.ps.gz) by Donald Knuth. To implement this algorithm, a sudoku is seen as an exact cover problem. 

### Exact cover problems

Before looking at the formal definition of an exact cover problem, let us attempt to understand the concept in plain english.

Imagine you are to to sort your cuttlery, a fork, a spoon and a knife each in a different spot in the drawer. Assume the 3 spots are names A, B and C
Each of the cuttleries must only occupy one of the spots - No spot shall have two items at any given time. There exist a number of possible solutions to this problem.
If we are to scale up the question to a set of 100 items and 100 spots then how are we to determine all possible configurations? This problem appears to be more complex as the number of
constraints has dramatically increased. The exact cover approach would go about solving this as such :
1) Create a list of all constraints that are to be satisfied - List 1
2) Create a list of potential ways to satisfy each of the constraints - List 2
3) Consturct a matrix, with list 1 being the column headers and list 2 populating the values of the matrix.
_AndyG (2016) Solving Sudoku, Revisited_

Using the simple problem we have constructed the following table :

![alt text](images/EmptyTable.png)

Since we cannot place all items at the same time, we require a constraint that states whether an item is placed or not. For all of the possible combinations of our simple problem, the matrix would look like this :\
![alt text](images/FullTable.png)

The exact cover solution is to find a subset of the rows that satisfies all the constraints. That is, choose a set of rows that contain exactly one "Yes" in each column.\ In computers, the "Yes/No" is represented by "1" or "0". The rows that satisfy the constraints are called the "solutions" of the problem.\



Formally, an exact cover is defined as follows :
>Given a collection `S` of subsets of `X`, an exact cover is a subcollection `S*` of `S` such that each element in `X` is contained within exactly one subset in `S*`"\
It should satisfy the following conditions : 
>1) The Intersection of any two subsets in S* should be empty. That is, each element of X should be contained in at most one subset of S*
>2) The Union of all subsets in S* should be equal to X. That is, each element of X should be contained in at least one subset of S*\


_Kumar (2018, p1) Exact Cover Problem and Algorithm X_

In the context of sudoku, the collection `S` is the set of all possible rows, columns and boxes in a sudoku grid. The subsets of `X` are the possible values that can be placed in each cell of the grid. A sudoku is represented as a matrix with 729 rows and 324 columns. Each row represents a possible value for a cell in the grid. A representation of an exact cover matrix for a sudoku is shown [here](https://www.stolaf.edu/people/hansonr/sudoku/exactcovermatrix.htm).

The simplest way to solve an exact cover problem is to use a brute force approach. This is to try all possible combinations of rows in the matrix and check if they satisfy the constraints. This is a very inefficient approach as the number of possible combinations increases exponentially with the number of rows. The number of possible combinations is given by the product of the number of elements in each subset of `S`. In the case of sudoku, the number of possible combinations is 2^729. This is a very large number and is not feasible to compute. A more elegant way to approach this is to use algorithm X, described below.

### Algorithm X

Algorithm X is a recursive algorithm that solves the exact cover problem. A great in-depth explanation of the algorith can be found [here](https://en.wikipedia.org/wiki/Knuth%27s_Algorithm_X). The goal is to select a subset of the rows such that the digit 1 appears in each column exactly once.

The algorith functions as:

>0) If the matrix is empty, the problem is solved, return the solution
>1) Select a column `c` deterministically.
>2) Choose a row `r` such that `A[r][c] = 1`.
>3) Include `r` in the partial solution.
>4) For each column `j` such that `A[r][j] = 1`, \
  &nbsp;&nbsp;&nbsp;&nbsp;for each row `i` such that `A[i][j] = 1`, \
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;delete row `i` from matrix. \
  &nbsp;&nbsp;&nbsp;&nbsp;delete column `j` from matrix.
>5) Repeat this algorithm recursively on the reduced matrix.

_Knuth's (2000, p.4) Algorithm X_

While any systematic rule to choose column `c` would will yield all solutions, Knuth suggested that the selection of column `c` would be made by selecting the column with the smallest number of 1s. This is a good heuristic as it reduces the number of recursive calls. This is because the number of 1s in a column is the number of rows that can be included in the solution. The fewer the number of rows, the fewer the number of recursive calls.

## Implementation

The implementation of the algorithm is done in Python. We require a way to use implement both the algorithm aswell as backtracking, therefore a class is created to represent a sudoku. The class has the following attributes :
>1) `grid` : A 9x9 matrix representing the sudoku grid
>2) `rows` : A list of all rows in the sudoku
>3) `columns` : A list of all columns in the sudoku
>4) `boxes` : A list of all boxes in the sudoku
>5) `units` : A list of all units in the sudoku

The class also has the following methods :
>1) `__init__` : The constructor for the class. Takes in a 9x9 matrix as input and creates the rows, columns, boxes and units.
>2) `get_row` : Returns the row at the specified index
>3) `get_column` : Returns the column at the specified index
>4) `get_box` : Returns the box at the specified index
>5) `get_unit` : Returns the unit at the specified index
>6) `get_units` : Returns a list of all units that contain the specified cell
>7) `pick_min` : Returns the column with the smallest number of 1s (condition of algorithm Step 1)
>8) `is_final` : Returns True if the sudoku has no more constraints to choose from, False otherwise
>9) `remove_constraints` : Application of algorithm Step 4
>10) `undo_remove` : Reverts the changes made by `remove_constraints`

There are also a handful of other methods used to aid in the implementation of the algorithm. In order to get to a solution we need to manipulate the object, by "moving" it generations forward (towards a solution) or backwords. Creating a copy of the object might seem like a good idea, however the deep copy function in Python is very slow. Therefore, we will use a different approach. We will create a few methods such that :
>1) `add_solution` : Adds a solution to the object (moves forward)
>2) `remove_solution` : Removes a solution from the object (moves back)



## References
http://www-cs-faculty.stanford.edu/~uno/papers/dancing-color.ps.gz

https://www.geeksforgeeks.org/exact-cover-problem-algorithm-x-set-1/

https://gieseanw.wordpress.com/2011/06/16/solving-sudoku-revisited/#:~:text=To%20solve%20the%20problem%20we,1%27%20in%20it%20exactly%20once.