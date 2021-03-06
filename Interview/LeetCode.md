## 几类常见算法
1. [单调栈](https://zhuanlan.zhihu.com/p/26465701)。
   1. 首先，选择合理的单调性，确定何时直接入栈，何时需要判断出栈。
   1. 接着，进行第一步中的出栈。出栈才计算弹出值对应的结果，结果一定依赖于比较值（即让自己出栈的值）。
   1. 注意：一定要考虑：进栈时要考虑相等的情况。
   
2. [并查集](https://blog.csdn.net/qq_41593380/article/details/81146850)。[常见题](https://leetcode-cn.com/problems/redundant-connection/)
   - 首先写出2个操作：并与查。
   - 接着进行初始化：如果两人有联系，于是进行关系合并。达到下图效果：
   - 最后
     - 情况一：根据现有的集合，进行结果获取。主流。
     - 情况二：在合并过程中也可以进行一个判断，找多余的关系。
   
3. 滑动窗口。滑动窗口最重要的是可以利用旧值，通过滑动计算新值，减少时间复杂度。[常见题](https://leetcode-cn.com/problems/max-consecutive-ones-iii/)
	- 首先，滑动窗口均在左边就位。
	- 接着，开始移动R，使得滑动窗口的区间满足给定的条件，然后，我们再移动L，直到滑动区间不再满足给定的条件。如此循环。在循环过程中记录最优值。
   
            while (right < length) {
                 sum 开始包含 array[right] => window.add(array[right])
                 while (sum不符合情况) {
                     sum中移除left => window.remove(array[left])
                     left++;
                 }
               // result的计算在while里面外面都可以
                 right++;
             }
   - 滑动窗口用来解决一些查找满足一定条件的连续区间的性质（长度等）问题。可以看做是一种双指针方法的特例，两个指针都起始于原点，并一前一后向终点前进。（双指针方法，其两个指针一始一终，并相向靠近）
   
4. BFS。BFS + Queue。记录level。模板。这里的visited可以用set存储。[常见题](https://leetcode-cn.com/problems/open-the-lock/)。与DFS不同，BFS适用于求level，即次数、圈数等层次关系。

	```
	while(q not empty){
		    int sz=q.size();
		    /*将当前队列中的所有节点向四周扩散*/ 即需要一个初始的种子，种入对列中
		    for(int i=0;i<sz;i++){
			Node cur=q.poll();
			/*划重点：这里判断是否到达终点*/
			if(cur is target)
			    return step;
			/*将cur的相邻节点加入队列*/
			for(Node x:cur.adj())
			    if(x not in visited){
				q.offer(x);
				visited.add(x);
			    }
		    }
		    /*划重点：更新步数在这里*/
		    step++;
	}
	```
	
5. DFS。DFS + Stack。[常见题](https://leetcode-cn.com/problems/reconstruct-itinerary/)。另外，对于BFS以及DFS的种子问题，如果是二位数组，可以考虑数据结构一次性加两个数字。简化操作。

6. [回溯](https://labuladong.gitbook.io/algo/)，DFS的一种。

    ```
    result = []
    	def backtrack(路径, 选择列表):
    		if 满足结束条件:
    			result.add(路径)
    			return
    		for 选择 in 选择列表:
    			做选择
    			backtrack(路径, 选择列表)
    			撤销选择
    ```

7. 前缀树。[常见](https://leetcode-cn.com/problems/short-encoding-of-words/)。[常见题](https://leetcode-cn.com/problems/implement-trie-prefix-tree/)
   - 遍历给定的字符串集合，插入到前缀树中。前缀树不一定非得是树，也许是map集合
   - 基于已有的，进行查。
   
8. 前缀和+Hash配套使用。给定一个数组A[1..n]，前缀和数组PrefixSum[1..n]定义为：PrefixSum[i] = A[0]+A[1]+...+A[i-1]；就是我平时使用的now[]数组。[常见题](https://leetcode-cn.com/problems/subarray-sum-equals-k/)
## 小技巧
1. 匹配题最好还是弄哈希映射,当匹配种类多时易于处理。[来源](https://leetcode-cn.com/problems/valid-parentheses/) 

