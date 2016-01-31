# Trees
The tree is one of the most important data structures in computer science. In this post, I will focus on Binary Search Tree and eliminate all of the basic definitions of a tree.

# Binary Search Trees
### Why Binary Search Trees
Binary Search Trees is the best data structure to organize a collection of elements which are ordered. 

Assume that we organize ordered elements as an array. We can find the n-th element in `O(1)` time, this is great. But how about insertion? 

```
insert(A[1..n], x):
    expand A by 1
    i = upper_bound(A[1..n], x)
    shift A[i..n] right by one
    A[i] = x
```

Note that `upper_bound` is a variant of `binary_search`.

Insertion costs `O(N)` time. When insertion is frequent, the cost is unacceptable. Yes, binary search is good, but the insertion itself costs `O(N)` because of shifting. This hints that we need a noncontiguous data structure. Further more, I can exclude linked lists for they don't support binary search.

Finally, We get the trees!

In order to support binary search, let's contrain how binary search trees are formed.

1. Each node has exactly two children(both of them could be null)
2. If left child of x exists, all elements of subtree rooted at left(x) have the property `node->value < x->value`
3. If right child of x exists, all elements of subtree rooted at right(x) have the property `x->value < node->value`

If these properties hold, we can search an arbitrary value in `O(h)` time where `h` is the height of the tree.

### Insertion for Binary Search Trees
Now consider how to insert a new element `x`.

First, locate the position to insert. If it exists, simply ignore it. We find a node `n` such that:

1. `x < n->value && left(n) == null`: `left(n) = new Node(x)`
2. `n->value < x && right(n) == null`: `right(n) = new Node(x)`

This operation costs `O(h)`.

### Deletion for Binary Search Tress
Now consider how to delete x in a BST.

First, locate `x` in the BST. If it don't exist, simply ignore it. Otherwise, assume `x` is in node `n`. 
There are 3 cases:

1. `n` has no child: erase `n`, fix `n->parent`'s pointers.
2. `n` has one child: erase `n`, set `n->parent`'s pointer which points to `n` to `n->left` or `n->right`
3. `n` has two children: in this case, things become a little more complex. Thinking about what we need to do? We should erase `n` while maintaining BST's properties, which means we need find another node `m` and put it on `n`'s position such that we can safely erase `n`. How do we find `m`? `m` is either the maximum of left subtree of `n` or the minimum of right subtree of `n`. Transplant `m` to `n`. Then we've done. It's no doubt that finding maximum and minimum costs only `O(h)`. I will prove this in the next section.

This operation costs `O(h)`.

### Maximum or Minimum
Given the properties of a BST, finding maximum requires: finding the rightmost leaf of the BST, which means we need to go down to the right child of a node recursively untill `n->right` is null.

```
find-maximum(root):
    while root->right != null:
        root = root->right
    return root
```

Finding minimum is analogous.

The time complexity is obviously `O(h)`.

# Balanced BST
Since all the significant operations of a BST is `O(h)`, we must keep the tree short to lower down the time complexity. In the worst case, a BST might decay into a linked list of height `O(N)`. This is why we need a balanced BST which has a height `O(lgN)`. I will discuss two balanced BST in this post.

### AVL Trees
An AVL tree has the property: **The heights of a node's left and right subtrees differ at most 1 or -1**.

#### Proof
It's not easy to know the height given the number of nodes, but it's easy to know the number of nodes given the height of the tree. Let `n(h)` be the number of nodes of a tree with the height `h`. We have:

`n(h) > n(h-1) + n(h-2) + 1 > F(h) = phi ^ h / sqrt(5) => h < log(phi, n(h) * sqrt(5)) = 1.44lgN`

Yeah, we only care about the lower bound and upper bound, so iliminate some trivial items is possible.

#### Implementation
Since an AVL is just a fix of a BST, we can implement it on top of a BST. Whenever we perform an operation on a BST which is already an AVL, we need one more step fixing to maintain AVL property because a BST operation may violate it.

##### Rebalance Operation
There are `4 + 1` primitives in the AVL implementation. First we define the height of a `null` to be -1.

1. Single rotation
    + left rotation
    + right rotation
