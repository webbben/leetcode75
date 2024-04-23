# Kids with the Greatest Number of Candies

[leetcode link](https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/)

4/23/2023

Difficulty: Easy

## Description

There are `n` kids with candies. You are given an integer array `candies`, where each `candies[i]` represents the number of candies the `i`th kid has, and an integer `extraCandies`, denoting the number of extra candies that you have.

Return a boolean array `result` of length `n`, where `result[i]` is `true` if, after giving the `i`-th kid all the `extraCandies`, they will have the greatest number of candies among all the kids, or `false` otherwise.

Note that multiple kids can have the greatest number of candies.

**Example 1:**

```
Input: candies = [2,3,5,1,3], extraCandies = 3
Output: [true,true,true,false,true]
Explanation: If you give all extraCandies to:
- Kid 1, they will have 2 + 3 = 5 candies, which is the greatest among the kids.
- Kid 2, they will have 3 + 3 = 6 candies, which is the greatest among the kids.
- Kid 3, they will have 5 + 3 = 8 candies, which is the greatest among the kids.
- Kid 4, they will have 1 + 3 = 4 candies, which is not the greatest among the kids.
- Kid 5, they will have 3 + 3 = 6 candies, which is the greatest among the kids.
```

**Example 2:**

```
Input: candies = [4,2,1,1,2], extraCandies = 1
Output: [true,false,false,false,false]
Explanation: There is only 1 extra candy.
Kid 1 will always have the greatest number of candies, even if a different kid is given the extra candy.
```

**Example 3:**

```
Input: candies = [12,1,12], extraCandies = 10
Output: [true,false,true]
```

**Constraints:**

-   `n == candies.length`
-   `2 <= n <= 100`
-   `1 <= candies[i] <= 100`
-   `1 <= extraCandies <= 50`

## Initial Thoughts / Planning

This one seems pretty straightforward; I think I just need to first find out the max number of candies any kid has in the `candies` array, and then loop back over it once again, checking if `candies[i] + extraCandies` is greater than or equal the max number of candies. Maybe I'm missing something, but honestly this seems the simplest so far. Maybe there's a twist I'm not noticing yet?

## Code Implementations, Optimizations

Well, it does seem to be very simple. I was able to write this solution up in about 5 minutes!

```go
func kidsWithCandies(candies []int, extraCandies int) []bool {
    // first, find the max number of candies any kid has
    maxCandies := 0
    for _, numCandies := range candies {
        if numCandies > maxCandies {
            maxCandies = numCandies
        }
    }
    // now, see if any kid would be boosted above this number by adding extraCandies to their number
    output := make([]bool, len(candies))
    for i, numCandies := range candies {
        output[i] = numCandies + extraCandies >= maxCandies
    }
    return output
}
```

Runtime: 2ms (beats 74% of go solutions)

Memory: 2.54 (beats 14% of go solutions)

I first loop over the candies array to find the max number of candies in the array, then I loop over it again to test each part of the array.

It seems that I might be using too much memory though, considering 75% of other go solutions used less than me. But I'm really not sure where my memory usage is coming from. Is it possible that looping over the array more than once is contributing to that? If so, that is contrary to how I currently understand memory usage. Maybe other solutions somehow avoid creating the output array?

## Let's ask Chat-GPT

Chat-GPT doesn't seem to be able to find any optimizations. I guess this problem really is that simple!

The thing that still confuses me is the extra memory usage. I tried asking about potential memory optimizations, but nothing came up.
It had some ideas at first, but those were dispelled after we discussed it slightly more in depth. We weren't able to come up with any
real ideas for where the extra memory usage comes from. My only slight hunch is maybe my second loop, which also allocations the `i` variable, is the cause of extra memory consumption. Maybe other implementions are not pre-allocating the `output` array, and instead appending to it. But I think even if that were the cause, my solution of pre-allocating is better since it's more efficient and doesn't need to do append operations on the array, which tend to be inefficient.

## Final Solution, Conclusion

No change from my first implementation. No big take aways here.
