---
title: leetcode 习题解析
urlname: algo_leetcode
#date: 2016-10-09
comments: true
top: true
#description: 
#tags: [git,hah,hha]
#keywords: git
categories: 03JVM&ART
tags:
    - java
    - leetcode
    - algo
---

## 1. Two Sum

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have ***exactly*** one solution, and you may not use the same element twice.

**Example:**

``` 
Given nums = [2, 7, 11, 15], target = 9,

return [0, 1].
Because nums[0] + nums[1] = 2 + 7 = 9,
```

**Tags:** Array, Hash Table

思路 0

题意是让你从给定的数组中找到两个元素的和为指定值的两个索引，最容易的当然是循环两次，复杂度为 `O(n^2)`，首次提交居然是 2ms，打败了 100% 的提交，谜一样的结果，之后再次提交就再也没跑到过 2ms 了。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for (int i = 0; i < nums.length; ++i) {
            for (int j = i + 1; j < nums.length; ++j) {
                if (nums[i] + nums[j] == target) {
                    return new int[]{i, j};
                }
            }
        }
        return null;
    }
}
```

思路 1

利用 HashMap 作为存储，键为目标值减去当前元素值，索引为值，比如 `i = 0` 时，此时首先要判断 `nums[0] = 2` 是否在 map 中，如果不存在，那么插入键值对 `key = 9 - 2 = 7, value = 0`，之后当 `i = 1` 时，此时判断 `nums[1] = 7` 已存在于 map 中，那么取出该 `value = 0` 作为第一个返回值，当前 `i` 作为第二个返回值，具体代码如下所示。

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int len = nums.length;
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < len; ++i) {
            if (map.containsKey(nums[i])) {
                return new int[]{map.get(nums[i]), i};
            }
            map.put(target - nums[i], i);
        }
        return null;
    }
}
```

<!-- more -->

## 2. Add Two Numbers

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Example**

```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```




```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode node = new ListNode(0);
        ListNode n1 = l1, n2 = l2, t = node;
        int sum = 0;
        while (n1 != null || n2 != null) {
            sum /= 10;
            if (n1 != null) {
                sum += n1.val;
                n1 = n1.next;
            }
            if (n2 != null) {
                sum += n2.val;
                n2 = n2.next;
            }
            t.next = new ListNode(sum % 10);
            t = t.next;
        }
        if (sum / 10 != 0) t.next = new ListNode(1);
        return node.next;
    }
```

## 3. Longest Substring Without Repeating Characters
Given a string, find the length of the longest substring without repeating characters.

Examples:

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.

Tags: Hash Table, Two Pointers, String

思路
题意是计算不带重复字符的最长子字符串的长度，开辟一个 hash 数组来存储该字符上次出现的位置，比如 hash[a] = 3 就是代表 a 字符前一次出现的索引在 3，遍历该字符串，获取到上次出现的最大索引（只能向前，不能退后），与当前的索引做差获取的就是本次所需长度，从中迭代出最大值就是最终答案。


```java
    public int lengthOfLongestSubstring(String s) {
        int[] m = new int[256];
        Arrays.fill(m, -1);
        int res = 0, left = -1;
        for (int i = 0; i < s.length(); ++i) {
            left = Math.max(left, m[s.charAt(i)]);
            m[s.charAt(i)] = i;
            res = Math.max(res, i - left);
        }
        return res;
    }
```

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int len;
        if (s == null || (len = s.length()) == 0) return 0;
        int preP = 0, max = 0;
        int[] hash = new int[128];
        for (int i = 0; i < len; ++i) {
            char c = s.charAt(i);
            if (hash[c] > preP) {
                preP = hash[c];
            }
            int l = i - preP + 1;
            hash[c] = i + 1;
            if (l > max) max = l;
        }
        return max;
    }
}
```


## 4. Median of Two Sorted Arrays

There are two sorted arrays nums1 and nums2 of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

Example 1:

```
nums1 = [1, 3]
nums2 = [2]
```

The median is 2.0

Example 2:

```
nums1 = [1, 2]
nums2 = [3, 4]
```

The median is (2 + 3)/2 = 2.5


思路：归并计数法，时间O(n) 空间O(1)

```java
public class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int len1 = nums1.length;
        int len2 = nums2.length;
        int total = len1 + len2;
        if(total % 2==0){
            return (findKth(nums1,nums2,total/2)+findKth(nums1,nums2,total/2+1))/2.0;
        } else {
            return findKth(nums1,nums2,total/2+1);
        }
    }
    private int findKth(int[] nums1, int[] nums2, int k){
        int p = 0, q = 0;
        for(int i = 0; i < k - 1; i++){
            if(p>=nums1.length && q<nums2.length){
                q++;
            } else if(q>=nums2.length && p<nums1.length){
                p++;
            } else if(nums1[p]>nums2[q]){
                q++;
            } else {
                p++;
            }
        }
        if(p>=nums1.length) {
            return nums2[q];
        } else if(q>=nums2.length) {
            return nums1[p];
        } else {
            return Math.min(nums1[p],nums2[q]);
        }
    }
}
```

分治法 Divide and Conquer 二分

复杂度
时间O(log(m+n)) 空间O(1)

思路，二分，每次删除一半的一半
题目要求O(log(m+n))的时间复杂度，一般来说都是分治法或者二分搜索。首先我们先分析下题目，假设两个有序序列共有n个元素（根据中位数的定义我们要分两种情况考虑），当n为奇数时，搜寻第(n/2+1)个元素，当n为偶数时，搜寻第(n/2+1)和第(n/2)个元素，然后取他们的均值。进一步的，我们可以把这题抽象为“搜索两个有序序列的第k个元素”。如果我们解决了这个k元素问题，那中位数不过是k的取值不同罢了。

那如何搜索两个有序序列中第k个元素呢，这里又有个技巧。假设序列都是从小到大排列，对于第一个序列中前p个元素和第二个序列中前q个元素，我们想要的最终结果是：p+q等于k-1,且一序列第p个元素和二序列第q个元素都小于总序列第k个元素。因为总序列中，必然有k-1个元素小于等于第k个元素。这样第p+1个元素或者第q+1个元素就是我们要找的第k个元素。

所以，我们可以通过二分法将问题规模缩小，假设p=k/2-1，则q=k-p-1，且p+q=k-1。如果第一个序列第p个元素小于第二个序列第q个元素，我们不确定二序列第q个元素是大了还是小了，但一序列的前p个元素肯定都小于目标，所以我们将第一个序列前p个元素全部抛弃，形成一个较短的新序列。然后，用新序列替代原先的第一个序列，再找其中的第k-p个元素（因为我们已经排除了p个元素，k需要更新为k-p），依次递归。同理，如果第一个序列第p个元素大于第二个序列第q个元素，我们则抛弃第二个序列的前q个元素。递归的终止条件有如下几种：

较短序列所有元素都被抛弃，则返回较长序列的第k个元素（在数组中下标是k-1）

一序列第p个元素等于二序列第q个元素，此时总序列第p+q=k-1个元素的后一个元素，也就是总序列的第k个元素

注意
每次递归不仅要更新数组起始位置（起始位置之前的元素被抛弃），也要更新k的大小（扣除被抛弃的元素）


```java
public class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int m = nums1.length, n = nums2.length;
        int k = (m + n) / 2;
        if((m+n)%2==0){
            return (findKth(nums1,nums2,0,0,m,n,k)+findKth(nums1,nums2,0,0,m,n,k+1))/2;
        }   else {
            return findKth(nums1,nums2,0,0,m,n,k+1);
        }
        
    }
    
    private double findKth(int[] arr1, int[] arr2, int start1, int start2, int len1, int len2, int k){
        // 保证arr1是较短的数组
        if(len1>len2){
            return findKth(arr2,arr1,start2,start1,len2,len1,k);
        }
        if(len1==0){
            return arr2[start2 + k - 1];
        }
        if(k==1){
            return Math.min(arr1[start1],arr2[start2]);
        }
        int p1 = Math.min(k/2,len1) ;
        int p2 = k - p1;
        if(arr1[start1 + p1-1]<arr2[start2 + p2-1]){
            return findKth(arr1,arr2,start1 + p1,start2,len1-p1,len2,k-p1);
        } else if(arr1[start1 + p1-1]>arr2[start2 + p2-1]){
            return findKth(arr1,arr2,start1,start2 + p2,len1,len2-p2,k-p2);
        } else {
            return arr1[start1 + p1-1];
        }
    }
}
```


## 5. Longest Palindromic Substring

Given a string s, find the longest palindromic substring in s. You may assume that the maximum length of s is 1000.
```
Example 1:

Input: "babad"
Output: "bab"
Note: "aba" is also a valid answer.

Example 2:

Input: "cbbd"
Output: "bb"
```
思路 中心扩散法 Spread From Center
时间 O(n^2) 空间 O(1)
动态规划虽然优化了时间，但也浪费了空间。实际上我们并不需要一直存储所有子字符串的回文情况，我们需要知道的只是中心对称的较小一层是否是回文。所以如果我们从小到大连续以某点为个中心的所有子字符串进行计算，就能省略这个空间。 这种解法中，外层循环遍历的是子字符串的中心点，内层循环则是从中心扩散，一旦不是回文就不再计算其他以此为中心的较大的字符串。由于中心对称有两种情况，一是奇数个字母以某个字母对称，而是偶数个字母以两个字母中间为对称，所以我们要分别计算这两种对称情况。



```java
public class Solution {
    String longest = "";
    
    public String longestPalindrome(String s) {
        for(int i = 0; i < s.length(); i++){
            //计算奇数子字符串
            helper(s, i, 0);
            //计算偶数子字符串
            helper(s, i, 1);
        }
        return longest;
    }
    
    private void helper(String s, int idx, int offset){
        int left = idx;
        int right = idx + offset;
        while(left>=0 && right<s.length() && s.charAt(left)==s.charAt(right)){
            left--;
            right++;
        }
        // 截出当前最长的子串
        String currLongest = s.substring(left + 1, right);
        // 判断是否比全局最长还长
        if(currLongest.length() > longest.length()){
            longest = currLongest;
        }
    }
}

```


更厉害的还有 马拉车算法 Manacher Algorithm

https://segmentfault.com/a/1190000002991199#articleHeader14


## 6. ZigZag Conversion 

The string "PAYPALISHIRING" is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)
```
P   A   H   N
A P L S I I G
Y   I   R
And then read line by line: "PAHNAPLSIIGYIR"

Write the code that will take a string and make this conversion given a number of rows:

string convert(string s, int numRows);
Example 1:

Input: s = "PAYPALISHIRING", numRows = 3
Output: "PAHNAPLSIIGYIR"
Example 2:

Input: s = "PAYPALISHIRING", numRows = 4
Output: "PINALSIGYAHRPI"
Explanation:

P     I    N
A   L S  I G
Y A   H R
P     I
```

思路 0
题意是让你把字符串按波浪形排好，然后返回横向读取的字符串。

听不懂的话，看下面的表示应该就明白了：

```
0                           2n-2                        4n-4
1                    2n-3   2n-1                 4n-5   4n-3
2              2n-4         2n               4n-6       .
.           .               .             .             .
.       n+1                 .          3n-1             .
n-2   n                     3n-4   3n-2                 5n-6
n-1                         3n-3                        5n-5
```


那么我们可以根据上面找规律，可以看到波峰和波谷是单顶点的，它们周期是 2 * (n - 1)，单独处理即可；中间的部分每个周期会出现两次，规律很好找，留给读者自己想象，不懂的可以结合以下代码。

```java
class Solution {
    public String convert(String s, int numRows) {
        if (numRows <= 1) return s;
        int len = s.length();
        char[] chars = s.toCharArray();
        int cycle = 2 * (numRows - 1);
        StringBuilder sb = new StringBuilder();
        for (int j = 0; j < len; j += cycle) {
            sb.append(chars[j]);
        }
        for (int i = 1; i < numRows - 1; i++) {
            int step = 2 * i;
            for (int j = i; j < len; j += step) {
                sb.append(chars[j]);
                step = cycle - step;
            }
        }
        for (int j = numRows - 1; j < len; j += cycle) {
            sb.append(chars[j]);
        }
        return sb.toString();
    }
}
```




## 7. Reverse Integer
Given a 32-bit signed integer, reverse digits of an integer.
```
Example 1:

Input: 123
Output: 321
Example 2:

Input: -123
Output: -321
Example 3:

Input: 120
Output: 21
```
题意是给你一个整型数，求它的逆序整型数，而且有个小坑点，当它的逆序整型数溢出的话，那么就返回 0，用我们代码表示的话可以求得结果保存在 long 中，最后把结果和整型的两个范围比较即可。
```java
class Solution {
    public int reverse(int x) {
        long res = 0;
        for (; x != 0; x /= 10)
            res = res * 10 + x % 10;
        return res > Integer.MAX_VALUE || res < Integer.MIN_VALUE ? 0 : (int) res;
    }
}
```

```java
public int reverse(int x) {
            long rev = 0;
            while (x != 0) {
                rev = rev * 10 + x % 10;
                x /= 10;
                if (rev > Integer.MAX_VALUE || rev < Integer.MIN_VALUE) {
                    return 0;
                }
            }
            return (int) rev;
        }
```


## 8. String to Integer (atoi)
Implement atoi which converts a string to an integer.

The function first discards as many whitespace characters as necessary until the first non-whitespace character is found. Then, starting from this character, takes an optional initial plus or minus sign followed by as many numerical digits as possible, and interprets them as a numerical value.

The string can contain additional characters after those that form the integral number, which are ignored and have no effect on the behavior of this function.

If the first sequence of non-whitespace characters in str is not a valid integral number, or if no such sequence exists because either str is empty or it contains only whitespace characters, no conversion is performed.

If no valid conversion could be performed, a zero value is returned.

Note:

Only the space character ' ' is considered as whitespace character.
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−231,  231 − 1]. If the numerical value is out of the range of representable values, INT_MAX (231 − 1) or INT_MIN (−231) is returned.
```
Example 1:
Input: "42"
Output: 42

Example 2:
Input: "   -42"
Output: -42
Explanation: The first non-whitespace character is '-', which is the minus sign.Then take as many numerical digits as possible, which gets 42.

Example 3:
Input: "4193 with words"
Output: 4193
Explanation: Conversion stops at digit '3' as the next character is not a numerical digit.
```

思路, 时间 O(n) 空间 O(1)

字符串题一般考查的都是边界条件、特殊情况的处理。所以遇到此题一定要问清楚各种条件下的输入输出应该是什么样的。这里已知的特殊情况有：

能够排除首部的空格，从第一个非空字符开始计算
允许数字以正负号(+-)开头
遇到非法字符便停止转换，返回当前已经转换的值，如果开头就是非法字符则返回0
在转换结果溢出时返回特定值，这里是最大/最小整数
注意
检查溢出时最大整数要先减去即将加的最末位再除以10，来处理"2147483648"类似的情况
可以参考glibc中stdlib/atoi.c的实现方法

```java
public class Solution {
    public int myAtoi(String str) {
        str = str.trim();
        int result = 0;
        boolean isPos = true;
        for(int i = 0; i < str.length(); i++){
            char c = str.charAt(i);
            if(i==0 && (c=='-'||c=='+')){
                isPos = c=='+'?true:false;
            } else if (c>='0' && c<='9'){
                // 检查溢出情况
                if(result>(Integer.MAX_VALUE - (c - '0'))/10){
                    return isPos? Integer.MAX_VALUE : Integer.MIN_VALUE;
                }
                result *= 10;
                result += c - '0';
            } else {
                return isPos?result:-result;
            }
        }
        return isPos?result:-result;
    }
}
```

## 9. Palindrome Number
Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.
```
Example 1:

Input: 121
Output: true

Example 2:

Input: -121
Output: false
Explanation: From left to right, it reads -121. From right to left, it becomes 121-. Therefore it is not a palindrome.

Example 3:

Input: 10
Output: false
Explanation: Reads 01 from right to left. Therefore it is not a palindrome.
```

思路 0
题意是判断一个有符号整型数是否是回文，也就是逆序过来的整数和原整数相同，首先负数肯定不是，接下来我们分析一下最普通的解法，就是直接算出他的回文数，然后和给定值比较即可。
```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0) return false;
        int copyX = x, reverse = 0;
        while (copyX > 0) {
            reverse = reverse * 10 + copyX % 10;
            copyX /= 10;
        }
        return x == reverse;
    }
}
```
思路 1
好好思考下是否需要计算整个长度，比如 1234321，其实不然，我们只需要计算一半的长度即可，就是在计算过程中的那个逆序数比不断除 10 的数大就结束计算即可，但是这也带来了另一个问题，比如 10 的倍数的数 10010，它也会返回 true，所以我们需要对 10 的倍数的数再加个判断即可，代码如下所示。

```java
class Solution {
    public boolean isPalindrome(int x) {
        if (x < 0 || (x != 0 && x % 10 == 0)) return false;
        int halfReverseX = 0;
        while (x > halfReverseX) {
            halfReverseX = halfReverseX * 10 + x % 10;
            x /= 10;
        }
        return halfReverseX == x || halfReverseX / 10 == x;
    }
}
```

## 10. Regular Expression Matching
Implement regular expression matching with support for '.' and'*'.

'.' Matches any single character. '*' Matches zero or more of the preceding element.

The matching should cover the entire input string (not partial).

The function prototype should be:

bool isMatch(const char *s, const char *p)
Some examples:
```
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "a*") → true
isMatch("aa", ".*") → true
isMatch("ab", ".*") → true
isMatch("aab", "c*a*b") → true

Example 3:
Input:
s = "ab"
p = ".*"
Output: true
Explanation: ".*" means "zero or more (*) of any character (.)".

Example 4:
Input:
s = "aab"
p = "c*a*b"
Output: true
Explanation: c can be repeated 0 times, a can be repeated 1 time. Therefore it matches "aab".

Example 5:
Input:
s = "mississippi"
p = "mis*is*p*."
Output: false
```

思路 1
我们可以把上面的思路更简单化，如下：

如果 s 和 p 都为空，那么返回 true；

如果 p 的第二个字符为 *，由于 * 前面的字符个数可以为任意，那么我们先递归调用个数为 0 的情况；或者当 s 不为空，如果他们的首字母匹配，那么我们就递归调用去掉去掉首字母的 s 和完整的 p；

如果 p 的第二个字符不为 *，那么我们就老老实实判断第一个字符是否匹配并且递归调用他们去掉首位的子字符串。

```java

class Solution {
    public boolean isMatch(String s, String p) {
        if (p.isEmpty()) return s.isEmpty();
        if (p.length() > 1 && p.charAt(1) == '*') {
            return isMatch(s, p.substring(2))
                    || (!s.isEmpty() && (p.charAt(0) == s.charAt(0) || p.charAt(0) == '.')
                    && isMatch(s.substring(1), p));
        }
        return !s.isEmpty() && (p.charAt(0) == s.charAt(0) || p.charAt(0) == '.')
                && isMatch(s.substring(1), p.substring(1));
    }
}
```


思路 2
另一种思路就是动态规划了，我们定义 dp[i][j] 的真假来表示 s[0..i) 是否匹配 p[0..j)，通过思路 1，我们可以确定其状态转移方程如下所示：

如果 p[j - 1] == '*', dp[i][j] = dp[i][j - 2] || (pc[j - 2] == sc[i - 1] || pc[j - 2] == '.') && dp[i - 1][j];；

如果 p[j - 1] != '*'，dp[i][j] = dp[i - 1][j - 1] && (pc[j - 1] == '.' || pc[j - 1] == sc[i - 1]);。

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if (p.length() == 0) return s.length() == 0;
        int sL = s.length(), pL = p.length();
        boolean[][] dp = new boolean[sL + 1][pL + 1];
        char[] sc = s.toCharArray(), pc = p.toCharArray();
        dp[0][0] = true;
        for (int i = 2; i <= pL; ++i) {
            if (pc[i - 1] == '*' && dp[0][i - 2]) {
                dp[0][i] = true;
            }
        }
        for (int i = 1; i <= sL; ++i) {
            for (int j = 1; j <= pL; ++j) {
                if (pc[j - 1] == '*') {
                    dp[i][j] = dp[i][j - 2] || (pc[j - 2] == sc[i - 1] || pc[j - 2] == '.') && dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j - 1] && (pc[j - 1] == '.' || pc[j - 1] == sc[i - 1]);
                }
            }
        }
        return dp[sL][pL];
    }
}
```



