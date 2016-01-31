# Disjoint Set or Partition
Partition是数据组织的一种重要的方式，在一个Partition中，我们只关心下面三种操作：

+ 数据属于哪一个Partition
+ 两个数据是否属于同一个Partition
+ 合并两个Partition

在计算机中，我们通常称这种数据结构为Disjoint Set，它由多个Set组成，这些Set组成Partition，两两间互不相交，并且，它应当具有这样的接口:

```
interface DisjointSet {
    SetIdentifier makeSet(Element e);
    SetIdentifier findSet(Element e);
    SetIdentitier union(SetIdentifier x, SetIdentitifer y);
}
```

# 实现
给定上面的接口后，要实现是比较简单的。直观的想法是为维护多个Set。此时，`makeSet`的成本为`O(1)`，而对于`findSet`，最笨的方法遍历所有的Set，检查此元素是否在其中，此时成本为`O(m)`，其中，m为总Set数。当然，一个比较好的方法是直接用一个map将元素映射到它所有的Set，这样，查找操作的成本仅为`O(1)`。然后，比较复杂的就是`union`操作了，如果使用HashSet，其成本为线性。这是我们无法接受的。考虑到最简单合并操作成本最低的是链表与树，均只需要`O(1)`操作，我们需要从这两种表示入手。

算法导论中给出了基于链表的实现，在这种实现下，`makeSet`与`findSet`均有`O(1)`的效率，但是`union`操作却表现不佳。于是，最终我们用有根树来表示一个Set，树的根作为Set的标识，此时，`makeSet`与`union`操作仅仅需要`O(1)`，但这却降低`findSEt`的性能，由于`findSet`需要由待查找元素对应的Node回溯到root，其成本与树的高度相关，为些，我们对它进行改进。我们的目标是降低树的平均高度。

### 按Rank合并
显然，可以为此作出贡献的出现在`union`操作中，我们总是将低的树挂在高的树下。这是不是意味着我们需要对树的高度进行维护呢？为此，我们引入rank的概念，每个Node的rank是其高度上界，合并时，我们将rank小的合并到rank的大的树下。维护rank的成本是很低的。

### 路径压缩
我们希望每个Node的父Node直接是root，这样，就可以在`O(1)`时间内判定其所有的Set，为此，我们需要引入一个调整操作，称之为路径压缩。然而，这样的调整需要对Node进行回溯。考虑到`findSet`操作中需要进行回溯，我们可以直接将路径压缩合并到`findSet`操作中，以避免额外的开销。

## 效率分析
使用了按Rank合并以及路径压缩后，`findSet`的成本近乎线性，由于其准确的分析比较复杂，因此不加多说。

## 参考实现
```D
template DisjointSet(T) {
	struct Node {
		uint rank;	// estimated upper bound of tree height
		Node* parent;
	}
	
	struct Representative {
		private Node* handle;
		
		bool opCast(T : bool)() const {
			return handle != null;
		}
	}
	
	final class DisjointSet {
		private {
			Node*[T] nodeMap_;
			uint size_;
		}
		
		@property
		uint size() const {
			return size_;
		}
		
		@property
		bool empty() const {
			return size_ == 0;
		}
		
		Representative makeSet(in T val) 
		in {
			assert(!find(val));
		}
		body {
			auto node = new Node;
			
			nodeMap_[val] = node;
			++size_;
			
			return Representative(node);
		}
		
		Representative find(in T val) {
			if (auto p = val in nodeMap_) {
				auto node = *p;
				auto root = node;
				
				// We use two pass strategy.
				// In the first pass, we backtrack to find the root.
				while (root.parent != null) {
					root = root.parent;
				}
				
				// In the second pass, we reset parents of nodes in the backtracking path.
				while (node.parent != null) {
					auto tmp = node.parent;
					node.parent = root;
					node = tmp;
				}
				
				return Representative(root);
			}
			else {
				return Representative();
			}
		}
		
		Representative unionSet(in T x, in T y) {
			return unionSet(find(x), find(y));
		}
		
		Representative unionSet(Representative x, Representative y) {
			if (x && y) {
				auto nodeX = x.handle;
				auto nodeY = y.handle;
				
				--size_;
				
				if (nodeX.rank > nodeY.rank) {
					++nodeX.rank;
					nodeY.parent = nodeX;
					return Representative(nodeX);
				}
				else {
					++nodeY.rank;
					nodeX.parent = nodeY;
					return Representative(nodeY);
				}
			}
			
			return Representative();
		}
		
		bool isUnion(in T x, in T y) {
			return isUnion(find(x), find(y));
		}
		
		bool isUnion(in Representative x, in Representative y) const {
			return x == y;
		}
	}
}
```