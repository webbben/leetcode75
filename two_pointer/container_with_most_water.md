# Container With Most Water

[leetcode link](https://leetcode.com/problems/container-with-most-water)

5/1/2024

Difficulty: Medium

## Description

You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `i`-th line are `(i, 0)` and `(i, height[i])`.

Find the two lines that together with the x-axis form a container, such that the container contains the most water.

Return the maximum amount of water a container can store.

Notice that you may not slant the container.

**Example 1:**

[link to diagram image](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/17/question_11.jpg)

```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49
Explanation: (pertaining to the image in the link above) The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7]. In this case, the max area of water (blue section) the container can contain is 49.
```

**Example 2:**

```
Input: height = [1,1]
Output: 1
```

**Constraints:**

-   `n == height.length`
-   `2 <= n <= 10^5`
-   `0 <= height[i] <= 10^4`

## Initial Thoughts / Planning

> Note: for this problem, you definitely need a visual aid to help understand the problem. If you can't see the linked image from the first example, then I highly recommend getting some scratch paper and drawing a visual representation of the problem.

First, let's clarify the problem and some of its details.

**Calculating the amount of water contained**

One important thing to note, which I didn't realize until I looked at the image for example 1:

If two lines are used in a solution, **no other lines will influence the amount of water being held**. For example, if there are "sub containers" in the middle that are taller, those don't hold extra water or anything. We pretend as if the other lines are simply not there when considering a specific solution of two lines as the bounds of a container. So, if we choose lines `i` and `j` (`i < j`), then the water contained is:

```
water = min(height[i], height[j]) * (j - i)
```

Or in otherwords, the area of the rectangle formed, using the smaller of the two heights as the height in the equation.

Ok, let's move on to planning a strategy.

Looking at the diagram more, and considering our above formula, I realize that for each line, we could calculate the maximum water volume if we used it by finding a corresponding line that is furthest away and has a height greater than or equal to its height. However, if the line is the tallest line, then it would never find a matching height, so it would have to just use whatever maximum calculated volume it found after comparing to all other lines.

The brute force approach to this problem would be to calculate the maximum water volume for each line, comparing it to every other line. But that would be O(n^2), which we don't want. So, perhaps there's a way to simplify, such that we only need to loop once?

Since the maximum volume of water for a line is usually going to be with a line on the opposite side of the array (if it can find a line of same or greater height), I'm wondering if a two pointer solution, where we have a pointer starting on both sides of the array, is a good approach.

For each pair of lines, we can calculate the maximum volume using those two lines and record it down as the current maximum. Then, we could move forward (or backwards) from whichever line is shorter, and repeat the same calculation, looking for the overall maximum volume. My reasoning for moving the smaller line is that it is the current limiting factor on volume, so we might as well look for a better option. I also think that this strategy would ensure that if there's a "hidden" super tall container in the middle, we will certainly find it since we are greedily looking for tall container walls.

## Code Implementations, Optimizations

```go
func maxArea(height []int) int {
    // two pointer approach starting from each side of the heights array, greedily looking for taller heights from each side.
    l, r := 0, len(height) - 1
    curMax := 0
    for r > l {
        vol := min(height[l], height[r]) * (r - l)
        if vol > curMax {
            curMax = vol
        }
        if height[l] < height[r] {
            l++
        } else {
            r--
        }
    }
    return curMax
}
```

Runtime: 73 ms (beats 28%)

Memory: 9.20 MB (beats 15%)

Well, looks like my idea worked! However, judging by the performance stats here, it makes me think there should be some room for improvement.

The first thing I'm wondering: is there an opportunity for early returning?
With my current solution, I'm not sure. I think we can't since we need to make sure we search the inside of the array too for a hidden "super tall" container.

I'm starting to wonder though, maybe if we knew from the beginning of the search where the tallest line was, that might reduce the amount of searching we have to do, allowing for early returns. However, to do that we'd need to traverse the array once initially to find the maximum height line, which in the end doesn't seem like it'd save us any time, since we are already traversing the whole array. Additionally, I don't think any of the operations in our current loop are expensive; pretty much just entirely comparisons of integers.

... Hm, I'm not really able to come up with any optimization ideas here. Let's see others' answers.

> After looking at leetcode answers, they all seem to be the same answer as mine - including ones that purport to beat 100%. If I resubmit my same solution a few times, I do get runtime that beats up to 70%, so it does seem a bit inconsistent. Maybe this is the optimal solution after all?

## Let's ask Chat-GPT

Chat-GPT supposed that we could try returning early if we encounter a volume that is less than the current max volume. However, I talked him out of that idea by explaining the possibility of the "hidden" extremely tall container in the middle of the array. Beyond that, Chat-GPT had no further ideas for optimization.

## Final Solution, Conclusion

```go
func maxArea(height []int) int {
    // two pointer approach starting from each side of the heights array, greedily looking for taller heights from each side.
    l, r := 0, len(height) - 1
    curMax := 0
    for r > l {
        vol := min(height[l], height[r]) * (r - l)
        if vol > curMax {
            curMax = vol
        }
        if height[l] < height[r] {
            l++
        } else {
            r--
        }
    }
    return curMax
}
```

Runtime: 73 ms (beats 28% - or sometimes up to 70%?)

Memory: 9.20 MB (beats 15%)

Time complexity: O(n)

Space complexity: O(1)

I think I've done this problem before, so I kind of already had a slight clue of what the solution would be. Also, of course since it's in the two pointer group, I assumed it would involve two pointers in some way or form.

Main takeaway: start by comfirming your understanding of the problem, and then considering the sub problems that go into the solution. Once you consider the problem from its sub problem, it becomes easier to understand how to tackle it.

> Note: Moving forward, I may put less emphasis on runtime and memory stats from leetcode, since they only seem to be a ballpark estimate of the actual performance of the algorithm, and are often slightly misleading. I think I'll put more emphasis on my own time and space complexity observations, and minimizing operations, iterations, etc, and just use the runtime stats as just a for-fun thing.

Ben's difficulty rating: 5/10.
