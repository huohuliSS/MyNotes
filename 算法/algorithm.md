# algorithm

## 面试题

### 字符串反转

```java
public static String reverse1(String s) {
        int length = s.length();
        if (length <= 1) {
            return s;
        }
        String left = s.substring(0, length / 2);
        String right = s.substring(length / 2, length);
        return reverse1(right) + reverse1(left);  //调用递归
    }
```

### TOP K问题（堆排）

> 给定一个非空的整数数组，返回其中出现频率前 **k** 高的元素。
>
> **示例 1:**
>
> ```
> 输入: nums = [1,1,1,2,2,3], k = 2
> 输出: [1,2]
> ```

```java
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        Map<Integer,Integer> map = new HashMap();
        for(int num : nums) {
            if(map.containsKey(num)) {
                map.put(num,map.get(num) + 1);
            }else {
                map.put(num,1);
            }
        }

//   用小顶堆进行排序（堆排）
        // PriorityQueue<Integer> pq = new PriorityQueue<>((a,b)->{return map.get(a)-map.get(b);});
        // for (Integer num : map.keySet()) {
        //     if(pq.size() < k) {
        //         pq.add(num);
        //     }else if(map.get(num) > map.get(pq.peek())) {
        //         pq.remove();
        //         pq.add(num);
        //     }
        // }
        // List<Integer> list = new ArrayList();
        // while(!pq.isEmpty()) {
        //     list.add(pq.poll());
        // }
        // return list;

        // 用桶排序实现
        List<Integer>[] list = new List[nums.length + 1];
        for(int key : map.keySet()) {
            // 获取出现次数的下标
            int i = map.get(key);
            if(list[i] == null) {
                list[i] = new ArrayList();
            }
            list[i].add(key);
        }

        List<Integer> res = new ArrayList();
        for(int i = list.length - 1; res.size() < k; i--) {
            if(list[i] == null) continue;
            res.addAll(list[i]);
        }
        return res;

    }
}
```



### 桶排序

给定一个字符串，请将字符串里的字符按照出现的频率降序排列。

> 示例 1:
>
> 输入:
> "tree"
>
> 输出:
> "eert"
>
> 解释:
> 'e'出现两次，'r'和't'都只出现一次。
> 因此'e'必须出现在'r'和't'之前。此外，"eetr"也是一个有效的答案。

```java
class Solution {
    public static String frequencySort(String s) {
        char[] ch = s.toCharArray();
        Map<Character,Integer> map = new HashMap();
        for(int i = 0; i < ch.length; i++) {
            if(map.containsKey(ch[i])) {
                map.put(ch[i],map.get(ch[i]) + 1);
            }else{
                map.put(ch[i],1);
            }
        }

        List<Character>[] list = new List[s.length() + 1];
        for (Character key : map.keySet()) {
                if(list[map.get(key)] == null) {
                    list[map.get(key)] = new ArrayList();
                }
                list[map.get(key)].add(key);
        }

        StringBuilder sd = new StringBuilder();
        for (int i = s.length(); i > 0; i--) {
            if (list[i] == null) continue;
            for (int j = 0; j < list[i].size(); j++) {
                for (int z = 0; z < i; z++) {
                    sd.append(list[i].get(j));
                }
            }
        }
        return sd.toString();
    }
}
```



### 荷兰国旗问题

给定一个包含红色、白色和蓝色，一共 n 个元素的数组，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

此题中，我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

```java
class Solution {
    public void sortColors(int[] nums) {
        int red = 0, white = 0, blue = nums.length - 1;
        while (white <= blue) {
            if(nums[white] == 0) {
                swap(nums,red++,white++);
            }else if(nums[white] == 2) {
                swap(nums,white,blue--);
            }else {
                white++;
            }
        }
    }

    public void swap(int[] nums, int start, int end){
        int temp = nums[start];
        nums[start] = nums[end];
        nums[end] = temp;
    }
}
```

### 加一

