
## 3. Longest Substring Without Repeating Characters
最长无重复字母的子串
双指针, 记录每个字母最后出现的位置,每次遇到有重复的就结算一次,遇到上个子串的重复的也没关系, 因为start 指针不会回溯
```py
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        dic, res, start, = {}, 0, 0
        for i, ch in enumerate(s):
            if ch in dic:
            
                res = max(res, i-start) # update the res
                start = max(start, dic[ch]+1)  #  这句是防止回溯, 不用清除掉前面的dict, 会结算到无意义的子串,但是不会影响最终结果

            dic[ch] = i
        return max(res, len(s)-start)  # return should consider the last non-repeated substring
```


5. Longest Palindromic Substring
最长回文子串
就是从中间向两边扩散判断子串 
    516. Longest Palindromic Subsequence
    最长回文子序列
    ! 子序列的问题, 想到动态规划, i,j 表示下标

    dp[i][j] 表示从i 到j 包括j的子串的最大回文长度
    状态转移方程是这个:
    #dp[i][j] = 1
    #dp[i][j] = dp[i+1][j-1] +2 if s[i] == s[j]
    #dp[i][j] = max(dp[i+1][j] ,dp[i][j-1])

    从状态转移方程可以看出当前元素来自于左, 下和左下
    所以遍历顺序是从左下到右上
    而且左下这个三角形都是0, i>j 无意义
    所以从对角线开始往右上遍历,从下往上,从左到右
    填入状态转移方程
    1 2 3 4 
    1 2 3 4
    1 2 3 4
    1 2 3 4
    dp[i][j] = s[i:j+1]
```py


class Solution:
def longestPalindromeSubseq(self, s: str) -> int:
    if len(s) ==0:
        return 0
    if len(s) ==1:
        return 1
    dp=[[0]*len(s) for _ in range(len(s))]
    #dp[i][j] = 1
    #dp[i][j] = dp[i+1][j-1] +2 if s[i] == s[j]
    #dp[i][j] = max(dp[i+1][j] ,dp[i][j-1])
    dp[0][0] = 1
    for i in range(1,len(s)):
        dp[i][i] = 1
            
    
    for i in range(len(s)-1,-1,-1):
        for j  in range(i+1,len(s)):
            if s[i] ==s[j]:
                dp[i][j] = dp[i+1][j-1] +2
            else:
                dp[i][j] = max(dp[i+1][j],dp[i][j-1])
        
        
        
    return dp[0][len(s)-1]     
    ```




11. Container With Most Water
数组城最多的水
双指针
方法一开始指针在最左边和最右边,得到一个面积,这个面积有可能是最大的,有可能不是,如果是最大的,就不用说了,我都记录下来了,如果不是,我就要往中间移动
我要怎么移动呢,我有两个选择, 左指针右移,有指针左移
但是一旦移动,底边必然要变短, 如果底边变短了, 你还用那个短边, 会更小,无法一步一步逼近最大值,只有大边才有潜力,小边当前状态已经是巅峰了,没有利用价值了
```py
class Solution:
    def area(height,i,j):
        if i>j:
            return 0
        return min(height[i],height[j])*(j-i)
    
    def maxArea(self, height: List[int]) -> int:
        i =0
        j = len(height)-1
        maxArea = 0
        while i < j:
            maxArea=max(maxArea, self.area(height,i,j))
            if height[i] < height[j]:
                i+=1
            else:
                j-=1
        return maxArea
            
            
```

17. Letter Combinations of a Phone Number
迪卡尓积,
class Solution:
    def letterCombinations(self, digits: str) -> List[str]:
        if len(digits)==0:
            return []
        d={'2':['a','b','c'],
        '3':['d','e','f'],
        '4':['g','h','i'],
        '5':['j','k','l'],
        '6':['m','n','o'],
        '7':['p','q','r','s'],
        '8':['t','u','v'],
        '9':['w','x','y','z']}
        l= []
        for i in digits:
            l.append(d[i])
        result =[]
        for element in itertools.product(*l): # 参数是数组的数组
            result.append(''.join(element))

        return result


22. Generate Parentheses
给一个整数n, 求所有的括号匹配方案
case:

Input: n = 3
Output: ["((()))","(()())","(())()","()(())","()()()"]

递归, 这就是个当前状态选择哪一个的造成的状态转移问题
需要考虑的情况 
左括号没满额, 当前放左括号右括号都行
左括号满额了, 就只能放左括号
一开始只能放左括号

```py
class Solution:
    def g(self,cur,left,right,n,result):
        if left == n and right == n:
            result.append(cur)
            return
        if left == n:
            self.g(cur+')',left,right+1,n,result)            
            return
        if left == right:
            self.g(cur+'(',left+1,right,n,result)
        else:
            self.g(cur+'(',left+1,right,n,result)
            self.g(cur+')',left,right+1,n,result)
            
        
        
    def generateParenthesis(self, n: int) -> List[str]:
        result =[]
        self.g("",0,0,n,result)
        return result
    
```

2116. Check if a Parentheses String Can Be Valid
括号匹配 给一个locked的字符串,说明哪些元素是不能更改的,其他元组则可以更改
那些能更改的元素,可以当做万能的字符使用,单独放到一个栈中,但是只有大于锁定元素的下标的才能匹配,小于的不能匹配
那些不能更改的被锁定的元素,放到另外一个栈中
遇到右括号结算一遍
最后处理两个栈里面剩下的元素, 锁定元素要找非锁定元素配对
锁定元素必须为空,非锁定元素必须为偶数

```py
class Solution:
    def canBeValid(self, s: str, locked: str) -> bool:
        if len(s) ==0:
            return True
        lockedStack=[]
        unlockedStack=[]
        
        for i in range(len(s)):
            if locked[i] =='0':# unlock
                unlockedStack.append(i) # just push , as a universal magic card
            else: # locked
                if s[i] =='(':
                    lockedStack.append(i)
                else: # ")" settlement 
                    if lockedStack:
                        lockedStack.pop()
                        continue
                    if unlockedStack: # elem in stack must before curent item
                        unlockedStack.pop()
                        continue         
                    return False
        # deal with remain stack
        while len(unlockedStack) > 0 and len(lockedStack) >0 and ( unlockedStack[-1] > lockedStack[-1]):
            unlockedStack.pop()
            lockedStack.pop()
            
        if len(lockedStack) !=0:
            return False
        if len(unlockedStack) & 1 == 1:
            return False
        return True
                
```



80. Remove Duplicates from Sorted Array II
删除调重复的元素,每个元素出现两个,不能另外开空间
Input: nums = [1,1,1,2,2,3]
Output: 5, nums = [1,1,2,2,3,_]

拿个指针一直往上增长就行
```py
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        if len(nums) <3:
            return len(nums)
        p=2
        for i in range(2,len(nums)):
            if nums[i] != nums[p-2]:
                nums[p] = nums[i]
                p+=1
        return p
```

28. Implement strStr()
找子串出现的位置
Input: haystack = "hello", needle = "ll"
Output: 2

一句就行:

```py
return haystack.find(needle)
```


459. Repeated Substring Pattern
判断一个子串是不是周期性的

s = "abab"
Output: true

This is based on the observation that a string is periodic if and only if it is equal to a nontrivial rotation of itself.
一个子串的非完全滚动 一定等于这个子串本身

比如abab 向右滚动两次  还是abab
然后两个abab的拼接 报所有的滚动子串 abab +abab = abababab

然后把第一个和最后一个元素干掉(不匹配完全滚动) 看能不能匹配得上s


```py
        i = (s+s).find(s,1,-1)
        
        if i>0:
            return True
        else:
            return False
```





29. Divide Two Integers
不用除法 除
就想办法减去大数, 但是这个大数必须是被除数的整数倍 不能用乘法,就只能用二进制了,减去多少倍 ,结果就加上多少
```py
class Solution:
    def divide(self, dividend: int, divisor: int) -> int:
        positive = (dividend<0) == (divisor<0)
        dividend= abs(dividend)
        divisor =abs(divisor)
        res  =0 
        while dividend >=divisor:
            d= divisor
            i=1
            while dividend >= d:
                dividend -= d
                res+=i
                i<<=1
                d<<=1
       return res  
        
```

33. Search in Rotated Sorted Array
81. Search in Rotated Sorted Array II
旋转数组中二分法找一个target

二分法用二分缩小规模,然后用线性搜索

```py
class Solution:
    #[1,2,3,4] [2,3,4,1] [4,1,2,3] 
    def binary_search(self,nums,i,j,target):
        if abs(i-j) <16: #最后直接搜索结果
            left = max(i-1,0)
            right = min(j+1,len(nums))
            return target in nums[left:right]
        

        mid  = (i+j)//2
        if nums[mid] ==target:
            return True
        
        if nums[i] < nums[j]: #如果第一个小于最后一个元素, 整体单调有序
            if nums[mid] > target:
                return self.binary_search(nums,i,mid-1,target)
            else:
                return self.binary_search(nums,mid+1,j,target)
        else:                   # 否则就区分哪边是有序的, 然后确认target 在哪边
            if nums[mid] > nums[i]: # left is in order
                if target > nums[mid]:
                    return self.binary_search(nums,mid+1,j,target)
                else:
                    return self.binary_search(nums,i, mid-1,target)
            if nums[mid] < nums[j]: # nums[mid] < nums[j]:
                if target < nums[mid]:
                    return self.binary_search(nums,i, mid-1,target)
                else:
                    return self.binary_search(nums,mid+1,j,target)
            
                
        
                
            
    def search(self, nums: List[int], target: int) -> int:
        nums= list(set(nums))
        if target >0:
            nums = [x for x in nums if x>0]
        else:
            nums = [x for x in nums if x<=0]
            
        return self.binary_search(nums,0,len(nums)-1,target)
```



34. Find First and Last Position of Element in Sorted Array
找到第一个和最后一个target 元素
同样是二分
二分法用二分缩小规模,然后用线性搜索
搜索两种结果, 即 第一个元素和最后一个元素

```py
class Solution:
    def s1(self,nums,target,i,j):
        if abs(j-i)<16:
            left = max(i-1,0)
            for i in range(left, j+1):
                if nums[i] == target:
                    return  i
            return -1
        mid =(i+j)//2
        if nums[mid] ==target:
            if mid ==0:
                return mid
            else:
                if nums[mid-1] ==nums[mid]:
                    return self.s1(nums,target,i,mid)
                
        if nums[mid]>target:
            return self.s1(nums,target,i,mid)
        else:
            return self.s1(nums,target,mid,j)
            
            
    def s2(self,nums,target,i,j):
        if abs(j-i)<16:
            right = min(j+1,len(nums)-1)
            for j in range(right,-1,-1):
                if nums[j] == target:
                    return  j
            return -1
        mid =(i+j)//2
        if nums[mid] ==target:
            if mid ==len(nums)-1:
                return mid
            else:
                if nums[mid+1] ==nums[mid]:
                    return self.s2(nums,target,mid,j)
                
        if nums[mid]>target:
            return self.s2(nums,target,i,mid)
        else:
            return self.s2(nums,target,mid,j)
             

        
    
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        return [self.s1(nums,target,0,len(nums)-1),self.s2(nums,target,0,len(nums)-1)]
```

36. Valid Sudoku
数独,检测是否是合法的
三个字典 一个是行 一个是列 一个是田字格
i//3 j//3 是田字格的坐标

i//3 * 3+  j//3  是田字格在字典中的坐标, 也可以用元组做key
```py
class Solution:
    def isValidSudoku(self, board: List[List[str]]) -> bool:
        dictc= [{} for i in range(9)]
        dictr= [{} for i in range(9)]
        dictb= [{} for i in range(9)]
        
        for i in range(9): # row
            for j in range(9): # column
                t= board[i][j]
                if t =='.':
                    continue
                a= i//3
                b= j//3
                if t in dictr[i] or t in dictc[j] or t in dictb[a*3+b]:
                    return False
                dictr[i][t]=1
                dictc[j][t] =1
                dictb[a*3+b][t] =1
                # 00 01 02 10 11 12 20 21 22
        return True
        
```


