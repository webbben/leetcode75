# Find the Highest Altitude

[leetcode link](https://leetcode.com/problems/find-the-highest-altitude/)

5/10/2024

Difficulty: Easy

## Description

There is a biker going on a road trip. The road trip consists of `n + 1` points at different altitudes. The biker starts his trip on point `0` with altitude `0`.

You are given an integer array `gain` of length `n` where `gain[i]` is the net gain in altitude between points `i` and `i + 1` for `0 <= i < n`. Return the highest altitude of a point.

## Initial Thoughts / Planning

Maybe I'm overlooking something, but this problem seems very simple. Don't you just simply add up all the values in the array and record the maximum value of the sum along the way?

## My Code Implementations, Optimizations

```go
func largestAltitude(gain []int) int {
    alt, maxAlt := 0, 0
    for i := 0; i < len(gain); i++ {
        alt += gain[i]
        maxAlt = max(maxAlt, alt)
    }
    return maxAlt
}
```

Time complexity: O(n)

Space complexity: O(1)

Yeah, it really was that simple lol.

At this point I'm kind of just thinking about how to minimize memory usage. I do think we need a variable to keep track of the altitude, and another to keep track of the maximum altitude value we've encountered so far. So maybe there isn't really anything to further micro-optimize here.

## Other Leetcode Solutions

All other leetcode solutions are the same as mine, pretty much.

## Final Solution, Conclusion

(final solution same as my first one)

Ben's difficulty rating: 1/10. Honestly I think this is the easiest leetcode problem I've ever done. Dead simple.
