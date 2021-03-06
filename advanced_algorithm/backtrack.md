# 回溯法

## 背景

回溯法（backtrack）常用于遍历列表所有子集，是 DFS 深度搜索一种，一般用于全排列，穷尽所有可能，遍历的过程实际上是一个决策树的遍历过程。时间复杂度一般 O(N!)，它不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高。

## 模板

```go
result = []
func backtrack(选择列表,路径):
    if 满足结束条件:
        result.add(路径)
        return
    for 选择 in 选择列表:
        做选择
        backtrack(选择列表,路径)
        撤销选择
```

核心就是从选择列表里做一个选择，然后一直递归往下搜索答案，如果遇到路径不通，就返回来撤销这次选择。

## 示例

### [subsets](https://leetcode-cn.com/problems/subsets/)

> 给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。

> 第一种是回溯法深度遍历
> 第二种是队列广度优先遍历


遍历过程

![image.png](https://img.fuiboom.com/img/backtrack.png)

```cpp
vector<vector<int>> subsets(vector<int>& nums) {
	if(nums.empty()) return {};
	//保存最终结果
	vector<vector<int>> res;
	vector<int> list;
	backtrack(nums, 0, list, res);
	return res;
}
// nums 给定的集合
// pos 下次添加到集合中的元素位置索引
// list 临时结果集合(每次需要复制保存)
// result 最终结果
void backtrack(vector<int>& nums, int idx, vector<int> list, vector<vector<int>>& res){
	// 把临时结果复制出来保存到最终结果
	res.push_back(list);
	// 选择、处理结果、再撤销选择
	for(int i = idx; i < nums.size(); i++){
		list.push_back(nums[i]);
		backtrack(nums, i + 1, list, res);
		list.pop_back();  //回溯
	}
}
```

```cpp
vector<vector<int>> subsets(vector<int>& nums) {
	int n = nums.size();
	vector<vector<int>> res;
	res.push_back({}); //先插入空集
	
	//广度遍历，插入数字
	for(int i = 0; i < n; i++){
		int num = nums[i];
		int k = res.size();
		for(int j = 0; j < k; j++){
			vector<int> tmp = res[j];
			tmp.push_back(num);
			res.push_back(tmp);
		}
	}

	return res;
}
```

### [subsets-ii](https://leetcode-cn.com/problems/subsets-ii/)

> 给定一个可能包含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。说明：解集不能包含重复的子集。

```cpp
vector<vector<int>> subsetsWithDup(vector<int>& nums) {
		if(nums.empty()) return {};
		sort(nums.begin(), nums.end());  //让重复元素紧挨着

		vector<vector<int>> res;
		res.push_back({});

		int last = res.size();
		for(int i = 0; i < nums.size(); i++){
				int n = res.size();
				int low = 0;
				if(i > 0 && nums[i] == nums[i-1]) low = last;

				for(int j = low; j < n; j++){
						vector<int> tmp = res[j];
						tmp.push_back(nums[i]);
						res.push_back(tmp);
				}
				last = n;
		}

		return res;
}
```

```cpp
vector<vector<int>> subsetsWithDup(vector<int>& nums) {
	if(nums.empty()) return {};
	sort(nums.begin(), nums.end());  //让重复元素紧挨着

	vector<vector<int>> res;
	vector<int> track;
	backtrack(nums, 0, track, res);
	return res;
}
// nums 给定的集合
// pos 下次添加到集合中的元素位置索引
// list 临时结果集合(每次需要复制保存)
// result 最终结果
void backtrack(vector<int>& nums, int idx, vector<int> track, vector<vector<int>>& res){
	res.push_back(track);
// 选择时需要剪枝、处理、撤销选择
	for(int i = idx; i < nums.size(); i++){
		if(i > idx && nums[i] == nums[i-1]) continue;  //剪枝去重
		track.push_back(nums[i]);
		backtrack(nums, i + 1, track, res);
		track.pop_back();
	}
}
```

### [permutations](https://leetcode-cn.com/problems/permutations/)

> 给定一个   没有重复   数字的序列，返回其所有可能的全排列。

思路：需要记录已经选择过的元素，满足条件的结果才进行返回

```cpp
vector<vector<int>> permute(vector<int>& nums) {
	int n = nums.size();
	vector<vector<int>> res;
	
	vector<int> track;
	vector<bool> visited(n, false);
	backtrack(nums, track, visited, res);

	return res;
}
// nums 输入集合
// visited 当前递归标记过的元素
// list 临时结果集(路径)
// result 最终结果
void backtrack(vector<int>& nums, vector<int> track, vector<bool>& visited, vector<vector<int>>& res){
	// 返回条件：临时结果和输入集合长度一致 才是全排列
	if(track.size() == nums.size()){
		res.push_back(track);
		return;
	}
	for(int i = 0; i < nums.size(); i++){
		if(visited[i]) continue;  //移除添加过的元素
		track.push_back(nums[i]);
		visited[i] = true;
		backtrack(nums, track, visited, res);
		visited[i] = false;
		track.pop_back();
	}
}
```


```cpp
vector<vector<int>> permute(vector<int>& nums) {
	int n = nums.size();
	vector<vector<int>> res;
	if(n <= 1){
		res.push_back(nums);
		return res;
	}
	queue<vector<int>> qe;
	qe.push({});
	for(int i = 0; i < nums.size(); i++){
		int m = qe.size();
		for(int k = 0; k < m; k++){
			vector<int> last = qe.front();
			qe.pop();
			for(int j = 0; j <= last.size(); j++){
				vector<int> tmp = last;
				tmp.insert(tmp.begin() + j, nums[i]);
				if(i < nums.size() - 1) qe.push(tmp);  //队列保存上一轮结果
				else res.push_back(tmp); //存到最终结果里
			}
		}
	}

	return res;
}

```

### [permutations-ii](https://leetcode-cn.com/problems/permutations-ii/)

> 给定一个可包含重复数字的序列，返回所有不重复的全排列。

```cpp
vector<vector<int>> permuteUnique(vector<int>& nums) {
	if(nums.size() < 2) return {nums};
	sort(nums.begin(), nums.end());
	int n = nums.size();

	vector<vector<int>> res;
	vector<int> track;
	vector<bool> visited(n, false);
	backtrack(nums, track, visited, res);
	return res;
}

void backtrack(vector<int>& nums, vector<int> track, vector<bool>& visited, vector<vector<int>>& res){
	if(track.size() == nums.size()){
		res.push_back(track);
		return;
	}
	for(int i = 0; i < nums.size(); i++){
		if(visited[i]) continue;
		// 上一个元素和当前相同，并且没有访问过就跳过
		if(i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue;
		visited[i] = true;
		track.push_back(nums[i]);
		backtrack(nums, track, visited, res);
		visited[i] = false;
		track.pop_back();

	}

}
```

### [ju-zhen-zhong-de-lu-jing-lcof](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)

> 请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。

```cpp
bool exist(vector<vector<char>>& board, string word) {
  for(int i = 0; i < board.size(); i++){
    for(int j = 0; j < board[0].size(); j++){
      if(bfs(board, i, j, word, 0)) return true;
    }
  }

  return false;
}

bool bfs(vector<vector<char>>& board, int i, int j, string word, int idx){
  if(idx >= word.length() || i < 0 || i >= board.size() || j < 0 || j >= board[0].size() || 
     board[i][j] != word[idx])
    return false;
  board[i][j] = '0';
  if(idx == word.length() - 1) return true;
  bool flag = bfs(board, i-1, j, word, idx+1) ||
    bfs(board, i+1, j, word, idx+1) ||
    bfs(board, i, j-1, word, idx+1) ||
    bfs(board, i, j+1, word, idx+1);
  board[i][j] = word[idx]; //别忘了回溯
  return flag;
}
```



## 练习

- [ ] [subsets](https://leetcode-cn.com/problems/subsets/)
- [ ] [subsets-ii](https://leetcode-cn.com/problems/subsets-ii/)
- [ ] [permutations](https://leetcode-cn.com/problems/permutations/)
- [ ] [permutations-ii](https://leetcode-cn.com/problems/permutations-ii/)

挑战题目

- [ ] [combination-sum](https://leetcode-cn.com/problems/combination-sum/)
- [ ] [letter-combinations-of-a-phone-number](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)
- [ ] [palindrome-partitioning](https://leetcode-cn.com/problems/palindrome-partitioning/)
- [ ] [restore-ip-addresses](https://leetcode-cn.com/problems/restore-ip-addresses/)
- [ ] [permutations](https://leetcode-cn.com/problems/permutations/)