2133. Check if Every Row and Column Contains All Numbers
给一个二维数组, 看每个行列是否都有从1到n(类似数独)
用位运算
```py
class Solution:
    def checkValid(self, matrix: List[List[int]]) -> bool:
        template =  2**len(matrix)-1
        for l in matrix:
            t= 0
            for i in l:
                t= t | 1<<(i-1)
                print(t)
            if t != template:
                return False
            

        matrix = [list(x) for x in zip(*reversed(matrix))]
        for l in matrix:
            t= 0
            for i in l:
                t= t | 1<<(i-1)
            if t != template:
                return False
        return True 
```

50. Pow(x, n)
一半一般的递归 背过

```py
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n ==0:
            return 1
        if n ==1:
            return x
        if n <0:
            return 1/self.myPow(x,-n)
        half = self.myPow(x,n//2)
        if n &1 ==1:
            return half*half*x
        return half*half
```



69. Sqrt(x)
二分缩小规模 加线性搜索

```py
class Solution:
    def mySqrt(self, x: int) -> int:
        if x ==0:
            return 0
        if x<0:
            return -1
        
        lower = 0
        upper = x
        while True:
            if upper - lower <10:
                for i in range(max(lower-5,0) , upper+5):
                    if i*i>x:
                        return i-1
                    
            mid = (lower+upper)//2
            if mid * mid == x:
                return mid
            
            if mid * mid > x:
                upper = mid
            else:
                lower = mid
                
                
```

### 前置状态取舍的问题, 做第二遍没印象,重要
53. Maximum Subarray
最大子数组和:
如果前面的的状态对当前的状态的影响是负面的, 那么就不要前面的状态
```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        if len(nums) ==0:
            return 0
        if len(nums) ==1:
            return nums[0]
        
        res= nums[0]
        s = nums[0]
        for i in range(1,len(nums)):
            if s<0:
                s=nums[i]
            else:
                s+=nums[i]
            res= max(res,s)
            
        return res
```

```py
class Solution:
    def maxSubArray(self, nums: List[int]) -> int:
        res= nums[0]
        s = nums[0]
        for i in range(1,len(nums)):
            s = max(s+nums[i],nums[i])
            res= max(s,res)
            
        return res
```

152. Maximum Product Subarray
最大的子数组乘积, 前置状态取舍的问题
```py
class Solution:
    def maxProduct(self, nums: List[int]) -> int:
        maxtemp,mintemp,p = nums[0],nums[0],nums[0]
        res=nums[0]
        for i in range(1,len(nums)):
            l = [mintemp*nums[i],maxtemp*nums[i],nums[i]]
            mintemp,maxtemp = min(l),max(l)
            res = max(res,maxtemp)
        return res
```



121. Best Time to Buy and Sell Stock
上 面同类型题目 (能直接做出来 可以不看)
```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        minimum = 2**64
        res = 0
        for i in range(len(prices)):
            minimum = min(minimum,prices[i])
            res= max(res,prices[i]-minimum)

        return res
```


697. Degree of an Array
找出一个数组中出现次数最多且包含这个最多的所有元素的最短的子数组
easy题 没做出来 就老老实实的统计就完了
```py
class Solution:
    def findShortestSubArray(self, nums: List[int]) -> int:
        d= {}
        for i in range(len(nums)):
            if nums[i] not in d:
                d[nums[i]] =[]
            d[nums[i]].append(i)
            
        l = max(d.values(),key=lambda x:len(x)*100000-(x[-1]-x[0]+1)) # 排序 先按照元组个数排, 元素个数一样的按照index的差值小的排
        return l[-1]-l[0]+1
```

978. Longest Turbulent Subarray
# 先检测 奇数大的 最长的, 在检测 偶数大的最长的 
# 没做出来 还觉得很难 卧槽实际上这么简单

```py
class Solution:
    def maxTurbulenceSize(self, arr: List[int]) -> int:
        if len(arr) <=0:
            return 0
        # 先检测 奇数大的 最长的, 在检测 偶数大的最长的
        max1 = 1
        max2 = 1
        s1=s2=1
        for i in range(1, len(arr)):
            if i & 1 ==1:
                if arr[i] >arr[i-1]:
                    s1+=1
                else:
                    s1=1
            else:
                if arr[i] < arr[i-1]:
                    s1+=1
                else:
                    s1=1
            max1= max(max1,s1)
        
        for i in range(1, len(arr)):
            if i & 1 ==1:
                if arr[i]  < arr[i-1]:
                    s2+=1
                else:
                    s2=1
            else:
                if arr[i] > arr[i-1]:
                    s2+=1
                else:
                    s2=1
            max2= max(max2,s2)
        
        
        
        return max(max1,max2)
```

1749. Maximum Absolute Sum of Any Subarray
# 最大(子数组和的绝对值)
# 和最大字数组和 最大子数组乘积一样的问题
```py
class Solution:
    def maxAbsoluteSum(self, nums: List[int]) -> int:
        if len(nums) ==0:
            return 0
        if len(nums)==1:
            return abs(nums[0])
        
        max1 = nums[0]
        min1 = nums[0]
        res= nums[0]
        s1=  nums[0]
        for i in range(1,len(nums)):
            l = [nums[i],nums[i] + max1, nums[i]+min1]
            max1 = max(l)
            min1= min(l)
            res = max([abs(max1),abs(min1),res])
        return res
```


54. Spiral Matrix
旋转遍历矩阵
只折一次,折完在检测一次,如果还是超界, 就返回

```py
class Solution:
    def f(self,matrix, i,j,visited,result,direction):
        if (i,j) in visited:
            return
        visited[(i,j)]=1
        result.append(matrix[i][j])
        
        d=[[0,1],[1,0],[0,-1],[-1,0]]
        a= i+d[direction][0]
        b= j+d[direction][1]

        # 第一次检测, 眼看原先的方向前进,行不通就换一个方向
        if a<0 or a >= len(matrix) or b<0 or b>=len(matrix[0]) or (a,b) in visited:
            direction = (direction+1) %4
            
        d=[[0,1],[1,0],[0,-1],[-1,0]]
        a= i+d[direction][0]
        b= j+d[direction][1]
        # 第二次检测 还是行不通就over
        if a<0 or a >= len(matrix) or b<0 or b>=len(matrix[0]) or (a,b) in visited:
            return
      
        self.f(matrix,a,b,visited,result,direction)

        
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        result =[]
        visited={}
        self.f(matrix,0,0,visited,result,0)
        return result
```


55. Jump Game
Input: nums = [2,3,1,1,4]
Output: true

从第一个格子开始跳,看能不能跳到在最后
每个格子 看做nums[i] +i ,然后遍历得最大值
如果某个格子不能达到 , 即 如果之前的最大值 比当前的index小 就直接返回False
# 注 不用便利最后一个 不care
```py
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        if len(nums) ==1:
            return True
        maxi = 0
        for i in range(len(nums)-1):
            if maxi<i:
                return False
            
            maxi = max(maxi,nums[i]+i)
            if maxi >= len(nums)-1:
                return True

        return False
```

45. Jump Game II
同上题 一定能跳到 求步数
也能模拟每一步的跳, 但是不太好算, 不太好想边界, 就用dp
[2,3,1,1,4]
nums[i] = 3
dp[4] = 0
dp[1] = min([dp[i+1]..dp[i+nums[i]]]) +1


```py
class Solution:
    def jump(self, nums: List[int]) -> int:
        if len(nums)==1:
            return 0
        
        
        dp=[0]*len(nums)
        dp[-1] = 0
        for i in range(len(nums)-2,-1,-1):
            left = min(i+1,len(nums)-1)
            right = min([nums[i] +i + 1,len(nums)])
            if nums[i] ==0 or left == right:
                dp[i] = 2**64
            else: 
                dp[i] = min(dp[left:right])+1
        return dp[0]
            
```



1871. Jump Game VII
贪吃蛇, 搞一堆0 , 然后不断不断的把这些0 吃掉
```py
class Solution:
    def canReach(self, s: str, minJump: int, maxJump: int) -> bool:
        zeros = [i for i in range(len(s)) if s[i]=='0']
        queue = deque()
        queue.append(zeros.pop(0))
        while queue:
            i = queue.popleft()
            if i ==len(s)-1:
                return True
            while zeros and i+minJump > zeros[0]:
                zeros.pop(0)
            while zeros and (i+minJump) <= zeros[0] <=  min(i + maxJump, len(s) - 1):
                queue.append(zeros.pop(0))
        return False
```




63. Unique Paths II
矩阵 从左上角移动到右下角, 每次只能横走一个格子, 或者往下走一个格子
obstacleGrid 是格子中的障碍,有障碍不能走不能越过
Input: obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
Output: 2


就一个动态规划,不用滚动数组,麻烦,有障碍的格子的dp为0
第一行 第一列 只有有一个格子障碍, 后面的一律为0
dp[i][j] = dp[i-1][j] +dp[i][j-1]
如果上面的格子是障碍 就不加 ,同理左边的格子
```py
 def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        if len(obstacleGrid) ==1:
            return int(1  not in obstacleGrid[0])
        if obstacleGrid[-1][-1] ==1  :
            return 0
        #dp[i][j] = dp[i-1][j] +dp[i][j-1]
        #dp[0][x] = 1
        #dp[x][0] =1
        dp = [[0] * len(obstacleGrid[0])  for _ in range(len(obstacleGrid))]
        
        f = False
        for i in range(len(obstacleGrid[0])):
            if obstacleGrid[0][i] ==1:
                f= True
            if f:
                dp[0][i] = 0
            else:
                dp[0][i] = 1
        f = False
        for i in range(len(obstacleGrid)):
            if obstacleGrid[i][0] ==1:
                f= True 
            if f:
                dp[i][0] = 0
            else:
                dp[i][0] = 1
                
                
                
                
        print("==>>dp",dp)
        for i in range(1,len(obstacleGrid)):
            for j in range(1,len(obstacleGrid[i])):
                t= 0
                if obstacleGrid[i-1][j]!=1:
                    t+=dp[i-1][j]
                if obstacleGrid[i][j-1]!=1:
                    t+=dp[i][j-1]
                    
                dp[i][j] = t
                    
                    
            
        return dp[-1][-1]
```


64. Minimum Path Sum
格子 坐上到右下路径和 这个太简单了 ,每次都能一边过

2087. Minimum Cost Homecoming of a Robot in a Grid
行列移动都需要花费, 求src 到dst 的最小距离

https://leetcode.com/problems/minimum-cost-homecoming-of-a-robot-in-a-grid/
从矩阵的一个坐标移动到另一个坐标,可以上下左右移动, 可以绕弯路,可以绕圈
但是如果转圈,往上移动 必然会往下移动, 互相就抵消了
所以就直接是直接移动行坐标的花费, 和列坐标的话费 , you cant not save cost


```py
class Solution:
    def minCost(self, startPos: List[int], homePos: List[int], rowCosts: List[int], colCosts: List[int]) -> int:
        return sum(rowCosts[startPos[0]+1:homePos[0]+1]) + \
                sum(colCosts[startPos[1]+1:homePos[1]+1]) + \
                    sum(rowCosts[homePos[0]:startPos[0]]) + \
                        sum(colCosts[homePos[1]:startPos[1]])
```




57. Insert Interval
intervals 中插入一个interval , 使仍然有序 不能重叠 要合并

不能使用二分, 因为二分完了还要一直加到最后,还特么不如直接从头开始加
Input: intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]


```py
class Solution:
    def f(self,intervals,newInterval,i,j):
        if abs(j-i)<16:
            left = max(i-1, 0)
            prev = intervals[:left]
            tempList = intervals[left:]
            tempList.append(newInterval)
            tempList.sort(key=lambda x:x[0]) # timsort  is log(n) when array is basicly in order

            st =[tempList[0]]
            for i in range(1,len(tempList)):
                if tempList[i][0] > st[-1][1]:
                    st.append(tempList[i])
                else:
                    st[-1][1] = max(tempList[i][1],st[-1][1] )
            return prev+st
                
            
                    
                    
                
                
            

                
        
        mid = (i+j)//2
        if intervals[mid][0] == newInterval[0]:
            return self.f(intervals,newInterval,mid-1,mid+1)
        elif intervals[mid][0] > newInterval[0]:
            return self.f(intervals,newInterval,i,mid)
        else:
            return self.f(intervals,newInterval,mid,j)
            
                
    def insert(self, intervals: List[List[int]], newInterval: List[int]) -> List[List[int]]:
        return self.f(intervals,newInterval,0,len(intervals)-1)
```


