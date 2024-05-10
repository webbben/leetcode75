# Max Consecutive Ones III

[leetcode link](https://leetcode.com/problems/max-consecutive-ones-iii/)

5/9/2024

Difficulty: Medium

## Description

Given a binary array `nums` and an integer `k`, return the maximum number of consecutive `1`'s in the array if you can flip at most `k` `0`'s.

**Example 1:**

```
Input: nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2
Output: 6
Explanation: [1,1,1,0,0,1,1,1,1,1,1]
Bolded numbers were flipped from 0 to 1. The longest subarray is underlined.
```

**Example 2:**

```
Input: nums = [0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], k = 3
Output: 10
Explanation: [0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,1,1,1,1]
Bolded numbers were flipped from 0 to 1. The longest subarray is underlined.
```

**Constraints:**

-   `1 <= nums.length <= 10^5`
-   `nums[i]` is either `0` or `1`.
-   `0 <= k <= nums.length`

## Initial Thoughts / Planning

Ok, so real quick let's clarify the problem:

One main thing that differs here compared to the previous two sliding window problems is that here, the sliding window is not a constant size; it can be however big we want. **But**, it can't just be the entire array's length, because we can **only allow** `k` zeroes in the window/sequence. Since `k` zeroes can be flipped, that means any more than `k` zeroes in the sequence and we no longer have a consecutive sequence of ones.

Now let's discuss our plan:

This will of course still be a sliding window problem I believe, but the main difference is that instead of always moving the front and back pointers forward, we will only move the front when the window has less zeroes than our limit; in other words, when `numZeroes < k`, we can proceed to expand the window. If `numZeroes >= k` then we instead need to move the back pointer forward to shrink the window (I add the `>` just as a "sanity check", even though it should be unnecessary...). We would continue to shrink the window until we've removed enough zeroes to be within `k` again, and then repeat this process of expanding and shrinking until we reach the end of the array, all the while recording the longest valid window size.

> We don't need to worry about _actually_ flipping any zeroes to ones, we just understand that that would implicitly be done, and in terms of solving this problem the idea of "flipping" is mostly a distraction. We can think of `k` instead as just a limit of allowed zeroes in our consecutive sequence of ones.

## Code Implementations, Optimizations

Here's my implementation so far:

```go
func longestOnes(nums []int, k int) int {
    if len(nums) == k {
        return k
    }
    left, right := 0, 0
    zeroCount := 0
    longest := 0
    if nums[0] == 0 {
        zeroCount++
    }

    for right < len(nums) - 1 {
        if zeroCount <= k {
            right++
            if nums[right] == 0 {
                zeroCount++
            }
        } else {
            left++
            if nums[left - 1] == 0 {
                zeroCount--
            }
        }
        // if this is a valid sequence, record its length
        if zeroCount <= k {
            if right - left + 1 > longest {
                longest = right - left + 1
            }
        }
    }
    return longest
}
```

Time Complexity: `O(n)`, since we don't have any nested looping going on for the `nums` array, and all other operations are constant time.

Space Complexity: `O(1)`

So in the beginning of the function we handle for some specific cases. First, if `k` is the same as the length of `nums`, then we already know that the maximum sequence length is `k`; even if the entire `nums` array were zeroes, all of those zeroes could be switched to ones.

Due to how the logic works for my for loop, we also have to check if the first number is a zero, since the logic of the for loop doesn't handle that initial step.

Then, inside the loop we:

-   check if we are within the zero limit `k`:
    -   if we are, we can expand the window; we also then check if a new zero has been incorporated into our sequence/window.
-   if we're actually **above** the zero limit, then we need to shrink the window by bringing up the caboose; so left moves forward and checks if a zero was dropped from the sequence/window.
-   then, after either left or right has moved (and potentially incorporated or dropped a zero), we once again check if we are within the zero limit `k`
    -   if we are, then we check if our current sequence size is the longest so far, and record it if so.

I have a gut sense that maybe this logic could be simplified a little bit, maybe since we are checking `zeroCount <= k` twice, but then again I think the logic is fine this way since it clearly has a "take a step" part, then a "check the size of the sequence" part, which is easy enough to understand by looking at it.

### Optimizing this solution

I think overall our solution is pretty good, and probably close to the optimal solution. But let's take a minute to explore any possible optimizations.

Perhaps there is a way we can return early? One basic early return case would be, if the current size of the window plus the remaining amount of unsearched space in the array is less than the current `longest` value, we know it's impossible to get a new `longest`. I don't know how much this will save us in reality though, because this would only ever be used probably near the very end of the array.

Adding this early return, our code looks like this now:

```go
func longestOnes(nums []int, k int) int {
    if len(nums) == k {
        return k
    }
    left, right := 0, 0
    zeroCount := 0
    longest := 0
    if nums[0] == 0 {
        zeroCount++
    }

    for right < len(nums) - 1 {
        // return early if there's no potential to get a larger sequence
        if (right - left + 1) + (len(nums) - right) < longest {
            return longest
        }
        if zeroCount <= k {
            right++
            if nums[right] == 0 {
                zeroCount++
            }
        } else {
            left++
            if nums[left - 1] == 0 {
                zeroCount--
            }
        }
        // if this is a valid sequence, record its length
        if zeroCount <= k {
            if right - left + 1 > longest {
                longest = right - left + 1
            }
        }
    }
    return longest
}
```

The condition we added looks a bit clunky:

```go
// return early if there's no potential to get a larger sequence
if (right - left + 1) + (len(nums) - right) < longest {
    return longest
}
```

`(right - left + 1)` is the size of our current window, and `len(nums) - right` is the amount of space that hasn't been searched yet. Adding those together tells you the largest possible sequence that could be made if all remaining values were included.

I'm not entirely convinced that this early return does much for the algorithm. I think in some special cases it might make a noticeable difference, but I think in most cases the difference is close to negligible. The most useful case would be if early on in the array there is a large sequence of 1s that goes to just about half of the array size. The least useful is when large sequences only appear near the end of the array. If most cases are somewhere between, where there are some middle sized sequences of ones scattered throughout, then this change probably just shaves off the last 5% (or less) iterations probably. I guess it's probably better than no early returns though?

## Other Leetcode Solutions

Spent some time looking at other solutions on leetcode, and found that everyone else is using the same approach as me. However, some people had managed to get their logic boiled down to just one or two conditions, which was interesting, so I think that's the only real improvement some of them had over my own solution. Still, none of them seemed to perform much better or worse than my own, at least according to Leetcode's performance stats. Didn't find anything new to incorporate into my solution.

## Let's ask Chat-GPT

Chat-GPT seems satisfied with my solution, and agrees with the optimization of the early return when there's no potential to get a bigger `longest` value. It doesn't give me any other ideas for making a more efficient algorithm either, so I think I'll stick with my current one.

## Final Solution, Conclusion

The same as my first solution, but with the micro-optimization of the early return added:

```go
func longestOnes(nums []int, k int) int {
    if len(nums) == k {
        return k
    }
    left, right := 0, 0
    zeroCount := 0
    longest := 0
    if nums[0] == 0 {
        zeroCount++
    }

    for right < len(nums) - 1 {
        // return early if there's no potential to get a larger sequence
        if (right - left + 1) + (len(nums) - right) < longest {
            return longest
        }
        if zeroCount <= k {
            right++
            if nums[right] == 0 {
                zeroCount++
            }
        } else {
            left++
            if nums[left - 1] == 0 {
                zeroCount--
            }
        }
        // if this is a valid sequence, record its length
        if zeroCount <= k {
            if right - left + 1 > longest {
                longest = right - left + 1
            }
        }
    }
    return longest
}
```

This was the first sliding window problem where we had to be a bit dynamic about how the window slides, where there are cases that it expands or shrinks along the way. That made it more interesting, but overall the approach still felt quite similar, just a bit more complicated logic inserted in.

Main Takeaway: Nothing to report here!

Ben's difficulty rating: 4.5/10. A bit more difficult than the other ones, but still pretty similar.
