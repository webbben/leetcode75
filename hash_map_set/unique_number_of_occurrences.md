# Unique Number of Occurrences

[leetcode link](https://leetcode.com/problems/unique-number-of-occurrences/)

5/13/2024

Difficulty: Easy

## Description

Given an array of integers `arr`, return `true` if the number of occurrences of each value in the array is unique or `false` otherwise.

**Example 1:**

```
Input: arr = [1,2,2,1,1,3]
Output: true
Explanation: The value 1 has 3 occurrences, 2 has 2 and 3 has 1. No two values have the same number of occurrences.
```

**Example 2:**

```
Input: arr = [1,2]
Output: false
```

**Example 3:**

```
Input: arr = [-3,0,1,-3,1,1,1,-3,10,0]
Output: true
```

**Constraints:**

-   `1 <= arr.length <= 1000`
-   `-1000 <= arr[i] <= 1000`

## Initial Thoughts / Planning

Just to quickly clarify the problem: we are finding the numbers that appear in the array a **unique number of times**. Unique means the frequency that number appears in the array is not the same as any other number's frequency. At first I thought this problem was for finding unique numbers or something, so I wanted to clarify that.

Ok, so we are looking to find if all numbers have **unique frequency** - or that no number appears at the same frequency as any other number. If any number has the same frequency as another number, we return false.

My first thought is I can use a map for this. We can create a `map[int]int`, which maps a number found in `arr` to the frequency it occurs. We simply iterate over `arr`, counting up the frequency of each number. This seems possibly in line with the intended solution, just because we are still in Hash Map and Set territory. However, once we have the frequency map, the problem becomes, how do we find if any of the frequencies match? I think there are a lot of possibilities here, but these come to mind:

1. We can do "brute force", where we compare all numbers frequencies to each other, until we find a match. This is not a good solution (is brute force ever a good solution?), as it would be `O(n^2)`.

2. We could create another map, one which maps a frequency value to the numbers that are of that frequency. It's basically like inverting the map we've already created. If any frequency value has more than one number being attempted to be assigned there, we return false. This solution isn't too bad, as we remain in linear time complexity overall, but we do have to create another map, potentially doubling our memory usage (space complexity remains linear though too).

This solution of using a map to get frequencies is probably the fastest, at least that I'm thinking of for now, but uses the most memory since we will need probably two maps to store values in.

Another idea I have is, what if we **sorted the array first?** We've done this in other problems to simplify things. We could sort the array so that we can count the frequency of each number as we progress, which would eliminate one of the maps. However, we would still need the "inverted frequency map" described in the last idea - mapping the frequency value to numbers that fall in that frequency bucket. Additionally, sorting the array will take `O(n logn)` time (previously we dug into Go's sorting, and found it uses a modified quicksort algorithm). This adds a bit of time to the overall solution, and only cuts down on part of the memory usage. If it were a constant memory solution, then I'd be more enticed to go this route.

For now, I think I'm going to tackle the first solution idea: making a map of numbers to their frequencies, then inverting it to bucket numbers by frequency.

## My Code Implementations, Optimizations

```go
func uniqueOccurrences(arr []int) bool {
    // first, map all numbers to their frequency
    freq := map[int]int{}
    for _, v := range arr {
        if _, ok := freq[v]; !ok {
            freq[v] = 1
        } else {
            freq[v] = freq[v] + 1
        }
    }
    freqBucket := map[int]int{}
    // next, put all numbers into their frequency buckets
    // if any frequency bucket already exists, then return false
    for num, f := range freq {
        if _, ok := freqBucket[f]; ok {
            return false
        }
        freqBucket[f] = num
    }
    return true
}
```

Time complexity: `O(n)`

Space complexity: `O(n)`

This solution works, and is pretty efficient I'd say. But the main areas for improvement I'd say would be to either reduce the looping down to one iteration over `arr`, or to eliminate one (or both) maps we are creating. I think the most plausible option is to eliminate a map, but as we pointed out before, it may require some extra work that would slow down the runtime a bit.

> After spending a few minutes, I haven't really come up with new ideas beyond what we've talked about above. I don't think ultimately the sorting idea is good, because that will just mean our time complexity goes up to `O(n logn)` while space complexity stays the same - a net negative. I also looked at the hints and they are both things we've already figured out for our first solution. Let's move on to looking at other leetcode solutions.

## Other Leetcode Solutions

