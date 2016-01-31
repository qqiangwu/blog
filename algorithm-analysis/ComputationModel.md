# Computation Model
To design and analyze an algorithm, we must figure out what operations are allowed and the cost for each operation. A Computation Model exactly defines these.

There are some popular computation models we should know.
1. Random Access Model
2. Pointer Model
3. Comparison Model

The first two models are often mixed and play an important role in the design and analysis of an algorithm. Comparison Model are mostly used to prove the lower bound of an algorithm. In the next section, I will mainly talk about Comparison Model and prove the lower bounds of searching and sorting using the model.

# Comparision Model
### Definition
In a Comparision Model:

1. Only comparisions are considered.
2. Each comparision costs exactly `O(1)` time.
3. All other operations are not considered, which means that we can do whatever we want.

### Usage
Prove lower bound.

Since only comparisions count, we only care about the most essential operations and we can only obtain the lower bound of an algorithm.

### Decision Tree
We use binary decision trees to simulate the execution of an algorithm on a Comparison Model.

1. The interior node represents a comparision
2. Each leaf represents an output
3. The length of a path represents the time of an execution.

The most significant part of the analysis is to find out the number of leaves. Then we can deduce the minimum height of the tree, i.e. the lower bound of the problem.

### Example 
1. Lower Bound of Searching: There are `O(N)` leaves in the tree, so we have `h > O(lgN)`.
2. Lower Bound of Sorting: There are `N!` permutations of N elements, so we have `h > O(N!) = O(NlgN)`