495. Teemo Attacking
提莫攻击的毒素持续
timeSeries 表示提莫在第几秒攻击,每次攻击持续时间是 duration秒,求最后被攻击的人受多少秒毒素
比较简单 变成interval 的overlap的问题
```py
class Solution:
    def findPoisonedDuration(self, timeSeries: List[int], duration: int) -> int:
        if len(timeSeries)==1:
            return duration
        st =[]
        st.append([timeSeries[0],timeSeries[0]+duration-1])
        
        for i in range(1,len(timeSeries)):
            t = [timeSeries[i],timeSeries[i]+duration-1]
            if t[0] <= st[-1][1]:
                st[-1][1] = max(t[1],st[-1][1])
            else:
                st.append(t)
        s=0    
        while st:
            t = st.pop()
            s+=(t[1] -t[0]+1)
        return s
            
```


763. Partition Labels
把一个字符串分成几部分, 每个字符只在其中一个部分出现
也是interval合并的问题,求每个字符出现的第一个位置和最后位置

有可能不能一下想起来是interval的问题
```py
class Solution:
    def partitionLabels(self, s: str) -> List[int]:
        d={}
        for i in range(len(s)):
            if s[i] not in d:
                d[s[i]] =[i,i] # 这里是[i,i]  一开始填的还是[i,0]
            else:
                d[s[i]][1] = i
        if len(d) ==1:
            return [1]
        
        intervals=[]
        for i in range(len(s)):
            intervals.append(d[s[i]])
        intervals.sort(key= lambda x:x[0])
        st=[intervals[0]]
        for i in range(1,len(intervals)):
            if intervals[i][0] <= st[-1][1]:
                st[-1][1] =max(st[-1][1], intervals[i][1])
            else:
                st.append(intervals[i])
        res =[]
        while st:
            t= st.pop(0)
            res.append(t[1]-t[0]+1)
        return res
```



986. Interval List Intersections
两个interval 数组的交集
一直维护一个最大的结尾端,  每次从两个list中挑一个最小的 和最大端求交,存入结果

```py
class Solution:
    def intervalIntersection(self, firstList: List[List[int]], secondList: List[List[int]]) -> List[List[int]]:
        maxend ,res= -1,[]
        firstList.append([2**64,2**64]) # 加一个堵头
        secondList.append([2**64,2**64])
        
        #print(secondList)
        while firstList  and secondList:
            minInterval = []
            
            # pick the smaller one
            if firstList[0][0]<=secondList[0][0]:
                minInterval = firstList.pop(0)
            else:
                minInterval = secondList.pop(0)
                
            #print(maxend,minInterval)
                
            l = [maxend,minInterval[1]]
            if minInterval[0] <= maxend:
                res.append([minInterval[0],min(l)])
                
            maxend = max(l)
        return res
```


bisect.insort_left 左插入
bisect.insort_right 右插入
bisect.bisect_left  左插入 位置
bisect.bisect_right  右插入 位置


43. Multiply Strings
手动模拟乘法
```py
class Solution:
    def add(self,l1,l2):
        l1 = l1[::-1]
        l2=l2[::-1]
        c =0
        res= ""
        while l1 and l2:
            a = l1[0]
            l1=l1[1:]
            b = l2[0]
            l2=l2[1:]
            t=(int(a)+int(b)+c)
            res = res+str(t %10)
            c= t//10
        l = l1 or l2
        while l:
            a= l[0]
            l=l[1:]
            t=int(a)+c
            res= res+ str(t%10)
            c= t//10
        if c!=0:
            res= res+str(c)
        return res[::-1]
        
    def multy1(self,s1,s2):
        n = int(s2)
        res = "0"
        while n:
            res = self.add(s1,res)
            n-=1
        return res
        
        
    def multiply(self, num1: str, num2: str) -> str:
        if num1=='0' or num2=='0':
            return '0'
        if len(num1) < len(num2):
            t = num1
            num1 = num2
            num2=t
        n=0
        res= '0'
        for i in range(len(num2)-1,-1,-1):
            tempvalue = self.multy1(num1,num2[i])
            tempvalue= tempvalue+'0'*n
            print(tempvalue)
            res = self.add(res,tempvalue)
            n+=1 
        return res
```


989. Add to Array-Form of Integer
一个数组组成的大数加一个小整数
num = [1,2,0,0], k = 34

虽然k 不是一位10进制, 但是我们可以一样算
1. 除  要用//
2.  split 判断边界 比如是0 的时候

自己列举一些case 比如
[0],1000

[9,9,9] ,1


```py
class Solution:
    def split(self,i):
        if i==0:
            return [0]
        res=[]
        while i:
            res.append(i%10)
            i//=10
        print(i,res[::-1])
        return res
    
    
    
    def addToArrayForm(self, num: List[int], k: int) -> List[int]:
        num =num[::-1]
        res=[]
        c=0
        t= num.pop(0)+k +c
        v = t%10
        res.extend(self.split(v))
            
        c = t//10
        while num:
            t= num.pop(0) + c
            res.append(t%10)
            c = t//10
        
        if c !=0:
            res.extend(self.split(c))  
        return res[::-1] 
```



367. Valid Perfect Square
69. Sqrt(x)
数值二分
完美平方数, 不能用内置的开方函数

```py
class Solution:
    def isPerfectSquare(self, num: int) -> bool:
        upper = num
        lower = 0
        while True:
            mid =( upper + lower) // 2
            t= mid*mid
            if abs(upper - lower) <16:
                lower  = max(1, upper-16)
                upper += 1
                for i in range(lower,upper):
                    if i*i == num:
                        return True
                return False
            
            if t == num:
                return True
                
            if t >num:
                upper = mid
            else:
                lower = mid
            
```

70. Climbing Stairs
爬梯子 一次能爬一步或者两步, 没啥好说的
```py
class Solution:
    def climbStairs(self, n: int) -> int:
        a = b = 1
        for i in range(2, n+1):
            a, b = b, a + b            
        return b
```

746. Min Cost Climbing Stairs
每个阶站上去要有花费,比如一开始在第0阶,先爬上第一阶,再爬两个到第三阶, 总共花费cost[1],如果在往上爬, 要算上cost[3]
f(n) = min(f(n-1)+c[n-1] ,f(n-2)+c[n-2])
f(0)=0
f[1] =0

最后的答案是f[len(cost)+1]
```py
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        #f(n) = min(f(n-1)+c[n-1] ,f(n-2)+c[n-2])
        dp = [0]*(len(cost)+1)
        for i in range(2,len(cost)+1):
            dp[i] = min(dp[i-1]+cost[i-1],dp[i-2]+cost[i-2])
        
        return dp[-1]
```


1137. N-th Tribonacci Number
斐波那契变种
T0 = 0, T1 = 1, T2 = 1
Tn+3 = Tn + Tn+1 + Tn+2
```py
class Solution:
    def tribonacci(self, n: int) -> int:
        if n ==0:
            return 0
        a,b,c = 0,1,1
        for i in range(3,n+1):
            a,b,c = b,c,a+b+c
        return c
```



148. Sort List
链表排序 归并
```py
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def merge(sef,l1,l2):
        p1 =l1
        p2=l2
        head= ListNode(0)
        p= head
        
        while p1 and p2:
            if p1.val < p2.val:
                temp= p1
                p1=p1.next               
            else:
                temp= p2
                p2=p2.next
                p.next = temp
            temp.next =None
            p.next = temp
            p= p.next
        l = p1 or p2
        p.next =l
        return head.next
    
        
            
    def sortList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        if not head:
            return head
        if head.next is None:
            return head
        
        prev= fast =mid = head
        
        while fast and fast.next:
            prev= mid
            fast = fast.next.next
            mid = mid.next
        prev.next =None
        
        return self.merge(self.sortList(head),self.sortList(mid))
```

324. Wiggle Sort II
给一个数组, 排成湍流数组
nums[0] < nums[1] > nums[2] < nums[3]....
 nums = [1,5,1,1,6,4]
 Output: [1,6,1,5,1,4]

 注意第一个元素是小值, 所以如果 两边划分不均匀,小的那边要多一个

```py
class Solution:
    def wiggleSort(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nums.sort()
        mid = len(nums)//2
        if len(nums)%2==1:
            mid+=1
        a= nums[:mid][::-1]
        b=nums[mid:][::-1]
        res=[]
        print(a,b)
        while a and b:
            res.append(a.pop(0))
            res.append(b.pop(0))
            
        l = a or b
        while l:
            res.append(l.pop(0))
        nums[:]= res
```


98. Validate Binary Search Tree
验证一个树是不是二叉排序数
二叉排序树的中序遍历是有序的, 只需要中序遍历 记录上一个值, 然后判断大小就行
```py
class Solution:
    def f(self,root,result):
        if not root:
            return True
        
        if not self.f(root.left,result):
            return False
        
        temp = result[0]
        if temp >= root.val:
            return False
        result[0]= root.val
        
        return self.f(root.right,result)
            

    
        
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        result =[-2**64]        
        return self.f(root,result)
```


230. Kth Smallest Element in a BST
求一个二叉排序树第k小的数
还是中序,计个数
```py
class Solution:
    def f(self,root, count,result,k):
        if not root:
            return
        
        self.f(root.left,count,result,k)
        count[0]+=1
        if count[0] ==k:
            result[0] = root.val
            return
        
        
        self.f(root.right,count,result,k)
        
        
        
    def kthSmallest(self, root: Optional[TreeNode], k: int) -> int:
        count=[0]
        result=[0]
        self.f(root,count,result,k)
        return result[0]
```

783. Minimum Distance Between BST Nodes
给一个二叉排序数 返回任意节点差的最小值

还是利用中序遍历的排序特性, 保存上一个值 然后减
一颗二叉排序树终结点的的最小距离差值
```py
class Solution:
    #-inf  1 2 3 4 6 inf
    def f(self,root,prev,result,t):
        if not root:
            return
        self.f(root.left,prev,result,t)
        if prev[0] != t:
            result[0] = min(root.val-prev[0],result[0])
        prev[0] = root.val
        
        
        self.f(root.right,prev,result,t)
    
    def minDiffInBST(self, root: Optional[TreeNode]) -> int:
        inf = 2**64
        result=[inf]
        prev=[-inf]
        
        self.f(root,prev,result,-inf)
        return result[0]
```


102. Binary Tree Level Order Traversal
二叉树层序遍历 每一层放到一起成为一个数组
#### 注意如果有其它业务条件的话 while循环完了还要处理一下

Input: root = [3,9,20,null,null,15,7]
Output: [[3],[9,20],[15,7]]
bfs , level 不用last 指针来做, 要用level 计数来做

```py
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        res=[[]]

        level = 1
        st=[(root,level)]
        currentLevel = 1
        while st:
            node,level =st.pop(0)
            if level == currentLevel:
                res[-1].append(node.val)
            else:
                res.append([node.val])
                currentLevel+=1
                
            if node.left:
                st.append( (node.left,level+1) )
            if node.right:
                st.append( (node.right,level+1) )
           
        return res
```

103. Binary Tree Zigzag Level Order Traversal
同上题 每一层,反转顺序
加个逻辑就行;
  if currentLevel &1==0:
        res[-1][:]=res[-1][::-1]

116. Populating Next Right Pointers in Each Node
层序遍历同一层的节点用指针连起来
层序遍历变种
太简单
```py
class Solution:
    def connect(self, root: 'Optional[Node]') -> 'Optional[Node]':
        if not root:
            return root

        level = 1
        st=[(root,level)]
        currentLevel = 1
        prev = TreeNode(0)
        while st:
            node,level =st.pop(0)
            if level == currentLevel:
                prev.next = node
            else:
                prev.next= None
                prev = TreeNode(0)

                currentLevel+=1
                
            prev=node
            if node.left:
                st.append( (node.left,level+1) )
            if node.right:
                st.append( (node.right,level+1) )
           
        return root
```