One interesting thing I found is, one solution uses arrays instead of maps again - just as the last hashmap/set problem did. I guess the theory here is that arrays, when no appending is needed, are a little more efficient overall than maps. On average they have the same time complexity for their common operations like accessing a value or setting, but arrays are just slightly faster and maps have some edge cases where they can slow down.

One again, since the constraints specify the range of values we can encounter (`-1000 <= x <= 1000`), we can make an array to hold the frequency of every possible value. we will make an `int[]` of size 2000, and add 1000 to all numbers encountered so that negative numbers all have a `>= 0` index. We then use this array in the same way as we use the `freq` map, and then once again attempt to bucket the numbers by frequency, this time using an array instead of map again. Let's try it out!

```go
func uniqueOccurrences(arr []int) bool {
    // first, map all numbers to their frequency
    freq := make([]int, 2002)
    for _, v := range arr {
        freq[v + 1001] += 1
    }
    freqBucket := make([]int, 1001)
    // next, put all numbers into their frequency buckets
    // if any frequency bucket already exists, then return false
    for num, f := range freq {
        if f == 0 {
            continue
        }
        if freqBucket[f] != 0 {
            return false
        }
        freqBucket[f] = num
    }
    return true
}
```

So as you can see, the only real difference now is `freq` and `freqBucket` are `[]int` arrays instead of `map[int]int` maps.

Some details about the array sizes, offsets to values, etc:

-   `freq` is initialized to length `2002` since it was going out of bounds at length `2001`. The reason is, we needed to make sure no value would be recorded as `0` in the `freq` array, since that's the "zero value" of an integer in Go. We are using the "zero value" to indicate nothing is set there, so that's why we are also doing:

```go
freq[v + 1001] += 1
```

This makes sure that even `-1000` would not end up stored as `0` in the `freq` array.

-   `freqBucket` stores the number that corresponds to the frequency defined by the index in the array. But, you'll notice we store these numbers as just:

```go
freqBucket[f] = num
```

Shouldn't it be `num - 1001`? Well, for our purposes it doesn't really matter. We just want to make sure the same number doesn't have the same frequency, but it doesn't actually matter if we keep track of the specific number's value correctly. So, to make things simpler, I decided to leave the numbers offset by `1001`. **But**, more importantly, it also prevents any number assigned to a frequency bucket as being recorded as `0` - since once again, that is the "zero value" for integers, and we interpret that as a "lack of a value" in this `freqBucket` too.

## Let's ask Chat-GPT

I showed Chat-GPT my solution above (after the leetcode inspired modification), and it started insisting there's a way to use one map instead of two arrays. But, after repeatedly giving me code with two maps (like my original solution), and me repeatedly trying to correct it, it repeatedly apologizing and continuing to give me the exact same code after "fixing it to truly use one map", I came to the conclusion that Chat-GPT has gone a little insane. So, I'm going to just assume that for now my solution works pretty well.

> I'm starting to question the value of Chat-GPT in this process, btw. I might start going without it, except for situations where I'm not able to find help elsewhere.

## Final Solution, Conclusion

```go
func uniqueOccurrences(arr []int) bool {
    // first, map all numbers to their frequency
    freq := make([]int, 2002)
    for _, v := range arr {
        freq[v + 1001] += 1
    }
    freqBucket := make([]int, 1001)
    // next, put all numbers into their frequency buckets
    // if any frequency bucket already exists, then return false
    for num, f := range freq {
        if f == 0 {
            continue
        }
        if freqBucket[f] != 0 {
            return false
        }
        freqBucket[f] = num
    }
    return true
}
```

Time complexity: `O(n)`

Space complexity: `O(1)` - given the constraints, we are able to do this with constant space, similar to last problem. If we expanded this problem to include much larger numbers, then perhaps we'd need to change the solution a bit.

I think this is probably pretty much the optimal solution. Leetcode does show it to do really well performance wise too, which is cool. This is another case where, given the restraints, we were able to replace maps with arrays, which are slightly faster in some cases. It also gives us the "illusion" of being more of a constant space solution, even though with the constraints, I think using a map is technically going to end up constant space too. Maybe I'm wrong, but it feels a little bit like "cheating" to say this solution is constant space, while the first one is linear space.

No major takeaway here. I think it's the same as last problem - arrays perform slightly better than maps when you can pre-allocate all the space and not need to do appends.

Ben's difficulty rating: 4/10. It was a bit more complicated than the last problem, but still felt manageable.
