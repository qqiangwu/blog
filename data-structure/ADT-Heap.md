# Heap or PriorityQueue
The PriorityQueue is one of the most important ADT. I will simply regard Heap as an alias to PriorityQueue. 

*Note: we I talk about PriorityQueues, I mean a maximum PriorityQueue for simplicity*

*Note: I will use Heap and PriorityQueue interchangablely*

For a PriorityQueue, we only care about its maximum element, and we would like the `query` and `extract` operation to be fast as possible as we can.

# Implementation
### BinaryHeap
A BinaryHeap is a nearly full complete binary tree. It has two representations: array representation and binary tree representation. Since array representation is more concise and almost leads to higher constant factor, we prefer arrays rather than a more straightforward tree representation. I will not get to its detail implementation and only illustrate the most important things.

#### Properties
The key properties of a BinaryHeap are:
  
1. The top element is the maximum of the heap. It is large than both its children, if exists.
2. A BinaryHeap is recursively defined, which means that the child node is also a top element with respect to its descendants.

#### Property Maintainance
These great properties make a BinaryHeap very fast to perform `query` and `extract` operations. But when we alter a BinaryHeap via operations such as `extract` or `delete-max`, this property invariant might be violated. Therefore, Fix operations are required.

We define two fix primitives which are both `O(lgN)`:  

1. `sift-up`: If a node and its descendants have the heap properties but the ascendants of the node don't, `sift-up` the the node.
2. `sift-down`: If a node and its ascendants have the heap properties but the descendants of the node don't, `sift-down` the the node.

##### Code demo for `sift-up`
```C++
void sift_up(HeapNode n)
{
    for (auto parent = n->parent; 
        !(n->value < parent->value); // heap property violation
         n = parent, parent = parent->parent) {
        std::swap(n->value, parent->value);
    }
}
```

##### Code demo for `sift-down`
```C++
void sift_down(HeapNode n)
{
    while (n->left) {
        auto c = n->right? 
            (n->left->value < n->right->value? n->left: n->right):
            n->left;
        if (!(n->value < c->value)) {
            std::swap(n->value, c->value);
            n = c;
        }
        else {
            break;
        }
    }
}
```

Especially note here how we pick the child to compare.

All other BinaryHeap operations can be implemented on top of these two primitives. For example:
```
MakeHeap(A[1..n]):
    for i = n / 2 downto 1:
        sift-down(A[1..n], i)
        
ExtractMax(A[1..n]):
    swap A[1] and A[n]
    sift-down(A[1..n-1], 1)
    return A[n]
    
Insert(A[1..n], x):
    A.append(x)
    sift-up(A[1..n+1], n+1)
```