```py
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        res=[[]]

        level = 1
        st=[(root,level)]
        currentLevel = 1
        while st:
            node,level =st.pop(0)
            if level == currentLevel:
                res[-1].append(node.val)
            else:
                if currentLevel &1==0:
                    res[-1][:]=res[-1][::-1]
                res.append([node.val])
                currentLevel+=1
                
            if node.left:
                st.append( (node.left,level+1) )
            if node.right:
                st.append( (node.right,level+1) )
        if currentLevel &1==0:
            res[-1][:]=res[-1][::-1]
            
        return res
```


111. Minimum Depth of Binary Tree
最小的从根节点到叶子节点的深度
注意如果 不是叶子节点的空孩子, 要剪枝,必须是叶子节点才算深度
模型是从下往上返回 逐层+1
```py
class Solution:
    def f(self,root):
        if not root.left and not root.right:
            return 1
        
        r= l = 2**64
        if root.left:
            l = self.f(root.left)
        if root.right:
            r = self.f(root.right)
                
        return min(l,r)+1
    
    def minDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        return self.f(root)
```


637. Average of Levels in Binary Tree
层序遍历每层的均值:

自带的均值函数 mean
return [mean(i) for i in res]




105. Construct Binary Tree from Preorder and Inorder Traversal
根据先序和中序建立二叉树
# 记住这个模板
直接背过  摘一个节点 然后分成左边和右边
```py
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        if  inorder:            
            val = preorder.pop(0)
            node = TreeNode(val)
            i = inorder.index(val)
            node.left =  self.buildTree(preorder , inorder[:i])
            node.right = self.buildTree(preorder , inorder[i+1:])
            return node
```   


106. Construct Binary Tree from Inorder and Postorder Traversal
根据中序和后序建立二叉树
##  重要:要先递归右边

```py
class Solution:
    def buildTree(self, inorder: List[int], postorder: List[int]) -> Optional[TreeNode]:
        if postorder and inorder: # ------------------------不一样的地方之一 要加inorder的判断
            t= postorder.pop()
            #print(inorder,postorder)
            i = inorder.index(t)
            node = TreeNode(t)
            node.right = self.buildTree(inorder[i+1:],postorder)
            node.left = self.buildTree(inorder[:i],postorder)

            return node
            
```
108. Convert Sorted Array to Binary Search Tree
根据排好顺序的数组建立二叉搜索树

```py
class Solution:
    def f(self,nums,i,j):
        if i>j:
            return None
        mid =(i+j)//2
        node = TreeNode(nums[mid])
        node.left = self.f(nums,i,mid-1)
        node.right = self.f(nums,mid+1,j)
        return node
        
        
        
    def sortedArrayToBST(self, nums: List[int]) -> Optional[TreeNode]:
        return self.f(nums,0,len(nums)-1)

```


118. Pascal's Triangle
     1
    1  1
  1   2   1
 1  3   3   1 
1  4  6   4   1

不想说了 太简单
```py
class Solution:
    def generate(self, numRows: int) -> List[List[int]]:
        if numRows ==1:
            return [[1]]
        res=[[1]]
        for i in range(1,numRows):
            l = [0]+res[-1].copy()+[0]
            temp=[]
            for j in range(1,len(l)):
                temp.append(l[j]+l[j-1])
            res.append(temp)
        return res
```


试试用英语说:
122. Best Time to Buy and Sell Stock II
Input: prices = [7,1,5,3,6,4]
Output: 7
(直接做出来的)

1 .记录下 到当前的最小值
2. 记录当前和最小值之间的利润,和上一次的利润
3. 如果利润在不断扩大,那么就继续
4. 如果利润比上一次减小,那么就结算, 并且最小值从当前开始 并且清空上一次的利润 
可以不断的买卖
5 使用end 来记录最后一次发生利润减小结算的位置,如果到最后还有一次没结算 ,就要结算
```py
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if len(prices) ==1:
            return 0
        m = prices[0]
        prevprofit = 0
        s = 0
        end = 0
        for i in range(1,len(prices)):
            m = min(m,prices[i])
            profit = prices[i] - m
            print(i,profit,prices[i])
            if profit < prevprofit:
                s += prevprofit
                m = prices[i]
                prevprofit =0
                end = i  
            else:
                prevprofit = profit
        if end != len(prices)-1:
            s+=profit
        return s
```

125. Valid Palindrome

isalpha
isdigit
isalnum
```py
        s = ''.join(i.lower() for i in s if i.isalnum())
        return s==s[::-1]
```

680. Valid Palindrome II
最多删除一个字母能不能组成回文子串
也可以写成递归

从两边向中间挤, 出现不相等的字母就给一次机会, 干掉左边或者干掉右边然后继续
```py
class Solution:
    def validPalindrome(self, s: str,f=False) -> bool:
        i =0
        j= len(s)-1
        while i<=j:
            #print(i,j,s[i],s[j])
            if s[i] ==s[j]:
                i+=1
                j-=1
            else:
                if not f: # 给一次机会
                    f= True
                    if i+1 <= j and s[i+1]==s[j]:
                        if self.validPalindrome(s[i+1:j+1],True):
                            return True
                    if j-1 >=i and s[j-1]==s[i]:
                        return self.validPalindrome(s[i:j],True)
                    
                        
                else:
                    return False
        return True
```


2002. Maximum Product of the Length of Two Palindromic Subsequences
hard


128. Longest Consecutive Sequence

写了一个O(n)的并查集, 还不如排序快
```py
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if len(nums)==0:
            return  0
        nums = sorted(list(set(nums)))
        maxlength = 0
        s=1
        for i in range(1,len(nums)):
            if nums[i] - nums[i-1] ==1:
                s+=1     
            else:
                maxlength = max(maxlength,s)
                s=1
        maxlength = max(maxlength,s)
        return maxlength
```



```py
class Solution:
    def longestConsecutive(self, nums: List[int]) -> int:
        if len(nums)==0:
            return  0
        d={}
        for i in nums:
            d[i]=1
        # uni-find set
        ufs = {}

        
        # construct tree
        for i in nums:
            if i-1 in d:
                ufs[i] = i-1
            else:
                ufs[i] = i

        # path compression
        visited={}
        for i in nums:
            if i in visited:
                continue

            c=[]
            t= i
            while t != ufs[t]:
                c.append(i)
                t=ufs[t]
                
            for j in c:
                ufs[j] =t
                visited[j]=1
            
            
        #print(ufs)  
        d1={}
        mx = 1
        visited={}

        for i in nums:
            if i in visited:
                continue
            visited[i] =1
            
            if ufs[i] not in d1:
                d1[ufs[i]]=1
            else:
                d1[ufs[i]]+=1
                mx = max(d1[ufs[i]],mx)
        return mx
```


2177. Find Three Consecutive Integers That Sum to a Given Number
```py
class Solution:
    def sumOfThree(self, num: int) -> List[int]:
        t = num//3-1
        while True:
            a = 3*t-3
            if a == num:
                return [t-2,t-1,t]
            if a>num:
                return []
            t+=1
            
```


130. Surrounded Regions
一个矩阵 完全被X包围起来的O 被同化成X:
先把边界的O 全部标记为F, 然后用bfs 以F为起点遍历所有开放边界的O的区域
没被遍历到的o 改成x


#####   bfs 记住入队, 要去重
```py
class Solution: 
    def solve(self, board):
        """
        Do not return anything, modify board in-place instead.
        """
        
        start=[0,0]
        for i in range(len(board)):
            if board[i][0] =='O':
                board[i][0] ='F'
                
            if board[i][-1] =='O':
                board[i][-1] ='F'
                
                
                
        for i in range(len(board[0])):
            if board[0][i] =='O':
                board[0][i] ='F'
            if board[-1][i] =='O':
                board[-1][i] ='F'
        
        
        
        visited={}
        for i in range(len(board)):
            for j in range(len(board[i])):
                if (i,j) in visited:
                    continue
                if board[i][j] == 'X' or board[i][j] == 'O':
                    continue
                    
                if board[i][j] =='F':
                    st=deque([(i,j)])
                    while st:
                        a,b = st.popleft()
                        #print("=====>",a,b)
                        board[a][b]='F'
                        
                        for d in [[0,1],[1,0],[0,-1],[-1,0]]:
                            row = a+d[0]
                            col = b+d[1]
                            if row <0 or row >=len(board) or col <0 or col >=len(board[0]):
                                  continue
                            if (row,col) in visited:
                                  continue
                            visited[(row,col)]=1
                            if board[row][col] !='X':
                                st.append((row,col))
       # print(board)
        for i in range(len(board)):
            for j in range(len(board[i])):
                if board[i][j]=='X':
                    continue
                elif board[i][j]=='F':
                    board[i][j] ='O'
                elif board[i][j]=='O':
                    board[i][j] ='X'
```


131. Palindrome Partitioning
把一个字符串分割成每个部分都是回文串的方案
Input: s = "aab"
Output: [["a","a","b"],["aa","b"]]

以s[0] 为开始字符 , 找一些回文, 然后递归后面的
```py
class Solution:
    # Palindrome string start with s[0] 
    def findPalindrome(self,s):
        if len(s)==1:
            return [s]
        result =[]
        for i in range(1,len(s)+1) :
            if s[:i] ==s[:i][::-1]:
                result.append(s[:i])
        
        return result     
        
    def f(self,s,index,temp,result):
        if index >=len(s):
            result.append(temp)
            return
            
        ts = self.findPalindrome(s[index:])
        for t in ts:
            a = temp.copy()
            a.append(t)
            self.f(s,index+len(t),a,result)
        
    def partition(self, s: str) -> List[List[str]]:
        result=[]
        self.f(s,0,[],result)
        return result
```

134. Gas Station
核心就是这条
gas[i] +left - cost[i] <0:



```py
class Solution:
    def canCompleteCircuit(self, gas: List[int], cost: List[int]) -> int:
        # 如果大于 则一定有解 ----------------   重要
        if sum(gas) < sum(cost):
            return -1
        gas = gas*2
        cost = cost*2
        start = 0
        left =0
        for i in range(len(gas)):
            if gas[i] +left - cost[i] <0:
                left = 0
                start = i+1
                continue   
            left +=gas[i]-cost[i]
        return start%(len(gas)//2)
```


137. Single Number II
除了一个数字只出现一次, 其他都出现3次,求那个一次的
Input: nums = [2,2,3,2]
Output: 3


统计每一位出现的次数 然后模3
```py
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        counts = [0] * 32
        for num in nums:
            for j in range(32):
                counts[j] += int((num & (2**j)) != 0)
        s = 0
        
        res, m = 0, 3
        for i in range(32):
            res += (counts[i] % m)*2**i
        return res if counts[31] % m == 0 else ~(res ^ 0xffffffff)  如果最后的结果是正数 直接返回res,否则返回 按符号位取反, 然后不带符号位取反
```

位运算
n&(n-1)   将最右边的1变成0
n&(-n) 只是保留二进制下最右边的1的位

260. Single Number III

```py
class Solution(object):
    def singleNumber(self, nums):
        diff = reduce(lambda x, y: x ^ y, nums, 0) # Get the XOR of the two unique elements.
        diff &= -diff # Find the last set bit.
        res = [0, 0]
        for num in nums:
            res[num & diff == 0] ^= num
        return res
```



268. Missing Number
### 数值求和
```
class Solution:
    def missingNumber(self, nums: List[int]) -> int:
        return int(((len(nums)+1)*len(nums))/2) - sum(nums)
```

138. Copy List with Random Pointer
深拷贝一个链表, 链表有一个random指针 随意指向某些节点


