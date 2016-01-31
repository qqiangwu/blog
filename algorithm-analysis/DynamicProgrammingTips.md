# Idea
1. Particularly for optimization problems
2. Clever brute force
3. Top-down(via memo) & bottom-up(topological sort of subproblem dependency DAG and fill a table)
4. Overlapped subproblems in recursion
5. Works only in DAGs
6. Solve a problem by check all possibilities in terms of subproblems. (The check is fairly cheap since all subproblem calls are constant)
7. Recursion + Memoization + guessing

# When to use
1. Overlapped subproblems in recursion
2. Optimized substructure

# Complexity
1. Bottom-up way: obvious
    + n table entries(subproblems)
    + fill each entry in O(m) time: subproblem complexity
    + O(nm)
2. Top-down
    + All subproblems will be solved only once!
    + When solve a subproblem, all subsubproblems have been solved, which means calls to them are constant
    + N subproblems
    + Each subproblem requires O(M): when there is subsubproblem call, it counts as a constant operation just like a '+'
    + O(MN)
    
# Examples
1. Shortest paths in DAGs: 
    + O(V + E)
    + Topo-sort and relax
    + Why to use DP: optimized subproblems
2. Text Justification
    + Define badness(i, j) for a line comprised of words[i: j]
    + Minimize the sum of badness for all lines.
    + Minimization -> Optimization Problem
    + Why to use DP: optimized subproblems
3. Perfect-Information Blackjack
    + Maximize benifit: BF[i]
    + Optimization
    + Why to use DP: optimized subproblems
4. Parenthesization
    + Minimize T[i:j] (the multiplications conducted in matrix[i:j])
    + Why to use DP: optimized subproblems
    + T[i:j] = max(T[i:k] + T[k:j] + t(k) for k in i .. j)
    + # of Problems: n^2
    + Complexity for each subproblem: O(n)
    + Total complexity: O(n^3)
5. Edit Distance
    + Application: Used for DNA comparison, diff, CVS/SVN/..., spellchecking (typos), plagiarism detection, etc.
    + Minimize Edit(i, j) for substring x[i:] and y[j:]
    + Why to use DP: optimized subproblems
    + # of Problems: mn
    + Complexity for each subproblem: O(1)
    + Total complexity: O(mn)
    + Guess (from x to y):
        - x[i] deleted
        - y[j] inserted
        - x[i] replaced by y[j]
    + Typically different operation costs differs
        + replacement: 0
        + insertion: 1
        + deletion: 1
6. Knapsack:
    + Minimize V(c, i) which means pick items from item[i:] with a space constraint c
    + Why to use DP: optimized subproblems
    + # of Problems: NC
    + Complexity for each subproblem: O(1)
7. Piano Fingering
    + Maximize C(i, j) which means the difficulty to play notes[i:] with finger j on notes[i] 
    + C(i, j) = max(d(j, i, k, i + 1) + C(i+1, k) for k in possible fingers)

# Reconstruct the solution to the optimized cost
+ DAG
+ Parent Pointers: save the decision in every guess step
    - To recover actual solution in addition to cost, store parent pointers (which guess used at each subproblem) & walk back
    - typically: remember argmin/argmax in addition to min/max