与问题的版本1类似。同样是BFS。

#出现的问题
### 结果不完全
由于此时需要的全部的结果，因此，我们需要系统化的搜索。在问题一的版本中，搜索完一个新的word之后，我直接将其设置为explored，但在此问题中，这个搜索到的word还可能会在其他路径的扩展中用到，因此，只能将其标识为frontier，而不能标识为explorerd。

BTW：

```
graph-search(problem):
    frontier = {Initial}
    explored = {}
    loop until frontier is empty:
        path = remove_choice(frontier)
        state = path.end
        explored.add(state)
        if (state is goal) return path
        for a in Actions(state):
            add [path + a -> Result(s, a) to frontier unless it is in frontier or explored]
```

### TLE
这个问题与1相同，按照1的模式来改。在找到第一个路径后，立即以它的长度为界，看与它长度相同的已有的path是否可以扩展到end，比它长的一律pass。可问题是，还是TLE。

### Runtime error
由于自作聪明，在代码中使用了大量的`std::move(path)`，结果导致数组为空而越界。