第一个方法 最好理解 最好写 
先当一个普通的链表拷贝 不管random, 但是记录下新旧节点的对应关系
然后便利一遍, 找到random 指向节点的的对应新节点 然后赋值
```py
"""
# Definition for a Node.
class Node:
    def __init__(self, x: int, next: 'Node' = None, random: 'Node' = None):
        self.val = int(x)
        self.next = next
        self.random = random
"""

class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        if not head:
            return None
        
        p = head
        p1 = result = Node(head.val)
        d={id(head):p1}

        p = p.next
        
        while p:
            newNode = Node(p.val)
            d[id(p)] = newNode
            p1.next = newNode
            p1=p1.next
            p=p.next
        
        
        p =head
        while p:
            if p.random is None:
                p=p.next
                continue
                
            newNode = d[id(p)]
            newRandomNode = d[id(p.random)]
            newNode.random = newRandomNode   
            p=p.next
        return result
```

1. 先复制每个节点 这把每个复制的节点先暂时挂在原先节点的后面
2. 然后指正random 指针指向愿的random的后面
3. 然后在把每个节点拆出来放到原先的节点后面


```py
class Solution:
    def copyRandomList(self, head: 'Optional[Node]') -> 'Optional[Node]':
        p = head
        
        d= {}
       # dup  every node and put it behind  next the the origin  
        while p:
            next = p.next
            n=Node(p.val,p.next,p.random)
            d[id(n)] = n
            p.next = n
            p= next
            
            
        # correct  the random to the  brand new node  random node
        
        p = head
        while p:
            if id(p) in d:
                if p.random is not None:
                    p.random = p.random.next
            p = p.next
        
        
        # split the new node to a new list
        p1 = head1 = Node(0)
        
        p =head
        
        while p:
            next = p.next
            if id(p) in d:
                p1.next = p
                p.next = None
                p1 = p1.next
            p=next
        
        return head1.next
```


133. Clone Graph
复制图


图结构
"""
# Definition for a Node.
class Node:
    def __init__(self, val = 0, neighbors = None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []
"""

bfs 遍历,每遍历到一个节点


```py
class Solution:
    def cloneGraph(self, node: 'Node') -> 'Node':
        if not node:
            return node
        queue = deque([node])
        root = Node(node.val)
        d={id(node):root}
        
        
        while queue:
            n = queue.pop()
            newGraphNode = d[id(n)]
            for nbr in n.neighbors:
                if id(nbr) not in d:
                    newnbr =  Node(nbr.val)
                    d[id(nbr)] =newnbr
                    queue.append(nbr)
                newGraphNode.neighbors.append(d[id(nbr)])
        return root        
```

139. Word Break

```py
class Solution:
    def wordBreak(self, s: str, wordDict: List[str]) -> bool:
        starts = [0]
        for i in range(1,len(s)+1):
            for start in starts:
                if s[start:i] in wordDict:
                    starts.append(i)
                    break # 没有这个break就会超时 为啥呢  为啥 break 是对的呢 
                    # 已经确认这个i 是一个 critical point , 目的已经达到了 没必要继续
        return starts[-1] == len(s)
            
```



142. Linked List Cycle II
返回链表成环的入口节点

hash 记录 最近单 但是最慢, 还有个方法
，当发现 slow 与 fast 相遇时，我们再额外使用一个指针它指向链表头部；随后，它和 slow 每次向后移动一个位置。最终，它们会在入环点相遇。


```py
class Solution:
    def detectCycle(self, head: Optional[ListNode]) -> Optional[ListNode]:
        d={}
        p = head
        cnt=0
        while p:
            if id(p) in d:
                return  p
            d[id(p)]=cnt
            p=p.next
            cnt+=1
            
        return None
```



146. LRU Cache
#   orderedDict !!!!!!!!!!


自己实现 hash + 双向链表
也很简单


```py
class LRUCache:

    def __init__(self, capacity: int):
        self.d= OrderedDict()
        self.cap= capacity

    def get(self, key: int) -> int:
        if key not in self.d:
            return -1
    
        self.d.move_to_end(key)
        return self.d[key]

    def put(self, key: int, value: int) -> None:
        if key in self.d:
            self.d.move_to_end(key)
        self.d[key]=value
        if len(self.d) > self.cap:
            self.d.popitem(last=False)
        
```
自己写的dict + list很慢
```py
class LRUCache:

    def __init__(self, capacity: int):
        self.st= []
        self.d={}
        self.cap= capacity

    def get(self, key: int) -> int:
        if key not in self.d:
            return -1
    
        self.st.remove(key)
        self.st.append(key)
        return self.d[key]

    def put(self, key: int, value: int) -> None:
        if self.get(key) ==-1:
            if len(self.st) == self.cap:
                v = self.st.pop(0)
                del self.d[v]
            self.st.append(key)
        self.d[key]=value
```


155. Min Stack
实现一个栈,有 push pop top  和getmin方法
getmin
另外加一个栈
新入栈的比当前小 就直接入站  否则栈顶dup一份入栈

```py
class MinStack:

    def __init__(self):
        self.st=[]
        self.minst=[]

    def push(self, val: int) -> None:
        self.st.append(val)
        if not self.minst:
            self.minst.append(val)
            return 
        if val >= self.minst[-1]:
            self.minst.append(self.minst[-1])
        else:
            self.minst.append(val)

    def pop(self) -> None:
        self.st.pop()
        self.minst.pop()

    def top(self) -> int:
        return self.st[-1]

    def getMin(self) -> int:
        return   self.minst[-1]
```


# dict intersect list((a & b).elements())
# def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
#         a,b = map(collections.Counter,(nums1,nums2))
#         return list((a & b).elements())



162. Find Peak Element
二分加线性搜索

```py
class Solution:
    def findPeakElement(self, nums: List[int]) -> int:
        if len(nums) ==1:
            return 0
        
        nums.insert(0,-2**64)
        nums.append(-2**64)
        i = 0
        j = len(nums)-1
        while True:
            if j-i < 32:
                for t in range(i,j+1):
                    if nums[t-1]< nums[t] > nums[t+1]:
                        return t-1
            
            
            mid = (i+j)//2
            if nums[mid-1]< nums[mid] > nums[mid+1]:
                return mid-1
            if  nums[mid-1]< nums[mid] < nums[mid+1]:
                i =mid+1
            else:
                j=mid-1
```


2210. Count Hills and Valleys in an Array
是个easy题 
严格大于两边的元素 或者严格小于两边的元素的元素的个数
len(nums)>3 且两头的肯定不是
如果有连续的相同的值, 那么一直比对到不同的值,比如说下面两个1, 的左右都是4,6
而且相同的值算做一个
就是可以是高原,或者盆地

Input: nums = [2,4,1,1,6,5]
Output: 3

```py
class Solution:
    def countHillValley(self, nums: List[int]) -> int:
        left=[0]*len(nums)
        left[0] = nums[0]
        right=[0]*len(nums)
        right[-1] = nums[-1]

        for i in range(1,len(nums)-1):
            if nums[i]==nums[i-1]:
                left[i]=left[i-1]
            else:
                left[i]=nums[i-1]
                
            j = len(nums)-i-1
            
            if nums[j]==nums[j+1]:
                right[j]=right[j+1]
            else:
                right[j]=nums[j+1]
        cnt = 0
        prev= 0
        for i in range(1,len(nums)-1):
            if nums[i] ==prev:
                continue
            if (left[i] < nums[i] and right[i] < nums[i]) or \
                (left[i] > nums[i] and right[i] > nums[i]):
                    cnt+=1
            prev = nums[i]
        return cnt
```
1901. Find a Peak Element II
没做

166. Fraction to Recurring Decimal

太难了不做


229. Majority Element II
数组中找到出现次数超过1/3总数的数组
# most_common 返回的tuple 按大小顺序
```py
class Solution:
    def majorityElement(self, nums: List[int]) -> List[int]:
        d= Counter(nums).most_common()
        c = len(nums)//3
        res=[]
        for i in d:
            if i[1] <= c:
                return res
            res.append(i[0])    
        return res
```

171. Excel Sheet Column Number
转换:

A -> 1
B -> 2
C -> 3
...
Z -> 26
AA -> 27
AB -> 28 
就进制不断乘加就行

168. Excel Sheet Column Title
反过来 数字转成字符串


不断% 和 除就行 ,但是 mod 等于0的时候单独处理, 0 变成26->z  并且向高位借一位

```py
class Solution:
    def convertToTitle(self, columnNumber: int) -> str:
        s=""
        while columnNumber:
            c= columnNumber%26
            f= False
            if c==0:
                if columnNumber !=0:
                    c =26
                    f= True
            l =chr(ord('A') + c-1)
            print(c,l)
            s=s+ l
            columnNumber = columnNumber// 26
            if f:
                columnNumber -=1
        return s[::-1]
```



1901. Find a Peak Element II
一个二维数组 找出peak
先找每行的max,然后在按照一维的标准去找 每行max的peak 就完事了
```py
class Solution:
    def findPeakGrid(self, mat: List[List[int]]) -> List[int]:
        k =  [ [mat[i].index(max(mat[i])),max(mat[i])] for i in range(len(mat))]
        k.insert(0,[0,-1])
        k.append([0,-1])
        
        i = 1
        j = len(k)-2
        while i <=j:
            if abs(j - i) <32:
                left = max(1,i)
                right = min(j+1,len(k)-1)
                for mid in range(left,right):
                    if k[mid-1][1] < k[mid][1] > k[mid+1][1]:
                        return [mid-1,k[mid][0]]
                return []
            
            mid = (i+j)//2
            if k[mid-1][1] < k[mid][1] > k[mid+1][1]:
                return [mid-1,k[mid][0]]
            
            if k[mid-1][1] <= k[mid][1] <= k[mid+1][1]:
                i = mid+1
            else:
                j= mid-1
```




2194. Cells in a Range on an Excel Sheet
Input: s = "K1:L2"
Output: ["K1","K2","L1","L2"]

两个范围展开所有的char
char和int  转来转去 easy题

```py
class Solution:
    def cellsInRange(self, s: str) -> List[str]:
        a,b = s.split(":")
        aa = a[0]
        ab = a[1]
        
        ba = b[0]
        bb = b[1]
        res=[]
        for i in range(ord(aa),ord(ba)+1):
            for j in range(int(ab),int(bb)+1):
                res.append(chr(i)+str(j))
                
        return res
```


172. Factorial Trailing Zeroes
    因为每隔 5 个数出现一个 5，所以计算出现了多少个 5，我们只需要用 n/5 就可以算出来。


每隔 25 个数字，出现的是两个 5，所以除了每隔 5 个数算作一个 5，每隔 25 个数，还需要多算一个 5。
```py
class Solution:
    def trailingZeroes(self, n: int) -> int:
        count = 0
        while n:
            n//=5
            count +=n
        return count
```



179. Largest Number
Input: nums = [10,2]
Output: "210"


主要是自定义排序函数
key = cmp_to_key
小于返回1 大于返回-1

```py
class Solution:
    
    def largestNumber(self, nums: List[int]) -> str:
        def cmp(a,b):
            a1= str(a)
            b1=str(b)
            if  (a1+b1) <(b1+a1):
                return 1
            return -1
        nums = sorted(nums,key=cmp_to_key(cmp))
        t=  ''.join(str(i) for i in nums)
        if int(t) ==0:
            return "0"
        return t
```


2165. Smallest Value of the Rearranged Number
一样的题 sort  reverse
第一个不能是0 , 如果第一个是0, 就往后面第一个不是0的换

```py
class Solution:
    def smallestNumber(self, num: int) -> int:
        if num ==0:
            return 0
        sign  = abs(num)/num
        num =abs(num)
        ss= sorted([s for s  in str(num)],reverse=False if sign == 1 else True)
        
        if ss[0]=='0':
            i =0
            while ss[i] =='0':
                i+=1
            ss[0]=ss[i]
            ss[i]='0'
        l = ''.join(ss)
        return str(int(int(l)*sign))
```


189. Rotate Array
余数 是0 不能转, 转了不对
```py
class Solution:
    def rotate(self, nums: List[int], k: int) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
       
        l = len(nums)
        k= k%l
        if k==0:
            return nums
        nums.extend(nums)
        nums[:] = nums[l-k:-k]

```


61. Rotate List
 while fast and k: 不能这么写
 while fast is not None and k!=0: 必须这么写
