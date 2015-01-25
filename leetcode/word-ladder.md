典型的BFS。

# 出现的问题-超时
当我按照正常的流程来写BFS时，当数据量大了会出现超时。经过修改后，算法才AC。分析一下原因。

在TLE的实现中，我在每次迭代开始时才检测当前word是否可以转换为end，而在探索到新word时，只是简单将将新路径放入queue。这会导致一个结果，可能有很多相同长的路径，而扩展这些路径时，又需要将整个剩下的dict再遍历一遍，这会在dic极大时带来很大的开销。因此，我在搜索到新word时，就检查新word是否可以转换为end，如果可以，直接返回，避免无谓的工作。

# 代码（未清理）
```C++
class Solution {
public:
    struct Path {
        int length;
        const std::string* word;
        Path(int l, const std::string* s)
            : length(l), word(s)
        {
        }
    };
    
    struct Path_cmp {
        bool operator()(Path x, Path y) const {
            return x.length > y.length;
        }
    };

    int ladderLength(string start, string end, unordered_set<string> &dict) {
        if (_transformable(start, end)) {
            return 2;
        }
        
        std::priority_queue<Path, std::vector<Path>, Path_cmp> q;
        
        q.push({1, &start});
        
        std::unordered_set<const std::string*> unexplored;
        for (const auto& s: dict) unexplored.insert(&s);
        
        while (!q.empty()) {
            const auto frontier = q.top();
            q.pop();
            
            for (auto iter = unexplored.cbegin(); iter != unexplored.cend(); ) {
                auto s = *iter;
                
                if (_transformable(*s, *frontier.word)) {
                    if (_transformable(*s, end)) {
                        return frontier.length + 2;
                    }
                    else {
                        q.push({frontier.length + 1, s});
                        iter = unexplored.erase(iter);
                    }
                }
                else {
                    ++iter;
                }
            }
        }
        
        return 0;
    }
    
    bool _transformable(const std::string& x, const std::string& y) {
        auto diff_occured = false;
        
        for (auto i = 0; i < x.size(); ++i) {
            if (x[i] != y[i]) {
                if (diff_occured) {
                    return false;
                }
                else {
                    diff_occured = true;
                }
            }
        }
        
        return true;
    }
};
```