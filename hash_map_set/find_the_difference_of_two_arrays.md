# Find the Difference of Two Arrays

[leetcode link](https://leetcode.com/problems/find-the-difference-of-two-arrays/)

5/12/2024

Difficulty: Easy

## Description

Given two 0-indexed integer arrays `nums1` and `nums2`, return a list `answer` of size `2` where:

-   `answer[0]` is a list of all distinct integers in `nums1` which are not present in `nums2`.
-   `answer[1]` is a list of all distinct integers in `nums2` which are not present in `nums1`.

Note that the integers in the lists may be returned in any order.

**Example 1:**

Input: nums1 = [1,2,3], nums2 = [2,4,6]
Output: [[1,3],[4,6]]
Explanation:
For nums1, nums1[1] = 2 is present at index 0 of nums2, whereas nums1[0] = 1 and nums1[2] = 3 are not present in nums2. Therefore, answer[0] = [1,3].
For nums2, nums2[0] = 2 is present at index 1 of nums1, whereas nums2[1] = 4 and nums2[2] = 6 are not present in nums2. Therefore, answer[1] = [4,6].

**Example 2:**

Input: nums1 = [1,2,3,3], nums2 = [1,1,2,2]
Output: [[3],[]]
Explanation:
For nums1, nums1[2] and nums1[3] are not present in nums2. Since nums1[2] == nums1[3], their value is only included once and answer[0] = [3].
Every integer in nums2 is present in nums1. Therefore, answer[1] = [].

**Constraints:**

-   `1 <= nums1.length, nums2.length <= 1000`
-   `-1000 <= nums1[i], nums2[i] <= 1000`

## Initial Thoughts / Planning

Ok, so basically we need to find the numbers that are present in `nums1` but not in `nums2`, and vice versa.

Of course the grouper title for these problems is a bit of a spoiler, but this screams "sets" to me since we want to just know the unique numbers of each array. With sets, you can - in constant time - check for the presence of a number.

With Go, I actually don't know if there is such a thing as a "Set". I know there are maps, which can be easily used as sets. Basically, you would make something like a `map[int]bool`, and when you want to test for the presense or absense of a number, you would do something like this:

```go
var setOrMap map[int]bool
// ... code that sets the values in setOrMap ...

if _, ok := setOrMap[someNumber]; ok {
    // someNumber exists in setOrMap
} else {
    // someNumber doesn't exist in setOrMap
}
```

Technically it doesn't really matter what the corresponding value is in the set; it could be a `bool`, which is what I normally would default to, but I've also seen people use `struct{}`. I'm actually not sure but maybe one is more lightweight than the other and is very slightly more memory efficient?

Anyway, here's what I'm thinking:

First, iterate over `nums1`, adding all the numbers that exist in it to a set - let's call it `numSet1`. Basically we'd just set `numSet1[val] = true` to "add" it to the "set".

Then, iterating over `nums2`, do the same thing, adding all numbers to a set called `numSet2`.

Finally, we will create two arrays for the return values. For each number in `numSet1`, if it's **not** in `numSet2`, add it to the array. Do the same thing but vice versa so get the numbers in `numSet2` that aren't in `numSet1`. Then, return these two arrays.

Right off the bat - I acknowledge that we are creating **2 maps and 2 arrays**, each of which consume `n` space. So while this is still an `O(n)` space solution, it's a "heavy" one. Maybe once we implement this we can find some room for improvement on minimizing memory usage.

## My Code Implementations, Optimizations

```go
func findDifference(nums1 []int, nums2 []int) [][]int {
    set1 := map[int]bool{}
    for _, v := range nums1 {
        set1[v] = true
    }
    set2 := map[int]bool{}
    for _, v := range nums2 {
        set2[v] = true
    }
    // now, see which values are in one set but not the other
    dist1 := []int{}
    for key := range set1 {
        if _, ok := set2[key]; !ok {
            dist1 = append(dist1, key)
        }
    }
    dist2 := []int{}
    for key := range set2 {
        if _, ok := set1[key]; !ok {
            dist2 = append(dist2, key)
        }
    }
    return [][]int{dist1, dist2}
}
```

Time complexity: `O(n)` - although there are 4 loops here, they each run after each other, not nested, so we are still in linear time complexity.

Space complexity: `O(n)` - again, although there are 2 maps and 2 arrays created (4 total "collections" that house `n` space), we are still using linear space complexity since they aren't creating space usage at more than a linear rate. The memory usage scales linearly with the size of the `nums` variables.

So this solution works pretty well. It does seem a bit verbose though, as we have 4 different loops going on, just to find the elements that are distinct between two arrays.

My first thought is, maybe we can reduce the amount of memory we are using here. I think we will need the 2 output arrays no matter what. But perhaps we don't actually need both of the sets/maps?
Maybe, instead of capturing the sets of values in both `nums` arrays independently, we could do sort of the opposite: capture the set of values that are common between them. Then, when deciding which values are distinct to one of the `nums` arrays, we just make sure it isn't in that set.

I think we could do something like this:

-   iterate over values in `nums1`, adding all the values to `commonSet`, setting their mapped value to `false`.
-   iterate over values in `nums2`, checking if they are in `commonSet` already. If they are, then change the value to `true`.

At this point, `commonSet` will have all values from `nums1` mapped in it, but those that are common between both `nums1` and `nums2` will be mapped as `true` (indicating they are "common" between the two arrays). We can then use this single map/set to figure out which values are distinct in the two arrays.

```go
func findDifference(nums1 []int, nums2 []int) [][]int {
    commonSet := map[int]bool{}
    dist1, dist2 := []int{}, []int{}
    for _, v := range nums1 {
        commonSet[v] = false
    }
    for _, v := range nums2 {
        if _, ok := commonSet[v]; ok {
            commonSet[v] = true // this value is shared in common between nums1 and nums2
        }
    }
    for _, v := range nums1 {
        // all nums1 values should be in commonSet
        // but, if it's mapped to false, it's distinct to nums1
        if !commonSet[v] {
            dist1 = append(dist1, v)
            commonSet[v] = true // mark it as true to skip later duplicates
        }
    }
    for _, v := range nums2 {
        // if a value from nums2 isn't found in commonSet, then we know it's distinct to nums2
        if _, ok := commonSet[v]; !ok {
            dist2 = append(dist2, v)
            commonSet[v] = true // mark it as true to skip later duplicates
        }
    }
    return [][]int{dist1, dist2}
}
```

This change seems to definitely improve memory usage, although space complexity of course remains the same. Now, instead of 7.7 MB, leetcode says the new solution takes about 7.1 MB memory. The runtime stays mostly the same however.

I do think, however, that technically the other solution might be very slightly more optimal for runtime: If you use super long arrays of numbers for `nums1` and `nums2` for example, then in this case the last two loops need to traverse the entirety of both arrays again. However in the first solution, the last two loops were just going through the unique value sets, which should usually be a lot smaller than the entire arrays. For example, if we were working on arrays that only consisted of a few different values, but were exceedingly long, then rather than doing 3 iterations over the distinct values at the end we'd need to do thousands over again.

Still, I personally like memory optimizations over runtime optimizations, especially if we are going after micro-optimizations and nothing that actually impacts the time or space complexity.

## Other Leetcode Solutions

I found an interesting solution on Leetcode that takes special note of the constraints to guide their approach.

Remember this constraint?

> `-1000 <= nums1[i], nums2[i] <= 1000`

This means that we know the range of values will be between `-1000` and `1000`. Given that information, this user's solution uses an bool slice of size `2000` to store all unique values encountered. Basically, since an array can't have negative indices, he just adds `1000` to all numbers to put them in the range of `0` to `2000` instead.

Then, he first goes over `nums1`, marking down the set of values in this array by setting the corresponding indices in this array to be `true`. Then, when iterating over `nums2`, he can already know if a given value is unique or not by checking this array.

The approach is more or less the same, but the interesting thing is they use an array instead of a map/set.

Here's the code, courtesy of leetcode user `d9rs`:

```go
func setDifference(a, b []int) []int {
    d := [2001]bool{}
    ans := []int{}
    for _, x := range b {
        d[x + 1000] = true
    }
    for _, x := range a {
        if !d[x + 1000] {
            ans = append(ans, x)
            d[x + 1000] = true // prevent duplicates from being added
        }
    }
    return ans
}

func findDifference(nums1 []int, nums2 []int) [][]int {
    return [][]int{ setDifference(nums1, nums2), setDifference(nums2, nums1) }
}
```

**Note about Space Complexity**

The interesting thing about this solution is - technically - the array we are creating to store all 2000 possible unique values is actually constant space. Since the array will always be `2000` length, I believe that qualifies it for constant space, since even if the number of values being analyzed in `nums1` and `nums2` increased, the size of this array would not. _However_, if we consider the constraints, then technically my own equivalent - the `commonSet` map/set, is also "constant space" since no matter how large `nums1` and `nums2` were scaled up, it similarly would be limited to having 2000 total entries. I think the constraints to this problem skew how we can calculate time and space complexity anyway, since it sets pretty low limits on even the size of the `nums` arrays themselves.

> All that said, I think technically a map will consume less space on average than the array. The array will always have 2000 indices since that's how its initialized, but the map will only have key/value pairs for the number of unique values encountered, which will always be less than or equal to 2000 at most.

**Is this array more efficient than a map?**

Moving on from space, let's think about time. The map and array both need to handle the following operations:

-   adding a new value to the array or map

When adding a new value to the array, it's very efficient since the array is already initialized to the full length that's needed. So we aren't doing any append operations on the array, just changing the value at an already allocated position in the array. This is always an `O(1)` operation and should be super fast.

When adding a new value to the map, we are creating a new key/value pair. This should generally be very fast too, but under the hood there is a little more complexity going on. When adding a new key/value pair to a map, there is a hash calculation and insertion operation. The hash calculation determines the bucket where it should be placed, and the key/value pair is stored there as long as there are no collisions, usually in `O(1)` time. Collisions should be pretty rare, but they can happen if more than one value has the same hash calculation. If a collision does occur, Go's map structure handles it pretty well and on average can still resolve it in around `O(1)` time. But in the worst case scenario (all the data having the same hash calculations, but being different actual values), it could degrade to as slow as `O(n)`.

... ok, so that's a lot of details just to discuss what ultimately is a very minor difference in speed. But there you have it, the array wins in this case.

-   accessing a value from the array or map

We need to do this to check if a value is unique or not (has it already been mapped?). In arrays, this is super fast and easy. Arrays provide direct access to elements at an index, and the operation has `O(1)` time complexity.

For maps, the average case is `O(1)`, but the worst case is `O(n)`. This once again comes down to how well the hash function works with the data we have stored and are looking for. If all the data hashes to the same bucket, then finding it may be slow since it would need to search through all the other values in that same bucket.

> So here we once again have a small difference that mainly depends on worst case scenarios. But, it does seem the array solution is slightly more efficient.

I think I'm going to see if changing my `commonSet` map to an array helps out runtime speeds.

## Final Solution, Conclusion

After considering the solution I found on leetcode, here's my final solution, replacing the map with an array, and simplifying the looping a bit:

```go
func findDifference(nums1 []int, nums2 []int) [][]int {
    commonSet := make([]bool, 4002)
    dist1, dist2 := []int{}, []int{}
    minVal, maxVal := math.MaxInt, math.MinInt
    for _, v := range nums1 {
        commonSet[v + 1000] = true
        minVal = min(minVal, v)
        maxVal = max(maxVal, v)
    }
    for _, v := range nums2 {
        // values in nums2 will be recorded in its own range of the array
        commonSet[v + 3001] = true
        minVal = min(minVal, v)
        maxVal = max(maxVal, v)
    }
    for i := minVal; i < maxVal + 1; i++ {
        // check if this number is unique to nums1 or nums2
        if commonSet[i + 1000] && !commonSet[i + 3001] {
            dist1 = append(dist1, i)
        } else if !commonSet[i + 1000] && commonSet[i + 3001] {
            dist2 = append(dist2, i)
        }
    }
    return [][]int{dist1, dist2}
}
```

Time complexity: `O(n)`

Space complexity: `O(n)`

As I mentioned before - analyizing time and space complexity on this problem is a bit weird because the constraints given in the problem description limit the size of `nums` to 1000. That's pretty small, but it also means we can't really consider scaling this solution up to higher magnitudes. But, if we pretended those constraints weren't there, this is what the complexity would be.

The main change from my first solution is I opted to use a `[]bool` slice instead of a `map[int]bool` to record unique values. I used a little trick too - I gave `nums1` and `nums2` their own ranges in this array, since we know there can only be at most `1000` values in each of these arrays. So I used an offset of `3001` for all values from `nums2`, so they wouldn't overlap at all with numbers from `nums2`.

Why use `3001` as the offset for values in `nums2`, you might ask? Well, the extra `1` is to keep them from overlapping. Values in both `nums` arrays are limited to the range `-1000 <= x <= 1000`, so if `nums1` included a `1000` and `nums2` included a `-1000`, if we add `1000` to the values in `nums1` and `3000` to values in `nums2`, these both would end up being `2000`. So that's why we are adding `3001` to values in `nums2` instead of `3000`.

After making this change, it allowed me to reduce the amount of looping we do as well. I even added a little micro-optimization of only checking values in the known range that we've encountered in `nums1/2`, by recording `minVal` and `maxVal`. I'm not actually sure if it really makes a big difference, but it seems to help a bit with leetcode's performance stats.

This very well could be an "over-optimized" solution that starts getting a little more complex than it needs to be, but I think the code is still readable enough as it is now. In fact, it could be a little easier to read than the first implemention I made.

Main Takeaway: **pay attention to the constraints**. They might simplify the approach you can take for solving the problem. Also, although it's a minor thing, prefer arrays over maps when possible. They are slightly more efficient at accessing data and setting data - as long as you aren't needing to do **append** operations on the array. Always avoid those.

Ben's difficulty rating: 3.4/10. The concept itself wasn't too hard, but there was definitely a lot of room to explore different ideas on how to solve this one. I discovered an idea I hadn't thought of before, using arrays as a map by mapping values into the array as their index.