```py
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def rotateRight(self, head: Optional[ListNode], k: int) -> Optional[ListNode]:
        if not head:
            return None
       
        fast = slow = p =head
        l = 0
        while p:
            l+=1
            p=p.next
            
        k =k %l
        if k ==0:
            return head
        fastprev= fast
        while fast is not None and k!=0:
            fastprev=fast
            fast = fast.next
            k-=1
        
        slowprev= slow
        while fast:
            fastprev=fast
            slowprev= slow
            fast = fast.next
            slow = slow.next
        
        slowprev.next = None
        fastprev.next = head
        head = slow
        return head
        
```


198. House Robber
一个数组里面pick一些数字 这些数字不能靠在一起, 求和的最大值
dpdpdppdpdpdpdp   没想到啊
```py
class Solution:
    def rob(self, nums: List[int]) -> int:
        if len(nums) ==0:
            return 0
        if len(nums) == 1:
            return nums[0]
        
        dp=[0]*len(nums)
        dp[0]  =nums[0]
        dp[1] = max(nums[0],nums[1])
        
        if len(nums) == 2:
            return dp[1]
        
        # at least 3 element
        for i in range(2,len(nums)):
            dp[i] = max(nums[i]+dp[i-2],dp[i-1])
            
            
        return dp[-1]
```


213. House Robber II
同上题 只不过房子首尾都连起来了
其实一样 选第一间或者不选第一间 比较大小就行
比较 rob(nums[1:])   和 rob(nums[:-1])  其中rob 是上面那提的求最大值函数



740. Delete and Earn

一个数组,每删掉一个值v 就可以得这个值的分数,但是要同时删掉 v-1 和v +1 ,连带删掉的不得分

同上题

一个值, 比如4 ,4删不删 和3有关系 和5有关系 但是和2没有关系
但是最终的值却要加上2的值

删了4,得到4得分, 就不能删3得3的分  4和3是互斥的 选4 就不能选3
所以删除4  还是删除3 要比较 dp[2] + 4 * count(4) 和 dp[3]


dp[i] = max(dp[i-1] ,dp[i-2]+count(i)*i)


先把有数组的状态方程实现了,在实现没数组的

```py
class Solution:
    def deleteAndEarn(self, nums: List[int]) -> int:
        #3 5 2
        #f(2)+f(4/3/5)
        d=Counter(nums) 
        if len(d) ==1:
            return sum(nums)
      
        mx = max(nums)
        a = 0
        
        b = d[1] if 1 in d else 0
        
        
        for i in range(2,mx+1):
            count  = 0
            if i in d:
                count = d[i]
            c = max(a+count*i,b)
            a = b
            b = c

            
        return b

```

2140. Solving Questions With Brainpower

