# title

[leetcode link](https://leetcode.com/problems/product-of-array-except-self/)

4/25/2024

Difficulty: Medium

## Description

Given an integer array `nums`, return an array `answer` such that `answer[i]` is equal to the product of all the elements of `nums` except `nums[i]`.

The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.

You must write an algorithm that runs in O(n) time and without using the division operation.

**Example 1:**

```
Input: nums = [1,2,3,4]
Output: [24,12,8,6]
```

**Example 2:**

```
Input: nums = [-1,1,0,-3,3]
Output: [0,0,9,0,0]
```

**Constraints:**

-   `2 <= nums.length <= 10^5`
-   `-30 <= nums[i] <= 30`
-   The product of any prefix or suffix of `nums` is guaranteed to fit in a 32-bit integer.

## Initial Thoughts / Planning

At first I went down a bit of a rabbit hole thinking that I would get the product of all the numbers in `nums`, and then divide out each number. Then I noticed this problem explicitly states that I **cannot use the division operator**...

Ok, so where does that leave us? I spent some time thinking and looking into "prefix/suffix products", like the description hints at. I think that might be how this problem could be solved.

Basically, to my understanding, a "prefix product" is the product of every value in the array up to a given index. The same applies to "suffix" product, just starting from the end of the array.

So, if we have this example array:

`[1, 2, 3, 4, 5]`

the prefix product array would be:

`[1, 2, 6, 24, 120]`

the suffix product array would be:

`[120, 120, 60, 20, 5]`

to know the product of all numbers excluding the number at index `i`, you can calculate it by:

`prefix[i-1] * suffix[i+1]`

to test this, let's calculate the total product excluding index 2, (value 3 from the array):

`prefix[1] * suffix[3]` => `2 * 20` = `40`

And we see this is correct by calculating `1 * 2 * 4 * 5` = `40`

## Code Implementations, Optimizations

Here's my implementation of the algorithm described above:

```go
func productExceptSelf(nums []int) []int {
    // calculate the prefix and suffix product arrays
    n := len(nums)
    prefix := make([]int, n)
    suffix := make([]int, n)
    for i := 0; i < n; i++ {
        if i == 0 {
            prefix[0] = nums[0]
            suffix[n-1] = nums[n - 1]
            continue
        }
        prefix[i] = prefix[i-1] * nums[i]
        suffix[n - i - 1] = suffix[n - i] * nums[n - i - 1]
    }

    // calculate the output
    answer := make([]int, n)
    for i := 0; i < n; i++ {
        if i == 0 {
            answer[0] = suffix[1]
            continue
        }
        if i == n - 1 {
            answer[i] = prefix[i-1]
            continue
        }
        answer[i] = prefix[i-1] * suffix[i+1]
    }
    return answer
}
```

Runtime: 39 ms (beats 22%)

Memory: 8.30 MB (beats 32%)

I think the first problem I want to look at is memory usage. Right now I'm allocating 3 different arrays: `prefix`, `suffix`, and `answer`.
I think `answer` may be necessary no matter what, but I wonder if there's a more space efficient way to store our prefix and suffix product values? maybe there's a way to calculate it such that I don't need to store the products in these arrays? I don't think calculating the prefix and suffix products on the fly each time is a good idea, since that's pretty "brute force" style and not efficient in terms of time.

Also, this is Hint 1, provided by leetcode:

> Think how you can efficiently utilize prefix and suffix products to calculate the product of all elements except self for each index. Can you pre-compute the prefix and suffix products in linear time to avoid redundant calculations?

So that makes me think we definitely shouldn't be considering doing away with our pre-computation in any way. But I also think it means we've already succeeded past what Hint 1 is for.

Hint 2:

> Can you minimize additional space usage by reusing memory or modifying the input array to store intermediate results?

I'm not sure if this is what the hint is pointing me towards, but I'm now wondering if we need both `prefix` and `suffix`.
For example, maybe we can first calculate `suffix`, and then loop through `nums`, effectively calculating the `prefix` values as we go, and using those values to fill out the `answers` array by multiplying the "running" `prefix` values with the pre-computed `suffix` values.

If this works like I'm assuming, then it would both eliminate an array. Let's give it a shot...

```go
func productExceptSelf(nums []int) []int {
    // calculate the suffix product array
    n := len(nums)
    suffix := make([]int, n)
    for i := 0; i < n; i++ {
        if i == 0 {
            suffix[n-1] = nums[n - 1]
            continue
        }
        suffix[n - i - 1] = suffix[n - i] * nums[n - i - 1]
    }

    // calculate the output
    answer := make([]int, n)
    runningPrefix := 1
    for i := 0; i < n; i++ {
        if i == 0 {
            answer[0] = suffix[1]
        } else if i == n - 1 {
            answer[i] = runningPrefix
        } else {
            answer[i] = runningPrefix * suffix[i+1]
        }
        runningPrefix *= nums[i]
    }
    return answer
}
```

> Seems like it might've helped a bit, but I'm not seeing a big difference in the memory usage or runtime from leetcode.

Let's move on from memory for a second and think about other optimizations. One idea I had is, zeroes make a big impact on the output.

If there is 1 zero:

`[0, 0, 0, ..., x, ..., 0, 0, 0]`

All indices will be zero **except** the index where the actual zero in `nums` is.

If there are 2 or more zeroes:

`[0, 0, ..., 0, 0]`

**ALL** positions will be zero.

```go
// ...
// for loop for building suffix product array
zeroCount := 0
for i := 0; i < n; i++ {
    // track zeroes for early return possibility
    if nums[i] == 0 {
        zeroCount++
        if zeroCount >= 2 {
            return make([]int, n)
        }
    }
    // compute suffix product array
    if i == 0 {
        suffix[n-1] = nums[n - 1]
        continue
    }
    suffix[n - i - 1] = suffix[n - i] * nums[n - i - 1]
}
// rest of the code is same
// ...
```

> similarly, this only yielded a minor improvement, if anything. I guess it is only an optimization for specific cases, and perhaps most of the cases don't include zeroes. I had code that handled the case for 1 zero too, but it ended up making the code a little messy and didn't seem like it would be all that much of an optimization in the end, judging by the performance stats returned by leetcode.

Looking around at other solutions people give on Leetcode, I don't see any big improvements in runtime or memory. I wonder if there's just something funky going on with Leetcode servers today? Or maybe it's not easy to optimize beyond this point?

## Let's ask Chat-GPT

Here's Chat-GPT's analysis of how we could optimize:

> **Memory Optimization:**
>
> 1. Avoid Creating Unnecessary Arrays: Instead of creating separate arrays for the suffix product and the final result, you can compute the product on-the-fly and store it directly in the result array. This way, you save memory by not needing additional arrays for intermediate calculations.
>
> **Runtime Speed Optimization:**
>
> 1. Reduce Redundant Calculations: Look for opportunities to eliminate redundant calculations. For instance, in your current solution, you calculate the suffix product array and then use it to compute the final result. You might be able to combine these steps to avoid redundant computations.
>
> 2. Reduce Loop Iterations: If possible, try to reduce the number of loop iterations or optimize the loop conditions. This can help improve runtime speed, especially for large input arrays.
>
> 3. Utilize Built-in Functions or Data Structures: Sometimes, built-in functions or data structures in the language can offer optimizations. Explore if there are any built-in functions or data structures in Golang that can help optimize your solution.

This all sounds fine and dandy, but I don't see many explicit suggestions, more just theoretical ideas to pursue.

_however..._

Upon trying the solutions Chat-GPT gave me as optimizations, it gave me a solution that used division (even after I kindly asked it not to use division), and then started giving me solutions that were full of bugs. So, I don't think there's much to learn here from Chat-GPT unfortunately.

## Final Solution, Conclusion

Here's my final solution, which incorporates a couple optimizations I came up with into my initial solution:

```go
func productExceptSelf(nums []int) []int {
    // calculate the suffix product array
    n := len(nums)
    suffix := make([]int, n)
    zeroCount := 0
    for i := 0; i < n; i++ {
        // track zeroes for early return possibility
        if nums[i] == 0 {
            zeroCount++
            if zeroCount >= 2 {
                return make([]int, n)
            }
        }
        // compute suffix product array
        if i == 0 {
            suffix[n-1] = nums[n - 1]
            continue
        }
        suffix[n - i - 1] = suffix[n - i] * nums[n - i - 1]
    }

    // calculate the output
    answer := make([]int, n)
    runningPrefix := 1
    for i := 0; i < n; i++ {
        if i == 0 {
            answer[0] = suffix[1]
        } else if i == n - 1 {
            answer[i] = runningPrefix
        } else {
            answer[i] = runningPrefix * suffix[i+1]
        }
        runningPrefix *= nums[i]
    }
    return answer
}
```

Runtime: 29 ms (beats 43%)

Memory: 8.35 MB (beats 29%)

These stat numbers are iffy because I've seen it vary a bit, but I think it more or less captures it accurately.
Unfortunately Chat-GPT wasn't much of a help this time - hopefully that's not a trend as the problems get more and more complicated.

While browsing other solutions on leetcode, I find some that purport to beat 90% of other solutions in runtime and memory, but the approach described sounds just about identical.

Main takeaway: Look for ways to combine work into the same loop if possible. Also, look for early return options, as those might be able to improve the performance and get rid of extra work if we can determine the answer early on based on some type of edge case.

Ben's difficulty rating: 6.4/10. The concept seemed easy at first, but proved a bit more complicated in the end. Also, it seemed hard to fully optimize this solution.