> 给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。
>
> 最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。
>
> 你可以假设除了整数 0 之外，这个整数不会以零开头。
>
> 示例 1:
>
> 输入: [1,2,3]
> 输出: [1,2,4]
> 解释: 输入数组表示数字 123。
> 示例 2:
>
> 输入: [4,3,2,1]
> 输出: [4,3,2,2]
> 解释: 输入数组表示数字 4321。

```java
class Solution {
    public int[] plusOne(int[] digits) {
        for(int i = digits.length - 1; i >= 0; i--) {
            digits[i]++;
            digits[i] = digits[i] % 10;
            if(digits[i] != 0) {
                return digits;
            }
        }
        digits = new int[digits.length + 1];
        digits[0] = 1;
        return digits;
    }
}
```

### 非递减数列

给定一个长度为 n 的整数数组，你的任务是判断在最多改变 1 个元素的情况下，该数组能否变成一个非递减数列。

我们是这样定义一个非递减数列的： 对于数组中所有的 i (1 <= i < n)，满足 array[i] <= array[i + 1]。

> 示例 1:
>
> 输入: [4,2,3]
> 输出: True
> 解释: 你可以通过把第一个4变成1来使得它成为一个非递减数列。
> 示例 2:
>
> 输入: [4,2,1]
> 输出: False
> 解释: 你不能在只改变一个元素的情况下将其变为非递减数列。

```java
class Solution {
    public boolean checkPossibility(int[] nums) {
        int count = 0;
        for(int i = 1; i < nums.length && count < 2; i++) {
            if(nums[i] >= nums[i - 1]) {
                continue;
            }
            count++;
            if(i - 2 >= 0 && nums[i - 2] > nums[i]){
                nums[i] = nums[i - 1];
            }else {
                nums[i - 1] = nums[i];
            }
        }
        return count < 2;
    }
}
```

### 合并两个有序链表

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

示例：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {

        if(l1 == null) {
            return l2;
        }
        if(l2 == null) {
            return l1;
        }
        if(l1.val < l2.val) {
            l1.next = mergeTwoLists(l1.next,l2);
            return l1;
        }else {
            l2.next = mergeTwoLists(l1, l2.next);
            return l2;
        }
    }
}
```



## 算法思想

### 双指针

> 给定一个非空字符串 s，最多删除一个字符。判断是否能成为回文字符串。
>
> 示例 1:
>
> 输入: "aba"
> 输出: True
> 示例 2:
>
> 输入: "abca"
> 输出: True
> 解释: 你可以删除c字符。

```java
class Solution {
    public boolean validPalindrome(String s) {
        int left = 0;
        int right = s.length()-1;
        while(left < right){
            if(s.charAt(left) != s.charAt(right)) {
                return isPalindrome(s,left+1,right) || isPalindrome(s,left,right -1);
            }
            left++;
            right--;
        }
        return true;
    }

    private boolean isPalindrome(String s,int left, int right){
        while(left < right){
            if(s.charAt(left++) != s.charAt(right--)) {
                return false;
            }
        }
        return true;
    }
}
```



### 排序（8种基本排序、桶排序）

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

> 示例 1:
>
> 输入: [3,2,1,5,6,4] 和 k = 2
> 输出: 5
> 示例 2:
>
> 输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
> 输出: 4

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        for(int val : nums) {
            pq.add(val);
            if(pq.size() > k) {
                pq.poll();
            }
        }
        return pq.peek();
    }
}
```

给定一个非空的整数数组，返回其中出现频率前 **k** 高的元素。

> 示例 1:
>
> 输入: nums = [1,1,1,2,2,3], k = 2
> 输出: [1,2]
> 示例 2:
>
> 输入: nums = [1], k = 1
> 输出: [1]