## 11. Container With Most Water
https://github.com/Blankj/awesome-java-leetcode/blob/master/note/010/README.md

Given *n* non-negative integers *a1*, *a2*, ..., *an*, where each represents a point at coordinate (*i*, *ai*). *n* vertical lines are drawn such that the two endpoints of line *i* is at (*i*, *ai*) and (*i*, 0). Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and *n* is at least 2.

**Tags:** Array, Two Pointers

思路

题意是给你 *a1*, *a2*, ..., *an* 这 *n* 个数，代表 (*i*, *ai*) 坐标，让你从中找两个点与 x 轴围成的容器可以容纳最多的水。

不明白的话可以看数据为 `1 8 6 2 5 4 8 3 7` 所示的图。

![](https://raw.githubusercontent.com/Blankj/awesome-java-leetcode/master/note/011/water.png)

如果用暴力法求每种情况的结果，其时间复杂度为 O(n^2)，相信肯定会超时，我们可以探索下是否有更巧妙的办法呢，题目的标签有双指针，是否就可以想到首尾各放一指针，然后根据条件来收缩。首先计算一次首尾构成的最大面积，然后分析下该移动哪个指针，如果移动大的那个指针的话，那样只会减小面积，所以我们要移动小的那个指针，小的那个指针移动到哪呢？当然是移动到大于之前的值的地方，否则面积不都比之前小么，然后继续更新最大值即可，借助如上分析写出如下代码应该不是什么难事了吧。


```java
class Solution {
    public int maxArea(int[] height) {
        int l = 0, r = height.length - 1;
        int max = 0, h = 0;
        while (l < r) {
            h = Math.min(height[l], height[r]);
            max = Math.max(max, (r - l) * h);
            while (height[l] <= h && l < r) ++l;
            while (height[r] <= h && l < r) --r;
        }
        return max;
    }
}


public int maxArea(int[] height) {
        int maxarea = 0, l = 0, r = height.length - 1;
        while (l < r) {
            maxarea = Math.max(maxarea, Math.min(height[l], height[r]) * (r - l));
            if (height[l] < height[r])
                l++;
            else
                r--;
        }
        return maxarea;
    }
```


## 12. Integer to Roman
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.
```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
For example, two is written as II in Roman numeral, just two one's added together. Twelve is written as, XII, which is simply X + II. The number twenty seven is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

I can be placed before V (5) and X (10) to make 4 and 9.
X can be placed before L (50) and C (100) to make 40 and 90.
C can be placed before D (500) and M (1000) to make 400 and 900.
Given an integer, convert it to a roman numeral. Input is guaranteed to be within the range from 1 to 3999.

Example 1:

Input: 3
Output: "III"
Example 2:

Input: 4
Output: "IV"
Example 3:

Input: 9
Output: "IX"
Example 4:

Input: 58
Output: "LVIII"
Explanation: C = 100, L = 50, XXX = 30 and III = 3.
Example 5:

Input: 1994
Output: "MCMXCIV"
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
```

思路
因为罗马数字都是由最大化的表示，比如10会表示成X而不是VV，所以我们可以从最大可能的表示向小的依次尝试。因为提示1-3999，所有的可能性不多，我们可以直接打表来帮助我们选择表示方法。在一个数量级中有四种可能的表示方法，以1-9为例，1-3是由I表示出来的，4是IV，5是V，6-8是V和若干个I，9是IX。

代码
```java
public class Solution {
    public String intToRoman(int num) {
        String[] symbol = {"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};
        int[] value = {1000,900,500,400,100,90,50,40,10,9,5,4,1};
        StringBuilder res = new StringBuilder();
        int i = 0;
        while(num>0){
            if(num>=value[i]){
                res.append(symbol[i]);
                num -= value[i];
            } else {
                i++;
            }
        }
        return res.toString();
    }
}
```

思路2：
思路
题意是整型数转罗马数字，范围从 1 到 3999，查看下百度百科的罗马数字介绍如下：

相同的数字连写，所表示的数等于这些数字相加得到的数，如 Ⅲ=3；

小的数字在大的数字的右边，所表示的数等于这些数字相加得到的数，如 Ⅷ=8、Ⅻ=12；

小的数字（限于 Ⅰ、X 和 C）在大的数字的左边，所表示的数等于大数减小数得到的数，如 Ⅳ=4、Ⅸ=9。

那么我们可以把整数的每一位分离出来，让其每一位都用相应的罗马数字位表示，最终拼接完成。比如 621 我们可以分离百、十、个分别为 6、2、1，那么 600 对应的罗马数字是 DC，20 对应的罗马数字是 XX，1 对应的罗马数字是 I，所以最终答案便是 DCXXI。

```java
class Solution {
    public String intToRoman(int num) {
        String M[] = {"", "M", "MM", "MMM"};
        String C[] = {"", "C", "CC", "CCC", "CD", "D", "DC", "DCC", "DCCC", "CM"};
        String X[] = {"", "X", "XX", "XXX", "XL", "L", "LX", "LXX", "LXXX", "XC"};
        String I[] = {"", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"};
        return M[num / 1000] + C[(num % 1000) / 100] + X[(num % 100) / 10] + I[num % 10];
    }
}
```



## 13. Roman to Integer
Roman numerals are represented by seven different symbols: I, V, X, L, C, D and M.
```
Symbol       Value
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
For example, two is written as II in Roman numeral, just two one's added together. Twelve is written as, XII, which is simply X + II. The number twenty seven is written as XXVII, which is XX + V + II.

Roman numerals are usually written largest to smallest from left to right. However, the numeral for four is not IIII. Instead, the number four is written as IV. Because the one is before the five we subtract it making four. The same principle applies to the number nine, which is written as IX. There are six instances where subtraction is used:

I can be placed before V (5) and X (10) to make 4 and 9.
X can be placed before L (50) and C (100) to make 40 and 90.
C can be placed before D (500) and M (1000) to make 400 and 900.
Given a roman numeral, convert it to an integer. Input is guaranteed to be within the range from 1 to 3999.

Example 1:

Input: "III"
Output: 3
Example 2:

Input: "IV"
Output: 4
Example 3:

Input: "IX"
Output: 9
Example 4:

Input: "LVIII"
Output: 58
Explanation: C = 100, L = 50, XXX = 30 and III = 3.
Example 5:

Input: "MCMXCIV"
Output: 1994
Explanation: M = 1000, CM = 900, XC = 90 and IV = 4.
Tags: Math, String
```

思路:
题意是罗马数字转整型数，范围从 1 到 3999，查看下百度百科的罗马数字介绍如下：

相同的数字连写，所表示的数等于这些数字相加得到的数，如 Ⅲ=3；

小的数字在大的数字的右边，所表示的数等于这些数字相加得到的数，如 Ⅷ=8、Ⅻ=12；

小的数字（限于 Ⅰ、X 和 C）在大的数字的左边，所表示的数等于大数减小数得到的数，如 Ⅳ=4、Ⅸ=9。

那么我们可以利用 map 来完成罗马数字的 7 个数字符号：I、V、X、L、C、D、M 和整数的映射关系，然后根据上面的解释来模拟完成即可。

```java
class Solution {
    public int romanToInt(String s) {
        Map<Character, Integer> map = new HashMap<>();
        map.put('I', 1);
        map.put('V', 5);
        map.put('X', 10);
        map.put('L', 50);
        map.put('C', 100);
        map.put('D', 500);
        map.put('M', 1000);
        int len = s.length();
        int sum = map.get(s.charAt(len - 1));
        for (int i = len - 2; i >= 0; --i) {
            if (map.get(s.charAt(i)) < map.get(s.charAt(i + 1))) {
                sum -= map.get(s.charAt(i));
            } else {
                sum += map.get(s.charAt(i));
            }
        }
        return sum;
    }
}


    public static class Solution1 {
        public int romanToInt(String s) {
            Map<Character, Integer> map = new HashMap();
            map.put('I', 1);
            map.put('V', 5);
            map.put('X', 10);
            map.put('L', 50);
            map.put('C', 100);
            map.put('D', 500);
            map.put('M', 1000);

            int result = 0;
            for (int i = 0; i < s.length(); i++) {
                if (i > 0 && map.get(s.charAt(i)) > map.get(s.charAt(i - 1))) {
                    result += map.get(s.charAt(i)) - 2 * map.get(s.charAt(i - 1));
                } else {
                    result += map.get(s.charAt(i));
                }
            }
            return result;
        }
    }
```

## 14. Longest Common Prefix
Write a function to find the longest common prefix string amongst an array of strings.

If there is no common prefix, return an empty string "".
```
Example 1:

Input: ["flower","flow","flight"]
Output: "fl"

Example 2:

Input: ["dog","racecar","car"]
Output: ""
Explanation: There is no common prefix among the input strings.
```

Note:

All given inputs are in lowercase letters a-z.


思路:
题意是让你从字符串数组中找出公共前缀，我的想法是找出最短的那个字符串的长度 minLen，然后在 0...minLen 的范围比较所有字符串，如果比较到有不同的字符，那么直接返回当前索引长度的字符串即可，否则最后返回最短的字符串即可。

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        int len = strs.length;
        if (len == 0) return "";
        int minLen = 0x7fffffff;
        for (String str : strs) minLen = Math.min(minLen, str.length());
        for (int j = 0; j < minLen; ++j)
            for (int i = 1; i < len; ++i)
                if (strs[0].charAt(j) != strs[i].charAt(j))
                    return strs[0].substring(0, j);
        return strs[0].substring(0, minLen);
    }
}
```

## 15. 3Sum
Given an array nums of n integers, are there elements a, b, c in nums such that a + b + c = 0? Find all unique triplets in the array which gives the sum of zero.

Note:

The solution set must not contain duplicate triplets.

Example:

Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
[
  [-1, 0, 1],
  [-1, -1, 2]
]


思路： 时间 O(N^2) 空间 O(1)
3Sum其实可以转化成一个2Sum的题，我们先从数组中选一个数，并将目标数减去这个数，得到一个新目标数。然后再在剩下的数中找一对和是这个新目标数的数，其实就转化为2Sum了。为了避免得到重复结果，我们不仅要跳过重复元素，而且要保证2Sum找的范围要是在我们最先选定的那个数之后的。

```java
public class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        for(int i = 0; i < nums.length - 2; i++){
            // 跳过重复元素
            if(i > 0 && nums[i] == nums[i-1]) continue;
            // 计算2Sum
            ArrayList<List<Integer>> curr = twoSum(nums, i, 0 - nums[i]);
            res.addAll(curr);
        }
        return res;
    }
    
    private ArrayList<List<Integer>> twoSum(int[] nums, int i, int target){
        int left = i + 1, right = nums.length - 1;
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        while(left < right){
            if(nums[left] + nums[right] == target){
                ArrayList<Integer> curr = new ArrayList<Integer>();
                curr.add(nums[i]);
                curr.add(nums[left]);
                curr.add(nums[right]);
                res.add(curr);
                do {
                    left++;
                }while(left < nums.length && nums[left] == nums[left-1]);
                do {
                    right--;
                } while(right >= 0 && nums[right] == nums[right+1]);
            } else if (nums[left] + nums[right] > target){
                right--;
            } else {
                left++;
            }
        }
        return res;
    }
}

```


## 18. 4Sum	


基本思想： 时间 O(N^3) 空间 O(1)

https://segmentfault.com/a/1190000003740669#articleHeader2
和3Sum的思路一样，在计算4Sum时我们可以先选一个数，然后在剩下的数中计算3Sum。而计算3Sum则同样是先选一个数，然后再剩下的数中计算2Sum。



```java
public class Solution {
    public List<List<Integer>> fourSum(int[] nums, int target) {
        Arrays.sort(nums);
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        for(int i = 0; i < nums.length - 3; i++){
            if(i > 0 && nums[i] == nums[i-1]) continue;
            List<List<Integer>> curr = threeSum(nums, i, target - nums[i]);
            res.addAll(curr);
        }
        return res;
    }
    
    private List<List<Integer>> threeSum(int[] nums, int i, int target) {
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        for(int j = i + 1; j < nums.length - 2; j++){
            if(j > i + 1 && nums[j] == nums[j-1]) continue;
            List<List<Integer>> curr = twoSum(nums, i, j, target - nums[j]);
            res.addAll(curr);
        }
        return res;
    }
    
    private ArrayList<List<Integer>> twoSum(int[] nums, int i, int j, int target){
        int left = j + 1, right = nums.length - 1;
        ArrayList<List<Integer>> res = new ArrayList<List<Integer>>();
        while(left < right){
            if(nums[left] + nums[right] == target){
                ArrayList<Integer> curr = new ArrayList<Integer>();
                curr.add(nums[i]);
                curr.add(nums[j]);
                curr.add(nums[left]);
                curr.add(nums[right]);
                res.add(curr);
                do {
                    left++;
                }while(left < nums.length && nums[left] == nums[left-1]);
                do {
                    right--;
                } while(right >= 0 && nums[right] == nums[right+1]);
            } else if (nums[left] + nums[right] > target){
                right--;
            } else {
                left++;
            }
        }
        return res;
    }
}
```


## 16. 3Sum Closet
Given an array S of n integers, find three integers in S such that the sum is closest to a given number, target. Return the sum of the three integers. You may assume that each input would have exactly one solution.

For example, given array S = {-1 2 1 -4}, and target = 1.

The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).

思路
题意是让你从数组中找出最接近 target 的三个数的和，该题和 3Sum 的思路基本一样，先对数组进行排序，然后遍历这个排序数组，用两个指针分别指向当前元素的下一个和数组尾部，判断三者的和与 target 的差值来移动两个指针，并更新其结果，当然，如果三者的和直接与 target 值相同，那么返回 target 结果即可。

```java
public class Solution {
    public int threeSumClosest(int[] nums, int target) {
        int delta = 0x7fffffff, res = 0;
        Arrays.sort(nums);
        int len = nums.length - 2;
        for (int i = 0; i < len; i++) {
            int left = i + 1, right = nums.length - 1;
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                int curDelta = Math.abs(sum - target);
                if (curDelta == 0) return sum;
                if (curDelta < delta) {
                    delta = curDelta;
                    res = sum;
                }
                if (sum > target) --right;
                else ++left;
            }
        }
        return res;
    }
}
```


## 17. Letter Combinations of a Phone Number
Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent.

A mapping of digit to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.

![](http://upload.wikimedia.org/wikipedia/commons/thumb/7/73/Telephone-keypad2.svg/200px-Telephone-keypad2.svg.png)
```
Example:

Input: "23"
Output: ["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].
Note:
```
Although the above answer is in lexicographical order, your answer could be in any order you want.


思路：dfs
题意是给你按键，让你组合出所有不同结果，首先想到的肯定是回溯了，对每个按键的所有情况进行回溯，回溯的终点就是结果字符串长度和按键长度相同。


```java

class Solution {
    private static String[] map = new String[]{"abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

    public List<String> letterCombinations(String digits) {
        if (digits.length() == 0) return Collections.emptyList();
        List<String> list = new ArrayList<>();
        helper(list, digits, "");
        return list;
    }

    private void helper(List<String> list, String digits, String ans) {
        if (ans.length() == digits.length()) {
            list.add(ans);
            return;
        }
        for (char c : map[digits.charAt(ans.length()) - '2'].toCharArray()) {
            helper(list, digits, ans + c);
        }
    }
}
```

## 19. Remove Nth Node From End of List
Given a linked list, remove the n-th node from the end of list and return its head.
```
Example:

Given linked list: 1->2->3->4->5, and n = 2.

After removing the second node from the end, the linked list becomes 1->2->3->5.
Note:

Given n will always be valid.
```
Follow up:

Could you do this in one pass?


典型的快慢指针。先用快指针向前走n步，这样快指针就领先慢指针n步了，然后快慢指针一起走，当快指针到尽头时，慢指针就是倒数第n个。不过如果要删除这个节点，我们要知道的是该节点的前面一个节点。另外，如果要删的是头节点，也比较难处理。这里使用一个dummy头节点，放在本来的head之前，这样我们的慢指针从dummy开始遍历，当快指针为空是，慢指针正好是倒数第n个的前一个节点。最后返回的时候，返回dummy的下一个。



```java
public class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode fast = head;
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        // 快指针先走n步
        for(int i = 0; i < n; i++){
            fast = fast.next;
        }
        // 从dummy开始慢指针和快指针一起走
        ListNode curr = dummy;
        while(fast != null){
            fast = fast.next;
            curr = curr.next;
        }
        // 删除当前的下一个节点
        curr.next = curr.next.next;
        return dummy.next;
    }
}
```


## 20. Valid Parentheses
Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

An input string is valid if:

Open brackets must be closed by the same type of brackets.
Open brackets must be closed in the correct order.
Note that an empty string is also considered valid.
```
Example 1:

Input: "()"
Output: true

Example 2:

Input: "()[]{}"
Output: true

Example 3:

Input: "(]"
Output: false

Example 4:

Input: "([)]"
Output: false

Example 5:

Input: "{[]}"
Output: true
```

思路: stack
题意是判断括号匹配是否正确，很明显，我们可以用栈来解决这个问题，当出现左括号的时候入栈，当遇到右括号时，判断栈顶的左括号是否何其匹配，不匹配的话直接返回 false 即可，最终判断是否空栈即可，这里我们可以用数组模拟栈的操作使其操作更快，有个细节注意下 top = 1;，从而省去了之后判空的操作和 top - 1 导致数组越界的错误。

```java
class Solution {
    public boolean isValid(String s) {
        char[] stack = new char[s.length() + 1];
        int top = 1;
        for (char c : s.toCharArray()) {
            if (c == '(' || c == '[' || c == '{') {
                stack[top++] = c; 
            } else if (c == ')' && stack[--top] != '(') {
                return false;
            } else if (c == ']' && stack[--top] != '[') {
                return false;
            } else if (c == '}' && stack[--top] != '{') {
                return false;
            }
        }
        return top == 1;
    }
}
```

## 21. Merge Two Sorted Lists

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.
```
Example:

Input: 1->2->4, 1->3->4
Output: 1->1->2->3->4->4
```


```java
思路
题意是用一个新链表来合并两个已排序的链表，那我们只需要从头开始比较已排序的两个链表，新链表指针每次指向值小的节点，依次比较下去，最后，当其中一个链表到达了末尾，我们只需要把新链表指针指向另一个没有到末尾的链表此时的指针即可。

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
        ListNode head = new ListNode(0);
        ListNode temp = head;
        while (l1 != null && l2 != null) {
            if (l1.val < l2.val) {
                temp.next = l1;
                l1 = l1.next;
            } else {
                temp.next = l2;
                l2 = l2.next;
            }
            temp = temp.next;
        }
        temp.next = l1 != null ? l1 : l2;
        return head.next;
    }
}
```


## 22. Generate Parentheses
Given n pairs of parentheses, write a function to generate all combinations of well-formed parentheses.
```
For example, given n = 3, a solution set is:

[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]
```

思路 0
题意是给你 n 值，让你找到所有格式正确的圆括号匹配组，题目中已经给出了 n = 3 的所有结果。遇到这种问题，第一直觉就是用到递归或者堆栈，我们选取递归来解决，也就是 helper 函数的功能，从参数上来看肯定很好理解了，leftRest 代表还有几个左括号可以用，rightNeed 代表还需要几个右括号才能匹配，初始状态当然是 rightNeed = 0, leftRest = n，递归的终止状态就是 rightNeed == 0 && leftRest == 0，也就是左右括号都已匹配完毕，然后把 str 加入到链表中即可。

```java
class Solution {
    public List<String> generateParenthesis(int n) {
        List<String> list = new ArrayList<>();
        helper(list, "", 0, n);
        return list;
    }

