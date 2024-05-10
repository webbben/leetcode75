# Longest Subarray of 1's After Deleting One Element

[leetcode link](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/)

5/10/2024

Difficulty: Medium

## Description

Given a binary array `nums`, you should delete one element from it.

Return the size of the longest non-empty subarray containing only 1's in the resulting array. Return `0` if there is no such subarray.

**Example 1:**

```
Input: nums = [1,1,0,1]
Output: 3
Explanation: After deleting the number in position 2, [1,1,1] contains 3 numbers with value of 1's.
```

**Example 2:**

```
Input: nums = [0,1,1,1,0,1,1,0,1]
Output: 5
Explanation: After deleting the number in position 4, [0,1,1,1,1,1,0,1] longest subarray with value of 1's is [1,1,1,1,1].
```

**Example 3:**

```
Input: nums = [1,1,1]
Output: 2
Explanation: You must delete one element.
```

**Constraints:**

-   `1 <= nums.length <= 10^5`
-   `nums[i]` is either `0` or `1`.

## Initial Thoughts / Planning

If I'm not mistaken... I think this might just be almost the exact same problem as the last one! It's effectively finding the longest subarray of 1's while permitting `k` number of zeroes, only `k = 1` in this scenario. Additionally, we would return the result minus 1, since the result from the other solution includes the zeroes in the subarray's size.

Feels kind of like cheating, but I think out of pure laziness I might just copy that solution over, and just subtract one from the result... lol.

## My Code Implementations, Optimizations

```go
func longestSubarray(nums []int) int {
    left, right := 0, 0
    zeroCount := 0
    longest := 0
    if nums[0] == 0 {
        zeroCount++
    }

    for right < len(nums) - 1 {
        if zeroCount <= 1 {
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
        if zeroCount <= 1 {
            if right - left + 1 > longest {
                longest = right - left + 1
            }
        }
    }
    return max(longest - 1, 0)
}
```

Lol! sure enough, just re-using the last problem's solution (I did that problem just a few minutes ago, so why not) and replacing `k` with 1, as well as a couple other tweaks, we have a working solution.

Time complexity: O(n)

Space complexity: O(1)

The performance on leetcode seems to be pretty average, but potentially better than most. But we already know how random those results can be.

Anyway, I think I'm... satisfied with this solution. I do feel like I cheated a bit lol. But honestly it's kind of mean for leetcode to put what is effectively the same problem back to back, because at this point it would be silly for me to spend extra time rewriting the same solution from scratch, and then go through all the effort of thoroughly researching it over again.

Basically, when Leetcode says this in the problem description:

> Given a binary array `nums`, you should **delete** one element from it.

That's just speaking "figuratively". Actually repeatedly deleting and then reinserting elements into the array would make the solution way less efficient, so we don't wanna do that at all.
We can just pretend we are deleting an element by keeping track of how many elements we would delete in our currently considered window - which is always at most 1 zero, if present. The last problem we dealt with was similar, where it was advising us to "flip zeroes to ones". No, we just want to keep a count of the number of zeroes we _would_ flip, and that's it.

So besides the fact that we are only tracking one zero in this version of the solution, we are also returning the size of the resultant subarray **without** including the zero, because well, it was "deleted". Those are the only differences.

## Final Solution, Conclusion

Kinda took a shortcut here by starting with the solution from the last problem, but I think it's still valid. See the implementation above. I decided to skip all the usual ChatGPT and Leetcode solutions research due to this.

Ben's difficulty rating: 4/10. Honestly since this problem only requires keeping track of a single zero in the window, it might qualify as being slightly easier than the other. But idk, maybe it could also change the logic a bit which would make it the same in the end.
