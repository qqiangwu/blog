# Graph
1. Definitions
2. Representations
    + Adjacency Lists
    + Implicit Graphs
    + Object-oriented Variations

# Graph Search
1. Exploring a graph systematically to find a particular vertex or to learn about properties of the graph.
2. Applications:
    + Web Crawling
    + Facebook friend finder
3. Example: Pocket Cube, implicitly generates a graph

# BFS 
1. Level based search
2. Implementations:
    + Queue based
    + Level based
3. Visit each vertex only once + Check all adj[v] for v belonging to V. => O(V + E) 
4. Application: Simple shortest paths.

# DFS
1. Backtrack
2. Implementations
3. The same as BFS: O(V + E)
4. Application: 
    + Exploring a maze
    + Detecting a cycle
5. Topological Sort: 
    + Reverse of DFS finishing times
    + Scheduling jobs
    + BFS: The order between vertice in the level is unspecified, which causes an error.

# Edge Classification
1. Tree edges: formed by parent
2. Forward edges: to descendant
3. Back edges: to ancestor
4. Cross edges: to another subtree

Only tree edges and back edges are allowed in undirected graphs.