    private void helper(List<String> list, String str, int rightNeed, int leftRest) {
        if (rightNeed == 0 && leftRest == 0) {
            list.add(str);
            return;
        }
        if (rightNeed > 0) helper(list, str + ")", rightNeed - 1, leftRest);
        if (leftRest > 0) helper(list, str + "(", rightNeed + 1, leftRest - 1);
    }
}
```

## 23. Merge k Sorted Lists
Merge k sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

Example:
```
Input:
[
  1->4->5,
  1->3->4,
  2->6
]
Output: 1->1->2->3->4->4->5->6
```

思路 1
还有另一种思路是利用优先队列，该数据结构用到的是堆排序，所以对总链表个数为 k 的复杂度为 logk，总元素为个数为 N 的话，其时间复杂度也为 O(Nlogk)。

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
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) return null;
        PriorityQueue<ListNode> queue = new PriorityQueue<>(lists.length, new Comparator<ListNode>() {
            @Override
            public int compare(ListNode o1, ListNode o2) {
                if (o1.val < o2.val) return -1;
                else if (o1.val == o2.val) return 0;
                else return 1;
            }
        });
        ListNode node = new ListNode(0), tmp = node;
        for (ListNode l : lists) {
            if (l != null) queue.add(l);
        }
        while (!queue.isEmpty()) {
            tmp.next = queue.poll();
            tmp = tmp.next;
            if (tmp.next != null) queue.add(tmp.next);
        }
        return node.next;
    }
}
```


思路 2 分治策略
题意是合并多个已排序的链表，分析并描述其复杂度，我们可以用分治法来两两合并他们，假设 k 为总链表个数，N 为总元素个数，那么其时间复杂度为 O(Nlogk)。

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
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists.length == 0) return null;
        return helper(lists, 0, lists.length - 1);
    }

    private ListNode helper(ListNode[] lists, int left, int right) {
        if (left >= right) return lists[left];
        int mid = left + right >>> 1;
        ListNode l0 = helper(lists, left, mid);
        ListNode l1 = helper(lists, mid + 1, right);
        return merge2Lists(l0, l1);
    }

    private ListNode merge2Lists(ListNode l0, ListNode l1) {
        ListNode node = new ListNode(0), tmp = node;
        while (l0 != null && l1 != null) {
            if (l0.val <= l1.val) {
                tmp.next = new ListNode(l0.val);
                l0 = l0.next;
            } else {
                tmp.next = new ListNode(l1.val);
                l1 = l1.next;
            }
            tmp = tmp.next;
        }
        tmp.next = l0 != null ? l0 : l1;
        return node.next;
    }
}
```

## 24. Swap Nodes in Pairs
Given a linked list, swap every two adjacent nodes and return its head.
```
Example:

Given 1->2->3->4, you should return the list as 2->1->4->3.
Note:

Your algorithm should use only constant extra space.
You may not modify the values in the list's nodes, only nodes itself may be changed.
```

思路 0
题意是让你交换链表中相邻的两个节点，最终返回交换后链表的头，限定你空间复杂度为 O(1)。我们可以用递归来算出子集合的结果，递归的终点就是指针指到链表末少于两个元素时，如果不是终点，那么我们就对其两节点进行交换，这里我们需要一个临时节点来作为交换桥梁，就不多说了。

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
    public ListNode swapPairs(ListNode head) {
        if (head == null || head.next == null) return head;
        ListNode node = head.next;
        head.next = swapPairs(node.next);
        node.next = head;
        return node;
    }
}
```

思路 1
另一种实现方式就是用循环来实现了，两两交换节点，也需要一个临时节点来作为交换桥梁，直到当前指针指到链表末少于两个元素时停止，代码很简单，如下所示。
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { vasl = x; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode preHead = new ListNode(0), cur = preHead;
        preHead.next = head;
        while (cur.next != null && cur.next.next != null) {
            ListNode temp = cur.next.next;
            cur.next.next = temp.next;
            temp.next = cur.next;
            cur.next = temp;
            cur = cur.next.next;
        }
        return preHead.next;
    }
}
```


## 26. Remove Duplicates from Sorted Array I

Given a sorted array, remove the duplicates in place such that each element appear only once and return the new length.

Do not allocate extra space for another array, you must do this in place with constant memory.

For example, Given input array nums = [1,1,2],

Your function should return length = 2, with the first two elements of nums being 1 and 2 respectively. It doesn't matter what you leave beyond the new length.


思路：两个指针，快指针指向当前数组遍历到的位置，慢指针指向不重复序列下一个存放的位置，这样我们一旦遍历到一个不重复的元素，就把它复制到不重复序列的末尾。判断重复的方法是记录上一个遍历到的数字，看是否一样。

```java
public int removeDuplicates(int[] nums) {
        if(nums.length == 0) return 0;
        int dup = nums[0];
        int end = 1;
        for(int i = 1; i < nums.length; i++){
            if(nums[i]!=dup){
                nums[end] = nums[i];
                dup = nums[i];
                end++;
            }
        }
        return end;
    }
```

不同写法：
题意是让你从一个有序的数组中移除重复的元素，并返回之后数组的长度。我的思路是判断长度小于等于 1 的话直接返回原长度即可，否则的话遍历一遍数组，用一个 tail 变量指向尾部，如果后面的元素和前面的元素不同，就让 tail 变量加一，最后返回 tail 即可。

```java

public int removeDuplicates(int[] nums) {
        int len = nums.length;
        if (len <= 1) return len;
        int tail = 1;
        for (int i = 1; i < len; ++i) {
            if (nums[i - 1] != nums[i]) {
                nums[tail++] = nums[i];
            }
        }
        return tail;
    }
```


##　27. Remove Element

Given an array nums and a value val, remove all instances of that value in-place and return the new length.

Do not allocate extra space for another array, you must do this by modifying the input array in-place with O(1) extra memory.

The order of elements can be changed. It doesn't matter what you leave beyond the new length.
```
Example 1:

Given nums = [3,2,2,3], val = 3,

Your function should return length = 2, with the first two elements of nums being 2.

It doesn't matter what you leave beyond the returned length.

Example 2:

Given nums = [0,1,2,2,3,0,4,2], val = 2,

Your function should return length = 5, with the first five elements of nums containing 0, 1, 3, 0, and 4.

Note that the order of those five elements can be arbitrary.

It doesn't matter what values are set beyond the returned length.
Clarification:

Confused why the returned value is an integer but your answer is an array?

Note that the input array is passed in by reference, which means modification to the input array will be known to the caller as well.

Internally you can think of this:

// nums is passed in by reference. (i.e., without making a copy)
int len = removeElement(nums, val);

// any modification to nums in your function would be known by the caller.
// using the length returned by your function, it prints the first len elements.
for (int i = 0; i < len; i++) {
    print(nums[i]);
}

```

思路
题意是移除数组中值等于 val 的元素，并返回之后数组的长度，并且题目中指定空间复杂度为 O(1)，我的思路是用 tail 标记尾部，遍历该数组时当索引元素不等于 val 时，tail 加一，尾部指向当前元素，最后返回 tail 即可。
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int tail = 0;
        for (int i = 0, len = nums.length; i < len; ++i) {
            if (nums[i] != val) {
                nums[tail++] = nums[i];
            }
        }
        return tail;
    }
}
```

## 28. Implement strStr()
Implement strStr().

Return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.
```
Example 1:

Input: haystack = "hello", needle = "ll"
Output: 2

Example 2:

Input: haystack = "aaaaa", needle = "bba"
Output: -1
```

Clarification:

What should we return when needle is an empty string? This is a great question to ask during an interview.

For the purpose of this problem, we will return 0 when needle is an empty string. This is consistent to C's strstr() and Java's indexOf().

思路
题意是从主串中找到子串的索引，如果找不到则返回-1，当子串长度大于主串，直接返回-1，然后我们只需要遍历比较即可。

```java

class Solution {
    public int strStr(String haystack, String needle) {
        int l1 = haystack.length(), l2 = needle.length();
        if (l1 < l2) return -1;
        for (int i = 0; ; i++) {
            if (i + l2 > l1) return -1;
            for (int j = 0; ; j++) {
                if (j == l2) return i;
                if (haystack.charAt(i + j) != needle.charAt(j)) break;
            }
        }
    }
}
```

KMP算法
复杂度
时间 O(N+M) 空间 O(M)

思路
KMP算法是较为高级的算法。它使用一个next数组，这个数组记录了模式串needle自身的前缀和后缀的重复情况。同样是双指针进行匹配，当失配时可以根据这个数组找到应该将模式串向后位移多少位，避免一些重复的比较。具体的解法这里有两篇文章比较好，一篇是详细讲解KMP算法。

```java
public class Solution {
    public int strStr(String haystack, String needle) {
        if(needle.length() == 0){
            return 0;
        }
        int[] next = new int[needle.length()];
        // 得到next数组
        getNextArr(next, needle);
        // i是匹配串haystack的指针，j是模式串needle的指针
        int i = 0, j = 0;
        // 双指针开始匹配
        while(i < haystack.length() && j < needle.length()){
            // 如果j=-1意味着刚刚失配过，下标+1后，下一轮就可以开始匹配各自的第一个了
            // 如果指向的字母相同，则下一轮匹配各自的下一个
            if(j == -1 || haystack.charAt(i) == needle.charAt(j)){
                i++;
                j++;
            // 如果失配，则将更新j为next[j]
            } else {
                j = next[j];
            }
        }
        return j == needle.length() ? i - j : -1;
    }
    
    private void getNextArr(int[] next, String needle){
        // k是前缀中相同部分的末尾，同时也是相同部分的长度，因为长度等于k-0。
        // j是后缀的末尾，即后缀相同部分的末尾 
        int k = -1, j = 0;
        next[0] = -1;
        while(j < needle.length() - 1){
            // 如果k=-1，说明刚刚失配了，则重新开始计算前缀和后缀相同的长度
            // 如果两个字符相等，则在上次前缀和后缀相同的长度加1就行了
            if (k == -1 || needle.charAt(j) == needle.charAt(k)){
                k++;
                j++;
                next[j] = k;
            } else {
            // 否则，前缀长度缩短为next[k]
                k = next[k];
            }
        }
    }
}

```

## 29. Divide Two Integers
Given two integers dividend and divisor, divide two integers without using multiplication, division and mod operator.

Return the quotient after dividing dividend by divisor.

The integer division should truncate toward zero.
```
Example 1:

Input: dividend = 10, divisor = 3
Output: 3

Example 2:

Input: dividend = 7, divisor = -3
Output: -2
```

Note:

Both dividend and divisor will be 32-bit signed integers.
The divisor will never be 0.
Assume we are dealing with an environment which could only store integers within the 32-bit signed integer range: [−2^31,  2^31 − 1]. For the purpose of this problem, assume that your function returns 2^31 − 1 when the division result overflows.

思路: 时间 O(N) 空间 O(1) 位操作

我们设想87 / 4，本来应该的得到21余3，那么如果我们把87忽略余数后分解一下，87 = 4 * 21 = 4 * 16 + 4 * 4 + 4 * 1，也就是87 = 4 * (1 * 2^4 + 0 * 2^3 + 1 * 2^2 + 0 * 2^1 + 1 * 2^0)，也就是把商分解为27 = 1 * 2^4 + 0 * 2^3 + 1 * 2^2 + 0 * 2^1 + 1 * 2^0，所以商的二进制是10101。我们可以不断的将4乘2的一次方，二次方，等等，直到找到最大那个次方，在这里是2的四次方。然后，我们就从四次方到零次方，按位看商的这一位该是0还是1。

```java
public class Solution {
    public int divide(int dividend, int divisor) {
        if(dividend == Integer.MIN_VALUE && (divisor == 1 || divisor == -1)){
            return divisor == 1 ? Integer.MIN_VALUE : Integer.MAX_VALUE;
        }
        return (int)divideLong(dividend, divisor);
    }
    
    public long divideLong(long dd, long dv){
        boolean isPos = (dd > 0 && dv > 0) || (dd < 0 && dv < 0); 
        dd = Math.abs(dd);
        dv = Math.abs(dv);
        int digits = 0;
        // 通过将除数乘2，乘4，乘8，一直乘下去，找到商的最高的次方
        // 比如87/4=21，商的最高次方是4，即2^4=16，即4 * 16 < 87
        while(dv <= dd){
            dv <<= 1;
            digits++;
        }
        // 重置除数，把最高次方减1得到实际最高位，刚才循环中多加了一次
        long res = 0;
        dv >>= digits;
        digits--;
        // 看商的每一位是否应该为1
        while(digits >= 0){
            if(dd >= (dv << digits)){
                dd -= dv << digits;
                res += 1 << digits;
            }
            digits--;
            System.out.println(res);
        }
        return isPos ? res : - res;
    }
}
```


## 30. Substring with Concatenation of All Words
You are given a string, s, and a list of words, words, that are all of the same length. Find all starting indices of substring(s) in s that is a concatenation of each word in words exactly once and without any intervening characters.
```
Example 1:

Input:
  s = "barfoothefoobarman",
  words = ["foo","bar"]
Output: [0,9]
Explanation: Substrings starting at index 0 and 9 are "barfoor" and "foobar" respectively.
The output order does not matter, returning [9,0] is fine too.

Example 2:

Input:
  s = "wordgoodstudentgoodword",
  words = ["word","student"]
Output: []
```


思路: 时间 O(NK) 空间 O(M)
由于数组中所有单词的长度都是一样的，我们可以像Longest Substring with At Most Two Distinct Characters中一样，把每个词当作一个字母来看待，但是要遍历K次，K是单词的长度，因为我们要分别统计从下标0开头，从下标1开头。。。直到下标K-1开头的字符串。举例来说foobarfoo，给定数组是[foo, bar]，那我们要对foo|bar|foo搜索一次，对oob|arf|oo搜索一次，对oba|rfo|o搜索一次，我们不用再对bar|foo搜索，因为其已经包含在第一种里面了。每次搜索中，我们通过哈希表维护一个窗口，比如foo|bar|foo中，我们先拿出foo。如果foo都不在数组中，那说明根本不能拼进去，则哈希表全部清零，从下一个词开始重新匹配。但是foo是在数组中的，所以给当前搜索的哈希表计数器加上1，如果发现当前搜索中foo出现的次数已经比给定数组中foo出现的次数多了，我们就要把上一次出现foo之前的所有词都从窗口中去掉，如果没有更多，则看下一个词bar，不过在这之前，我们还要看看窗口中有多少个词了，如果词的个数等于数组中词的个数，说明我们找到了一个结果。

注意
先判断输入是否为空，为空直接返回空结果

```java
public class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        List<Integer> res = new LinkedList<Integer>();
        if(words == null || words.length == 0 || s == null || s.equals("")) return res;
        HashMap<String, Integer> freq = new HashMap<String, Integer>();
        // 统计数组中每个词出现的次数，放入哈希表中待用
        for(String word : words){
            freq.put(word, freq.containsKey(word) ? freq.get(word) + 1 : 1);
        }
        // 得到每个词的长度
        int len = words[0].length();
        // 错开位来统计
        for(int i = 0; i < len; i++){
            // 建一个新的哈希表，记录本轮搜索中窗口内单词出现次数
            HashMap<String, Integer> currFreq = new HashMap<String, Integer>();
            // start是窗口的开始，count表明窗口内有多少词
            int start = i, count = 0;
            for(int j = i; j <= s.length() - len; j += len){
                String sub = s.substring(j, j + len);
                // 看下一个词是否是给定数组中的
                if(freq.containsKey(sub)){
                    // 窗口中单词出现次数加1
                    currFreq.put(sub, currFreq.containsKey(sub) ? currFreq.get(sub) + 1 : 1);
                    count++;
                    // 如果该单词出现次数已经超过给定数组中的次数了，说明多来了一个该单词，所以要把窗口中该单词上次出现的位置及之前所有单词给去掉
                    while(currFreq.get(sub) > freq.get(sub)){
                        String leftMost = s.substring(start, start + len);
                        currFreq.put(leftMost, currFreq.get(leftMost) - 1);
                        start = start + len;
                        count--;
                    }
                    // 如果窗口内单词数和总单词数一样，则找到结果
                    if(count == words.length){
                        String leftMost = s.substring(start, start + len);
                        currFreq.put(leftMost, currFreq.get(leftMost) - 1);
                        res.add(start);
                        start = start + len;
                        count--;
                    }
                // 如果截出来的单词都不在数组中，前功尽弃，重新开始
                } else {
                    currFreq.clear();
                    start = j + len;
                    count = 0;
                }
            }
        }
        return res;
    }
}

```



## 31. Next Permutation
```
Implement next permutation, which rearranges numbers into the lexicographically next greater permutation of numbers.

If such arrangement is not possible, it must rearrange it as the lowest possible order (ie, sorted in ascending order).

The replacement must be in-place, do not allocate extra memory.

Here are some examples. Inputs are in the left-hand column and its corresponding outputs are in the right-hand column.
1,2,3 → 1,3,2
3,2,1 → 1,2,3
1,1,5 → 1,5,1
```

​    

```java
public class Solution {
    public void nextPermutation(int[] nums) {
        if(nums.length <= 1){
            return;
        }
        int i = nums.length - 2;
        // 找到第一个下降点，我们要把这个下降点的值增加一点点
        // 对于511这种情况，要把前面两个1都跳过，所以要包含等于
        while(i >= 0 && nums[i] >= nums[i + 1]){
            i--;
        }
        // 如果这个下降点还在数组内，我们找到一个比它稍微大一点的数替换
        // 如果在之外，说明整个数组是降序的，是全局最大了
        if(i >= 0){
            int j = nums.length - 1;
            // 对于151，这种情况，要把最后面那个1跳过，所以要包含等于
            while(j > i && nums[j] <= nums[i]){
                j--;
            }
            swap(nums, i, j);
        }
        // 将下降点之前的部分倒序构成一个最小序列
        reverse(nums, i + 1, nums.length - 1);
    }
    
    private void swap(int[] nums, int i, int j){
        int tmp = nums[j];
        nums[j] = nums[i];
        nums[i] = tmp;
    }
    
    private void reverse(int[] nums, int left, int right){
        while(left < right){
            swap(nums, left, right);
            left++;
            right--;
        }
    }
}
```




## 32. Longest Valid Parentheses
Given a string containing just the characters '(' and ')', find the length of the longest valid (well-formed) parentheses substring.
```
Example 1:

Input: "(()"
Output: 2
Explanation: The longest valid parentheses substring is "()"

Example 2:

Input: ")()())"
Output: 4
Explanation: The longest valid parentheses substring is "()()"
```

思路1: 栈
这道求最长有效括号比之前那道 Valid Parentheses 验证括号难度要大一些，这里我们还是借助栈来求解，需要定义个start变量来记录合法括号串的起始位置，我们遍历字符串，如果遇到左括号，则将当前下标压入栈，如果遇到右括号，如果当前栈为空，则将下一个坐标位置记录到start，如果栈不为空，则将栈顶元素取出，此时若栈为空，则更新结果和i - start + 1中的较大值，否则更新结果和i - 栈顶元素中的较大值，代码如下：

```cpp
class Solution {
public:
    int longestValidParentheses(string s) {
        int res = 0, start = 0;
        stack<int> m;
        for (int i = 0; i < s.size(); ++i) {
            if (s[i] == '(') m.push(i);
            else if (s[i] == ')') {
                if (m.empty()) start = i + 1;
                else {
                    m.pop();
                    res = m.empty() ? max(res, i - start + 1) : max(res, i - m.top());
                }
            }
        }
        return res;
    }
};
```



## 33. Search in Rotated Sorted Array
Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.

(i.e., [0,1,2,4,5,6,7] might become [4,5,6,7,0,1,2]).

You are given a target value to search. If found in the array return its index, otherwise return -1.

You may assume no duplicate exists in the array.

Your algorithm's runtime complexity must be in the order of O(log n).
```
Example 1:

Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Example 2:

Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1
```

思路：二分，判断左半边是否有序的条件是：nums[min] <= nums[mid]，而判断是否在某一个有序区间，则要包含那个区间较远的那一头。判断是否在有序的那一方内，不再就循环继续找无序的一部分。

```java
public class Solution {
    public int search(int[] nums, int target) {
        int min = 0, max = nums.length - 1, mid = 0;
        while(min <= max){
            mid = (min + max) / 2;
            if(nums[mid] == target){
                return mid;
            }
            // 如果左半部分是有序的
            if(nums[min] <= nums[mid]){
                if(nums[min] <= target && target < nums[mid]){
                    max = mid - 1;
                } else {
                    min = mid + 1;
                } 
            // 如果右半部份是有序的
            } else {
                if(nums[mid] < target && target <= nums[max]){
                    min = mid + 1; 
                } else {
                    max = mid - 1;
                }
            }
        }
        // 不满足min <= max条件时，返回-1
        return -1;
    }
}
```





