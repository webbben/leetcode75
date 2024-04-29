# Move Zeroes

[leetcode link](https://leetcode.com/problems/move-zeroes)

4/29/2024

Difficulty: Easy

## Description

Given an integer array `nums`, move all `0`'s to the end of it while maintaining the relative order of the non-zero elements.

Note that you must do this in-place wihtout making a copy of the array.

**Example 1:**

```
Input: nums = [0,1,0,3,12]
Output: [1,3,12,0,0]
```

**Example 2:**

```
Input: nums = [0]
Output: [0]
```

**Constraints:**

-   `1 <= nums.length <= 10^4`
-   `-2^31 <= nums[i] <= 2^31 - 1`

**Follow up:** Could you minimize the total number of operations done?

## Initial Thoughts / Planning

The overall concept of the problem doesn't sound too complicated: if you encounter a zero, count it, and then remove it from the array. then, once you get to the end of the array, append all the zeroes you counted.

The thing I'm not quite sure on is what the most efficient way is. Maybe removing the zero from the array really is the most efficient way, because of course shifting over all other values left one at a time sounds pretty inefficient. Let's go ahead and just try it out.

## Code Implementations, Optimizations

```go
func moveZeroes(nums []int)  {
    zeroCount := 0
    i := 0
    for i < len(nums) {
        if nums[i] == 0 {
            zeroCount++
            nums = append(nums[:i], nums[i+1:]...)
        } else {
            i++
        }
    }

    nums = append(nums, make([]int, zeroCount)...)
}
```

Runtime: 27 ms (beats 17%)

Memory: 6.59 MB (beats 94%)

So this solution is apparently quite slow, but it does beat most solutions in terms of memory, which is nice.

I think the main efficiency problem is probably all the `append` commands. This command isn't very efficient because it's effectively creating a new array each time. In theory, the number of elements in this array shouldn't be changing, since we are just moving the zeroes to the end. So we should probably find a way to remove all these appends and instead be moving the numbers to new indices.

To do this (as hinted by the problem group name), I'm guessing we will use a two pointer approach of some sort.

Maybe one pointer represents a "searching" pointer that finds the next non-zero number, and another pointer represents the "placement" index, where the non-zero numbers are placed. once we get to the end of the array with the searching pointer, we know we've accounted for all the numbers. So, the rest of the positions from the placement pointer onward are converted to zeroes. If this concept works, I think it would be O(n) time complexity and O(1) space complexity.

```go
func moveZeroes(nums []int)  {
    finder := 0
    placer := 0

    // first, find all non-zero elements and move them up
    for finder < len(nums) {
        if nums[finder] != 0 {
            // move this number to placer index
            nums[placer] = nums[finder]
            placer++
        }
        finder++
    }
    // next, fill in remaining places after placer with zeroes
    for placer < len(nums) {
        nums[placer] = 0
        placer++
    }
}
```

Runtime: 14 ms (beats 88%)

Memory: 6.66 MB (beats 87%)

So it looks like this solution indeed speeds up the algorithm quite a bit. Now, instead of removing zero elements from the array and then appending them back on at the end, we are just moving them around, so there's no need to add or remove space to the array.

## Let's ask Chat-GPT

When I showed Chat-GPT my solution, and asked for a further optimized one, this is what it gave me:

```go
func moveZeroes(nums []int) {
    // Initialize a pointer to keep track of the position to place the next non-zero element
    nextNonZero := 0

    // Iterate through the array
    for i := 0; i < len(nums); i++ {
        // If the current element is non-zero, swap it with the nextNonZero element
        if nums[i] != 0 {
            nums[nextNonZero], nums[i] = nums[i], nums[nextNonZero]
            nextNonZero++
        }
    }
}
```

> This optimized version simplifies the logic and eliminates the need for a separate loop to fill in the remaining places with zeroes. It also reduces the number of operations by avoiding unnecessary assignments when the current element is already non-zero.
>
> Overall, both versions have the same time complexity of O(n) and use constant extra space. However, the optimized version might offer slight improvements in runtime speed and readability.

So, it basically just moves non-zero elements to the front of the array, swapping whatever was there before. This effectively moves all zeroes to the end of the array. It also seems this might use less operations, which coincidentally answers the follow-up in the problem description. It seems this solution also has a slightly faster runtime. But oddly enough, it might be consuming a little more memory. Not sure why it would be though - might just be a leetcode judging problem.

## Final Solution, Conclusion

```go
func moveZeroes(nums []int) {
    // Initialize a pointer to keep track of the position to place the next non-zero element
    nextNonZero := 0

    // Iterate through the array
    for i := 0; i < len(nums); i++ {
        // If the current element is non-zero, swap it with the nextNonZero element
        if nums[i] != 0 {
            nums[nextNonZero], nums[i] = nums[i], nums[nextNonZero]
            nextNonZero++
        }
    }
}
```

Runtime: 13 ms (beats 90%)

Memory: 7.12 MB (beats 35%)

I think the memory usage here is probably not as bad as reported here; I ran the solution (and my other original one) a few times, and got varying results, but overall I think the runtime here was a bit faster. But regardless of outputs from leetcode's performance judge, I think logically this solution is a little more efficient than the other one. They both have the same time and space complexity, but this one eliminates the second loop.

Main takeaway: **avoid appending arrays, as it's inefficient**. moving elements to new positions is a lot better of an option.
