# Peek Finding
Though peek finding is a very simple problem, it's worth thinking. I will briefly discuss it.

# One Dimension
For the 1D problem, the straightforward solution is linear search with time complexity `O(N)` which is not good enough. An intuitive idea is to use binary search with time complexity `O(logN)`, let's try it.

### Solution
1. Check the middle element x of the range
2. If x is a peak, simply return x
3. If x is not a peak, let a, b be its left and right neighbor respectively:
    + if x <= a, apply this algorithm to [1, n / 2 - 1] recursively
    + if x <= b, apply this algorithm to [n / 2 - 1, n] recursively
    
Clearly the time complexity is `O(logN)`. Now we have to prove its correctness. 

### Proof
When we examine the middle element, say `x`, of the range. Let its left and right neighbor be `a` and `b`.

1. If `a <= x` and `b <= x` are both true, we can affirmatively confirm that `x` is a peak.
2. If either `a > x` or `b > x` is true, we prove as follows. For simplicity, assuming `a > x`, we need to prove that there is a peak in [1 .. n / 2 - 1]. We view `A[x]` as a function mapping an index `x` to a value `A[x]`. If the function is monotonic, the endpoints are both peaks. Otherwise, there must be two indice with the same value. By Role Theorem, we obtain that there must be a peak.

# Two Dimension
I directly present the solution, and will discuss its correctness later.

1. If we only have one column, return its global maximum.
2. Else pick the middle column j, find its global maximum x at row i.
3. If x is a peak of row i, return x
4. Let a, b, be x's left and right neighbor respectively.
5. If x <= a, solve this problem recursively on columns [1, j - 1]
6. If x <= b, solve this problem recursively on columns [j + 1, n]

It's time complexity is `O(NlogN)`. 

Apparently 1-D is a specialization of 2-D. If 2-D algorithm is true, then 1-D version must be also true.

### Proof
I will prove the following two statements:

1. The algorithm will halt which means that It will never recurse on an empty subproblem. This is obvious.
2. If the algorithm returns a location, it must be a peak in the original question.
    The key idea is that we solve this problem by reducing the problem into 1-D version. Assume that we ends at column c1 and we're about to return a location (r1, c1). Its adjacent column is (r1, c2). According to the algorithm, `(r1, c1) >= (r2, c1) > (r2, c2) >= (r1, c2)` where (r2, c2) is the maximum in column c2, or we will recurse on the subproblem containing column c2.

### Question
Why must we find the global maximum instead of a peak?

According to out proof, it's quite straightforward.

### A linear time solution