## 34. Search for a Range
Given a sorted array of integers, find the starting and ending position of a given target value.

Your algorithm's runtime complexity must be in the order of O(log n).

If the target is not found in the array, return [-1, -1].

For example, Given [5, 7, 7, 8, 8, 10] and target value 8, return [3, 4].

思路1：二分找到该元素位置，然后由中间向两边扩散，最坏情况是o(n)

```
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int idx = search(nums, 0, nums.size() - 1, target);
        if (idx == -1) return {-1, -1};
        int left = idx, right = idx;
        while (left > 0 && nums[left - 1] == nums[idx]) --left;
        while (right < nums.size() - 1 && nums[right + 1] == nums[idx]) ++right;
        return {left, right};
    }
    int search(vector<int>& nums, int left, int right, int target) {
        if (left > right) return -1;
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) return search(nums, mid + 1, right, target);
        else return search(nums, left, mid - 1, target);
    }
};
```


思路2：两次二分。时间 O(logN) 空间 O(1)

其实就是执行两次二分搜索，一次专门找左边边界，一次找右边边界。特别的，如果找左边边界时，要看mid左边一位的的数是否和当前mid相同，如果相同要继续在左半边二分搜索。如果找右边边界，则要判断右边一位的数是否相同。

```java
public class Solution {
    public int[] searchRange(int[] nums, int target) {
        // 找到左边界
        int front = search(nums, target, "front");
        // 找到右边界
        int rear = search(nums, target, "rear");
        int[] res = {front, rear};
        return res;
    }
    
    public int search(int[] nums, int target, String type){
        int min = 0, max = nums.length - 1;
        while(min <= max){
            int mid = min + (max - min) / 2;
            if(nums[mid] > target){
                max = mid - 1;
            } else if(nums[mid] < target){
                min = mid + 1;
            } else {
                // 对于找左边的情况，要判断左边的数是否重复
                if(type == "front"){
                    if(mid == 0) return 0;
                    if(nums[mid] != nums[mid - 1]) return mid;
                    max = mid - 1;
                } else {
                // 对于找右边的情况，要判断右边的数是否重复
                    if(mid == nums.length - 1) return nums.length - 1;
                    if(nums[mid] != nums[mid + 1]) return mid;
                    min = mid + 1;
                }
            }
        }
        //没找到该数返回-1
        return -1;
    }
}
```



## 35. Search Insert Position

Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.

You may assume no duplicates in the array.
```
Example 1:

Input: [1,3,5,6], 5
Output: 2

Example 2:

Input: [1,3,5,6], 2
Output: 1

Example 3:

Input: [1,3,5,6], 7
Output: 4

Example 4:

Input: [1,3,5,6], 0
Output: 0
```

思路：二分查找
```java
public class Solution {
    public int searchInsert(int[] nums, int target) {
        int min = 0, max = nums.length - 1;
        // 条件是min <= max
        while(min <= max){
            int mid = min + (max - min) / 2;
            // 找到了
            if(nums[mid] == target){
                return mid;
            }
            // 在左边
            if(nums[mid] > target){
                max = mid - 1;
            } else {
            // 在右边
                min = mid + 1;
            }
        }
        return min;
    }
}
```


## 36 Valid Sudoku	
```
 * 36. Valid Sudoku
 *
 * Determine if a Sudoku is valid, according to: Sudoku Puzzles - The Rules.
 * The Sudoku board could be partially filled, where empty cells are filled with the character '.'.
 *
 * A partially filled sudoku which is valid.
```
求解数独当前状态是否正确
思路是： 判断每行，每列，每个小方块是否有重复(数字1~9且不重复)。

双重循环搞定：HashSet或位图均可判断重复。
https://blog.csdn.net/mine_song/article/details/70207326
```java
	public boolean isValidSudoku(char[][] board) {
		for (int i = 0; i < 9; i++) {
			HashSet<Character> row = new HashSet<>();
			HashSet<Character> column = new HashSet<>();
			HashSet<Character> cube = new HashSet<>();
			for (int j = 0; j < 9; j++) {
				// 检查第i行，在横坐标位置
				if (board[i][j] != '.' && !row.add(board[i][j]))
					return false;
				// 检查第i列，在纵坐标位置
				if (board[j][i] != '.' && !column.add(board[j][i]))
					return false;
				// 行号+偏移量
				int RowIndex = 3 * (i / 3) + j / 3;
				// 列号+偏移量
				int ColIndex = 3 * (i % 3) + j % 3;
				//每个小九宫格，总共9个
				if (board[RowIndex][ColIndex] != '.' 
						&& !cube.add(board[RowIndex][ColIndex]))
					return false;
			}
		}
		return true;
	}

位图思路： bit_map_row[board[i][j] - '1'] == 1

```


## 37 Sudoku Solver	
```
 * 37. Sudoku Solver
 *
 * Write a program to solve a Sudoku puzzle by filling the empty cells.
 * Empty cells are indicated by the character '.'.
 * You may assume that there will be only one unique solution.
```
求解当前状态数独是否有解
思路是：dfs，递归，尝试设置未填充位置上每一个数字，判断该数字，如果可行递归下一个位置，不可行换下一个数字。
代码思路：
https://blog.csdn.net/zdavb/article/details/48165059
```java
    private boolean solve(char[][] board){
         for(int row = 0;row<9;row++){
             for(int col = 0;col<9;col++){
                 if(board[row][col] ==  '.'){
                     for(char i = '1';i<='9';i++){
                         board[row][col] = i;
                         if(isValid(board,row,col) && solve(board))
                            return true;
                         board[row][col] = '.';
                     }
                     return false;
                 }
             }
         }
         return true;
    }
    private boolean isValid(char[][] board,int row,int col){
        for(int i = 0;i<9;i++){
            if(i!=col && board[row][i] == board[row][col])
                return false;
        }
        for(int i = 0;i<9;i++){
            if(i!=row && board[i][col] == board[row][col])
                return false;
        }
        int beginRow = 3*(row/3);
        int beginCol = 3*(col/3);
        for(int i = beginRow;i<beginRow+3;i++){
            for(int j = beginCol;j<beginCol+3;j++){
                if(i!=row && j!=col && board[i][j] == board[row][col])
                    return false;
            }
        }
        return true;
    }

```

## 38 Count and Say

The count-and-say sequence is the sequence of integers with the first five terms as following:

```
1.     1
2.     11
3.     21
4.     1211
5.     111221
```

`1` is read off as `"one 1"` or `11`.

`11` is read off as `"two 1s"` or `21`.

`21` is read off as `"one 2`, then `one 1"` or `1211`.

Given an integer *n*, generate the *n*<sup>th</sup> term of the count-and-say sequence.

Note: Each term of the sequence of integers will be represented as a string.

**Example 1:**

```
Input: 1
Output: "1"
```

**Example 2:**

```
Input: 4
Output: "1211"
```

**Tags:** String

思路

题意是数和说，根据如下序列 `1, 11, 21, 1211, 111221, ...`，求第 n 个数，规则很简单,就是数和说，数就是数这个数数字有几个，说就是说这个数，所以 `1` 就是 1 个 1：`11`,`11` 就是有 2 个 1：`21`，`21` 就是 1 个 2、1 个 1：`1211`，可想而知后面就是 `111221`，思路的话就是按这个逻辑模拟出来即可。

```java
class Solution {
    public String countAndSay(int n) {
        String str = "1";
        while (--n > 0) {
            int times = 1;
            StringBuilder sb = new StringBuilder();
            char[] chars = str.toCharArray();
            int len = chars.length;
            for (int j = 1; j < len; j++) {
                if (chars[j - 1] == chars[j]) {
                    times++;
                } else {
                    sb.append(times).append(chars[j - 1]);
                    times = 1;
                }
            }
            str = sb.append(times).append(chars[len - 1]).toString();
        }
        return str;
    }
}
```


## 39. Combination Sum
Combination Sum： https://segmentfault.com/a/1190000003743112

```
Given a set of candidate numbers (candidates) (without duplicates) and a target number (target), find all unique combinations in candidates where the candidate numbers sums to target.

The same repeated number may be chosen from candidates unlimited number of times.

Note:

All numbers (including target) will be positive integers.
The solution set must not contain duplicate combinations.
Example 1:

Input: candidates = [2,3,6,7], target = 7,
A solution set is:
[
  [7],
  [2,2,3]
]
Example 2:

Input: candidates = [2,3,5], target = 8,
A solution set is:
[
  [2,2,2,2],
  [2,3,3],
  [3,5]
]
```
思路： dfs， 递归+剪枝

 Path Sum II 二叉树路径之和之二，Subsets II 子集合之二，Permutations 全排列，Permutations II 全排列之二，Combinations 组合项等等，如果仔细研究这些题目发现都是一个套路，都是需要另写一个递归函数，这里我们新加入三个变量，start记录当前的递归到的下标，out为一个解，res保存所有已经得到的解，每次调用新的递归函数时，此时的target要减去当前数组的的数。


```java
public class Solution {
    
    List<List<Integer>> res;
    
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        res = new LinkedList<List<Integer>>();
        List<Integer> tmp = new LinkedList<Integer>();
        // 先将数组排序避免重复搜素
        Arrays.sort(candidates);
        helper(candidates, target, 0, tmp);
        return res;
    }
    
    private void helper(int[] nums, int target, int index, List<Integer> tmp){
        // 如果当前和已经大于目标，说明该路径错误 
        if(target < 0){
            return;
        // 如果当前和等于目标，说明这是一条正确路径，记录该路径
        } else if(target == 0){
            List<Integer> oneComb = new LinkedList<Integer>(tmp);
            res.add(oneComb);
        // 否则，对剩余所有可能性进行深度优先搜索
        } else {
            // 选取之后的每个数字都是一种可能性
            for(int i = index; i < nums.length; i++){
                // 典型的先加入元素，再进行搜索，递归回来再移出元素的DFS解法 
                tmp.add(nums[i]);
                helper(nums, target - nums[i], i, tmp);
                tmp.remove(tmp.size() - 1);
            }
        }
    }
}
```


```java


public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    if (candidates == null || candidates.length < 1)
        return res;
    Arrays.sort(candidates);

    backtrack(candidates, target, res, new ArrayList<Integer>(), 0);

    return res;
}

// 如果不加参数start，会出现：input:[2,3,6,7]7
// 结果：[[2,2,3],[2,3,2],[3,2,2],[7]]instead of [[2,2,3],[7]]的情况
private void backtrack(int[] nums, int target, List<List<Integer>> res, ArrayList<Integer> list, int start) {
    if (target < 0)
        return;
    if (target == 0) {
        res.add(list);
        return;
    }
    //加上target >= nums[i]：33ms to 23ms
    for (int i = start; i < nums.length && target >= nums[i]; i++) {
        list.add(nums[i]);
        backtrack(nums, target - nums[i], res, new ArrayList<Integer>(list), i);
        list.remove(list.size() - 1);
    }
}


或者，

public class Solution {
    private void solve(List<List<Integer>> res,int currentIndex,int count,List<Integer> tmp,int[] candidates,int target){
        if(count>=target) {
            if(count==target)
                res.add(new LinkedList<>(tmp));
            return;
        }

        for(int i = currentIndex;i<candidates.length;i++){
            if(count+candidates[i]>target){
                break;
            }
            tmp.add(candidates[i]);
            solve(res,i,count+candidates[i],tmp,candidates,target);
            tmp.remove(tmp.size()-1);
        }
    }
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        List<List<Integer>> res = new LinkedList<List<Integer>>();
        List<Integer> tmp = new LinkedList<>();
        Arrays.sort(candidates);

        solve(res,0,0,tmp,candidates,target);
        return res;
    }
}
```

## 40. Combination Sum II 

Given a collection of candidate numbers (candidates) and a target number (target), find all unique combinations in candidates where the candidate numbers sums to target.

**Each number in candidates may only be used once in the combination.**

Note:

All numbers (including target) will be positive integers.
The solution set must not contain duplicate combinations.
```
Example 1:

Input: candidates = [10,1,2,7,6,1,5], target = 8,
A solution set is:
[
  [1, 7],
  [1, 2, 5],
  [2, 6],
  [1, 1, 6]
]

Example 2:
Input: candidates = [2,5,2,1,2], target = 5,
A solution set is:
[
  [1,2,2],
  [5]
]
```

思路：

只需要改动一点点即可，之前那道题给定数组中的数字可以重复使用，而这道题不能重复使用，只需要在之前的基础上修改两个地方即可，首先在递归的for循环里加上if (i > start && num[i] == num[i - 1]) continue; 这样可以防止res中出现重复项，然后就在递归调用combinationSum2DFS里面的参数换成i+1，这样就不会重复使用数组中的数字了

```java
public class Solution {
    
    List<List<Integer>> res;
    
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        res = new LinkedList<List<Integer>>();
        List<Integer> tmp = new LinkedList<Integer>();
        Arrays.sort(candidates);
        helper(candidates, target, 0, tmp);
        return res;
    }
    
    private void helper(int[] nums, int target, int index, List<Integer> tmp){
        if(target < 0){
            return;
        } else if(target == 0){
            List<Integer> oneComb = new LinkedList<Integer>(tmp);
            res.add(oneComb);
        } else {
            for(int i = index; i < nums.length; i++){
                tmp.add(nums[i]);
                // 递归时下标加1
                helper(nums, target - nums[i], i+1, tmp);
                tmp.remove(tmp.size() - 1);
                // 跳过本轮剩余的重复元素
                while(i < nums.length - 1 && nums[i] == nums[i + 1]){
                    i++;
                }
            }
        }
    }
}
```


```java

public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    if (candidates == null || candidates.length < 1)
        return res;
    Arrays.sort(candidates);

    backtrack(candidates, target, res, new ArrayList<Integer>(), 0);

    return res;
}

// 如果不加参数start，会出现：input:[2,3,6,7]7
// 结果：[[2,2,3],[2,3,2],[3,2,2],[7]]instead of [[2,2,3],[7]]的情况
private void backtrack(int[] nums, int target, List<List<Integer>> res, ArrayList<Integer> list, int start) {
    if (target == 0) {
        res.add(new ArrayList<Integer>(list));
        return;
    }
    for (int i = start; i < nums.length && target >= nums[i]; i++) {
        if(i>start&&nums[i]==nums[i-1])
            continue;
        list.add(nums[i]);
        backtrack(nums, target - nums[i], res, list, i + 1);
        list.remove(list.size() - 1);
    }
}

```

## 41. First Missing Positive 

Given an unsorted integer array, find the smallest missing positive integer.
```
Example 1:
Input: [1,2,0]
Output: 3

Example 2:
Input: [3,4,-1,1]
Output: 2

Example 3:
Input: [7,8,9,11,12]
Output: 1
```

Note:

Your algorithm should run in O(n) time and uses constant extra space.

思路：
1、hashset
```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int mx = 0;
        unordered_set<int> s;
        for (int num : nums) {
            if (num <= 0) continue;
            s.insert(num);
            mx = max(mx, num);
        }
        for (int i = 1; i <= mx; ++i) {
            if (!s.count(i)) return i;
        }
        return mx + 1;
    }
};
```

2、原地交换，A[i] = i + 1; 注意交换条件A[i] != (i+1) &&  A[A[i]-1] != A[i] (防止重复)
https://www.cnblogs.com/AnnieKim/archive/2013/04/21/3034631.html
```java
class Solution {
public:
    int firstMissingPositive(int A[], int n) {
        int i = 0;
        while (i < n)
        {
            if (A[i] != (i+1) && A[i] >= 1 && A[i] <= n && A[A[i]-1] != A[i])
                swap(A[i], A[A[i]-1]);
            else
                i++;
        }
        for (i = 0; i < n; ++i)
            if (A[i] != (i+1))
                return i+1;
        return n+1;
    }
};
```

## 42. Trapping Rain Water
Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.


