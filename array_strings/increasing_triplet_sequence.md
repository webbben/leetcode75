# Increasing Triplet Sequence

[leetcode link](https://leetcode.com/problems/increasing-triplet-subsequence/)

4/26/2024

Difficulty: Medium

## Description

Given an integer array `nums`, return `true` if there exists a triple of indices `(i, j, k)` such that `i < j < k` and `nums[i] < nums[j] < nums[k]`. If no such indices exist, return `false`.

**Example 1:**

```
Input: nums = [1,2,3,4,5]
Output: true
Explanation: Any triplet where i < j < k is valid.
```

**Example 2:**

```
Input: nums = [5,4,3,2,1]
Output: false
Explanation: No triplet exists.
```

**Example 3:**

```
Input: nums = [2,1,5,0,4,6]
Output: true
Explanation: The triplet (3, 4, 5) is valid because nums[3] == 0 < nums[4] == 4 < nums[5] == 6.
```

**Constraints:**

-   `1 <= nums.length <= 5 * 10^5`
-   `-2^31 <= nums[i] <= 2^31 - 1`

**Follow up:** Could you implement a solution that runs in O(n) time complexity and O(1) space complexity?

## Initial Thoughts / Planning

This one is a bit tricky, because if each index `i, j, k` had to be right next to each other, it would be a trivial problem. But since they could have other elements between them, I'm now not sure how this is going to work, because we have to avoid getting "distracted" by other elements midway that aren't increasing, but decreasing. It's also hard to figure out what number should be the "starting point" to start searching based on, and which number should be picked as the "middle" (if you choose one that's too big, then we might fail to find the 3rd element).

... Ok, after some more thinking, I've realized something: both `i` and `k` depend on `j`, and the value of `j` will determine the range of possible values for those two. **but**, `i`, for example, will only determine where `j` can start from, and doesn't tell us much about `k`.

My point is that perhaps the brute-force method is to iterate through the array, treating the current index we are looking at as `j` (middle value), and then search below and above for a smaller value and a larger value. if they exist, then we've found the triplet. If not? then keep it moving.

```
[1, 5, 3, 2, 1, 6]
       j
<-i?>      <-k?->
```

This would be `O(n^2)` though, which isn't a very satisfying solution.

I guess if we were to base our solution on this concept, the key to making it `O(n)` would be to eliminate the extra search that happens above and below `j`. If we could just know that there is a value less than 3 below `j`, and a value greater than 3 above `j`, then no more work is necessary of course.

One way to know about the less-than portion would be to track the minimum number encountered as we iterate. Then, for any given number we use as `j`, we would know if there exists a number less than that before it. That would leave us with finding `k`, but this would still leave us with technically an O(n^2) solution since we aren't eliminating all the extra iterating.

So, an idea there would be to do some pre-processing to make an array that records the maximum value encountered _after_ a given index.
We would iterate backwards over the source array, tracking the maximum value encountered. When we go to the next index in the array (moving in reverse, still) we record the current max value, check the value at the current index to see if it's a new maximum, and then move on to the next one.

Here's how this array would look using our previous example source array:

```
[1, 5, 3, 2, 1, 6]

...

[6, 6, 6, 6, 6, 0]
```

there's no value after 6, so it's "after-max" is just 0. But since 6 is the maximum value, and at the end of the array, it means it's the "after-max" for all over values. For another example, if our source array were `[3, 2, 1]`, then the "after-max" array would be `[2, 1, 0]`.

This solution would be O(n) time complexity, but would be O(n) memory too. Even though we need to do an extra iteration before hand to calculate "after-max", they aren't nested loops so we treat it as O(n) time.

For now, I think I'll try this solution out.

## Code Implementations, Optimizations

Here's my solution:

```go
func increasingTriplet(nums []int) bool {
    if len(nums) < 3 {
        return false
    }
    // calculate the "after-max" array
    afterMax := make([]int, len(nums))
    curMax := nums[len(nums) - 1]
    for i := len(nums) - 1; i >= 0; i-- {
        if i == len(nums) - 1 {
            afterMax[i] = 0
            continue
        }
        afterMax[i] = curMax
        if nums[i] > curMax {
            curMax = nums[i]
        }
    }

    // now, walk through the nums array, finding the correct middle value
    curMin := nums[0]
    for i := 1; i < len(nums); i++ {
        if curMin < nums[i] && afterMax[i] > nums[i] {
            return true
        }
        if nums[i] < curMin {
            curMin = nums[i]
        }
    }
    return false
}
```

Runtime: 104 ms (beats 73%)

Memory: 23.02 MB (beats 6%)

Runtime is pretty good, but memory obviously is pretty bad. Right off the bat, I assume a large portion of the memory usage comes from the `afterMax` array we create in the pre-computing stage. But, unless we want to introduce a nested for loop that checks for the 3rd value for each middle value we test, I'm not sure if we can do away with it.

...

Ok, so after spending more time than I care to admit iterating over a couple ideas relating to using pointers, tracking mins and maxes, etc, I cheated and took a peek at a solution on leetcode. Now that I'm reading the solution, I feel kinda dumb:

Basically, you just need to track 2 the minimum values you've encountered so far. and if you find a value greater than the second minimum, you have a solution! Here's the full explanation and solution, courtesy of leetcode user `singhsourav73` ([link here](https://leetcode.com/problems/increasing-triplet-subsequence/solutions/4620669/increasing-triplet-subsequence-go-solution)):

> **Intuition**
>
> To find a triplet (i, j, k) where nums[i] < nums[j] < nums[k], we can iterate through the array and maintain two variables, min and secondMin. The idea is to find a pair of indices (i, j) such that min represents the minimum value encountered so far and secondMin represents the second minimum value encountered.
>
> If we encounter a value smaller than or equal to min, we update min. If we encounter a value greater than min but smaller than or equal to secondMin, we update secondMin. If we encounter a value greater than secondMin, we have found a triplet, and we return true.
>
> **Approach**
>
> 1. Initialize triplet1 and triplet2 to positive infinity.
> 2. Iterate through the array.
>
> -   If the current element is less than or equal to triplet1, update triplet1.
> -   If the current element is greater than triplet1 but less than or equal to triplet2, update triplet2.
> -   If the current element is greater than triplet2, return true as we have found a triplet.
>
> 3. If no triplet is found during the iteration, return false.

```go
func increasingTriplet(nums []int) bool {
    triplet1, triplet2 := math.MaxInt32, math.MaxInt32

    for i := 0; i < len(nums); i++ {
        if nums[i] <= triplet1 {
            triplet1 = nums[i]
        } else if nums[i] <= triplet2 {
            triplet2 = nums[i]
        } else {
            return true
        }
    }
    return false
}
```

This solution performs a lot better than my own, and also clearly uses less time and space complexity (linear time and constant space).

## Let's ask Chat-GPT

Chat-GPT seems pretty satisfied with this solution. No big suggestions for improving the existing algorithm, it seems.

## Final Solution, Conclusion

The final solution:

```go
func increasingTriplet(nums []int) bool {
    triplet1, triplet2 := math.MaxInt32, math.MaxInt32

    for i := 0; i < len(nums); i++ {
        if nums[i] <= triplet1 {
            triplet1 = nums[i]
        } else if nums[i] <= triplet2 {
            triplet2 = nums[i]
        } else {
            return true
        }
    }
    return false
}
```

I was able to come up with a different solution on my own, but it turned out to be overly complicated; it involved using extra memory and doing extra calculations, when in reality this problem is just a matter of tracking a couple minimum values. I think I was starting to get close to something like this final solution on my own, but it was still overly complicated and the logic was turning out messy.

Main takeaway: hmm... nothing too philosophical to offer here. I guess **don't overthink it?** I think I have a little ways to go studying leetcode challenges, because this one felt a little more difficult than it should've been.

Ben's difficulty rating: 7.8/10. This one felt like it almost approached "Hard" level territory. I definitely struggled to figure out the optimal solution here.
