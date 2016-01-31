# Introduction
1. Motivations
2. Weighted Graphs
3. Single Source Shortest Paths
4. Negative Weights/Cycles

# Attempts
BFS-based shortest path solution: okay and straightforward, but fails in graphs with negative cycles. Actually, it is exactly the Dijkstra algorithm. Then how to cope with graphs with negative cycles?

### General Solution
It's an iterative solution. 

We relax an edge once a time to compute a tighter upper bound to a vertex repeatedly until no RELAX operations can be performed. It don't apply to negative cycles.

If we choose edges improperly, the complexity is exponetial.

### Optimal substructure
This helps! It means if we want find shortest path from s to t, we just need to find one from s to v from which we can reach t. If we properly arrange edges, we can relax each edge only once!

# Topologial Sort
Topological-sort a graph and relax edges in order.

# Dijkstra
Start at the source s, relax its neighbors and mark them as frontier. Pick the nearest neighbor from the processed one and start to visit it.

```
Dijkstra(G, W, s)
    initialize(G, s)
    explored = {}
    unexplored = PriorityQueue(V)
    while explored is nonempty:
        v = extract-min(unexplored)
        explored.add(v)
        foreach u in Adj[v]:
            relax(v, u, w) // There is a implicit decrease-key op if an edge is relaxed.
``` 

By induction we can prove that the algorithm is effective.

### Complexity
1. V insertions
2. V extract-min
3. E decrease-key

So we have O(V + VlgV + E) = O(VlgV + E) when employing Fib heap.

# Bellman-ford
Dijkstra is nice but it doesn't work in negative cycles. Bellman-ford can at a higher complexcity.

We can easily prove that there are k vertices in the shortest path between s and u, we need to relax all edges at most k - 1 times to find the shortest path of u by induction. So we can find all shortest paths at V - 1 times. If there is a negative cycle, we can definitely find it.

```
Bellman-ford(G, W, s):
    initialize()
    
    repeat V - 1 times:
        foreach e in E:
            relax(e)
            
    foreach e in E:
        if can relax e:
            report a negative cycle
```

# Longest path problem
The shortest path problem for negative cycles is NP-hard.

In the longest path problem, we can negate all weights which may produce negative cycles and run bellman-ford, it may fails. The problem is also NP-hard in general. But if we can ensure that the transformed graph has no negative cycles, it works!

# Speeding up Dijkstra
### Single source, single target
```
Initialize()
Q = V
while Q is nonempty:
    u = EXTRACT-MIN(Q)
    if u is t: stop
    for each vertex v t Adj[u]
        RELAX(u, v, w)
```

### Others
1. Birectional Search
2. Goal-directed Search or A star

# Summary
<table border="1">
    <tr>
        <th> Algorithms </th>
        <th> Negative Weights </th>
        <th> Cycles </th>
        <th> Negative Cycles </th>
        <th> Time </th>
    </tr>
    <tr>
        <th> Dijkstra </th>
        <td> Yes </td>
        <td> Yes </td>
        <td> No </td>
        <td> VlgV + E </td>
    </tr>
    <tr>
        <th> Bellman-ford </th>
        <td> Yes </td>
        <td> Yes </td>
        <td> Yes </td>
        <td> VE </td>
    </tr>
    <tr>
        <th> Topological </th>
        <td> Yes </td>
        <td> No </td>
        <td> No </td>
        <td> V + E </td>
    </tr>
</table>