The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped. Thanks Marcos for contributing this image!
![](http://www.leetcode.com/static/images/problemset/rainwatertrap.png)

Example:
```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6 
```

思路： 
1、dp，遍历两遍
这种方法是基于动态规划Dynamic Programming的，我们维护一个一维的dp数组，这个DP算法需要遍历两遍数组，第一遍遍历dp[i]中存入i位置左边的最大值，然后开始第二遍遍历数组，第二次遍历时找右边最大值，然后和左边最大值比较取其中的较小值，然后跟当前值A[i]相比，如果大于当前值，则将差值存入结果，代码如下：
```java
public class Solution {
    public int trap(int[] height) {
        int res = 0, mx = 0, n = height.length;
        int[] dp = new int[n];
        for (int i = 0; i < n; ++i) {
            dp[i] = mx;
            mx = Math.max(mx, height[i]);
        }
        mx = 0;
        for (int i = n - 1; i >= 0; --i) {
            dp[i] = Math.min(dp[i], mx);
            mx = Math.max(mx, height[i]);
            if (dp[i] - height[i] > 0) res += dp[i] - height[i];
        }
        return res;
    }
}

优化思路：
	public int trap(int[] height) {
		if (height == null || height.length < 2)
			return 0;
		int leftMax = height[0], rightMax = height[height.length - 1], left = 0, right = height.length - 1;
 
		int sum = 0;
		while (left < right) {
			leftMax = Math.max(height[left], leftMax);
			rightMax = Math.max(height[right], rightMax);
			if (leftMax < rightMax) {
				sum += leftMax - height[left];
				left++;
			} else {
				sum += rightMax - height[right];
				right--;
			}
 
		}
 
		return sum;
	}

```

2 stack实现
遍历高度，如果此时栈为空，或者当前高度小于等于栈顶高度，则把当前高度的坐标压入栈，注意我们不直接把高度压入栈，而是把坐标压入栈，这样方便我们在后来算水平距离。当我们遇到比栈顶高度大的时候，就说明有可能会有坑存在，可以装雨水。此时我们栈里至少有一个高度，如果只有一个的话，那么不能形成坑，我们直接跳过，如果多余一个的话，那么此时把栈顶元素取出来当作坑，新的栈顶元素就是左边界，当前高度是右边界，只要取二者较小的，减去坑的高度，长度就是右边界坐标减去左边界坐标再减1，二者相乘就是盛水量啦，参见代码如下
```java
class Solution {
    public int trap(int[] height) {
        Stack<Integer> s = new Stack<Integer>();
        int i = 0, n = height.length, res = 0;
        while (i < n) {
            if (s.isEmpty() || height[i] <= height[s.peek()]) {
                s.push(i++);
            } else {
                int t = s.pop();
                if (s.isEmpty()) continue;
                res += (Math.min(height[i], height[s.peek()]) - height[t]) * (i - s.peek() - 1);
            }
        }
        return res;
    }
}
```


## 43 Multiply Strings	 
Given two non-negative integers num1 and num2 represented as strings, return the product of num1 and num2, also represented as a string.
```
Example 1:

Input: num1 = "2", num2 = "3"
Output: "6"

Example 2:

Input: num1 = "123", num2 = "456"
Output: "56088"
```

Note:

The length of both num1 and num2 is < 110.
Both num1 and num2 contain only digits 0-9.
Both num1 and num2 do not contain any leading zero, except the number 0 itself.
You must not use any built-in BigInteger library or convert the inputs to integer directly.

```java

	public String multiply(String num1, String num2) {
		if (num1 == null || num2 == null || num1.length() == 0 || num2.length() == 0)
			return null;
		if(num1.equals("0")||num2.equals("0"))
			return "0";
		int len1 = num1.length(), len2 = num2.length();
		int[] arr1 = new int[len1], arr2 = new int[len2];
		// 首尾交换，便于计算
		for (int i = 0; i < len1; i++) {
			arr1[len1 - 1 - i] = num1.charAt(i) - '0';
		}
		for (int i = 0; i < len2; i++) {
			arr2[len2 - 1 - i] = num2.charAt(i) - '0';
		}
		// 两数相乘结果的长度介于len1*len2-1到len1*len2
		int[] res = new int[len1 + len2];
		for (int i = 0; i < len1; i++) {
			for (int j = 0; j < len2; j++) {
				res[i + j] += arr1[i] * arr2[j];
			}
		}
		for (int i = 0; i < len1 + len2; i++) {
			while (res[i] >9) {
				res[i + 1] += res[i] / 10;
				res[i] = res[i] % 10;
			}
		}
		StringBuilder ans=new StringBuilder();
		for (int i = len1 + len2-1; i >=0; i--) {
			ans.append(res[i]);
		}
		//如果第一个元素是0，去除
		return ans.charAt(0) == '0' && ans.length() != 1 ? ans.substring(1) : ans.toString();
	}


    public String multiply(String num1, String num2) {
        if (isZero(num1) || isZero(num2)) {
            return "0";
        }
        int[] a1 = new int[num1.length()];
        int[] a2 = new int[num2.length()];
        int[] product = new int[num1.length() + num2.length()];

        for (int i = a1.length - 1; i >= 0; i--) {
            for (int j = a2.length - 1; j >= 0; j--) {
                int thisProduct = Character.getNumericValue(num1.charAt(i)) * Character.getNumericValue(num2.charAt(j));
                product[i + j + 1] += thisProduct % 10;
                if (product[i + j + 1] >= 10) {
                    product[i + j + 1] %= 10;
                    product[i + j]++;
                }
                product[i + j] += thisProduct / 10;
                if (product[i + j] >= 10) {
                    product[i + j] %= 10;
                    product[i + j - 1]++;
                }
            }
        }

        StringBuilder stringBuilder = new StringBuilder();
        for (int i = 0; i < product.length; i++) {
            if (i == 0 && product[i] == 0) {
                continue;
            }
            stringBuilder.append(product[i]);
        }
        return stringBuilder.toString();
    }

    private boolean isZero(String num) {
        for (char c : num.toCharArray()) {
            if (c != '0') {
                return false;
            }
        }
        return true;
    }


```


## 44. Wildcard Matching

Implement wildcard pattern matching with support for '?' and '*'.

'?' Matches any single character.
'*' Matches any sequence of characters (including the empty sequence).

The matching should cover the entire input string (not partial).

The function prototype should be:
bool isMatch(const char *s, const char *p)
```
Some examples:
isMatch("aa","a") → false
isMatch("aa","aa") → true
isMatch("aaa","aa") → false
isMatch("aa", "*") → true
isMatch("aa", "a*") → true
isMatch("ab", "?*") → true
isMatch("aab", "c*a*b") → false
```

思路 0

题意是让让你从判断 `s` 字符串是否通配符匹配于 `p`，这道题和[Regular Expression Matching][010]很是相似，区别在于 `*`，正则匹配的 `*` 不能单独存在，前面必须具有一个字符，其意义是表明前面的这个字符个数可以是任意个数，包括 0 个；而通配符的 `*` 是可以随意出现的，跟前面字符没有任何关系，其作用是可以表示任意字符串。在此我们可以利用 *贪心算法* 来解决这个问题，需要两个额外指针 `p` 和 `match` 来分别记录最后一个 `*` 的位置和 `*` 匹配到 `s` 字符串的位置，其贪心体现在如果遇到 `*`，那就尽可能取匹配后方的内容，如果匹配失败，那就回到上一个遇到 `*` 的位置来继续贪心。

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if (p.length() == 0) return s.length() == 0;
        int si = 0, pi = 0, match = 0, star = -1;
        int sl = s.length(), pl = p.length();
        char[] sc = s.toCharArray(), pc = p.toCharArray();
        while (si < sl) {
            if (pi < pl && (pc[pi] == sc[si] || pc[pi] == '?')) {
                si++;
                pi++;
            } else if (pi < pl && pc[pi] == '*') {
                star = pi++;
                match = si;
            } else if (star != -1) {
                si = ++match;
                pi = star + 1;
            } else return false;
        }
        while (pi < pl && pc[pi] == '*') pi++;
        return pi == pl;
    }
}
```

思路 1

另一种思路就是动态规划了，我们定义 `dp[i][j]` 的真假来表示 `s[0..i)` 是否匹配 `p[0..j)`，其状态转移方程如下所示：

* 如果 `p[j - 1] != '*'`，`P[i][j] = P[i - 1][j - 1] && (s[i - 1] == p[j - 1] || p[j - 1] == '?');`

* 如果 `p[j - 1] == '*'`，`P[i][j] = P[i][j - 1] || P[i - 1][j]`

```java
class Solution {
    public boolean isMatch(String s, String p) {
        if (p.length() == 0) return s.length() == 0;
        int sl = s.length(), pl = p.length();
        boolean[][] dp = new boolean[sl + 1][pl + 1];
        char[] sc = s.toCharArray(), pc = p.toCharArray();
        dp[0][0] = true;
        for (int i = 1; i <= pl; ++i) {
            if (pc[i - 1] == '*') dp[0][i] = dp[0][i - 1];
        }
        for (int i = 1; i <= sl; ++i) {
            for (int j = 1; j <= pl; ++j) {
                if (pc[j - 1] != '*') {
                    dp[i][j] = dp[i - 1][j - 1] && (sc[i - 1] == pc[j - 1] || pc[j - 1] == '?');
                } else {
                    dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
                }
            }
        }
        return dp[sl][pl];
    }
}
```

1.  Jump Game II

Given an array of non-negative integers, you are initially positioned at the first index of the array.

Each element in the array represents your maximum jump length at that position.

Your goal is to reach the last index in the **minimum number of jumps**.

Example:

Input: [2,3,1,1,4]
Output: 2
Explanation: The minimum number of jumps to reach the last index is 2.
    Jump 1 step from index 0 to 1, then 3 steps to the last index.
Note:

You can assume that you can always reach the last index.


思路：
cur是当前能到达的最远位置，last是上一步能到达的最远位置，我们遍历数组，首先用i + nums[i]更新cur，这个在上面解法中讲过了，然后判断如果当前位置到达了last，即上一步能到达的最远位置，说明需要再跳一次了，我们将last赋值为cur，并且步数res自增1，这里我们小优化一下，判断如果cur到达末尾了，直接break掉即可，代码如下

```
class Solution {
public:
    int jump(vector<int>& nums) {
        int res = 0, n = nums.size(), last = 0, cur = 0;
        for (int i = 0; i < n - 1; ++i) {
            cur = max(cur, i + nums[i]);
            if (i == last) {
                last = cur;
                ++res;
                if (cur >= n - 1) break;
            }
        }
        return res;
    }
};
```


## 46. Permutations

全排列系列： https://blog.csdn.net/ll15311257617/article/details/79858510

Given a collection of numbers, return all possible permutations.
```
For example,
[1,2,3] have the following permutations:
[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], and [3,2,1].
```

思路：


```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    boolean[] used = null;

    public List<List<Integer>> permuteUnique(int[] nums) {
        res.clear();
        if(nums.length==0) return res;
        used = new boolean[nums.length];
        permuteAux(nums,0, new ArrayList<>());
        return res;
    }

    private void permuteAux(int[] nums,int index,List<Integer> st) {
        if(index == nums.length){
            List<Integer> t = new ArrayList<>(st);
            res.add(t);
            return;
        }

        for(int i=0;i<nums.length;i++){
            if(used[i]) continue;
            st.add(nums[i]);
            used[i] = true;
            permuteAux(nums,index+1,st);
            used[i] = false;
            st.remove(st.size()-1);

        }
        return;
    }
}
```

另一种交换思路
```
class Solution {
public:
    vector<vector<int> > permute(vector<int> &num) {
        vector<vector<int> > res;
        permuteDFS(num, 0, res);
        return res;
    }
    void permuteDFS(vector<int> &num, int start, vector<vector<int> > &res) {
        if (start >= num.size()) res.push_back(num);
        for (int i = start; i < num.size(); ++i) {
            swap(num[start], num[i]);
            permuteDFS(num, start + 1, res);
            swap(num[start], num[i]);
        }
    }
};
```


## 47. Permutations II
Given a collection of numbers that might contain **duplicates**, return **all possible unique** permutations.
```
Example:

Input: [1,1,2]
Output:
[
  [1,1,2],
  [1,2,1],
  [2,1,1]
] 
```

思路：解题思路与46图题基本一致，但是却必须做出一些，改变，首先必须使得数组有序，在对数组中的每个元素进行全排列的时候，如果该元素与前一个元素相同，且前面一个元素已经完成了全排列，则跳过这个元素。

```java
class Solution {

    List<List<Integer>> res = new ArrayList<>();
    boolean[] used = null;

    public List<List<Integer>> permuteUnique(int[] nums) {
        res.clear();
        if(nums.length==0) return res;
        used = new boolean[nums.length];

        Arrays.sort(nums);
        permuteAux(nums,0, new ArrayList<>());
        return res;
    }

    private void permuteAux(int[] nums,int index,List<Integer> st) {
        if(index == nums.length){

            List<Integer> t = new ArrayList<>(st);
            res.add(t);
            return;
        }

        for(int i=0;i<nums.length;i++){
            if(used[i]) continue;
            if(i>0 &&nums[i-1]==nums[i] && !used[i-1]) continue;

            st.add(nums[i]);
            used[i] = true;
            permuteAux(nums,index+1,st);
            used[i] = false;
            st.remove(st.size()-1);

        }

        return;
    }
}
```

或者直接使用46交换方法+set去重复即可。



## 48. Rotate Image
You are given an n x n 2D matrix representing an image.

Rotate the image by 90 degrees (clockwise).

Note:

You have to rotate the image in-place, which means you have to modify the input 2D matrix directly. DO NOT allocate another 2D matrix and do the rotation.
```
Example 1:

Given input matrix = 
[
  [1,2,3],
  [4,5,6],
  [7,8,9]
],

rotate the input matrix in-place such that it becomes:
[
  [7,4,1],
  [8,5,2],
  [9,6,3]
]
```


```java
    public static class Solution1 {
        public void rotate(int[][] matrix) {
            /**First swap the elements on the diagonal, then reverse each row:
             * 1, 2, 3                    1, 4, 7                      7, 4, 1
             * 4, 5, 6         becomes    2, 5, 8           becomes    8, 5, 2
             * 7, 8, 9                    3, 6, 9                      9, 6, 3
             This is done in O(1) space!
             **/
            int m = matrix.length;
            for (int i = 0; i < m; i++) {
                for (int j = i; j < m; j++) {
                    /**ATTN: j starts from i, so that the diagonal changes with itself, results in no change.*/
                    int tmp = matrix[i][j];
                    matrix[i][j] = matrix[j][i];
                    matrix[j][i] = tmp;
                }
            }

            /**then reverse*/
            for (int i = 0; i < m; i++) {
                int left = 0;
                int right = m - 1;
                while (left < right) {
                    int tmp = matrix[i][left];
                    matrix[i][left] = matrix[i][right];
                    matrix[i][right] = tmp;
                    left++;
                    right--;
                }
            }
        }
    }
```



## 49. Group Anagrams
Given an array of strings, group anagrams together.

Example:
```
Input: ["eat", "tea", "tan", "ate", "nat", "bat"],
Output:
[
  ["ate","eat","tea"],
  ["nat","tan"],
  ["bat"]
]
```

Note:

All inputs will be in lowercase.
The order of your output does not matter.

思路：错位词，所谓的错位词就是两个字符串中字母出现的次数都一样，只是位置不同，比如abc，bac, cba等它们就互为错位词，那么我们如何判断两者是否是错位词呢


```java
public class _49 {

  public static class Solution1 {
    public List<List<String>> groupAnagrams(String[] strs) {
      Map<String, List<String>> map = new HashMap<>();
      for (String word : strs) {
        char[] c = word.toCharArray();
        Arrays.sort(c);
        String key = new String(c);
        if (!map.containsKey(key)) {
          map.put(key, new ArrayList<>());
        }
        map.get(key).add(word);
      }
      return new ArrayList<>(map.values());
    }
  }
}
```

## 50. Pow(x, n)
Implement pow(x, n), which calculates x raised to the power n (xn).
```
Example 1:
Input: 2.00000, 10
Output: 1024.00000

Example 2:
Input: 2.10000, 3
Output: 9.26100

Example 3:
Input: 2.00000, -2
Output: 0.25000
Explanation: 2^-2 = 1/2^2 = 1/4 = 0.25
```

Note:

-100.0 < x < 100.0
n is a 32-bit signed integer, within the range [−2^31, 2^31 − 1]


思路：二分
```
class Solution {
public:
    double myPow(double x, int n) {
        if (n < 0) return 1 / power(x, -n);
        return power(x, n);
    }
    double power(double x, int n) {
        if (n == 0) return 1;
        double half = power(x, n / 2);
        if (n % 2 == 0) return half * half;
        return x * half * half;
    }
};
```


```
class Solution {
public:
    double myPow(double x, int n) {
        double res = 1.0;
        for (int i = n; i != 0; i /= 2) {
            if (i % 2 != 0) res *= x;
            x *= x;
        }
        return n < 0 ? 1 / res : res;
    }
};
```

## 51. N-Queens
https://segmentfault.com/a/1190000003762668
The n-queens puzzle is the problem of placing n queens on an n×n chessboard such that no two queens attack each other.

![](https://leetcode.com/static/images/problemset/8-queens.png)

Given an integer n, return all distinct solutions to the n-queens puzzle.

Each solution contains a distinct board configuration of the n-queens' placement, where 'Q' and '.' both indicate a queen and an empty space respectively.
```
Example:

Input: 4
Output: [
 [".Q..",  // Solution 1
  "...Q",
  "Q...",
  "..Q."],

 ["..Q.",  // Solution 2
  "Q...",
  "...Q",
  ".Q.."]
]
Explanation: There exist two distinct solutions to the 4-queens puzzle as shown above.
```

思路：dfs爆破
```java
public class Solution {
    
    List<List<String>> res;
    
    public List<List<String>> solveNQueens(int n) {
        res = new LinkedList<List<String>>();
        int[] nqueens = new int[n];
        helper(nqueens, n, 0);
        return res;
    }
    
    public void helper(int[] nqueens, int n, int i){
        if(i == nqueens.length){
            List<String> one = new LinkedList<String>();
            // 构成表示整个棋盘的字符串
            for(int num : nqueens){
                // 构成一个形如....Q....的字符串
                StringBuilder sb = new StringBuilder();
                for(int j = 0; j < num; j++){
                    sb.append('.');
                }
                sb.append('Q');
                for(int j = num + 1; j < n; j++){
                    sb.append('.');
                }
                one.add(sb.toString());
            }
            res.add(one);
        } else {
            //选择下一列的数字
            // 比如之前已经选了13xxxxxx，下一列可以选6，形成136xxxxx
            for(int num = 0; num < n; num++){
                nqueens[i] = num;
                // 如果是有效的，继续搜索
                if(isValid(nqueens, i)){
                    helper(nqueens, n, i+1);
                }
            }
        }
    }
    
    private boolean isValid(int[] nqueens, int i){
        for(int idx = 0; idx < i; idx++){
            // 检查对角线只要判断他们差的绝对值和坐标的差是否相等就行了
            if(nqueens[idx] == nqueens[i] || Math.abs(nqueens[idx] - nqueens[i]) ==  i - idx){
                return false;
            }
        }
        return true;
    }
}
```


## 52. N-Queens II

The n-queens puzzle is the problem of placing n queens on an n×n chessboard such that no two queens attack each other.

Given an integer n, **return the number** of distinct solutions to the n-queens puzzle.

```java
public class Solution {
    
    List<List<String>> res;
    int cnt = 0;
    
    public int totalNQueens(int n) {
        int[] nqueens = new int[n];
        helper(nqueens, n, 0);
        return cnt;
    }
    
    public void helper(int[] nqueens, int n, int i){
        if(i == nqueens.length){
            cnt++;
        } else {
            for(int num = 0; num < n; num++){
                nqueens[i] = num;
                if(isValid(nqueens, i)){
                    helper(nqueens, n, i+1);
                }
            }
        }
    }
    
    private boolean isValid(int[] nqueens, int i){
        for(int idx = 0; idx < i; idx++){
            if(nqueens[idx] == nqueens[i] || Math.abs(nqueens[idx] - nqueens[i]) ==  i - idx){
                return false;
            }
        }
        return true;
    }
}
```

## 53 Maximum Subarray
Find the contiguous subarray within an array (containing at least one number) which has the largest sum.

For example, given the array [−2,1,−3,4,−1,2,1,−5,4], the contiguous subarray [4,−1,2,1] has the largest sum = 6.

思路：动态规划+空间优化
这是一道非常典型的动态规划题，为了求整个字符串最大的子序列和，我们将先求较小的字符串的最大子序列和。这里我们从后向前、从前向后计算都是可以的。在从前向后计算的方法中，我们将第i个元素之前最大的子序列和存入一个一维数组dp中，对第i+1个元素来说，它的值取决于dp[i]，如果dp[i]是负数，那就没有必要加上它，因为这只会拖累子序列的最大和。如果是正数就加上它。最后我们再讲第i+1个元素自身加进去，就得到了第i+1个元素之前最大的子序列和。
```java
public class Solution {
    public int maxSubArray(int[] nums) {
        int[] dp = new int[nums.length];
        int max = nums[0];
        dp[0] = nums[0];
        for(int i = 1; i < nums.length; i++){
            dp[i] = dp[i-1]>0? dp[i-1] + nums[i] : nums[i];
            max = Math.max(dp[i],max);
        }
        return max;
    }
}
```
空间优化，dp[i]只与dp[i-1]有关
```java
public class Solution {
    public int maxSubArray(int[] nums) {
        int max = nums[0];
        int sum = nums[0];
        for(int i = 1; i < nums.length; i++){
            sum = sum < 0 ? nums[i] : sum + nums[i];
            max = Math.max(sum, max);
        }
        return max;
    }
}
```

## 54. Spiral Matrix
螺旋矩阵
Given a matrix of m x n elements (m rows, n columns), return all elements of the matrix in spiral order.
```
Example 1:

Input:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
Output: [1,2,3,6,9,8,7,4,5]
Example 2:

Input:
[
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9,10,11,12]
]
Output: [1,2,3,4,8,12,11,10,9,5,6,7]
```

思路：顺序打印即可，注意计算
1、圈数是宽和高中较小的那个，加1再除以2
2、对应最后的行列计算int lastRow = m - i - 1; int lastCol = n - i - 1;
```java
public class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> res = new LinkedList<Integer>();
        if(matrix.length == 0) return res;
        int m = matrix.length, n = matrix[0].length;
        // 计算圈数
        int lvl = (Math.min(m, n) + 1) / 2;
        for(int i = 0; i < lvl; i++){
            // 计算相对应的该圈最后一行
            int lastRow = m - i - 1;
            // 计算相对应的该圈最后一列
            int lastCol = n - i - 1;
            // 如果该圈第一行就是最后一行，说明只剩下一行
            if(i == lastRow){
                for(int j = i; j <= lastCol; j++){
                    res.add(matrix[i][j]);
                }
            // 如果该圈第一列就是最后一列，说明只剩下一列
            } else if(i == lastCol){
                for(int j = i; j <= lastRow; j++){
                    res.add(matrix[j][i]);
                }
            } else {
                // 第一行
                for(int j = i; j < lastCol; j++){
                    res.add(matrix[i][j]);
                }
                // 最后一列
                for(int j = i; j < lastRow; j++){
                    res.add(matrix[j][lastCol]);
                }
                // 最后一行
                for(int j = lastCol; j > i; j--){
                    res.add(matrix[lastRow][j]);
                }
                // 第一列
                for(int j = lastRow; j > i; j--){
                    res.add(matrix[j][i]);
                }
            }
        }
        return res;
    }
}
```



## 55. Jump Game

Given an array of non-negative integers, you are initially positioned at the first index of the array.

Each element in the array represents your maximum jump length at that position.

Determine if you are **able to reach the last index**.
```
Example 1:
Input: [2,3,1,1,4]
Output: true
Explanation: Jump 1 step from index 0 to 1, then 3 steps to the last index.

Example 2:
Input: [3,2,1,0,4]
Output: false
Explanation: You will always arrive at index 3 no matter what. Its maximum
             jump length is 0, which makes it impossible to reach the last index.
