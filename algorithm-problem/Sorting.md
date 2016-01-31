# Sort
Sort and search are the main operations for data structures. Now I'm going to talk about sort for linear data structures. I will only discuss practical sort algorithms and won't cover all of them. For example, I won't talk about Buble Sort.

# Sort without auxiliary data structures
Recursion is the major method to solve a problem, so I will use recursion to devise sort algorithms for liner data structures. In this section, I will focus on sort a array of integers, say, `A[1..n]`

### Induction
Assume that we've already had `A[1 .. n - 1]` sorted, and how do we deal with A[n] to make `A[1..n]` sorted? The answer is quite straightforward, just find a pos `i` in `A[1..n-1]` such that for all x in `A[1..i-1]` we have `x < A[n]`,  shift `A[i..n-1]` right by one, and insert `A[n]` to `A[i]`. Then we've done.

```C++
template <class RandomAccessIterator>
void insertion_sort(RandomAccessIterator first, RandomAccessIterator last)
{
    for (auto current = first; current != last; ++current) {
        auto i = std::upper_bound(first, current, *current);
        
        // bring_front is an alias for std::rotate which is somewhat misnamed.
        bring_front(i, current, current + 1);
    }
}
```

Obviously, the time complexity is `O(N^2)`, and it costs constant space.

### Divide and Conquer
As above, we have an unsorted array of integers, `A[1..n]`, we can break the array into two parts `a` and `b`, sort them recursively such that we have both `a` and `b` sorted, then we merge these two parts.

```C++
template <class InputIterator, class OutputIterator>
OutputIterator merge(
        InputIterator first1, InputIterator last1, 
        InputIterator first2, InputIterator last2,
        OutputIterator result)
{
    for (; first1 != last1 && first2 != last2; ++result) {
        if (*first1 < *first2) {
            *result = *first1;
            ++first1;
        }
        else {
            *result = *first2;
            ++first2;
        }
    }
    
    result = std::copy(first1, last1, result);
    result = std::copy(first2, last2, result);
    
    return result;
}
```

Obviously, `N = (last1 - first1) + (last2 - first2)` elements are copied, so its time complexity is `O(N)`. Put them all together, we have `T(n) = 2T(n/2) + O(n)`, the result is `O(NlgN)`, but `O(N)` space is consumed.

Actually, there do exists inplace merge sort which takes no extra space. But it is too complex, making the constant factor too large, to be practical.

# Sort with auxiliary data structures
1. Heap sort
2. Binary Search Tree sort

# Linear Time Sort
Linear time sort is super attractive. But sorting based on comparisons and swaps has the lower bound `O(nlgn)`, which means we cannot find out a linear time sort algorithm using comparisions.

Counting sort is a sorting algorithm built on top of RAM. It's stable. Both its time complexity and space requirement is `big-theta(n + k)`. K is the number of all possbile values.

Radix sort uses a stable sorting algorithm as its subroutine. Imagine each integer is in base b, and there are `d = log(b, k)` digits. 

Its time complexity is `O(td)`. We Counting sort is applied, `O(td) = O((n+b)log(b, k)) = O(nlog(n, k))`.

If `k = n ^ O(1)`, then we have `O(nlog(n,k)) = O(nO(1)) = O(cn)`.