```java
class Solution {
    public List<Integer> topKFrequent(int[] nums, int k) {
        Map<Integer,Integer> map = new HashMap();
        for(int num : nums) {
            if(map.containsKey(num)) {
                map.put(num,map.get(num) + 1);
            }else {
                map.put(num,1);
            }
        }

//   用小顶堆进行排序（堆排）
        // PriorityQueue<Integer> pq = new PriorityQueue<>((a,b)->{return map.get(a)-map.get(b);});
        // for (Integer num : map.keySet()) {
        //     if(pq.size() < k) {
        //         pq.add(num);
        //     }else if(map.get(num) > map.get(pq.peek())) {
        //         pq.remove();
        //         pq.add(num);
        //     }
        // }
        // List<Integer> list = new ArrayList();
        // while(!pq.isEmpty()) {
        //     list.add(pq.poll());
        // }
        // return list;

        // 用桶排序实现
        List<Integer>[] list = new List[nums.length + 1];
        for(int key : map.keySet()) {
            // 获取出现次数的下标
            int i = map.get(key);
            if(list[i] == null) {
                list[i] = new ArrayList();
            }
            list[i].add(key);
        }

        List<Integer> res = new ArrayList();
        for(int i = list.length - 1; res.size() < k; i--) {
            if(list[i] == null) continue;
            res.addAll(list[i]);
        }
        return res;
    }
}
```



### 贪心思想

> 贪心算法总是作出在当前看来最好的选择，也就是说贪心算法并不从整体最优考虑，它所作出的选择只是在某种意义上的**局部最优选择**。

- 给定一个区间的集合，找到需要移除区间的最小数量，使剩余区间互不重叠。

> 注意:
>
> ​		可以认为区间的终点总是大于它的起点。
> ​		区间 [1,2] 和 [2,3] 的边界相互“接触”，但没有相互重叠。
>
> 示例 1:
>
> 输入: [ [1,2], [2,3], [3,4], [1,3] ]
>
> 输出: 1
>
> 解释: 移除 [1,3] 后，剩下的区间没有重叠。
> 示例 2:
>
> 输入: [ [1,2], [1,2], [1,2] ]
>
> 输出: 2
>
> 解释: 你需要移除两个 [1,2] 来使剩下的区间没有重叠。
>

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        if(intervals.length == 0) return 0;
        // 按照end升序排序
        Arrays.sort(intervals,new Comparator<int[]>(){
            public int compare(int[] a, int[] b) {
                return a[1] - b[1];
            }
        });

        //至少有一个区间不想交
        int count =1;
        // 排序后，第一个区间就是x
        int x_end = intervals[0][1];
        for(int[] intv : intervals) {
            int start = intv[0];
            if(start >= x_end) {
                count++;
                x_end = intv[1];
            }
        }
        return intervals.length - count;
    }
}
```

- 假设有打乱顺序的一群人站成一个队列。 每个人由一个整数对(h, k)表示，其中h是这个人的身高，k是排在这个人前面且身高大于或等于h的人数。 编写一个算法来重建这个队列。

> 注意：
> 总人数少于1100人。
>
> 示例
>
> 输入:
> [[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]
>
> 输出:
> [[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people,(o1,o2)->{
            return o1[0] == o2[0]? o1[1] - o2[1]: o2[0] - o1[0];});
        
        List<int[]> list = new ArrayList();
        for(int[] p : people) {
            list.add(p[1],p);
        }
        return list.toArray(new int[list.size()][]);
    }
}
```

- 在二维空间中有许多球形的气球。对于每个气球，提供的输入是水平方向上，气球直径的开始和结束坐标。由于它是水平的，所以y坐标并不重要，因此只要知道开始和结束的x坐标就足够了。开始坐标总是小于结束坐标。平面内最多存在104个气球。

  一支弓箭可以沿着x轴从不同点完全垂直地射出。在坐标x处射出一支箭，若有一个气球的直径的开始和结束坐标为 xstart，xend， 且满足  xstart ≤ x ≤ xend，则该气球会被引爆。可以射出的弓箭的数量没有限制。 弓箭一旦被射出之后，可以无限地前进。我们想找到使得所有气球全部被引爆，所需的弓箭的最小数量。