```

思路：我们只希望知道能否到达末尾，也就是说我们只对最远能到达的位置感兴趣，所以我们维护一个变量reach，表示最远能到达的位置，初始化为0。遍历数组中每一个数字，如果当前坐标大于reach或者reach已经抵达最后一个位置则跳出循环，否则就更新reach的值为其和i + nums[i]中的较大值，其中i + nums[i]表示当前位置能到达的最大位置，代码如下：


```
class Solution {
public:
    bool canJump(vector<int>& nums) {
        int n = nums.size(), reach = 0;
        for (int i = 0; i < n; ++i) {
            if (i > reach || reach >= n - 1) break;
            reach = max(reach, i + nums[i]);
        }
        return reach >= n - 1;
    }
};

```


```java
public static class Solution1 {
        public boolean canJump(int[] nums) {
            int farthest = nums[0];
            for (int i = 0; i < nums.length; i++) {
                if (i <= farthest && nums[i] + i > farthest) {
                    // i <= farthest is to make sure that this current i is within the current range
                    // i > fathest 说明上一步最远也不能到达i这个位置
                    // nums[i]+i > farthest is to make sure that it's necessary to update farthest with current nums[i]+i
                    farthest = nums[i] + i;
                }
            }
            return farthest >= nums.length - 1;
        }
    }
```

## 56. Merge Intervals

Given a collection of intervals, merge all overlapping intervals.
```
Example 1:
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].

Example 2:
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considerred overlapping.
```


```java
/**
 * Definition for an interval.
 * public class Interval {
 *     int start;
 *     int end;
 *     Interval() { start = 0; end = 0; }
 *     Interval(int s, int e) { start = s; end = e; }
 * }
 */
```
思路：
首先根据Interval的起点，我们**将其排序**，这样能合并的Interval就一定是**相邻**的了：
[1,3] [5,6] [2,3]
---> [1,3] [2,3] [5,6]
然后我们就顺序遍历这个列表，并记录一个当前待合并的Interval，如果遍历到的Interval和当前待合并的Interval有重复部分，我们就将两个合并，如果没有重复部分，就将待合并的Interval加入结果中，并用新的Interval更新这个待合并的Interval。因为数组已经排过序，**前面的Interval的起点肯定小于后面Interval的起点**，所以在判断是否有重叠部分时，只要考虑待合并的Interval的终点和遍历到的Interval的起点的关系就行了。

```java
public class Solution {
    public List<Interval> merge(List<Interval> intervals) {
        List<Interval> res = new LinkedList<Interval>();
        if(intervals.size() == 0){
            return res;
        }
        // 按照起点排序
        Collections.sort(intervals, new Comparator<Interval>(){
            public int compare(Interval i1, Interval i2){
                return i1.start - i2.start;
            }
        });
        // 拿出第一个待合并的Interval
        Interval curr = intervals.get(0);
        for(Interval itv : intervals){
            // 如果有重叠部分，更新待合并的Interval的起点和终点
            if(curr.end >= itv.start){
                curr.start = Math.min(curr.start, itv.start);
                curr.end = Math.max(curr.end, itv.end);
            } else {
            // 否则将待合并的Interval加入结果中，并选取新的待合并Interval
                res.add(curr);
                curr = itv;
            }
        }
        // 将最后一个待合并的加进结果
        res.add(curr);
        return res;
    }
}

```

## 57. Insert Interval

Given a set of **non-overlapping** intervals, insert a new interval into the intervals (merge if necessary).

You may assume that the intervals were **initially sorted** according to their start times.
```
Example 1: Given intervals [1,3],[6,9], insert and merge [2,5] in as [1,5],[6,9].

Example 2: Given [1,2],[3,5],[6,7],[8,10],[12,16], insert and merge [4,9] in as [1,2],[3,10],[12,16].
```

This is because the new interval [4,9] overlaps with [3,5],[6,7],[8,10].

```java
public class _57 {

  public static class Solution1 {
    public List<Interval> insert(List<Interval> intervals, Interval newInterval) {
      List<Interval> result = new ArrayList<>();
      int i = 0;
      // add all the intervals ending before newInterval starts
      while (i < intervals.size() && intervals.get(i).end < newInterval.start) {
        result.add(intervals.get(i++));
      }
      // merge all overlapping intervals to one considering newInterval
      while (i < intervals.size() && intervals.get(i).start <= newInterval.end) {
        newInterval = new Interval( // we could mutate newInterval here also
            Math.min(newInterval.start, intervals.get(i).start),
            Math.max(newInterval.end, intervals.get(i).end));
        i++;
      }
      result.add(newInterval);
      // add all the rest
      while (i < intervals.size()) {
        result.add(intervals.get(i++));
      }
      return result;
    }
  }
}
```


## 58. Length of Last Word

Given a string s consists of upper/lower-case alphabets and empty space characters ' ', return the length of last word in the string.

If the last word does not exist, return 0.

Note: A word is defined as a character sequence consists of non-space characters only.

Example:
```
Input: "Hello World"
Output: 5
```

```java
    public static class Solution1 {
        public int lengthOfLastWord(String s) {
            if (s == null || s.length() == 0) {
                return 0;
            }
            s = s.trim();
            int n = s.length() - 1;
            while (n >= 0 && s.charAt(n) != ' ') {
                n--;
            }
            return s.length() - n - 1;
        }
    }

```


## 59. Spiral Matrix II

Given a positive integer n, generate a square matrix filled with elements from 1 to n2 in spiral order.

Example:

Input: 3
Output:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]

```java
public class Solution {
    public int[][] generateMatrix(int n) {
        int[][] res = new int[n][n];
        int left = 0, right = n - 1, bottom = n - 1, top = 0, num = 1;
        while(left < right && top < bottom){
            // 添加该圈第一行
            for(int i = left; i < right; i++){
                res[top][i] = num++;
            }
            // 添加最后一列
            for(int i = top; i < bottom; i++){
                res[i][right] = num++;
            }
            // 添加最后一行
            for(int i = right; i > left; i--){
                res[bottom][i] = num++;
            }
            // 添加第一列
            for(int i = bottom; i > top; i--){
                res[i][left] = num++;
            }
            top++;
            bottom--;
            left++;
            right--;
        }
        // 如果是奇数，加上中间那个点
        if(n % 2 == 1){
            res[n / 2][n / 2] = num;
        }
        return res;
    }
}
```


## 60. Permutation Sequence
The set [1,2,3,...,n] contains a total of n! unique permutations.
```
By listing and labeling all of the permutations in order, we get the following sequence for n = 3:

"123"
"132"
"213"
"231"
"312"
"321"
Given n and k, return the kth permutation sequence.
```
Note:

Given n will be between 1 and 9 inclusive.
Given k will be between 1 and n! inclusive.
```
Example 1:

Input: n = 3, k = 3
Output: "213"

Example 2:

