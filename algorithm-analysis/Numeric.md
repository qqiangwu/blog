# Newton's Method
In order to implements Newton's method, we need high-precision computation.

If we want d-digit precision when calculate the square root of 2. We need `floor(10^d * sqrt(2)) == floor(sqrt(2*10^2d)`. We can just use Newton's method to find out square root of 2 times 10 raise 2d to eliminate fraction part and achieve all things in terms of integers.

###  Error Analysis
e(n+1) = e(n)^2 / 2(1+e(n)) = e(n)^2 since e(n) -> 0 so 1 + e(n) -> 1

Quadratic convergence, as # correct digits doubles each step.

# High-precision multiplication
1. Straightforward idea: recursion. => O(n^2)
2. Karatsuba’s Method: T(n) = 3T(n/2) => O(n^log3) = O(n^1.58...)
    + z0 = x0 * y0
    + z2 = x1 * y1
    + z1 = (x0 + x1)(y0 + y1) - z0 - z2 = x0y1 + x1y0

# High-precision division via multiplication
a/b => 1/b => floor(R/b) where R is a large value s.t. it is easy to divide by R(typically 2^k)

Use Newton's method to compute R/b:

f(x) = 1/x - b/R

x(i+1) = 2xi - bxi^2 / R <= (How to choose R?)

Complexity: equals the complexity of multiplication.

# Complexity of computing square roots
O(d^a)