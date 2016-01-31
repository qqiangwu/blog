# P, EXP, R
1. Most problems are unsolvable

# NP
```
NP = {Decision problems solvable in polynomial time via a "lucky" algorithm}
   = {Decision problems with solutions that can be "checked" in polynomial time}
```

P <(?) NP <(?) EXP < R

# Hardness & completeness
1. NP-hard: as hard as every problem in NP (> NP)
2. NP-complete: in both NP and NP-hard
3. Exp-hard & Exp-complete

# Reductions
1. The most common algorithm design technique
2. NP-complete examples:
    + Knapsack (pseudopoly, not poly)
    + 3\-Partition: given n integers, can you divide them into triples of equal sum?
    + Traveling Salesman Problem: shortest path that visits all vertices of a given graph
    + longest common subsequence of k strings
    + SAT: given a Boolean formula (and, or, not), is it ever true
    + 3\-coloring a given graph