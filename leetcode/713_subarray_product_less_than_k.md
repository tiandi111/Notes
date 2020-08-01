713.Subarray Product Less Than K
---
问题:
Your are given an array of positive integers nums.

Count and print the number of (contiguous) subarrays where the product of all the elements in the subarray is less than k.

关键条件：
1. 必须是连续的subarray
2. 元素全部是正整数

思路：
subarray可以这样不重叠的分组：所有以arr[0]结尾的subarray，所有以arr[1]结尾的subarray，所有以arr[2]结尾的subarray...

问题变成了 => 对每个元素arr[i]，找到在它前面的，离他最远的一个元素arr[j]，则j-i+1就是所有以arr[i]结尾的满足条件的subarray

用双指针解决很方便。

Leetcode Solution：

Algorithm

Our loop invariant is that left is the smallest value so that the product in the window prod = nums[left] * nums[left + 1] * ... * nums[right] is less than k.

For every right, we update left and prod to maintain this invariant. Then, the number of intervals with subarray product less than k and with right-most coordinate right, is right - left + 1. We'll count all of these for each value of right.