2. Double rotation
    + left-right rotation
    + right-left rotation
3. AVL-fix

Rotations are extensively used to reshape a tree. AVL-fix is used to repair a node that violate the AVL property. Since rotations are so fundamental, I won't talk about them and focus on AVL-fix.

```
AVL-fix(node):
    requires: node is not null
    requires: node.height != max(node.left, node.right) + 1
    
    oldh = node.height
    
    diff = height(node.left) - height(node.right)
    if diff == -2:  # right tree is higher
        subtree = node.right
        if height(subtree.right) > height(subtree.left):
            left_rotate(node)
        else:
            right_rotate(subtree)
            left_rotate(node)
    elif diff == 2:  # left tree is higher
        symmetric
    
    if oldn != height(node) and node.parent is not null: 
        AVL-fix(node.parent)
```

#### Insertion
When a node is successfully inserted, its parent's height might changed, simply AVL-fix it.

#### Deletion
When a node is successfully deleted, its parent's height might changed, simply AVL-fix it.

### Red-black Trees
Red-black trees are sort of obscure than avl trees, but they do work and gain more popularity than avl trees.

First I will introduce the properties of a rb-tree.

1. All nodes are either red or black.
2. The root is black.
3. All leaves are black.
4. Red parents cannot have red children.
5. The numbers of black nodes in all paths are the same.

A BST with these properties holding produces a balanced BST. I will prove this later. Now I will discuss insertion and deletion operations.

Note that we can view all null children of nodes as leaves and paint them black implicitly. Thus we can safely ignore property 3 and focus on the rest 4 properties. In practice, we replace all null by a Nil node whose color is black. Also, this will significantly ease bound checking.

#### Insertion
When a node is successfully inserted, we simply paint it red. If we paint it black, property 5 is broken. Because property 5 is a global property, fix it might be hard. Besides, a black node always needs fix. So we paint it red, and begin to fix it. The general idea is to move the color red up or to parent's sibling until property 4 is not violated. Actually, a picture will be of great help. You can draw it yourself.

```
try-insert-fix(node):
    if node.color == red and node.parent.color == red:
        insert-fix(node)

    if node is root:
        node.color = black
       
insert-fix(node):
    requires: node.color == red
    requires: node.parent.color == red
    
    p = node.parent
    if p.is_left():
        switch:
        case p.sibling.color == red:
            assert: p.parent.color == black
            
            p.color = black
            p.sibling.color = black
            p.parent.color = red
            
            try-insert-fix(p.parent)
            
        case p.sibling.color == black:
            if node.is_right():
                p.left_rotate()
                p, node = node, p
            
            assert: node.color == red
            assert: p.color == red
            assert: node.parent is p
            
            p.parent.right_rotate()
  
            p.color = black
            p.right.color = red
            
    else: # p.is_right()
        symmetric
```

Note that when the red is move up to the root, simply paint it black, then we've done. This means that a black can only be generated at the root.

#### Deletion
Compare with insertion, deletion might be more complex.

If a node is deleted and the color of the deleted node is red, no fix is required for no property is violated. Otherwise, call `try-delete-fix` on it and move an extra black up to its parent and `delete-fix` it until the extra black is either moved to the root or consumed by painting a red to black. 

```
try-delete-fix(node):
    if node is root:
        node.color = black
    else:
        delete-fix(node)
    
delete-fix(node):
    requires: node has an extra black
    requires: node is not root
    
    if node.is_left():
        if node.sibling.color == red:
            node.sibling.color = black
            node.parent.color = red
            node.parent.left_rotate()
        
        assert node.sibling.color == black
        
        switch:
        case node.sibling has two black children:
            # the black is moved to its parent
            node.sibling.color = red 
            try-delete-fix(node.parent)
        case node.sibling.left is red:
            # convert to the next case
            node.sibling.color = red
            node.sibling.left.color = red
            node.sibling.right_rotate()
            goto next case
        case node.sibling.right is red:
            node.sibling.color = node.parent.color
            node.parent.color = black
            node.sibling.right = black
            
            node.parent.left_rotate()
            # we've done
    else:
        symmetric
```