Input: questions = [[3,2],[4,3],[4,4],[2,5]]
Output: 5
每个元素 [a,b] 代表一道题, a 是解决这道题所得的分数,b是表示解决了这道题后面就n题就不能动,求分手最大值
dp[i] = max(d[i+1],dp[i+2]....dp[i+questons[i][1]] ,dp[i+questons[i][1]+1] + questons[i][0]

dp 应该是个递增数组 所以
dp[i] = max(dp[i+1],dp[i+questons[i][1]+1] + questons[i][0])
完事

```py
class Solution:
    def mostPoints(self, questions: List[List[int]]) -> int:
        if len(questions) ==1:
            return questions[0][0]
        if len(questions) == 2:
            return max(questions[0][0],questions[1][0])
        
        dp = [0]*len(questions)
        dp[-1] = questions[-1][0]
        
        for i in range(len(questions)-2,-1,-1):
            t = questions[i][0]
            if i + questions[i][1]+1<=len(questions)-1:
                t+= dp[i + questions[i][1]+1]
            dp[i] = max(dp[i+1],t)
        #print(dp)
        return dp[0]         
```


1905. Count Sub Islands

两个矩阵, 0是水,1是岛屿 ,1连在一起是一个岛屿
如果 第二个矩阵的岛屿是第一个矩阵岛屿的子集 ,就算做一个岛屿


这题就是 如果要返回false  不能阻断后面的遍历,不能直接return False  因为要把一整个岛屿都标记上 ,必须在最后返回

```py
class Solution:
    def f(self,grid,grid1, i,j,visited):
        if i<0 or i >=len(grid) or j<0 or j >=len(grid[i]):
            return True
        if (i,j) in visited:
            return True
        visited[(i,j)]=1

        if grid[i][j]==0:
            return True
        
        f =  f= True
        if grid1[i][j] == 0:
            f = False


        for d in [[0,1],[0,-1],[1,0],[-1,0]]: 
            if not self.f(grid,grid1,i+d[0],j+d[1], visited):
                f=  False
        if not f:
            return False
        return True
        
    def countSubIslands(self, grid1: List[List[int]], grid2: List[List[int]]) -> int:
        cnt = 0
        visited={}
        for i in range(len(grid2)):
            for j in range(len(grid2[i])):
                if (i,j) in visited:
                    continue
                if grid2[i][j] ==0:
                    continue

                result =[0]
                if self.f(grid2,grid1,i,j,visited):
                    cnt+=1


        return cnt
```


丑数

```py
    def nthUglyNumber(self, n: int) -> int:
        d=[1]
        cnt = 0
        maxlength = 1
        while True:
            maxlength = max(maxlength,len(d))
            cnt+=1
            top = heappop(d)
            if cnt == n:
                print("maxlength",maxlength)
                return top


            l = [top*2,top*3,top*5]
            for i in l:
                if i not in d:
                    heappush(d,i)
        
```


dp 三指针:
背过背过
```py
class Solution:
    def nthUglyNumber(self, n: int) -> int:
        dp =[0]*(n+1)
        dp[1] =1
        p2,p3,p5 =1,1,1
        
        
        for i in range(2,n+1):
            dp[i] = min(dp[p2]*2,dp[p3]*3,dp[p5]*5)
            if dp[i] == dp[p2]*2:
                p2+=1
            if dp[i] == dp[p3]*3:
                p3+=1
            if dp[i] == dp[p5]*5:
                p5+=1
        
        return dp[n]
```

204. Count Primes

质数个数
```py
class Solution:
    def countPrimes(self, n: int) -> int:
        prime = [1] *(n+1)
        prime[:3]=[0]*3
        t = int(n**(1/2) +1)
        for i in range(2,t):
            prime[i+i:n:i] = [0]*len(prime[i+i:n:i])
        return sum(prime)
            
```

279. Perfect Squares
Input: n = 12
Output: 3
Explanation: 12 = 4 + 4 + 4.

求平方数的和


先想到底是深度还是宽度,这题明显是 宽度更好
加个去重 就完事了


```py
class Solution:
    def numSquares(self, n: int) -> int:
        queue = deque([(0,1)])
        t = int(n**(1/2)+1)
        while queue:
            v, level = queue.popleft()
            
            visited={}
            for i in range(t):
                value = v + i**2
                if value == n:
                    return level
                if value > n:
                    break
                if value not in visited:
                    visited[value] =1
                    queue.append((value,level+1))
        return 0
```


92. Reverse Linked List II
翻转中间一段链表 
开始加一个空白

Input: head = [1,2,3,4,5], left = 2, right = 4
Output: [1,4,3,2,5]


从左数 第二个 到 从左数第四个  翻转这一段

right 要加一

这题凑数的
```py
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        headNode = ListNode(0)
        headNode.next =head
        head = headNode
        
        
        leftprev = rightprev =  p1 = p2 = head
        right+=1
        
        while left:
            leftprev  = p1
            p1=p1.next
            left-=1
            
        while right:
            rightprev  = p2
            p2=p2.next
            right-=1
        
        leftprev.next =None
        rightprev.next = None
        headMid = ListNode(0)
        p = p1
        
        while p:
            next = p.next
            p.next =None
            
            
            hnext = headMid.next
            headMid.next = p
            headMid.next.next = hnext
            
            
            p=next
            
        leftprev.next = headMid.next
        p1.next = p2
        return head.next
        
        
```


207. Course Schedule
拓扑排序
构建一个graph的  和一个indgree, 提取出 classes


```py
class Solution:
    def canFinish(self, numCourses: int, prerequisites: List[List[int]]) -> bool:
        graph = {}
        indegree ={}
        
        for i in prerequisites:
            if i[0] not in indegree:
                indegree[i[0]] =0
            indegree[i[0]]+=1
            
            if i[1]  not in graph:
                graph[i[1]]=[]
            graph[i[1]].append(i[0])
            
        classes = set(list(indegree.keys()) + list(graph.keys()))
        
        
        while classes:
            f =False
            t= 0
            for i in classes:
                if i not in indegree or indegree[i] ==0:
                    t = i
                    classes.remove(i)
                    f = True
                    break
            
            if t in graph:
                for i in graph[t]:
                    indegree[i]-=1
            
                    
            if not f:
                return False
        return True            
```


210. Course Schedule II
输出拓扑排序的结果
上面的 加一段这个

没出现在prerequisites中的也要输出 且顺序随意
```py
 l = [i for i in range(numCourses)]
        for i in l:
            if i not in res:
                res.append(i)
```



310. Minimum Height Trees
一个连起来的图,随便选一个节点当根节点,怎么选树的高度最低
拓扑排序, 从外向内, 每次去掉度为1的 剩下两个节点或1个节点 就是最终答案

# 重要!!!!   更新度的时候要先全部删完在更新, 否则变化的过程中会删的不对

```py
class Solution:
    def findMinHeightTrees(self, n: int, edges: List[List[int]]) -> List[int]:
        if len(edges) == 0:
            return [0]
        
        graph ={}
        degree ={}
        nodes={}
        for i in edges:
            if i[0] not in degree:
                graph[i[0]] = []
                degree[i[0]] = 0
            if i[1] not in graph:
                degree[i[1]] = 0
                graph[i[1]] = []
                
            degree[i[0]]+=1
            degree[i[1]]+=1
            graph[i[0]].append(i[1])
            graph[i[1]].append(i[0])
            
            
            
            if i[0] not in nodes:
                nodes[i[0]]=1
            if i[1] not in nodes:
                nodes[i[1]]=1
                
                
                
        minimum = 2**64
        res=[]
        l = list(nodes.keys())
        while l:
            #print(l,degree)
            if len(l) <=2:
                return l
            removed = []
            for i in l.copy():
                if degree[i] <=1:
                    l.remove(i)
                    removed.append(i)
                    
            for j in removed: # ------------------------------------------------就是这里
                for c in graph[j]:
                    degree[c]-=1
        return l
```


前缀树
定义个节点做这个事
每个字符串最后加个堵头`@`



```py
class TrieNode:
    def __init__(self,val =0):
        self.val = val
        self.children= {}

class Trie:

    def __init__(self):
        self.root = TrieNode()
        

    def insert(self, word: str) -> None:
        r = self.root
        for i in word:
            if i in r.children:
                r = r.children[i]
            else:
                node = TrieNode(i)
                r.children[i] = node
                r = node
        r.children['@'] =  TrieNode('@')

    def search(self, word: str) -> bool:
        r = self.root
        for i in word:
            if i in r.children:
                r = r.children[i]
            else:
                return False
        return '@' in r.children
    

    def startsWith(self, prefix: str) -> bool:
        r = self.root
        for i in prefix:
            if i in r.children:
                r = r.children[i]
            else:
                return False

        return True
    
# Your Trie object will be instantiated and called as such:
# obj = Trie()
# obj.insert(word)
# param_2 = obj.search(word)
# param_3 = obj.startsWith(prefix)
```


648. Replace Words
Input: dictionary = ["cat","bat","rat"], sentence = "the cattle was rattled by the battery"
Output: "the cat was rat by the bat"

给一个字典, 然后把后面句子里面的单词换成字典的前缀


字典树,拿一个 字典层层嵌套就可以做到
```py
class Solution:
    def replaceWords(self, dictionary: List[str], sentence: str) -> str:
        trie={}
        for word in dictionary:
            t = trie
            for i in word:
                if i not in t:
                    t[i]={}
                t = t[i]
            t['#']='#'
            
        sentences = sentence.split(' ')
        res=[]
        for s in sentences:
            t= trie
            f= False
            for i in range(len(s)):
                if '#' in t: # ====== 到了结尾 匹配上了, 放入前缀
                    f= True
                    res.append(s[:i])
                    break 
                elif s[i] not in t: # 没匹配上 到了结尾 放入整个单词
                    f= True
                    res.append(s)
                    break
                else:
                    t= t[s[i]]
            if not f: #============================最后也要判断一次 是不是字典比单词长, 如果是, 就必须放到里面去
                res.append(s)
                
        return ' '.join(res)
```


676. Implement Magic Dictionary
返回是否有正好改变一个字母就能匹配的字母, 不能不改变也不能改变超过一个


比如字典中, 有 hello , 那么 hellx 返回true

直接循环判断长度必须相等，然后再判断不相同字符只有一个即可
```py
class MagicDictionary:

    def __init__(self):
        self.l=[]

    def buildDict(self, dictionary: List[str]) -> None:
        for word in dictionary:
            self.l.append(word)
        

    def search(self, searchWord: str) -> bool:
        for w in self.l:
            if len(w) != len(searchWord):
                continue
            cnt = 0
            for a,b in zip(searchWord,w):
                if a != b:
                    cnt+=1
                if cnt >1:
                    break
            if cnt ==1:
                return True
        return False
```



广义邻居
hello 产生5个字符串
*ello  h*llo ...
必须加* , *代表位置信息

其次必须记录*ello 是由什么转变过来 就是*ello 可以代表 aello ,bello ,就是不能代表 hello, 即 hello 不能匹配

所以字典中有
hallo 和bello  的时候 字典的是这样的 
h*llo [a,e]

判断能不能匹配的时候

比如hello , h*llo in dict and (if e not in d[h*llo]  return true elif len(d[h*llo] >1 return true)) 

```py
class MagicDictionary:

    def __init__(self):
        self.d={}

    def buildDict(self, dictionary: List[str]) -> None:
        for word in dictionary:
            for i in range(len(word)):
                temps = word[:i]+"*" +word[i+1:]
                if temps not in self.d:
                    self.d[temps] = []
                self.d[temps].append(word[i])
            
        
        

    def search(self, searchWord: str) -> bool:
        for i in range(len(searchWord)):
            temps = searchWord[:i]+"*"+searchWord[i+1:]
            if temps in self.d:
                if searchWord[i] not in self.d[temps] : #==============================这里的判断是重点
                    return True
                else:
                    if len(self.d[temps]) >1:
                        return True
                    
        
        return False
```

215. Kth Largest Element in an Array
heapq.nlargest


```py
class Solution:
    def findKthLargest(self, nums: List[int], k: int) -> int:
        heapify(nums)
        return heapq.nlargest(k,nums)[-1]
```





703. Kth Largest Element in a Stream
维护一个k那么大小的最大堆和最大堆

heap数组里面第0 个元组 就是最小的
```py
class KthLargest:        
    def __init__(self, k: int, nums: List[int]):
        self.l= []
        for i in nums:
            if len(self.l) <k:
                heapq.heappush(self.l,i)
            else:
                heapq.heappushpop(self.l,i)
        self.k = k
    
    
            
            
    def add(self, val: int) -> int:
        if len(self.l) <self.k:
            heapq.heappush(self.l,val)
        else:
            heapq.heappushpop(self.l,val)
        return self.l[0]

```


973. K Closest Points to Origin
返回坐标里原点最近的k个元素

heapq 没有compare key, 就把元组做成元组, 排序key 放在元组的第一个

```py
class Solution:
    def kClosest(self, points: List[List[int]], k: int) -> List[List[int]]:
        nums = []
        for p in points:
            t = p[0] **2 + p[1] **2
            item = (-t,p[0],p[1])
            if len(nums) < k:
                heapq.heappush(nums,item)
            else:
                heapq.heappushpop(nums,item)

        return [ [t[1],t[2]] for t in nums[:k]]
```


1985. Find the Kth Largest Integer in the Array
Input: nums = ["3","6","7","10"], k = 4
Output: "3"


还是topk的问题
```py
class Solution:
    def kthLargestNumber(self, nums: List[str], k: int) -> str:
        #def cmp(a,b):
        #    if len(a) ==len(b):
        #            return 1 if a<b else -1
        #    return 1 if len(a) < len(b) else -1
        #nums = sorted(nums, key = cmp_to_key(cmp))
        #return nums[k-1]
        

        hp = []
        for i in nums:
            if len(hp) < k:
                heapq.heappush(hp,(int(i),i))
            else:
                heapq.heappushpop(hp,(int(i),i))
        return hp[0][1]
```


2099. Find Subsequence of Length K With the Largest 
找前k大的数 不能改变原来的顺序


记录index

```py
class Solution:
    def maxSubsequence(self, nums: List[int], k: int) -> List[int]:
        hp = []
        for i in range(len(nums)):
            if len(hp) < k:
                heapq.heappush(hp,(nums[i],i))
            else:
                heapq.heappushpop(hp,(nums[i],i))
        indexes = []
        for i in range(k):
            indexes.append(hp[i][1])
        indexes.sort()
        
        return [nums[i] for i in indexes]
```


219. Contains Duplicate II
k那么大的窗口里面是否有重复的数字

Input: nums = [1,2,3,1], k = 3
Output: true
滑动窗口
不断地把前一个干掉
```py
class Solution:
    def containsNearbyDuplicate(self, nums: List[int], k: int) -> bool:
        d={}
        if len(nums)  ==1:
            return False
        d[nums[0]]=1
        start = 0
        for i in range(1,len(nums)):
            if i - start > k:
                d[nums[start]]-=1
                start +=1
            if nums[i] in d and d[nums[i]]>0:
                    return True

            d[nums[i]] =1
        return False
```



220. Contains Duplicate III
k大小的窗口中, 是否有数值相差 小于t的两个元素

二分:
利用二分维护一个大小为k的 有序数组,如果当前元素大于数组最大值, 或数组最小值 就直接判断两端, 否则获取二分的插入位置判断两边
```py
class Solution:
    def containsNearbyAlmostDuplicate(self, nums: List[int], k: int, t: int) -> bool:
        if len(nums)==1:
            return False
        if k==0:
            return False
        
        st = [nums[0]]

        start = 0
        for i in range(1, len(nums)):
            #print(nums[i],st)
            if nums[i] <= st[0] or nums[i] >= st[-1]:
                if abs(st[0]-nums[i]) <=t or abs(st[-1]-nums[i]) <=t:
                    return True
            else:
                index = bisect.bisect_left(st,nums[i])
                left = max(0,index-1)
                right = index
                #print("===>>",abs(st[left]-nums[i]) <=t , abs(st[right]-nums[i]) <=t,nums[right],right,nums[i],st)

                if abs(st[left]-nums[i]) <=t or abs(st[right]-nums[i]) <=t:
                    return True
                
            bisect.insort_left(st,nums[i])
                   
            if i - start >= k:
                st.remove(nums[start])
                start+=1       
            
                
                   
        return False
```


桶排序
每个桶 t 那么大 如果是一个桶,肯定符合条件 ,否则查看相邻的桶
bucket 只存一个元素就够了 ,因为如果有多个元素直接满足条件返回true了

```py
class Solution:
    def containsNearbyAlmostDuplicate(self, nums: List[int], k: int, t: int) -> bool:
        if len(nums)==1:
            return False

        if k ==0:
            return False
        
        bucket ={nums[0]//(t+1):nums[0]}
        start = 0
        #print(bucket)
        for i in range(1,len(nums)):
            b = nums[i]//(t+1)
            #print(i,b,bucket)
            if b not in bucket:
                bucket[b]  = nums[i]
                if (b-1 in bucket and abs(bucket[b-1] - nums[i]) <=t) \
                    or (b + 1 in bucket and abs(bucket[b + 1] - nums[i]) <=t):
                        return True
            
            else:
                return True

        
            if i - start +1 >= k + 1:
                del bucket[nums[start]//(t+1)]
                start+=1

        return False
```

230. Kth Smallest Element in a BST
二叉排序树 第k大的 中序遍历


671. Second Minimum Node In a Binary Tree
一个二叉树 每个节点的子节点不是2个就是0个, 而且每个节点的值等于子节点中的最小,求第二小的值
根节点是最小的 , 只要找严格大于根节点的就行

```py
class Solution:
    def f(self,root,result,rootval):
        if not root:
            return 
        if root.val > rootval:
            result[0] = min(result[0], root.val)
            
        self.f(root.left,result,rootval)
        self.f(root.right,result,rootval)
        
    def findSecondMinimumValue(self, root: Optional[TreeNode]) -> int:
        result=[2**64]
        self.f(root,result,root.val)
        if result[0] == 2**64:
            return -1
        return result[0]
```

235. Lowest Common Ancestor of a Binary Search Tree
二叉搜索树找公共父节点


如果两个节点值都小于根节点，说明他们都在根节点的左子树上，我们往左子树上找
如果两个节点值都大于根节点，说明他们都在根节点的右子树上，我们往右子树上找
如果一个节点值大于根节点，一个节点值小于根节点，说明他们他们一个在根节点的左子树上一个在根节点的右子树上，那么根节点就是他们的最近公共祖先节点。

```py
class Solution:
    def lowestCommonAncestor(self, root: 'TreeNode', p: 'TreeNode', q: 'TreeNode') -> 'TreeNode':
        if not root:
            return None
        
        if p.val > q.val:
            temp =p
            p=q
            q=temp
            
        if p.val < root.val and q.val < root.val:
            return self.lowestCommonAncestor(root.left,p,q)
            
        if p.val > root.val and q.val > root.val:
            return self.lowestCommonAncestor(root.right,p,q)
        return root
```


2096. Step-By-Step Directions From a Binary Tree Node to Another
找二叉树中两个节点的路径
向父节点走一格 打印U
否则打印R 或 L


```py
class Solution:
    def f(self,root,direction, st,r1,r2,start,dest):
        if not root:
            return
        
        st.append((direction,root.val))
        #print(st,root.val)
        if root.val == start:
             r1[:] = st.copy()
        if root.val == dest:
             r2[:] = st.copy()
                
        
        if r1 and r2:
            return
        
        self.f(root.left,"L",st,r1,r2,start,dest)
        self.f(root.right,"R",st,r1,r2,start,dest)

        st.pop()
                
        
    def getDirections(self, root: Optional[TreeNode], startValue: int, destValue: int) -> str:
        r1,r2,st=[],[],[]
        self.f(root, "",st,r1,r2,startValue,destValue)
        left=""
        right =""
        while r1 and r2 and r1[0] ==r2[0]:
            a =r1.pop(0)
            b= r2.pop(0)
        
        while r1:
            r1.pop()
            left = left+'U'        
        while r2:
            t = r2.pop(0)
            right = right + t[0]
    
        return left+right
```


438. Find All Anagrams in a String
字谜

Input: s = "cbaebabacd", p = "abc"
Output: [0,6]
Explanation:
The substring with start index = 0 is "cba", which is an anagram of "abc".
The substring with start index = 6 is "bac", which is an anagram of "abc".
看有没有子串是目标子串的重排列

counter 的值可以直接比对, 比比对字符串效率高


```py
class Solution:
     def findAnagrams(self, s, p):
        result = []
        dicp=  Counter(p)
        dics = Counter(s[:len(p)])
        if dicp == dics:
            result.append(0)
            
        for i in range(1, len(s)-len(p)+1):
            dics[s[i-1]]-=1
            if dics[s[i-1]] ==0:
                del dics[s[i-1]]
            dics[s[i+len(p)-1]]+=1
            if dicp == dics:
                result.append(i)
        return result
```



287. Find the Duplicate Number


找到重复的一个数字

Input: nums = [1,3,4,2,2]
Output: 2
快慢指针

从0 开始也不对

```py
class Solution:
    def findDuplicate(self, nums: List[int]) -> int:
        i = j =0
        while nums[i] ==i:
            i+=1
        i = nums[i]
        j = nums[j]
        j =nums[j]
        start = i
        while  nums[i] != nums[j]:
            i = nums[i]
            j =nums[j]
            j =nums[j]
        i = 0
        while nums[i] != nums[j]:
            i = nums[i]
            j = nums[j]
            
        return nums[i]
```


# 迪卡尓积
# for element in itertools.product(*l):
    #print(element)

1980. Find Unique Binary String



长度为2 但是不存在在数组中得数:
Input: nums = ["01","10"]
Output: "11"


长度为3 但是不存在在数组中得数:
Input: nums = ["111","011","001"]
Output: "101"
Explanation: "101" does not appear in nums. "000", "010", "100", and "110" would also be correct.


```py
class Solution:
    def findDifferentBinaryString(self, nums: List[str]) -> str:
        a=['0','1']
        r = [a]*len(nums)
        for l in itertools.product(*r):
            t = ''.join(l)
            if  t not in nums:
                return t
```


448. Find All Numbers Disappeared in an Array
找一个数组中没有的数字
Input: nums = [4,3,2,7,8,2,3,1]
Output: [5,6]

数组长度是n,  然后数组中的数字原本应该是1~n,现在有的数字重复了 但是有的数字没有

找出那些没有的 

每个数加上一个长度,然后找出那些没加的
太妙了

```py
class Solution:
    def findDisappearedNumbers(self, nums: List[int]) -> List[int]:
        nums.insert(0,len(nums)+1)
        for i in nums[1:]:
            nums[i]+= len(nums)
            
        return [i for i in range(len(nums)) if nums[i]<len(nums)]
```


1980. Find Unique Binary String
找不存在的排列序列

Input: nums = ["01","10"]
Output: "11"
长度是2, 每个字符串的长度就也为2, 都是一致的
然后 
00 01 10 11 中给出的数组只有两个, 给出数组中没有的直接返回


### itertools.product(*r) 用法

```py
class Solution:
    def findDifferentBinaryString(self, nums: List[str]) -> str:
        a=['0','1']
        r = [a]*len(nums)
        for l in itertools.product(*r):
            t = ''.join(l)
            if  t not in nums:
                return t
```


300. Longest Increasing Subsequence
最长递增子序列
```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if len(nums) ==1:
            return 1
        dp = [0]*(len(nums))
        dp[0]=1
        mx = -2**64
        for i in range(1, len(nums)):
            c= i-1
            while c>=0: # 写这么多, 就是 找事 , 不如直接用函数式编程
                if nums[c] == nums[i]:
                    dp[i] =dp[c]
                    break
                if nums[c] < nums[i]:
                    dp[i] =dp[c] + 1
                    break
                c-=1
            if dp[i] ==0:
                dp[i]=1
            mx = max(mx, dp[i])
        print(dp)
        return  mx
```       


```py
class Solution:
    def lengthOfLIS(self, nums: List[int]) -> int:
        if len(nums) ==1:
            return 1
        dp = [0]*(len(nums))
        dp[0]=1
        mx = -2**64
        for i in range(1, len(nums)):
            l = [dp[k] for k in range(len(nums[:i])) if nums[k] < nums[i]]
            if len(l) ==0:
                dp[i]=1
            else:
                dp[i] = max(l)+1

        return max(dp)

```


334. Increasing Triplet Subsequence

Input: nums = [1,2,3,4,5]
Output: true
Explanation: Any triplet where i < j < k is valid.




firstMin 总是迄今为止最小的值, 先维护firstMin, 然后维护secondMin , 如果有比secondMin大的,就直接返回True
```py
class Solution:
    def increasingTriplet(self, nums: List[int]) -> bool:
        firstMin  = 2**64
        secondMin = 2**64+1
        
        
        for i in  nums:
            if i > secondMin:
                return True
            if firstMin >= i:                       # firstMin 总是迄今为止最小的值, 先维护firstMin,
                firstMin = i
            else:                                   #然后维护secondMin 
                if secondMin >i:
                    secondMin = i
        return False
```

712. Minimum ASCII Delete Sum for Two Strings
两个字符串的子序列相等需要删除的字符的总和
Input: s1 = "sea", s2 = "eat"
Output: 231

explain: ord(s) +ord(t)
删除掉s 和t 以后两个字符串相等
```py
class Solution(object):
    def minimumDeleteSum(self, s1, s2):
        dp = [[0] * (len(s2) + 1) for _ in range(len(s1) + 1)]

        for i in range(len(s1) - 1, -1, -1):
            dp[i][len(s2)] = dp[i+1][len(s2)] + ord(s1[i]) #   其中一个字符串是空串 就要把另外一个全部都删除掉
        for j in range(len(s2) - 1, -1, -1):
            dp[len(s1)][j] = dp[len(s1)][j+1] + ord(s2[j])

        for i in range(len(s1) - 1, -1, -1):
            for j in range(len(s2) - 1, -1, -1):
                if s1[i] == s2[j]:
                    dp[i][j] = dp[i+1][j+1]
                else:
                    dp[i][j] = min(dp[i+1][j] + ord(s1[i]),
                                   dp[i][j+1] + ord(s2[j]))

        return dp[0][0]
```

322. Coin Change
背包问题,求几个数相加的目标数, 每个数可以用无数次
类似丑数的问题
dp[coin[i]] =1
dp[i] = min(dp[i-coin[1..n]])+1



```py
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        if amount ==0:
            return 0
        inf,dp =2**64,{}
        for i in coins:
            dp[i] =1

        for i in range(1, amount+1):
            mi = inf
            if i not in dp:
                dp[i] = inf
            for c in coins:
                if i-c in dp and i >= c :
                    mi = min(dp[i-c],mi)

            dp[i] = min(dp[i],mi+1)

        return -1 if dp[amount] >=inf else dp[amount]
```


bfs
```py
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        if amount==0:
            return 0
        queue  = deque([(0,0)])
        dic ={}
        while queue:
            value ,step = queue.popleft()
            for i in coins:
                t = value + i
                if t == amount:
                       return step +1
                if t>amount:
                       continue
                if t < amount:
                    if t not in dic:
                        queue.append((t,step+1))
                        dic[t]=1
                    
        return -1
```


983. Minimum Cost For Tickets
Input: days = [1,4,6,7,8,20], costs = [2,7,15]
days表示第几天要出行, costs表示日票周票月票,求最小花费
比如第一天要出行,买个日票, 然后第四天买个周票,第20天买个日票共花费11块钱


状态转移方程:
dp[i] 表示第i天有要出行共花费了多少钱
dp[i] = 
dp[i-1]+costs[0] + # 昨天之前共花了dp[i-1] 块钱  今天还要买个日票
dp[i-7]+costs[1] + # 七天前一共花了dp[i - 7] 块钱, 然后买了个周票 7天内不用再花钱
dp[i-30]+costs[0] + # 同理


如果当天某一天不用出行 dp[i] =dp[i-1] 要继承之前的钱


```py
class Solution:
    def mincostTickets(self, days: List[int], costs: List[int]) -> int:
        dp=[0]*366
        for i in range(1,366):
            if i > days[-1]:
                break
            if i not in days:
                dp[i] =dp[i-1]
                continue
            dp[i] = min(dp[i-1] + costs[0],dp[max(0,i-7)] +costs[1],dp[max(0,i-30)] +costs[2])
        #print(dp)
        return dp[days[-1]]
```

326. Power of Three

```py
class Solution:
    def isPowerOfThree(self, n: int) -> bool:
        return n > 0 and 3486784401 % n == 0
```



350. Intersection of Two Arrays II
counter 求交集
(a & b).elements()
```py
class Solution:
    def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
            a, b = map(collections.Counter, (nums1, nums2))
            return list((a & b).elements())

```

1002. Find Common Characters

忘了使用这个技巧

```py
class Solution:
    def commonChars(self, words: List[str]) -> List[str]:
        d= Counter(words[0])
        for w in words[1:]:
            d&= Counter(w)

        return list(d.elements())
```


2215. Find the Difference of Two Arrays
两个数组 找出a有b没有的, 和b有a没有的
不包括重复元素
如果要包括,就用counter

```py
class Solution:
    def findDifference(self, nums1: List[int], nums2: List[int]) -> List[List[int]]:
        a= set(nums1) 
        b= set(nums2) 
        return [list((a-b)),list((b-a))]
```


380. Insert Delete GetRandom O(1)
random采样

```py
return random.sample(self.s, 1)[0]
```

random.shuffle(self.shuffled)

384. Shuffle an Array

```py
    def __init__(self, nums: List[int]):
        self.origin = nums.copy()
        self.shuffled = nums.copy()
        

    def reset(self) -> List[int]:
        self.shuffled = self.origin.copy()
        return self.origin

    def shuffle(self) -> List[int]:
        random.shuffle(self.shuffled)
        return self.shuffled
```


395. Longest Substring with At Least K Repeating Characters

Input: s = "aaabb", k = 3
Output: 3

找出一个子串, 每个字母都至少重复k次
先counter一次, 找出少于k次的字符,从这几个中间断开

```py
class Solution:
    def longestSubstring(self, s: str, k: int) -> int:
        if len(s) ==0:
            return 0
        if k ==0:
            return len(s)
        if len(s) <k:
            return 0
        
        dic = Counter(s)
        c = min(dic,key=dic.get)
        if dic[c] >= k:
            return len(s)
        ss = s.split(c)
        maxlength = 0
        for temp in ss:
            maxlength =max(self.longestSubstring(temp,k), maxlength)
        return maxlength
```


412. Fizz Buzz
```py
class Solution:
    def fizzBuzz(self, n: int) -> List[str]:
        return ['Fizz' * (not i % 3) + 'Buzz' * (not i % 5) or str(i) for i in range(1, n+1)]
```


454. 4Sum II
函数式二重循环

总结，看到形如：A+B....+N=0的式子，要转换为(A+...T)=-((T+1)...+N)再计算，这个T的分割点一般是一半，特殊情况下需要自行判断。定T是解题的关键。

```py
class Solution:
    def fourSumCount(self, nums1: List[int], nums2: List[int], nums3: List[int], nums4: List[int]) -> int:
        AB = Counter([i+j for i in nums1 for j in nums2])
        return sum([AB[-i-j] for i in nums3 for j in nums4])
```



18. 4Sum
4sum
双指针双指双指针双指针针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针双指针
数组选出4个数 加起来等于target

Input: nums = [1,0,-1,0,-2,2], target = 0
Output: [[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

```py
class Solution:
    def f(self,nums,target,i,j,d):
        t = target - nums[i]- nums[j]
        a = i + 1
        b= j - 1
        res=[]
        while a < b:
            #print(i,j,a,b,nums[a] , nums[b] , t)
            if nums[a] + nums[b] == t:
                l = [nums[i],nums[a],nums[b],nums[j]]
                key = ''.join([str(i) for i in l])
                if key not in d:
                    d[key]=1
                    res.append(l)
                a+=1
                b-=1
            elif nums[a] + nums[b] < t:
                a+=1
            else:
                b-=1
        return res
          
                
    def fourSum(self, nums: List[int], target: int) -> List[List[int]]:
        nums.sort()
        res,d =[],{}
        i,j = 0, len(nums)-1
        res=[]
        for i in range(len(nums)-1):
            for j in range(i+1,len(nums)):
                res.extend(self.f(nums,target,i,j,d))
            
        return res
```



1661. Average Time of Process per Machine
easy
每台机器平均处理程序的时间
不是连续两两配对的 必须各自查出来
Activity table:
+------------+------------+---------------+-----------+
| machine_id | process_id | activity_type | timestamp |
+------------+------------+---------------+-----------+
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |
| 1          | 1          | start         | 0.430     |
| 1          | 1          | end           | 1.420     |
| 2          | 0          | start         | 4.100     |
| 2          | 0          | end           | 4.512     |
| 2          | 1          | start         | 2.500     |
| 2          | 1          | end           | 5.000     |
+------------+------------+---------------+-----------+
Output: 
+------------+-----------------+
| machine_id | processing_time |
+------------+-----------------+
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |
+------------+-----------------+

```sql
select machine_id ,round(avg(diff),3) as processing_time from (
    select b.* , b.timestamp - a.timestamp as diff from 
    (select  * from Activity where activity_type='start') a
    inner join
    (select  * from Activity where activity_type='end') b
    on a.machine_id =b.machine_id and a.process_id = b.process_id 
    
) b group by machine_id
```