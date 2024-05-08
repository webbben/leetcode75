# Maximum Average Subarray I

[leetcode link](https://leetcode.com/problems/maximum-average-subarray-i/)

5/8/2024

Difficulty: Easy

## Description

You are given an integer array nums consisting of `n` elements, and an integer `k`.

Find a contiguous subarray whose length is equal to `k` that has the maximum average value and return this value. Any answer with a calculation error less than `10^-5` will be accepted.

**Example 1:**

```
Input: nums = [1,12,-5,-6,50,3], k = 4
Output: 12.75000
Explanation: Maximum average is (12 - 5 - 6 + 50) / 4 = 51 / 4 = 12.75
```

**Example 2:**

```
Input: nums = [5], k = 1
Output: 5.00000
```

**Constraints:**

-   `n == nums.length`
-   `1 <= k <= n <= 105`
-   `-10^4 <= nums[i] <= 10^4`

## Initial Thoughts / Planning

So, I think the first obvious hint is that this is the first problem of the sliding window section - so, drumroll: gonna assume we are using a sliding window approach.

I think this will be pretty simple overall. We will have a sliding window of size `k` (two pointers - a `left` and `right`, `right` leading the front of the window).
First, we actually need to get the window to size `k` to initialize things. Once it's been expanded to encompass indices `0` to `k - 1`, we then begin sliding the window.

Sliding the window just means that `right` moves over and incorporates a new number by adding it to a running `sum`, and `left` slides over, _removing_ the number it was previously on from `sum`.
Then, we quickly calculate the current average by dividing `sum / k`, and if this average is greater than the current running `maxAve`, we set it as the new `maxAve`.

At the end, we return `maxAve`.

## Code Implementations, Optimizations

```go
func findMaxAverage(nums []int, k int) float64 {
    left, right := 0, k - 1
    // calculate average of first k values
    curSum := 0
    for i := 0; i < k; i++ { curSum += nums[i] }
    maxAve := float64(curSum) / float64(k)

    // now find averages for all the rest of the values in nums
    for right < len(nums) - 1 {
        right++
        curSum += nums[right]
        curSum -= nums[left]
        left++
        maxAve = max(maxAve, float64(curSum) / float64(k))
    }
    return maxAve
}
```

Time Complexity: O(n)

Space Complexity: O(1)

I think in terms of time and space complexity, there isn't any way to further improve this, since we need to iterate over the array at least once. It also looks like this solution performs pretty well according to the performance stats on leetcode.

I think we'll skip the usual song and dance of talking to Chat-GPT since I don't think there is much to improve here beyond maybe some micro-optimizations.

## Final Solution, Conclusion

(final solution is the same as our first implementation above)

Big takeaways: sliding windows! the sliding window will be useful for problems like this, where we are finding a subarray or a continguous group of elements in a larger element. The main idea is that you can keep track of that "sub-array" / group of elements by only doing operations to add or remove them to the group, rather than repeatedly iterating over all the elements in the group we are maintaining.
