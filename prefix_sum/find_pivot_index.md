# Find Pivot Index

[leetcode link](https://leetcode.com/problems/find-pivot-index/)

5/10/2024

Difficulty: Easy

## Description

Given an array of integers `nums`, calculate the **pivot index** of this array.

The pivot index is the index where the sum of all numbers strictly to the left of the index is equal to the sum of all the numbers strictly to the index's right.

If the index is on the left edge of the array, then the left sum is `0` because there are no elements to the left. This also applies to the right edge of the array.

Return the leftmost pivot index. If no such index exists, return `-1`.

**Example 1:**

```
Input: nums = [1,7,3,6,5,6]
Output: 3
Explanation:
The pivot index is 3.
Left sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11
Right sum = nums[4] + nums[5] = 5 + 6 = 11
```

**Example 2:**

```
Input: nums = [1,2,3]
Output: -1
Explanation:
There is no index that satisfies the conditions in the problem statement.
```

**Example 3:**

```
Input: nums = [2,1,-1]
Output: 0
Explanation:
The pivot index is 0.
Left sum = 0 (no elements to the left of index 0)
Right sum = nums[1] + nums[2] = 1 + -1 = 0
```

**Constraints:**

-   `1 <= nums.length <= 10^4`
-   `-1000 <= nums[i] <= 1000`

## Initial Thoughts / Planning

For this problem, we need to find the index where the sum of elements before it is equal to the sum of elements after it.

We already know how to get the sum of the values up to a certain index in the array; we did just that in the last problem. However, to also know the sum of the elements after that element is a bit more complicated.

We _could_ do a brute force solution where for each element, we subsequently count all the rest of the elements in the array after it. If we did this repeatedly for each element in the array though, it would be a time complexity of O(n^2) (more or less). And that's no good.

But if you think about it more, you can start to see the solution. Suppose we are calculating for position `i` in `nums`. We know the current sum of values before `i`, as we've been counting it up as we go. Now, to get the sum of all elements after `i`, we go ahead and iterate through the rest of the array, counting up the values. This means we know the sum from `i+1` to `n`.

What if we move to the next position - `i + 1` - and start the same process? Doesn't it seem like we could re-use the information we got from the last iteration for finding the sum from `i + 2` to `n`?
What if we found a way so we don't have to recount all the same values over and over again?

One idea I have is, we could first iterate over the whole array, and get the sum of all elements in the array. Then, I believe we could subtract the running sum we are gathering from the left side, and also subtract the current element at index `i`, to get the rest of the sum from `i + 1` onward.

Let's take this example array:

`[1,5,2,-3,6,3]`

The `totalSum` of all elements in the array is `14`.

If we are at index `i = 2`, then the running sum to the left would be `6` (1 + 5). So, in theory if we took `totalSum - 6 - 2`, I think it should be the sum of all elements to the right of `i = 2`.
And sure enough it is! `14 - 6 - 2` indeed equals `6`.

## My Code Implementations, Optimizations

```go
func pivotIndex(nums []int) int {
    totalSum := 0
    for i := 0; i < len(nums); i++ {
        totalSum += nums[i]
    }
    leftSum := 0
    for i := 0; i < len(nums); i++ {
        if leftSum == totalSum - leftSum - nums[i] {
            return i
        }
        leftSum += nums[i]
    }
    return -1
}
```

Time Complexity: `O(n)` - even though there are two loops over `nums`, since they aren't nested it's still linear time.

Space Complexity: `O(1)` - only constant space is used.

I'm pretty satisfied with this solution. I think - hence the title of this group being "prefix sum" - that one potential solution is to first calculate the "suffix sums" of all indices in `nums`. This means you would calculate the sum of all numbers _after_ a given position. Then you could use that to know the sum after position `i`. But here we actually ended up skipping that altogether for a better, constant space solution.

I'm pretty confident in this solution, so I think I'll skip the extra research steps and stick with this solution.

## Final Solution, Conclusion

Final solution is same as the first solution!

This problem was a cool example of finding the optimal solution by first considering the brute force solution, analyzing it further to identify the sub-problems involved, and then finding a way to pre-calculate so we don't have to repeatedly do the same calculations and searches each iteration.

Ben's difficulty rating: 2.7/10. This could potentially be a little difficult if you aren't already used to the idea of prefix sums and things like that. Lucky for me I think I've done a similar problem not too long ago.
