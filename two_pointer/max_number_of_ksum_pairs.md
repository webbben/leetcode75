# Max Number of K-Sum Pairs

[leetcode link](https://leetcode.com/problems/max-number-of-k-sum-pairs)

5/7/2024

Difficulty: Medium

## Description

You are given an integer array `nums` and an integer `k`.

In one operation, you can pick two numbers from the array whose sum equals `k` and remove them from the array.

Return the maximum number of operations you can perform on the array.

**Example 1:**

```
Input: nums = [1,2,3,4], k = 5
Output: 2
Explanation: Starting with nums = [1,2,3,4]:
- Remove numbers 1 and 4, then nums = [2,3]
- Remove numbers 2 and 3, then nums = []
There are no more pairs that sum up to 5, hence a total of 2 operations.
```

**Example 2:**

```
Input: nums = [3,1,3,4,3], k = 6
Output: 1
Explanation: Starting with nums = [3,1,3,4,3]:
- Remove the first two 3's, then nums = [1,4,3]
There are no more pairs that sum up to 6, hence a total of 1 operation.
```

**Constraints:**

-   `1 <= nums.length <= 10^5`
-   `1 <= nums[i] <= 10^9`
-   `1 <= k <= 10^9`

## Initial Thoughts / Planning

With this problem, it seems like there are two things you have to handle for:

1. finding a pair that adds up to `k`

I can imagine a couple different ways to find pairs. We could of course do a "brute force" style approach where for each element, we compare it to the other elements to find another number that would sum up to `k`. But that would be an O(n^2) solution, which is no good.

I can also imagine a two pointer solution where we point at the first element, then dive deeper to find a corresponding number that would sum to `k`: but I'm not sure exactly on what the logic would be for when to skip a number. I also think there must be some additional trick necessary to make that work.

2. removing those pairs as they are found, or somehow avoiding re-using them

I think the "brute force" would be to actual modify the array and remove them from it. However, as we know, removing elements from an array is a bit inefficient, so it would be best to avoid that; otherwise, we'd need to do that operation on potentially every element in the array, which would slow things down considerably.

Another idea is, we could use some extra memory to track which indices have been used in a pair already. Perhaps just a `map[int]bool`, where as an index is used in a sum, we map it to `true`. This would be more efficient since we avoid the expensive appends/removal of items from arrays, but we also would incur an O(n) space complexity.

So, to summarize these two solution ideas:

**Brute Force:**

For each number, iterate over the `nums` array to find a matching number that sums up to `k`. Once found, remove both of those numbers from `nums`. Repeat until you get to the end of `nums`, or `nums` becomes empty. Removing elements from the array, while maintaining the relative order of elements, is actually pretty inefficient; it seems that it takes linear time in Go ([source](https://programming.guide/go/delete-element-slice.html)). So actually, this solution might be more in the ballpark of **O(n^3)**.

We could use extra memory (the index map idea) to reduce this back down to O(n^2), but anyway, not ideal either way.

**Two Pointer:**

For this, I actually just had an idea: if we first sort the array, then it becomes a lot easier to find matches. Firstly, we know that all numbers in `nums` are greater than or equal to 1, so we don't need to worry about handling negative values. We can have two pointers, one on the `left` side (lower value numbers) and one on the `right` side (higher value numbers), and we can adjust them based on if the current sum of `nums[left] + nums[right]`. If the sum is too large, then we decrement `right` to find a smaller value, and if the sum is too low, we increment `left` to find a larger value.
If the two pointers meet, then the entire array has been searched, and we return the number of matches that were found.

This solution would be linear time complexity and constant space complexity, since we only iterate over the array once, and we only have a couple variables we use for pointers and keeping a count of matches.

## Code Implementations, Optimizations

Here's my implementation of the two pointer solution described above:

```go
import (
    "sort"
)

func maxOperations(nums []int, k int) int {
    sort.Ints(nums)

    left, right := 0, len(nums) - 1
    count := 0

    for left < right {
        sum := nums[left] + nums[right]
        if sum == k {
            count++
            left++
            right--
            continue
        }
        if sum < k {
            // too small
            left++
        } else {
            // too big
            right--
        }
    }
    return count
}
```

This solution appears to work pretty average-ly compared to other submissions on leetcode. I think that overall this is probably on the right track for the optimal solution, but let's dig into it a bit more.

First, the sorting:

I looked into how `sort.Ints` works, and found the [source code](https://go.dev/src/slices/zsortordered.go#:~:text=func-,pdqsortOrdered,-%5BE%20cmp.Ordered) which describes it as a `pdquicksort` algorithm ("pattern defeating quicksort"). I believe this is a quicksort algorithm that has some additional optimization done to it, and a quick google search tells me that quicksort, on average, is a time complexity of `O(n*logn)`. This is what I was pretty sure is the standard time complexity for good sorting algorithms, but good to check.

From there, the actual body of the code is just a two pointer solution that loops once over the contents of `nums`. This means a time complexity there of O(n). Space complexity is O(1), since we only create a few variables to track the pointers, count, and also the sum (for more readability).

Putting this together: `O(n*logn)` + `O(n)` would just be simplified as `O(n*logn)`, since nothing is nested here.

### Optimizing this solution

So the first thing I'd look at is if there is a way to minimize the number of operations we do, probably via some early return cases.

One idea is to quit early once `nums[left]` and `nums[right]` are equal, but don't sum up to `k`.

Another idea would be to early return if `k` is less than or equal to `nums[left]`. On a similar note, we could also return early if `k` is greater than `2 * nums[right]`, since at that point we know there's no way to get a pair that would reach the value of `k`.

Here's the updated solution, with these small optimizations that may reduce the number of operations:

```go
import (
    "sort"
)

func maxOperations(nums []int, k int) int {
    sort.Ints(nums)

    left, right := 0, len(nums) - 1
    count := 0

    for left < right {
        if nums[left] >= k {
            return count
        }
        if k > nums[right] * 2 {
            return count
        }
        sum := nums[left] + nums[right]
        if sum == k {
            count++
            left++
            right--
            continue
        }
        if nums[left] == nums[right] {
            return count
        }
        if sum < k {
            // too small
            left++
        } else {
            // too big
            right--
        }
    }
    return count
}
```

## Let's ask Chat-GPT

Showing Chat-GPT my solution, it seemed to like it. However, when I pushed it for further optimization ideas, it suggested that instead of sorting the `nums` array first, we could use a different approach where we count the frequency of each number, and then iterate through the frequency map to find pairs. Here's the solution it provided:

```go
func maxOperations(nums []int, k int) int {
    freq := make(map[int]int)

    for _, num := range nums {
        freq[num]++
    }

    count := 0
    for num, f := range freq {
        complement := k - num
        if complement == num {
            count += f / 2
        } else if freq[complement] > 0 {
            minFreq := min(f, freq[complement])
            count += minFreq
            freq[num] -= minFreq
            freq[complement] -= minFreq
        }
    }

    return count
}
```

Time Complexity: O(n)

Space Complexity: O(n)

Compared to our first solution, this is a bit more optimized for time complexity, but at the expense of space complexity, since it needs to keep a map of the frequencies of each number in the array.
I think that overall, I prefer my first solution, just because I don't know that the potential speed improvement justifies the increased memory usage. However, I do like how this solution takes a different approach I hadn't considered.

## Final Solution, Conclusion

Sticking with the slightly optimized version of our original solution:

```go
import (
    "sort"
)

func maxOperations(nums []int, k int) int {
    sort.Ints(nums)

    left, right := 0, len(nums) - 1
    count := 0

    for left < right {
        if nums[left] >= k {
            return count
        }
        if k > nums[right] * 2 {
            return count
        }
        sum := nums[left] + nums[right]
        if sum == k {
            count++
            left++
            right--
            continue
        }
        if nums[left] == nums[right] {
            return count
        }
        if sum < k {
            // too small
            left++
        } else {
            // too big
            right--
        }
    }
    return count
}
```

Time Complexity: `O(n*logn)`

Space Complexity: `O(1)`

Main Takeaway: Sometimes you can do some pre-processing to simplify a problem, or ultimately make it more efficient. In this case, working with a sorted array makes the problem dramatically simpler, so although it does add a little bit of extra work, in the end it actually improves the overall solution. I think in general, when it comes to a problem where you are searching for some sort of mathematical pair, sorting the array should be a consideration so that you can then mathematically determine when to do things like early returns too.

Ben's difficulty rating: 6/10.