Input: n = 4, k = 9
Output: "2314" 
```

思路：固定技巧

这道题是让求出n个数字的第k个排列组合，由于其特殊性，我们不用将所有的排列组合的情况都求出来，然后返回其第k个，我们可以只求出第k个排列组合即可，那么难点就在于如何知道数字的排列顺序，可参见网友喜刷刷的博客，首先我们要知道当n = 3时，其排列组合共有3! = 6种，当n = 4时，其排列组合共有4! = 24种，我们就以n = 4, k = 17的情况来分析，所有排列组合情况如下：

1234
1243
1324
1342
1423
1432
2134
2143
2314 
2341
2413
2431
3124
3142
3214
3241
3412	<--- k = 17
3421
4123
4132
4213
4231
4312
4321

我们可以发现，每一位上1,2,3,4分别都出现了6次，当第一位上的数字确定了，后面三位上每个数字都出现了2次，当第二位也确定了，后面的数字都只出现了1次，当第三位确定了，那么第四位上的数字也只能出现一次，那么下面我们来看k = 17这种情况的每位数字如何确定，由于k = 17是转化为数组下标为16：
```
最高位可取1,2,3,4中的一个，每个数字出现3！= 6次，所以k = 16的第一位数字的下标为16 / 6 = 2，即3被取出
第二位此时从1,2,4中取一个，k = 16是此时的k' = 16 % (3!) = 4，而剩下的每个数字出现2！= 2次，所以第二数字的下标为4 / 2 = 2，即4被取出
第三位此时从1,2中去一个，k' = 4是此时的k'' = 4 % (2!) = 0，而剩下的每个数字出现1！= 1次，所以第三个数字的下标为 0 / 1 = 0，即1被取出
第四位是从2中取一个，k'' = 0是此时的k''' = 0 % (1!) = 0，而剩下的每个数字出现0！= 1次，所以第四个数字的下标为0 / 1= 0，即2被取出

那么我们就可以找出规律了

a1 = k / (n - 1)!
k1 = k

a2 = k1 / (n - 2)!
k2 = k1 % (n - 2)!
...

an-1 = kn-2 / 1!
kn-1 = kn-2 / 1!

an = kn-1 / 0!
kn = kn-1 % 0!
```

```cpp
string getPermutation(int n, int k) {
        string res;
        string num = "123456789";
        vector<int> f(n, 1);
        for (int i = 1; i < n; ++i) f[i] = f[i - 1] * i;
        --k;
        for (int i = n; i >= 1; --i) {
            int j = k / f[i - 1];
            k %= f[i - 1];
            res.push_back(num[j]);
            num.erase(j, 1);
        }
        return res;
    }

```

```java
public String getPermutation(int n, int k) {
            int[] nums = new int[n + 1];
            int permcount = 1;
            for (int i = 0; i < n; i++) {
                nums[i] = i + 1; // put 1, 2, 3 ... n into nums[]
                permcount *= (i + 1);
            }

            k--;
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < n; i++) {
                permcount = permcount / (n - i);
                int idx = k / permcount;// the index that this position should
                // choose
                sb.append(nums[idx]);
                // left shift nums[] by one bit
                for (int j = idx; j < n - i; j++) {
                    nums[j] = nums[j + 1];
                }
                k %= permcount;
            }
            return sb.toString();
        }
```



## 61. Rotate List
Given a linked list, rotate the list to the right by k places, where k is non-negative.
```
Example 1:
Input: 1->2->3->4->5->NULL, k = 2
Output: 4->5->1->2->3->NULL
Explanation:
rotate 1 steps to the right: 5->1->2->3->4->NULL
rotate 2 steps to the right: 4->5->1->2->3->NULL

Example 2:
Input: 0->1->2->NULL, k = 4
Output: 2->0->1->NULL
Explanation:
rotate 1 steps to the right: 2->0->1->NULL
rotate 2 steps to the right: 1->2->0->NULL
rotate 3 steps to the right: 0->1->2->NULL
rotate 4 steps to the right: 2->0->1->NULL
```

思路1：
一个指针就够了，原理是先遍历整个链表获得链表长度n，然后此时把链表头和尾链接起来，在往后走n - k % n个节点就到达新链表的头结点前一个点，这时断开链表即可，代码如下:

```java
ListNode *rotateRight(ListNode *head, int k) {
        if (!head) return NULL;
        int n = 1;
        ListNode *cur = head;
        while (cur->next) {
            ++n;
            cur = cur->next;
        }
        cur->next = head;
        int m = n - k % n;
        for (int i = 0; i < m; ++i) {
            cur = cur->next;
        }
        ListNode *newhead = cur->next;
        cur->next = NULL;
        return newhead;
    }
```

思路2：
快慢指针,
旋转链表的本质，就是把链表的后半部分放到前面来，所以关键在于如何找链表后半部分的起始节点。实际上，我们用一个快指针先走k步，然后快慢指针同时走，这样当快指针到末尾时，慢指针就到链表后半部分的起始节点了。不过，由于k可能大于链表的长度，所以我们要先对k取链表长度的模。要计算链表长度只要遍历一遍链表就行了。

```java
public ListNode rotateRight(ListNode head, int k) {
        ListNode fast = head;
        int length = 0;
        // 计算链表长度
        while(fast != null){
            length++;
            fast = fast.next;
        }
        // 如果不旋转或者原链表为空 则直接返回
        if(head == null || k % length == 0){
            return head;
        }
        fast = head;
        // 让快指针先走k步
        for(int i = 0; i < k % length; i++){
            fast = fast.next;
        } 
        // 找到新头节点的前一个节点
        ListNode slow = head;
        while(fast.next != null){
            fast = fast.next;
            slow = slow.next;
        }
        // 将后半部分放到前面来
        ListNode newHead = slow.next;
        slow.next = null;
        fast.next = head;
        return newHead;
    }
```




## 62. Unique Paths

A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

**How many** possible unique paths are there?

思路：动态规划+扫描空间优化
动态规划Dynamic Programming来解，我们可以维护一个二维数组dp，其中dp[i][j]表示到当前位置不同的走法的个数，然后可以得到递推式为: dp[i][j] = dp[i - 1][j] + dp[i][j - 1]，这里为了节省空间，我们使用一维数组dp，一行一行的刷新也可以.

```java
public int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        for(int i = 0; i < m; i++){
            dp[i][0] = 1;
        }
        for(int i = 0; i < n; i++){
            dp[0][i] = 1;
        }
        for(int i = 1; i < m; i++){
            for(int j = 1; j < n; j++){
                dp[i][j] = dp[i-1][j] + dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
```

```java
    public int uniquePaths(int m, int n) {
        int[] dp = new int[n];
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                // 每一行的第一个数都是1
                dp[j] = j == 0 ? 1 : dp[j - 1] + dp[j];
            }
        }
        return dp[n - 1];
    }
```

```
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<int> dp(n, 1);
        for (int i = 1; i < m; ++i) {
            for (int j = 1; j < n; ++j) {
                dp[j] += dp[j - 1]; 
            }
        }
        return dp[n - 1];
    }
};
```



## 63. Unique Paths II
A robot is located at the top-left corner of a m x n grid (marked 'Start' in the diagram below).

The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid (marked 'Finish' in the diagram below).

Now consider if **some obstacles** are added to the grids. How many unique paths would there be?

![](https://leetcode.com/static/images/problemset/robot_maze.png)

An obstacle and empty space is marked as 1 and 0 respectively in the grid.

Note: m and n will be at most 100.
```
Example 1:

Input:
[
  [0,0,0],
  [0,1,0],
  [0,0,0]
]
Output: 2
Explanation:
There is one obstacle in the middle of the 3x3 grid above.
There are two ways to reach the bottom-right corner:
1. Right -> Right -> Down -> Down
2. Down -> Down -> Right -> Right
```

思路：直接改造上一道题即可。

```java
public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int[] dp = new int[obstacleGrid[0].length];
        for(int i = 0; i < obstacleGrid.length; i++){
            for(int j = 0; j < obstacleGrid[0].length; j++){
                dp[j] = obstacleGrid[i][j] == 1 ? 0 : (j == 0 ? (i == 0 ? 1 : dp[j]) : dp[j - 1] + dp[j]); 
            }
        }
        return dp[obstacleGrid[0].length - 1];
    }
```

```
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int> > &obstacleGrid) {
        if (obstacleGrid.empty() || obstacleGrid[0].empty()) return 0;
        int m = obstacleGrid.size(), n = obstacleGrid[0].size();
        if (obstacleGrid[0][0] == 1) return 0;
        vector<int> dp(n, 0);
        dp[0] = 1;
        for (int i = 0; i < m; ++i) {
            for (int j = 0; j < n; ++j) {
                if (obstacleGrid[i][j] == 1) dp[j] = 0;
                else if (j > 0) dp[j] += dp[j - 1];
            }
        }
        return dp[n - 1];
    }
};
```






## 64. Minimum Path Sum
Given a m x n grid filled with non-negative numbers, find a path from top left to bottom right which minimizes the sum of all numbers along its path.

Note: You can only move either down or right at any point in time.

Example:
```
Input:
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
Output: 7
Explanation: Because the path 1→3→1→1→1 minimizes the sum.
```

思路：动态规划
```java
    public int minPathSum(int[][] grid) {
      if (grid == null || grid.length == 0) {
        return 0;
      }

      int height = grid.length;
      int width = grid[0].length;
      int[][] dp = new int[height][width];
      dp[0][0] = grid[0][0];
      for (int i = 1; i < height; i++) {
        dp[i][0] = dp[i - 1][0] + grid[i][0];
      }
      for (int j = 1; j < width; j++) {
        dp[0][j] = dp[0][j - 1] + grid[0][j];
      }
      for (int i = 1; i < height; i++) {
        for (int j = 1; j < width; j++) {
          dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
        }
      }
      return dp[height - 1][width - 1];
    }
```



## 65. Valid Number
验证数字 http://www.cnblogs.com/grandyang/p/4084408.html
Validate if a given string is numeric.

Some examples:
"0" => true
" 0.1 " => true
"abc" => false
"1 a" => false
"2e10" => true

Note: It is intended for the problem statement to be ambiguous. You should gather all requirements up front before implementing one.

思路：
首先，从题目中给的一些例子可以分析出来，我们所需要关注的除了数字以外的特殊字符有空格 ‘ ’， 小数点 '.', 自然数 'e/E', 还要加上正负号 '+/-"， 除了这些字符需要考虑意外，出现了任何其他的字符，可以马上判定不是数字。下面我们来一一分析这些出现了也可能是数字的特殊字符：

1. 空格 ‘ ’： 空格分为两种情况需要考虑，一种是出现在开头和末尾的空格，一种是出现在中间的字符。出现在开头和末尾的空格不影响数字，而一旦中间出现了空格，则立马不是数字。解决方法：预处理时去掉字符的首位空格，中间再检测到空格，则判定不是数字。

2. 小数点 '.'：小数点需要分的情况较多，首先的是小数点只能出现一次，但是小数点可以出现在任何位置，开头(".3"), 中间("1.e2"), 以及结尾("1." ), 而且需要注意的是，小数点不能出现在自然数 'e/E' 之后，如 "1e.1" false, "1e1.1" false。还有，当小数点位于末尾时，前面必须是数字，如 "1."  true，" -." false。解决方法：开头中间结尾三个位置分开讨论情况。

3. 自然数 'e/E'：自然数的前后必须有数字，即自然数不能出现在开头和结尾，如 "e" false,  ".e1" false, "3.e" false, "3.e1" true。而且小数点只能出现在自然数之前，还有就是自然数前面不能是符号，如 "+e1" false, "1+e" false. 解决方法：开头中间结尾三个位置分开讨论情况。

4. 正负号 '+/-"，正负号可以再开头出现，可以再自然数e之后出现，但不能是最后一个字符，后面得有数字，如  "+1.e+5" true。解决方法：开头中间结尾三个位置分开讨论情况。

```
class Solution {
public:
    bool isNumber(string s) {
        bool num = false, numAfterE = true, dot = false, exp = false, sign = false;
        int n = s.size();
        for (int i = 0; i < n; ++i) {
            if (s[i] == ' ') {
                if (i < n - 1 && s[i + 1] != ' ' && (num || dot || exp || sign)) return false;
            } else if (s[i] == '+' || s[i] == '-') {
                if (i > 0 && s[i - 1] != 'e' && s[i - 1] != ' ') return false;
                sign = true;
            } else if (s[i] >= '0' && s[i] <= '9') {
                num = true;
                numAfterE = true;
            } else if (s[i] == '.') {
                if (dot || exp) return false;
                dot = true;
            } else if (s[i] == 'e') {
                if (exp || !num) return false;
                exp = true;
                numAfterE = false;
            } else return false;
        }
        return num && numAfterE;
    }
};
```


其他思路：
利用有限自动机Finite Automata Machine的程序写的简洁优雅 (http://blog.csdn.net/kenden23/article/details/18696083), 还有利用正则表达式，更是写的丧心病狂的简洁 (http://blog.csdn.net/fightforyourdream/article/details/12900751)

```java
	public boolean isNumber(String s) {
        if(s.trim().isEmpty()){
            return false;
        }
        String regex = "[-+]?(\\d+\\.?|\\.\\d+)\\d*(e[-+]?\\d+)?";
        if(s.trim().matches(regex)){
            return true;
        } else{
            return false;
        }
    }

```


## 66. Plus One
Given a non-empty array of digits representing a non-negative integer, plus one to the integer.

The digits are stored such that the most significant digit is at the head of the list, and each element in the array contain a single digit.

You may assume the integer does not contain any leading zero, except the number 0 itself.
```
Example 1:
Input: [1,2,3]
Output: [1,2,4]
Explanation: The array represents the integer 123.

Example 2:
Input: [4,3,2,1]
Output: [4,3,2,2]
Explanation: The array represents the integer 4321.
```

```java
public int[] plusOne(int[] digits) {
        int n = digits.length;
        for (int i = digits.length - 1; i >= 0; --i) {
            if (digits[i] < 9) {
                ++digits[i];
                return digits;
            }
            digits[i] = 0;
        }
        int[] res = new int[n + 1];
        res[0] = 1;
        return res;
    }
```

进位法
```java
public int[] plusOne(int[] digits) {
        if (digits.length == 0) return digits;
        int carry = 1, n = digits.length;
        for (int i = digits.length - 1; i >= 0; --i) {
            if (carry == 0) return digits;
            int sum = digits[i] + carry;
            digits[i] = sum % 10;
            carry = sum / 10;
        }
        int[] res = new int[n + 1];
        res[0] = 1;
        return carry == 0 ? digits : res;
    }
```


## 67. Add Binary
Given two binary strings, return their sum (also a binary string).

The input strings are both non-empty and contains only characters 1 or 0.
```
Example 1:

Input: a = "11", b = "1"
Output: "100"

Example 2:

Input: a = "1010", b = "1011"
Output: "10101"
```

思路：i >=0 || j >=0 循环，字符串长度不同，可以填0处理。

```java
public String addBinary(String a, String b) {
        int i = a.length() - 1, j = b.length() - 1, carry = 0;
        StringBuilder sb = new StringBuilder();
        while(i >=0 || j >=0){
            int m = i >= 0 ? a.charAt(i) - '0' : 0;
            int n = j >= 0 ? b.charAt(j) - '0' : 0;
            int sum = m + n + carry;
            carry = sum / 2;
            sb.insert(0, String.valueOf(sum % 2));
            i--;
            j--;
        }
        if(carry != 0) sb.insert(0, '1');
        return sb.toString();
    }
```


## 68. Text Justification
Given an array of words and a length L, format the text such that each line has exactly L characters and is fully (left and right) justified.

You should pack your words in a greedy approach; that is, pack as many words as you can in each line. Pad extra spaces ' ' when necessary so that each line has exactly Lcharacters.

Extra spaces between words should be distributed as evenly as possible. If the number of spaces on a line do not divide evenly between words, the empty slots on the left will be assigned more spaces than the slots on the right.

For the last line of text, it should be left justified and no extra space is inserted between words.
```
For example,
words: ["This", "is", "an", "example", "of", "text", "justification."]
L: 16.

Return the formatted lines as:

[
   "This    is    an",
   "example  of text",
   "justification.  "
]
```

文本对齐：http://www.cnblogs.com/grandyang/p/4350381.html



## 69. Sqrt(x)	
Description
Implement int sqrt(int x).

Compute and return the square root of x, where x is guaranteed to be a non-negative integer.

Since the return type is an integer, the decimal digits are truncated and only the integer part of the result is returned.

Example 1:

Input: 4
Output: 2
Example 2:

Input: 8
Output: 2
Explanation: The square root of 8 is 2.82842..., and since 
             the decimal part is truncated, 2 is returned.

二分
```java
public int mySqrt(int x) {
        long low = 0 , high = x / 2;
        while(low <= high){
            int mid = low + (high - low) / 2;
            if(mid * mid < x){
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        return (int)high;
    }
```

牛顿
其中x1是上次计算结果，x2是本次计算结果，当他的误差小于一定值时返回。初始x值可选n/2，或者神奇数0x5f37642f。

https://segmentfault.com/a/1190000003697204
```java
public int sqrt(int x) {
        // 如果初始值取0x5f37642f，则会进一步加快计算速度
        double x1 = x/2.0;
        double x2 = 0.0, err = x2 - x1;
        while(Math.abs(err)>0.00000001){
            x2 = x1 - (x1 * x1 - x) / (2 * x1);
            err = x2 - x1;
            x1 = x2;
        }
        return (int)x2;
    }
```

## 70. Climbing Stairs
You are climbing a stair case. It takes n steps to reach to the top.

Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

Note: Given n will be a positive integer.

题意是爬楼梯，每次你只能爬一步或者两步，问到顶层共有多少种方案。我们假设到顶层共有 f(n) 种，那么 f(n) = f(n - 1) + f(n - 2) 肯定是成立的.

```java
public int climbStairs(int n) {
        int a = 1, b = 1;
        while (--n > 0) {
            b += a;
            a = b - a;
        }
        return b;
    }
```


## 71. Simplify Path
Given an absolute path for a file (Unix-style), simplify it.
```
For example,
path = "/home/", => "/home"
path = "/a/./b/../../c/", => "/c"

Corner Cases:

Did you consider the case where path = "/../"?
In this case, you should return "/".
Another corner case is the path might contain multiple slashes '/' together, such as "/home//foo/".
In this case, you should ignore redundant slashes and return "/home/foo".
```


```java
public class Solution {
    public String simplifyPath(String path) {
        Stack<String> s = new Stack<>();
        String[] p = path.split("/");
        for (String t : p) {
            if (!s.isEmpty() && t.equals("..")) {
                s.pop();
            } else if (!t.equals(".") && !t.equals("") && !t.equals("..")) {
                s.push(t);
            }
        }
        List<String> list = new ArrayList(s);
        return "/" + String.join("/", list);
    }
}
```


## 72. Edit Distance
Given two words word1 and word2, find the minimum number of operations required to convert word1 to word2.
```
You have the following 3 operations permitted on a word:

Insert a character
Delete a character
Replace a character

Example 1:

Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation: 
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e')

Example 2:

Input: word1 = "intention", word2 = "execution"
Output: 5
Explanation: 
intention -> inention (remove 't')
inention -> enention (replace 'i' with 'e')
enention -> exention (replace 'n' with 'x')
exention -> exection (replace 'n' with 'c')
exection -> execution (insert 'u')
```




思路：dp[i-1][j-1]表示一个长为i-1的字符串str1变为长为j-1的字符串str2的最短距离

这道题让求从一个字符串转变到另一个字符串需要的变换步骤，共有三种变换方式，插入一个字符，删除一个字符，和替换一个字符。根据以往的经验，对于字符串相关的题目十有八九都是用动态规划Dynamic Programming来解，这道题也不例外。这道题我们需要维护一个二维的数组dp，其中dp[i][j]表示从word1的前i个字符转换到word2的前j个字符所需要的步骤。那我们可以先给这个二维数组dp的第一行第一列赋值，这个很简单，因为第一行和第一列对应的总有一个字符串是空串，于是转换步骤完全是另一个字符串的长度。跟以往的DP题目类似，难点还是在于找出递推式，我们可以举个例子来看，比如word1是“bbc"，word2是”abcd“，那么我们可以得到dp数组如下：

```
Ø a b c d
Ø 0 1 2 3 4
b 1 1 1 2 3
b 2 2 1 2 3
c 3 3 2 1 2
```

我们通过观察可以发现，当word1[i] == word2[j]时，dp[i][j] = dp[i - 1][j - 1]，其他情况时，dp[i][j]是其左，左上，上的三个值中的最小值加1，那么可以得到递推式为：
```
if word1[i - 1] == word2[j - 1]
    dp[i][j] =  dp[i - 1][j - 1];
else
    dp[i][j] =min(dp[i - 1][j - 1], min(dp[i - 1][j], dp[i][j - 1])) + 1;
```

```cpp
int minDistance(string word1, string word2) {
        int n1 = word1.size(), n2 = word2.size();
        int dp[n1 + 1][n2 + 1];
        for (int i = 0; i <= n1; ++i) dp[i][0] = i;
        for (int i = 0; i <= n2; ++i) dp[0][i] = i;
        for (int i = 1; i <= n1; ++i) {
            for (int j = 1; j <= n2; ++j) {
                if (word1[i - 1] == word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = min(dp[i - 1][j - 1], min(dp[i - 1][j], dp[i][j - 1])) + 1;
                }
            }
        }
        return dp[n1][n2];
    }
```

```java
public int minDistance(String word1, String word2) {
        int m = word1.length(), n = word2.length();
        int[][] dp = new int[m + 1][n + 1];
        // 初始化空字符串的情况
        for(int i = 1; i <= m; i++){
            dp[i][0] = i;
        }
        for(int i = 1; i <= n; i++){
            dp[0][i] = i;
        }
        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                // 增加操作：str1a变成str2后再加上b，得到str2b
                int insertion = dp[i][j-1] + 1;
                // 删除操作：str1a删除a后，再由str1变为str2b
                int deletion = dp[i-1][j] + 1;
                // 替换操作：先由str1变为str2，然后str1a的a替换为b，得到str2b
                int replace = dp[i-1][j-1] + (word1.charAt(i - 1) == word2.charAt(j - 1) ? 0 : 1);
                // 三者取最小
                dp[i][j] = Math.min(replace, Math.min(insertion, deletion));
            }
        }
        return dp[m][n];
    }
```

## 73. Set Matrix Zeroes
Given a m x n matrix, if an element is 0, set its entire row and column to 0. Do it in-place.
```
Example 1:

Input: 
[
  [1,1,1],
  [1,0,1],
  [1,1,1]
]
Output: 
[
  [1,0,1],
  [0,0,0],
  [1,0,1]
]

Example 2:

Input: 
[
  [0,1,2,0],
  [3,4,5,2],
  [1,3,1,5]
]
Output: 
[
  [0,0,0,0],
  [0,4,5,0],
  [0,3,1,0]
]
```

思路：
实际上，我们只需要直到哪些行，哪些列需要被置0就行了，最简单的方法就是建两个大小分别为M和N的数组，来记录哪些行哪些列应该被置0。那有没有可能不用额外空间呢？我们其实可以借用原矩阵的首行和首列来存储这个信息。这个方法的缺点在于，如果我们直接将0存入首行或首列来表示相应行和列要置0的话，我们很难判断首行或者首列自己是不是该置0。这里我们用两个boolean变量记录下首行和首列原本有没有0，然后在其他位置置完0后，再单独根据boolean变量来处理首行和首列，就避免了干扰的问题。

```java
public void setZeroes(int[][] matrix) {
        if(matrix.length == 0) return;
        boolean firstRowZero = false, firstColZero = false;
        // 记录哪些行哪些列需要置0，并判断首行首列是否需要置0
        for(int i = 0; i < matrix.length; i++){
            for(int j = 0; j < matrix[0].length; j++){
                if(i != 0 && j != 0 && matrix[i][j] == 0){
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                } else if (matrix[i][j] == 0){
                    // 如果首行或首列出现0，则标记其需要置0，否则沿用上次值
                    firstRowZero = i == 0 ? true : firstRowZero;
                    firstColZero = j == 0 ? true : firstColZero;
                }
            }
        }
        // 将除首行首列的位置置0
        for(int i = 1; i < matrix.length; i++){
            for(int j = 1; j < matrix[0].length; j++){
                if(matrix[0][j] == 0 || matrix[i][0] == 0){
                    matrix[i][j] = 0;
                }
            }
        }
        // 如果必要，将首列置0
        for(int i = 0; firstColZero && i < matrix.length; i++){
            matrix[i][0] = 0;
        }
        // 如果必要，将首行置0
        for(int j = 0; firstRowZero && j < matrix[0].length; j++){
            matrix[0][j] = 0;
        }
    }
```


## 74. Search a 2D Matrix
Write an efficient algorithm that searches for a value in an m x n matrix. This matrix has the following properties:

Integers in each row are sorted from left to right.
The first integer of each row is greater than the last integer of the previous row.
```
Example 1:
Input:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 3
Output: true

Example 2:
Input:
matrix = [
  [1,   3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
]
target = 13
Output: false
```

思路：二分
```java
public boolean searchMatrix(int[][] matrix, int target) {
        int m = matrix.length;
        if(m == 0) return false;
        int n = matrix[0].length;
        int min = 0, max = m * n - 1;
        while(min <= max){
            int mid = min + (max - min) / 2;
            int row = mid / n;
            int col = mid % n;
            if(matrix[row][col] == target){
                return true;
            } else if (matrix[row][col] < target){
                min = mid + 1;
            } else {
                max = mid - 1;
            }
        }
        return false;
    }
```


## 75. Sort Colors
Given an array with n objects colored red, white or blue, sort them in-place so that objects of the same color are adjacent, with the colors in the order red, white and blue.

Here, we will use the integers 0, 1, and 2 to represent the color red, white, and blue respectively.

Note: You are not suppose to use the library's sort function for this problem.
```
Example:

Input: [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
```

Follow up:

A rather straight forward solution is a two-pass algorithm using counting sort.
First, iterate the array counting number of 0's, 1's, and 2's, then overwrite array with total number of 0's, then 1's and followed by 2's.
Could you come up with a **one-pass algorithm** using only constant space?

思路：
我们先用两个指针，一个指向已经排好序的0的序列的后一个点，一个指向已经排好序的2的序列的前一个点。这样在一开始，两个指针是指向头和尾的，因为我们还没有开始排序。然后我们遍历数组，当遇到0时，将其和0序列后面一个数交换，然后将0序列的指针向后移。当遇到2时，将其和2序列前面一个数交换，然后将2序列的指针向前移。遇到1时，不做处理。这样，当我们遍历到2序列开头时，实际上我们已经排好序了，因为所有0都被交换到了前面，所有2都被交换到了后面。

```java
public class Solution {
    public void sortColors(int[] nums) {
        int left = 0, right = nums.length - 1;
        int i = 0;
        while(i <= right){
            // 遇到0交换到前面
            if(nums[i] == 0){
                swap(nums, i, left);
                left++;
                // 因为左边必定有序，所以可以直接i++
                i++;
            // 遇到2交换到后面
            } else if(nums[i] == 2){
                swap(nums, i, right);
                right--;
            } else {
            // 遇到1跳过 
                i++;
            }
        }
    }
    
    private void swap(int[] nums, int i1, int i2){
        int tmp = nums[i1];
        nums[i1] = nums[i2];
        nums[i2] = tmp;
    }
}
```


## 76. Minimum Window Substring
Given a string S and a string T, find the minimum window in S which will contain all the characters in T in complexity O(n).
```
Example:

Input: S = "ADOBECODEBANC", T = "ABC"
Output: "BANC"
```

Note:

If there is no such window in S that covers all characters in T, return the empty string "".
If there is such window, you are guaranteed that there will always be only one unique minimum window in S.

思路
用一个哈希表记录目标字符串每个字母的个数，一个哈希表记录窗口中每个字母的个数。先找到第一个有效的窗口，用两个指针标出它的上界和下界。然后每次窗口右界向右移时，将左边尽可能的右缩，右缩的条件是窗口中字母的个数不小于目标字符串中字母的个数。

注意
用一个数组来保存每个字符出现的次数，比哈希表容易

保存结果子串的起始点初值为-1，方便最后判断是否有正确结果

```java
public class Solution {
    public String minWindow(String S, String T) {
        int[] srcHash = new int[255];
        // 记录目标字符串每个字母出现次数
        for(int i = 0; i < T.length(); i++){
            srcHash[T.charAt(i)]++;
        }
        int start = 0,i= 0;
        // 用于记录窗口内每个字母出现次数 
        int[] destHash = new int[255];
        int found = 0;
        int begin = -1, end = S.length(), minLength = S.length();
        for(start = i = 0; i < S.length(); i++){
            // 每来一个字符给它的出现次数加1
            destHash[S.charAt(i)]++;
            // 如果加1后这个字符的数量不超过目标串中该字符的数量，则找到了一个匹配字符
            if(destHash[S.charAt(i)] <= srcHash[S.charAt(i)]) found++;
            // 如果找到的匹配字符数等于目标串长度，说明找到了一个符合要求的子串    
            if(found == T.length()){
                // 将开头没用的都跳过，没用是指该字符出现次数超过了目标串中出现的次数，并把它们出现次数都减1
                while(start < i && destHash[S.charAt(start)] > srcHash[S.charAt(start)]){
                    destHash[S.charAt(start)]--;
                    start++;
                }
                // 这时候start指向该子串开头的字母，判断该子串长度
                if(i - start < minLength){
                    minLength = i - start;
                    begin = start;
                    end = i;
                }
                // 把开头的这个匹配字符跳过，并将匹配字符数减1
                destHash[S.charAt(start)]--;
                found--;
                // 子串起始位置加1，我们开始看下一个子串了
                start++;
            }
        }
        // 如果begin没有修改过，返回空
        return begin == -1 ? "" : S.substring(begin,end + 1);
    }
}
```


## 77. Combinations

Given two integers n and k, return all possible combinations of k numbers out of 1 ... n.

Example:
```
Input: n = 4, k = 2
Output:
[
  [2,4],
  [3,4],
  [2,3],
  [1,2],
  [1,3],
  [1,4],
]
```

思路：
通过深度优先搜索，回溯出所有可能性即可。时间 O(N) 空间 O(K)


```java
public class Solution {
    
    List<List<Integer>> res = new ArrayList<List<Integer>>();
    
    public List<List<Integer>> combine(int n, int k) {
        dfs(1, k, n, new ArrayList<Integer>());
        return res;
    }
    
    private void dfs(int start, int k, int n, List<Integer> tmp){
        // 当已经选择足够数字时，将tmp加入结果
        if(k == 0){
            res.add(new ArrayList<Integer>(tmp));
            System.out.println(tmp);
            return;
        }
        // 每一种选择数字的可能
        for(int i = start; i <= n; i++){
            tmp.add(i);
            dfs(i + 1, k - 1, n, tmp);
            tmp.remove(tmp.size() - 1);
        }
    }
}

[1, 2]
[1, 3]
[1, 4]
[2, 3]
[2, 4]
[3, 4]
```


## 78. Subsets
Given a set of distinct integers, nums, return all possible subsets (the power set).

Note: The solution set must not contain duplicate subsets.

Example:
```
Input: nums = [1,2,3]
Output:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]
```

思路：dfs即可，时间 O(NlogN) 空间 O(N) 递归栈空间
https://github.com/fishercoder1534/Leetcode/blob/master/src/main/java/com/fishercoder/solutions/_78.java
```java
    public static class Solution2 {
        public List<List<Integer>> subsets(int[] nums) {
            List<List<Integer>> result = new ArrayList();
            backtracking(result, new ArrayList(), nums, 0);
            return result;
        }

        void backtracking(List<List<Integer>> result, List<Integer> list, int[] nums, int start) {
            result.add(new ArrayList(list));
            for (int i = start; i < nums.length; i++) {
                list.add(nums[i]);
                backtracking(result, list, nums, i + 1);
                list.remove(list.size() - 1);
            }
        }
    }
```

```java
public class Solution {
    public List<List<Integer>> subsets(int[] S) {
        Arrays.sort(S);
        List<List<Integer>> res = new ArrayList<List<Integer>>();
        List<Integer> tmp = new ArrayList<Integer>();
        //先加入空集
        res.add(tmp);
        helper(S, 0, res, tmp);
        return res;
    }
    
    private void helper(int[] S,int index,List<List<Integer>> res, List<Integer> tmp){
        if(index>=S.length) return;
        // 不加入当前元素产生的集合，然后继续递归
        helper(S, index+1, res, tmp);
        List<Integer> tmp2 = new ArrayList<Integer>(tmp);
        tmp2.add(S[index]);
        res.add(tmp2);
        // 加入当前元素产生的集合，然后继续递归
        helper(S, index+1, res, tmp2);
    }
}

```


## 79. Word Search
Given a 2D board and a word, find if the word exists in the grid.

The word can be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once.

Example:
```
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

Given word = "ABCCED", return true.
Given word = "SEE", return true.
Given word = "ABCB", return false.
```

思路：dfs, 时间 O(N^2) 空间 O(N)


```java
public class Solution {
    public boolean exist(char[][] board, String word) {
        if(board.length == 0) return false;
        for(int i = 0; i < board.length; i++){
            for(int j = 0; j < board[0].length; j++){
                // 从i,j点作为起点开始搜索
                boolean isExisted = search(board, i, j, word, 0);
                if(isExisted) return true;
            }
        }
        return false;
    }
    
    private boolean search(char[][] board, int i, int j, String word, int idx){
        if(idx >= word.length()) return true;
        if(i < 0 || i >= board.length || j < 0 || j >= board[0].length || board[i][j] != word.charAt(idx)) return false;
        // 将已经搜索过的字母标记一下，防止循环。只要变成另外一个字符，就不会再有循环了。
        board[i][j] ^= 255;
        boolean res = search(board, i-1, j, word, idx+1) || search(board, i+1, j, word, idx+1) || search(board, i, j-1, word, idx+1) || search(board, i, j+1, word, idx+1);
        // 再次异或255就能恢复成原来的字母
        board[i][j] ^= 255;
        return res;
    }
}

```


## 80. Remove Duplicates from Sorted Array II
Follow up for "Remove Duplicates": What if **duplicates are allowed at most twice**?

For example, Given sorted array nums = [1,1,1,2,2,3],

Your function should return length = 5, with the first five elements of nums being 1, 1, 2, 2 and 3. It doesn't matter what you leave beyond the new length.


思路：双指针，记录前两个遍历到的数字来帮助我们判断是否出现了第三遍。如果当前数字和前一个数字的前一个一样的话，说明出现了第三次。

```java
public int removeDuplicates(int[] nums) {
        if(nums.length <= 2) return nums.length;
        int dup1 = nums[0];
        int dup2 = nums[1];
        int end = 2;
        for(int i = 2; i < nums.length; i++){
            if(nums[i]!=dup1){
                nums[end] = nums[i];
                dup1 = dup2;
                dup2 = nums[i];
                end++;
            }
        }
        return end;
    }
```

## Search in Rotated Sorted Array II	
https://segmentfault.com/a/1190000003811864

Suppose an array sorted in ascending order is rotated at some pivot unknown to you beforehand.

(i.e., [0,0,1,2,2,5,6] might become [2,5,6,0,0,1,2]).

You are given a target value to search. If found in the array return true, otherwise return false.
```
Example 1:

Input: nums = [2,5,6,0,0,1,2], target = 0
Output: true

Example 2:

Input: nums = [2,5,6,0,0,1,2], target = 3
Output: false
```

Follow up:

This is a follow up problem to Search in Rotated Sorted Array, where nums may contain duplicates.
Would this affect the run-time complexity? How and why?

思路
如果可能有重复，那我们之前判断左右部分是否有序的方法就失效了，因为可能有这种13111情况，虽然起点小于等于中间，但不代表右边就不是有序的，因为中点也小于等于终点，所有右边也是有序的。所以，如果遇到这种中点和两边相同的情况，我们两边都要搜索。

```java
public class Solution {
    public boolean search(int[] nums, int target) {
        return helper(nums, 0, nums.length - 1, target);
    }
    
    public boolean helper(int[] nums, int min, int max, int target){
        int mid = min + (max - min) / 2;
        // 不符合min <= max则返回假
        if(min > max){
            return false;
        }
        if(nums[mid] == target){
            return true;
        }
        boolean left = false, right = false;
        // 如果左边是有序的
        if(nums[min] <= nums[mid]){
            // 看目标是否在左边有序数列中
            if(nums[min] <= target && target < nums[mid]){
                left = helper(nums, min, mid - 1, target);
            } else {
                left = helper(nums, mid + 1, max, target);
            }
        }
        // 如果右边也是有序的
        if(nums[mid] <= nums[max]){
            // 看目标是否在右边有序数列中
            if(nums[mid] < target && target <= nums[max]){
                right = helper(nums, mid + 1, max, target);
            } else {
                right = helper(nums, min, mid - 1, target);
            }
        }
        // 左右两边有一个包含目标就返回真
        return left || right;
    }
}
```


## 82. Remove Duplicates from Sorted List II
Given a sorted linked list, delete all nodes that have duplicate numbers, leaving only distinct numbers from the original list.
```
Example 1:

Input: 1->2->3->3->4->4->5
Output: 1->2->5

Example 2:

Input: 1->1->1->2->3
Output: 2->3
```


```java
public static class Solution1 {
    public ListNode deleteDuplicates(ListNode head) {
      if (head == null) {
        return head;
      }
      ListNode fakeHead = new ListNode(-1);
      fakeHead.next = head;
      ListNode pre = fakeHead;
      ListNode curr = head;
      while (curr != null) {
        while (curr.next != null && curr.val == curr.next.val) {
          curr = curr.next;
        }
        if (pre.next == curr) {
          pre = pre.next;
        } else {
          pre.next = curr.next;
        }
        curr = curr.next;
      }
      return fakeHead.next;
    }
  }

```


## 83. Remove Duplicates from Sorted List

Given a sorted linked list, delete all duplicates such that each element appear only once.
```
Example 1:

Input: 1->1->2
Output: 1->2

Example 2:

Input: 1->1->2->3->3
Output: 1->2->3
```


```java
  public static class Solution1 {
    public ListNode deleteDuplicates(ListNode head) {
      ListNode ret = new ListNode(-1);
      ret.next = head;
      while (head != null) {
        while (head.next != null && head.next.val == head.val) {
          head.next = head.next.next;
        }
        head = head.next;
      }
      return ret.next;
    }
  }

  public static class Solution2 {
    public ListNode deleteDuplicates(ListNode head) {
      ListNode curr = head;
      while (curr != null && curr.next != null) {
        if (curr.val == curr.next.val) {
          curr.next = curr.next.next;
        } else {
          curr = curr.next;
        }
      }
      return head;
    }
  }
```



## 84. Largest Rectangle in Histogram
同leetcode 11不同要区分清楚概念。
https://leetcode.com/problems/largest-rectangle-in-histogram/description/
Given n non-negative integers representing the histogram's bar height where the width of each bar is 1, find the area of largest rectangle in the histogram.
![](https://leetcode.com/static/images/problemset/histogram.png)
Above is a histogram where width of each bar is 1, given height = [2,1,5,6,2,3].

![](https://leetcode.com/static/images/problemset/histogram_area.png)
The largest rectangle is shown in the shaded area, which has area = 10 unit.

```
Example:
Input: [2,1,5,6,2,3]
Output: 10
```

思路参考：
http://www.cnblogs.com/grandyang/p/4322653.html

1、遍历数组，每找到一个局部峰值，然后向前遍历所有的值，算出共同的矩形面积，每次对比保留最大值
```
class Solution {
public:
    int largestRectangleArea(vector<int> &height) {
        int res = 0;
        for (int i = 0; i < height.size(); ++i) {
            if (i + 1 < height.size() && height[i] <= height[i + 1]) {
                continue;
            }
            int minH = height[i];
            for (int j = i; j >= 0; --j) {
                minH = min(minH, height[j]);
                int area = minH * (i - j + 1);
                res = max(res, area);
            }
        }
        return res;
    }
};
```

2、stack 同1方法思想雷同
```java
    /**
     * credit: https://leetcode.com/articles/largest-rectangle-histogram/#approach-5-using-stack-accepted
     * and https://discuss.leetcode.com/topic/7599/o-n-stack-based-java-solution
     */
    public int largestRectangleArea(int[] heights) {
      int len = heights.length;
      Stack<Integer> s = new Stack<>();
      int maxArea = 0;
      for (int i = 0; i <= len; i++) {
        int h = (i == len ? 0 : heights[i]);
        if (s.isEmpty() || h >= heights[s.peek()]) {
          s.push(i);
        } else {
          int tp = s.pop();
          maxArea = Math.max(maxArea, heights[tp] * (s.isEmpty() ? i : i - 1 - s.peek()));
          i--;
        }
      }
      return maxArea;
    }
  }

```


## 85. Maximal Rectangle

Given a 2D binary matrix filled with 0's and 1's, find the largest rectangle containing only 1's and return its area.
```
Example:

Input:
[
  ["1","0","1","0","0"],
  ["1","0","1","1","1"],
  ["1","1","1","1","1"],
  ["1","0","0","1","0"]
]
Output: 6
```

解析1，https://www.cnblogs.com/lichen782/p/leetcode_maximal_rectangle.html
复杂度：o(n^3)
![](https://images0.cnblogs.com/blog/466943/201307/20223403-64ab2f344d24409786bca2cdb184042b.png)
如上从上到下遍历，参考代码，然后对每一个坐标进行遍历寻找最大即可。
```java
/**
     * 以给出的坐标作为左上角，计算其中的最大矩形面积
     * @param matrix
     * @param row 给出坐标的行
     * @param col 给出坐标的列
     * @return 返回最大矩形的面积
     */
    private int maxRectangle(char[][] matrix, int row, int col) {
        int minWidth = Integer.MAX_VALUE;
        int maxArea = 0;
        for (int i = row; i < matrix.length && matrix[i][col] == '1'; i++) {
            int width = 0;
            while (col + width < matrix[row].length
                    && matrix[i][col + width] == '1') {
                width++;
            }
            if (width < minWidth) {// 如果当前宽度小于了以前的最小宽度，更新它，为下面的矩形计算做准备
                minWidth = width;
            }
            int area = minWidth * (i - row + 1);
            if (area > maxArea)
                maxArea = area;
        }
        return maxArea;
    }

    public int maximalRectangle(char[][] matrix) {
        // Start typing your Java solution below
        // DO NOT write main() function
        int m = matrix.length;
        int n = m == 0 ? 0 : matrix[0].length;
        int maxArea = 0;
        for(int i = 0; i < m; i++){//row
            for(int j = 0; j < n; j++){//col
                if(matrix[i][j] == '1'){
                    int area = maxRectangle(matrix, i, j);
                    if(area > maxArea) maxArea = area;
                }
            }
        }
        return maxArea;
     }
```

思路：把每一行都计算上边的直方图，然后对每一行求解最大即可。
这题的解法基于上题。要求最大的矩形，实际上可以将矩阵的每一行，转化为上一题的直方图，而直方图的每个竖条的数字，就是该行对应坐标正上方，向上方向有多少个连续的1。要转化为直方图，方法是每一行的数值都累加上一行计算出来的数值，而第一行的数值就是本身。如果原始矩阵中遇到0，则累加中断，重新置0。

![](https://images0.cnblogs.com/blog/466943/201307/20232015-42cfb4fde2c34119b1811953e34d4193.png)

```java
public class Solution {
    public int maximalRectangle(char[][] matrix) {
        int max = 0;
        if(matrix.length == 0) return 0;
        int[][] dp = new int[matrix.length][matrix[0].length];
        for(int i = 0; i < matrix.length; i++){
            for(int j = 0; j < matrix[0].length; j++){
                // 如果是第一行就是自身，如果遇到0则停止累加
                dp[i][j] =  i == 0 ? matrix[i][j] - '0' : matrix[i][j] == '1' ? dp[i-1][j] + matrix[i][j] - '0' : 0;
            }
        }
        for(int i = 0; i < dp.length; i++){
            //找每行的最大矩形
            int tmp = findRowMax(i, dp);
            max = Math.max(max, tmp);
        }
        return max;
    }
    
    private int findRowMax(int row, int[][] matrix){
        if(matrix[row].length== 0) return 0;
        Stack<Integer> stk = new Stack<Integer>();
        int i = 1, max = matrix[row][0];
        stk.push(0);
        while(i < matrix[row].length || (i == matrix[row].length && !stk.isEmpty())){
            if(i != matrix[row].length && ( stk.isEmpty() || matrix[row][i] >= matrix[row][stk.peek()] )){
                stk.push(i);
                i++;
            } else {
                int top = matrix[row][stk.pop()];
                int currMax = !stk.isEmpty() ? top * (i - stk.peek() - 1) : top * i;
                max = Math.max(currMax, max);
            }
        }
        return max;
    }
}

```

## 86. Partition List
Given a linked list and a value x, partition it such that all nodes less than x come before nodes greater than or equal to x.

You should preserve the original relative order of the nodes in each of the two partitions.
```
Example:

Input: head = 1->4->3->2->5->2, x = 3
Output: 1->2->2->4->3->5
```

思路：
就是将所有小于给定值的节点取出组成一个新的链表，此时原链表中剩余的节点的值都大于或等于给定值，只要将原链表直接接在新链表后即可，代码如下：
```java
  public static class Solution1 {
    public ListNode partition(ListNode head, int x) {
      if (head == null || head.next == null) {
        return head;
      }
      ListNode left = new ListNode(0);
      ListNode right = new ListNode(0);
      ListNode less = left;
      ListNode greater = right;
      while (head != null) {
        if (head.val < x) {
          less.next = head;
          less = less.next;
        } else {
          greater.next = head;
          greater = greater.next;
        }
        head = head.next;
      }
      greater.next = null;
      less.next = right.next;
      return left.next;
    }
  }
```

另一种思路是把所有小于给定值的节点都移到前面，大于该值的节点顺序不变，相当于一个局部排序的问题。那么可以想到的一种解法是首先找到第一个大于或等于给定值的节点，用题目中给的例子来说就是先找到4，然后再找小于3的值，每找到一个就将其取出置于4之前即可。


##　87. Scramble String

```
Given a string s1, we may represent it as a binary tree by partitioning it to two non-empty substrings recursively.

Below is one possible representation of s1 = "great":

    great
   /    \
  gr    eat
 / \    /  \
g   r  e   at
           / \
          a   t
To scramble the string, we may choose any non-leaf node and swap its two children.

For example, if we choose the node "gr" and swap its two children, it produces a scrambled string "rgeat".

    rgeat
   /    \
  rg    eat
 / \    /  \
r   g  e   at
           / \
          a   t
We say that "rgeat" is a scrambled string of "great".

Similarly, if we continue to swap the children of nodes "eat" and "at", it produces a scrambled string "rgtae".

    rgtae
   /    \
  rg    tae
 / \    /  \
r   g  ta  e
       / \
      t   a
We say that "rgtae" is a scrambled string of "great".

Given two strings s1 and s2 of the same length, determine if s2 is a scrambled string of s1.

Example 1:

Input: s1 = "great", s2 = "rgeat"
Output: true

Example 2:

Input: s1 = "abcde", s2 = "caebd"
Output: false
```


## 88. Merge Sorted Array	

Given two sorted integer arrays nums1 and nums2, merge nums2 into nums1 as one sorted array.

Note:

The number of elements initialized in nums1 and nums2 are m and n respectively.
You may assume that nums1 has enough space (size that is greater or equal to m + n) to hold additional elements from nums2.
```
Example:

Input:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

Output: [1,2,2,3,5,6]
```


算法思想是：由于合并后A数组的大小必定是m+n，所以从最后面开始往前赋值，先比较A和B中最后一个元素的大小，把较大的那个插入到m+n-1的位置上，再依次向前推。如果A中所有的元素都比B小，那么前m个还是A原来的内容，没有改变。如果A中的数组比B大的，当A循环完了，B中还有元素没加入A，直接用个循环把B中所有的元素覆盖到A剩下的位置。代码如下:

```c
class Solution {
public:
    void merge(int A[], int m, int B[], int n) {
        int count = m + n - 1;
        --m; --n;
        while (m >= 0 && n >= 0) A[count--] = A[m] > B[n] ? A[m--] : B[n--];
        while (n >= 0) A[count--] = B[n--];
    }
};
```

## 89. Gray Code	
格雷码

The gray code is a binary numeral system where two successive values differ in only one bit.

Given a non-negative integer n representing the total number of bits in the code, print the sequence of gray code. A gray code sequence must begin with 0.


```
Example 1:

Input: 2
Output: [0,1,3,2]
Explanation:
00 - 0
01 - 1
11 - 3
10 - 2

For a given n, a gray code sequence may not be uniquely defined.
For example, [0,2,3,1] is also a valid gray code sequence.

00 - 0
10 - 2
11 - 3
01 - 1

Example 2:

Input: 0
Output: [0]
Explanation: We define the gray code sequence to begin with 0.
             A gray code sequence of n has size = 2n, which for n = 0 the size is 20 = 1.
             Therefore, for n = 0 the gray code sequence is [0].
```

格雷码相互转换：
https://www.cnblogs.com/zhihongyu/archive/2012/08/14/2638781.html

```
        static int DecimaltoGray(int x)
        {
            return x^(x>>1);
        }

        static unsigned int GraytoDecimal(unsigned int x)
        {
            unsigned int y = x;
            while(x>>=1) 
                y ^= x;
            return y;
        }
```

那么本题得解法直接利用int to gray num即可
```c
// Binary to grey code
class Solution {
public:
    vector<int> grayCode(int n) {
        vector<int> res;
        for (int i = 0; i < pow(2,n); ++i) {
            res.push_back((i >> 1) ^ i);
        }
        return res;
    }
};
```

## 90. Subsets II	
Given a collection of integers that might contain duplicates, nums, return all possible subsets (the power set).

Note: The solution set must not contain duplicate subsets.
```
Example:

Input: [1,2,2]
Output:
[
  [2],
  [1],
  [1,2,2],
  [2,2],
  [1,2],
  []
]
```

```java
    public static class Solution2 {
        public List<List<Integer>> subsetsWithDup(int[] nums) {
            List<List<Integer>> result = new ArrayList();
            Arrays.sort(nums);
            backtrack(nums, 0, new ArrayList(), result);
            return result;
        }

        void backtrack(int[] nums, int start, List<Integer> curr, List<List<Integer>> result) {
            result.add(new ArrayList(curr));
            for (int i = start; i < nums.length; i++) {
                if (i > start && nums[i] == nums[i - 1]) {
                    continue;
                }
                curr.add(nums[i]);
                backtrack(nums, i + 1, curr, result);
                curr.remove(curr.size() - 1);
            }
        }
    }
```

## 91. Decode Ways	
A message containing letters from A-Z is being encoded to numbers using the following mapping:
```
'A' -> 1
'B' -> 2
...
'Z' -> 26
Given a non-empty string containing only digits, determine the total number of ways to decode it.

Example 1:

Input: "12"
Output: 2
Explanation: It could be decoded as "AB" (1 2) or "L" (12).

Example 2:

Input: "226"
Output: 3
Explanation: It could be decoded as "BZ" (2 26), "VF" (22 6), or "BBF" (2 2 6).
```

思路：dp
```java

public class Solution {
    public int numDecodings(String s) {
        if(s.length() == 0) return s.length();
        int[] dp = new int[s.length() + 1];
        // 初始化第一种解码方式
        dp[0] = 1;
        // 如果第一位是0，则无法解码
        dp[1] = s.charAt(0) == '0' ? 0 : 1;
        for(int i = 2; i <= s.length(); i++){
            // 如果字符串的第i-1位和第i位能组成一个10到26的数字，说明我们可以在第i-2位的解码方法上继续解码
            if(Integer.parseInt(s.substring(i-2, i)) <= 26 && Integer.parseInt(s.substring(i-2, i)) >= 10){
                dp[i] += dp[i - 2];
            }
            // 如果字符串的第i-1位和第i位不能组成有效二位数字，在第i-1位的解码方法上继续解码
            if(Integer.parseInt(s.substring(i-1, i)) != 0){
                dp[i] += dp[i - 1];
            }
        }
        return dp[s.length()];
    }
}

    public static class Solution1 {
      public int numDecodings(String s) {
        if (s == null || s.length() == 0) {
          return 0;
        }
        int[] dp = new int[s.length() + 1];
        dp[0] = 1;
        dp[1] = (s.charAt(0) != '0') ? 1 : 0;
        for (int i = 2; i <= s.length(); i++) {
          int first = Integer.valueOf(s.substring(i - 1, i));
          int second = Integer.valueOf(s.substring(i - 2, i));
          if (first > 0 && first <= 9) {
            dp[i] += dp[i - 1];
          }
          if (second >= 10 && second <= 26) {
            dp[i] += dp[i - 2];
          }
        }
        return dp[s.length()];
      }
    }
```


## 92. Reverse Linked List II



## 94. Binary Tree Inorder Traversal
```
Given a binary tree, return the inorder traversal of its nodes' values.

Example:

Input: [1,null,2,3]
   1
    \
     2
    /
   3

Output: [1,3,2]
Follow up: Recursive solution is trivial, could you do it iteratively?
```

```java
    public static class Solution1 {
        public List<Integer> inorderTraversal(TreeNode root) {
            return inorder(root, new ArrayList());
        }

        List<Integer> inorder(TreeNode root, List<Integer> result) {
            if (root == null) {
                return result;
            }
            inorder(root.left, result);
            result.add(root.val);
            return inorder(root.right, result);
        }
    }

    public static class Solution2 {
        //iterative approach
        public List<Integer> inorderTraversal(TreeNode root) {
            List<Integer> result = new ArrayList();
            Stack<TreeNode> stack = new Stack();
            while (root != null || !stack.isEmpty()) {
                while (root != null) {
                    stack.push(root);
                    root = root.left;
                }
                root = stack.pop();
                result.add(root.val);
                root = root.right;
            }
            return result;
        }
    }
```

## 
