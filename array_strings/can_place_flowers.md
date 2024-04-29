# Can Place Flowers

[leetcode link](https://leetcode.com/problems/can-place-flowers/)

4/24/2024

Difficulty: Easy

## Description

You have a long flowerbed in which some of the plots are planted, and some are not. However, flowers cannot be planted in **adjacent** plots.

Given an integer array `flowerbed` containing `0`'s and `1`'s, where `0` means empty and `1` means not empty, and an integer `n`, return `true` _if `n` new flowers can be planted in the `flowerbed` without violating the no-adjacent-flowers rule and `false` otherwise._

**Example 1:**

```
Input: flowerbed = [1,0,0,0,1], n = 1
Output: true
```

**Example 2:**

```
Input: flowerbed = [1,0,0,0,1], n = 2
Output: false
```

**Constraints:**

-   `1 <= flowerbed.length <= 2 * 10^4`
-   `flowerbed[i]` is `0` or `1`
-   there are no adjacent flowers in `flowerbed`
-   `0 <= n <= flowerbed.length`

## Initial Thoughts / Planning

This doesn't sound too complicated; basically we are going to iterate over the `flowerbed` array and check if a given position is 0, and also surrounded by 0 on both the indices before and after. if so, then we can count a flower inserted, and also go ahead and set that position in `flowerbed` to 1, so that we remember a flower was placed there. In the end, if our count of flowers placed is greather than or equal to `n`, we return true.

I think this would be a pretty efficient algorith - O(n), and also I think it would use constant memory space - O(1) - since we aren't creating a new array to record placed flowers, but just modifying the given array.

## Code Implementations, Optimizations

Here's my first solution, which was pretty quick and easy to code up:

```go
func canPlaceFlowers(flowerbed []int, n int) bool {
    placed := 0
    for i := 0; i < len(flowerbed); i++ {
        flowerVal := flowerbed[i]
        if flowerVal == 1 {
            continue
        }
        if i > 0 && flowerbed[i-1] == 1 {
            continue
        }
        if i < len(flowerbed) - 1 && flowerbed[i+1] == 1 {
            continue
        }
        flowerbed[i] = 1
        placed++
    }
    return placed >= n
}
```

Runtime: 15 ms (beats 32% of go users)

Memory: 6.27 MB (beats 62% of go users)

So, it seems like this solution might be a bit slow. I'm a little surprised, since I thought the time complexity was O(n), since we are only iterating over the data once, and there don't seem to be any operations in the for loop that would require iterating on its own. I guess one thing is that each iteration, we are repeatedly checking the adjacent positions for flowers, which perhaps is slowing things down a little. Maybe accessing an index in an array is a bit inefficient? I could imagine a solution where instead, we just count how many zeroes we've encountered as we iterate, and if that number becomes 3, we know we've found an empty spot, and we reset the count and continue that process.

For memory, I'm also a bit stumped on where my excess memory usage is. as far as I can tell, we re-use the flowerbed input array, and only create one counter array. I guess we do a for loop, which includes the `i` variable too.

## Let's ask Chat-GPT

Here's one optimization Chat-GPT had for me:

> Optimization: Your solution is efficient and straightforward, but there's a slight optimization that can be made. Currently, you're iterating over the entire flowerbed, checking each plot individually. However, you can skip some iterations if you've already planted enough flowers to satisfy the requirement n. Once you've planted enough flowers, you don't need to continue checking the remaining plots.

That seems like a no-brainer! I forgot to consider early returning once the condition has already been met. That might save a lot of extra iterating for many of the test cases.

## Final Solution, Conclusion

Here's the final, optimized code:

```go
func canPlaceFlowers(flowerbed []int, n int) bool {
    placed := 0
    for i := 0; i < len(flowerbed); i++ {
        flowerVal := flowerbed[i]
        if flowerVal == 1 {
            continue
        }
        if i > 0 && flowerbed[i-1] == 1 {
            continue
        }
        if i < len(flowerbed) - 1 && flowerbed[i+1] == 1 {
            continue
        }
        flowerbed[i] = 1
        placed++
        if placed >= n {
            return true
        }
    }
    return placed >= n
}
```

Runtime: 9 ms (beats 90%)

Memory: 6.05 MB (beats 90%)

The only change I made to optimize from the original solution is the early return in the for loop, checking `placed >= n`.
With this tiny change, runtime and memory improved quite a bit.

Main takeaway: **Don't forget to look for early return chances from loops!**

Ben's difficulty rating: 3/10.