> Example:
>
> 输入:
> [[10,16], [2,8], [1,6], [7,12]]
>
> 输出:
> 2
>
> 解释:
> 对于该样例，我们可以在x = 6（射爆[2,8],[1,6]两个气球）和 x = 11（射爆另外两个气球）。

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        if(points.length < 2) {
            return points.length;
        }
        Arrays.sort(points,new Comparator<int[]>(){
            public int compare(int[] a, int[] b){
                if(a[0] != b[0]) {
                    return a[0] - b[0];
                }
                return a[1] - a[1];
            }
        });

        int minCount = 1;
        int raw = points[0][1];
        // 贪心法，基于上一个箭，记录当前能够射穿的所有区间
        for(int i = 1; i < points.length; i++) {
            if(raw < points[i][0]) {
                raw = points[i][1];
                minCount++;
            }else{
                raw = Math.min(raw, points[i][1]);
            }
        }

        return minCount;  
    }
}
```

- 给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

  设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）

> 示例 1:
>
> 输入: [7,1,5,3,6,4]
> 输出: 7
> 解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
>      随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。

```java
class Solution {
    public int maxProfit(int[] prices) {
        int num = 0;
        for(int i = 1; i < prices.length; i++) {
            if(prices[i - 1] < prices[i]) {
                num += prices[i] - prices[i - 1];
            }
        }
        return num;
    }
}
```

- 假设你有一个很长的花坛，一部分地块种植了花，另一部分却没有。可是，花卉不能种植在相邻的地块上，它们会争夺水源，两者都会死去。

  给定一个花坛（表示为一个数组包含0和1，其中0表示没种植花，1表示种植了花），和一个数 n 。能否在不打破种植规则的情况下种入 n 朵花？能则返回True，不能则返回False。

> 示例 1:
>
> 输入: flowerbed = [1,0,0,0,1], n = 1
> 输出: True
> 示例 2:
>
> 输入: flowerbed = [1,0,0,0,1], n = 2
> 输出: False

```java
class Solution {
    public boolean canPlaceFlowers(int[] flowerbed, int n) {
       int len = flowerbed.length;
       int count = 0;
       for(int i = 0; i < len && count < n; i++) {
           int pre = (i == 0)? 0 : flowerbed[i - 1];
           int next = (i == len - 1)? 0 : flowerbed[i + 1];
           if(flowerbed[i] == 0 && pre == 0 && next == 0) {
               count++;
               flowerbed[i] = 1;
           }
       }
       return count >= n;
    }
}
```

- 字符串 `S` 由小写字母组成。我们要把这个字符串划分为尽可能多的片段，同一个字母只会出现在其中的一个片段。返回一个表示每个字符串片段的长度的列表。

> 示例 1:
>
> 输入: S = "ababcbacadefegdehijhklij"
> 输出: [9,7,8]
> 解释:
> 划分结果为 "ababcbaca", "defegde", "hijhklij"。
> 每个字母最多出现在一个片段中。
> 像 "ababcbacadefegde", "hijhklij" 的划分是错误的，因为划分的片段数较少。

```java
class Solution {
    public List<Integer> partitionLabels(String S) {

        // 贪心
        int[] arr = new int[26];
        for(int i = 0; i < S.length(); i++) {
            arr[S.charAt(i) - 'a'] = i;
        }
        List<Integer> list = new ArrayList();
        int end = 0, start = 0;
        for(int j = 0; j < S.length(); j++) {
            end = Math.max(arr[S.charAt(j) - 'a'],end);
            if(j == end) {
                list.add(end - start + 1);
                start = j + 1;
            }
        }
        return list;

        // List<Integer> list = new ArrayList<>();
        // int start = 0;
        // int index = S.lastIndexOf(S.charAt(start));
        // while (index < S.length()) {
        //     int left = start;
        //     while (left < index) {
        //         left++;
        //         String str = S.substring(index + 1);
        //         if(str.contains(S.charAt(left) + "")) {
        //             index = S.lastIndexOf(S.charAt(left));
        //         }
        //     }
        //     list.add(index - start + 1);
        //     if(index == S.length() - 1) {
        //         break;
        //     }
        //     start = index + 1;
        //     index = S.lastIndexOf(S.charAt(start));
        // }
        // return list;
    }
}
```

### 二分查找

给定一个只包含小写字母的有序数组letters 和一个目标字母 target，寻找有序数组里面比目标字母大的最小字母。

数组里字母的顺序是循环的。举个例子，如果目标字母target = 'z' 并且有序数组为 letters = ['a', 'b']，则答案返回 'a'。

```java
class Solution {
    public char nextGreatestLetter(char[] letters, char target) {

        if(letters[0]-target > 0 || letters[letters.length-1]-target <= 0) {
            return letters[0];
        }
        // 二分法
        int n = letters.length;
        int l = 0, h = n - 1;
        while(l <= h) {
            int m = (h + l) / 2;
            if(letters[m] <= target) {
                l = m + 1;
            }else {
                h = m - 1;
            }
        }
        return letters[l];

        // 普通解决办法
        // int index = 0;
        // for(char c : letters) {
        //     if(c - target > 0) {
        //         break;
        //     }else if(c - target == 0) {
        //         index++;
        //         continue;
        //     }
        //     index++;
        // }
        // return letters[index];
    }
}
```

假设按照升序排序的数组在预先未知的某个点上进行了旋转。

( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

请找出其中最小的元素。

你可以假设数组中不存在重复元素。

```java
class Solution {
    public int findMin(int[] nums) {
        // Arrays.sort(nums);
        // return nums[0];

//        二分法
        int len = nums.length;
        int start = 0, end = len - 1;
        while(start < end) {
            int mid = start + (end - start) / 2;
            if(nums[mid] > nums[end]) {
                start = mid + 1;
            }else {
                end = mid;
            }
        }
        return nums[start];
    }
}
```



### 分治

### 搜索

### 动态规划

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意：你不能在买入股票前卖出股票。

> 示例 1:
>
> 输入: [7,1,5,3,6,4]
> 输出: 5
> 解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
>      注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
> 示例 2:
>
> 输入: [7,6,4,3,1]
> 输出: 0
> 解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

```java
class Solution {
    public int maxProfit(int[] prices) {
        // 动态规划
        int max = 0, min = Integer.MAX_VALUE;
        for(int i = 0; i < prices.length; i++) {
            if(prices[i] < min) {
                min = prices[i];
            }else if(prices[i] - min > max) {
                max = prices[i] - min;
            }
        }
        return max;
        // 暴力解决
        // int end = prices.length - 1;
        // int max = 0;
        // for(int i = 0; i < end; i++) {

        //     for(int j = end; j > i; j--) {
        //         if(prices[i] > prices[j]) {
        //             continue;
        //         }
        //         max = Math.max(max, prices[j] - prices[i]);
        //     }

        // }
        // return max;
    }
}
```

给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

> **示例:**
>
> ```
> 输入: [-2,1,-3,4,-1,2,1,-5,4],
> 输出: 6
> 解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
> ```

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int sum = nums[0];
        if(nums.length <= 1) {return sum;}
        int maxSum = sum;
        for(int i = 1; i < nums.length; i++) {
            if(sum <= 0) {
                sum = nums[i];
            }else {
                sum += nums[i];
            }
            if(sum > maxSum) {
                maxSum = sum;
            }
        }
        return maxSum;
    }
}
```

#### 斐波那契问题

你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。

给定一个代表每个房屋存放金额的非负整数数组，计算你在不触动警报装置的情况下，能够偷窃到的最高金额。

> 示例 1:
>
> 输入: [1,2,3,1]
> 输出: 4
> 解释: 偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
>      偷窃到的最高金额 = 1 + 3 = 4 。
> 示例 2:
>
> 输入: [2,7,9,3,1]
> 输出: 12
> 解释: 偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
>      偷窃到的最高金额 = 2 + 9 + 1 = 12 。

```java
public int rob(int[] num) {
   // f(k) = max(f(k-2) + Ai, f(k-1))
    int prevMax = 0;
    int currMax = 0;
    for (int x : num) {
        int temp = currMax;
        currMax = Math.max(prevMax + x, currMax);
        prevMax = temp;
    }
    return currMax;
}
```


复杂度分析

时间复杂度：O(n)O(n)。其中 nn 为房子的数量。
空间复杂度：O(1)O(1)。

### 数学