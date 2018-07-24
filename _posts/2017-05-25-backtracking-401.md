---
title: 401. Binary Watch
date: 2017-05-25 22:26:58
tags: 
    - backtracking
categories:
    - leetcode
---

# 401. Binary Watch

## 使用backtracking的方法

穷举所有的可能，注意hour < 12, minute < 60
递归实现,退出条件是当num为0时，生成相应的时间。

```cpp
class Solution {
    vector<int> hour = {1,2,4,8};
    vector<int> minute = {1,2,4,8,16,32};
public:
    vector<string> readBinaryWatch(int num) {
        vector<string> res;
        helper(res, make_pair(0,0), num, 0);
        return res;
    }
    
private:
    void helper(vector<string>& res, pair<int, int> time, int num, int start_point) {
        if (num == 0) {
            string time_str = to_string(time.first) +  (time.second < 10 ?  ":0" : ":") + to_string(time.second);
            res.push_back(time_str);
        }
        for (int i = start_point; i < hour.size() + minute.size(); ++i) {
            if (i < hour.size()) {
                time.first += hour[i];
                if (time.first < 12) {
                    helper(res, time, num - 1, i + 1);
                }
                time.first -= hour[i];
            } else {
                time.second += minute[i - hour.size()];
                if (time.second < 60) {
                    helper(res, time, num - 1, i + 1);
                }
                time.second -= minute[i - hour.size()];
            }
        }
    }
};
```

