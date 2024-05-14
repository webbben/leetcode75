# Determine if Two Strings are Close

[leetcode link](https://leetcode.com/problems/determine-if-two-strings-are-close/)

5/14/2024

Difficulty: Medium

## Description

Two strings are considered "close" if you can attain one from the other using the following operations:

-   Operation 1: swap any two existing characters
    -   For example `abcde -> aecdb` (`b` and `e` were swapped)
-   Operation 2: transform every occurrence of one existing character into another existing character, and do the same with the other character.
    -   For example, `aacabb -> bbcbaa` (all `a`'s turn into `b`'s, and vice versa)

You can use the operations on either string as many times as necessary.

Given two strings, `word1` and `word2`, return `true` if `word1` and `word2` are "close", and `false` otherwise.

**Example 1:**

```
Input: word1 = "abc", word2 = "bca"
Output: true
Explanation: You can attain word2 from word1 in 2 operations.
Apply Operation 1: "abc" -> "acb"
Apply Operation 1: "acb" -> "bca"
```

**Example 2:**

```
Input: word1 = "a", word2 = "aa"
Output: false
Explanation: It is impossible to attain word2 from word1, or vice versa, in any number of operations.
```

**Example 3:**

```
Input: word1 = "cabbba", word2 = "abbccc"
Output: true
Explanation: You can attain word2 from word1 in 3 operations.
Apply Operation 1: "cabbba" -> "caabbb"
Apply Operation 2: "caabbb" -> "baaccc"
Apply Operation 2: "baaccc" -> "abbccc"
```

**Constraints:**

-   `1 <= word1.length, word2.length <= 10^5`
-   `word1` and `word2` contain only lowercase English letters.

## Initial Thoughts / Planning

Hmm, this is pretty interesting. After reading this, I'm actually pretty unsure how to proceed.

So, we have these two operations - both of which involve swapping characters - and we need to determine if it's possible to convert the strings into each other using these operations.

> Right off the bat, let's just say that no, I'm 90% sure the solution isn't repeatedly applying these operations until one string is transformed into the other. That just seems to me to be very complicated and not really feasible even. Firstly, how would you know when to stop applying operations and give up/return false?

So disregarding that idea, the first thing that stands out to me is, after doing these operations, the strings still are pretty similar: if you swap all `a` characters to `b` (and vice versa), then in the end there are still the same number of characters in certain groups - just different characters. Maybe a good way to put it is the "relative frequency" of different chars remains the same.

Let's look at some of the example cases:

`abc` and `bca`: in both strings there are 3 letters, each appearing once. We can just manually swap all of them to get from one string to another.

`cabbba` and `abbccc`: in both of these strings, there is one letter with a frequency of 1, one letter with a frequency of 2, and one letter with a frequency of 3.

So, is the solution that we need to count the frequencies of each letter, then, sort of similar to the last problem, "bucket" each character by frequency, so we know how many characters in each string have a certain frequency? My guess here is that, if both strings have the same number of characters for each frequency bucket, then it should be possible to convert them into each other.

Ah, and by the way, a base case we can apply right away is if the length of the strings isn't equal, we return `false`, since of course they con't be converted into each other.

Anyway, I'm gonna dive into this solution and see how it goes.

## My Code Implementations, Optimizations

Ok, so the solution described above apparently passes 145/167 test cases, but here's one that it fails on:

```
word1 = "uau"
word2 = "ssx"

Output: true
Expects: false
```

So, what's going on here?

Well, here the answer should be `false`: the set of letters in `word1` is different from `word2`! So of course, no matter how much swapping you could attempt, it's impossible since there's no way to turn a `u` to an `s` or an `a` to an `x`. So, if I'm not mistaken, another requirement is the set of letters in `word1` must be the same as `word2`.

Alright, here's my first solution, applying the above logic to pass the remaining cases:

```go
func closeStrings(word1 string, word2 string) bool {
    if len(word1) != len(word2) {
        return false
    }
    if word1 == word2 {
        return true
    }
    // get the frequencies of each character in each string
    freq1 := getFreq(word1)
    freq2 := getFreq(word2)
    // make sure the set of characters for each string is the same
    if len(freq1) != len(freq2) {
        return false
    }
    for c, _ := range freq1 {
        if _, ok := freq2[c]; !ok {
            return false
        }
    }
    // make sure there are the same number of characters for each frequency bucket
    bucket1 := getFreqBucket(freq1)
    bucket2 := getFreqBucket(freq2)
    if len(bucket1) != len(bucket2) {
        return false
    }
    for f, numChars := range bucket1 {
        numChars2, ok := bucket2[f]
        if !ok {
            return false
        }
        if numChars != numChars2 {
            return false
        }
    }
    return true
}

func getFreq(word string) map[byte]int {
    freq := map[byte]int{}
    for i := 0; i < len(word); i++ {
        if _, ok := freq[word[i]]; !ok {
            freq[word[i]] = 1
        } else {
            freq[word[i]] += 1
        }
    }
    return freq
}

func getFreqBucket(freq map[byte]int) map[int]int {
    bucket := map[int]int{}
    for _, f := range freq {
        if _, ok := bucket[f]; !ok {
            bucket[f] = 1
        } else {
            bucket[f] += 1
        }
    }
    return bucket
}
```

Time complexity: `O(n)` - The solution is quite lengthy, but still scales linearly with the size of the words.

Space complexity: `O(n)` - Again, a lot of memory is being used, but the memory usage is linear still.

This solution is pretty long, and doesn't seem to perform that great judging by the leetcode performance stats. The memory usage is actually alright, but the runtime is pretty slow apparently.

Let's break down the algorithm we have here:

-   First, get the frequencies of each character for both words. This maps a character to its number of occurrences, but also effectively acts as a Set of all characters in the word.
-   Next, make sure the set of characters in both strings are the same. i.e. both words have the same characters, disregarding their frequencies
-   Then, bucket the characters for each word by frequency. So if `a` appears 3 times, it's put into the `3` bucket. In the end, we will know how many letters had the frequency `3`, or `4`, etc.
-   with the frequency buckets, we then make sure that both words have the same number of characters for each frequency.
-   If all the above checks pass, we know these two strings are "close".

### Talking through optimizations with Chat-GPT

> at this point I was still pretty stumped, so I started talking to Chat-GPT about some of the helper functions, seeing if there is room to improve efficiency there. I got some interesting results actually.

It turns out that this code in `getFreq` was slowing our runtime down considerably:

```go
func getFreq(word string) map[byte]int {
    freq := map[byte]int{}
    for i := 0; i < len(word); i++ {
        if _, ok := freq[word[i]]; !ok { // here we are checking if the character has been encountered yet. we don't need this!
            freq[word[i]] = 1
        } else {
            freq[word[i]] += 1
        }
    }
    return freq
}
```

Our code is first checking if the given character has been mapped yet, and if not it's handling it specially to initialize that key/value pair. Apparently, this isn't necessary for `int` types in Go, as a Go map automatically initializes zero values for `int` when the key doesn't exist. I'm not sure if this applies to all types, but I think we can start assuming it is until proven otherwise.

> Making this small adjustment, our runtime went from 120 ms to just around 70 ms:

```go
func getFreq(word string) map[byte]int {
    freq := map[byte]int{}
    for i := 0; i < len(word); i++ {
        freq[word[i]]++ // just directly increment it, without checking for its presence first
    }
    return freq
}
```

That's a pretty dramatic improvement for just one little change.

Similarly, we can make this adjustment to `getFreqBucket`:

```go
func getFreqBucket(freq map[byte]int) map[int]int {
    bucket := map[int]int{}
    for _, f := range freq {
        bucket[f]++
    }
    return bucket
}
```

However, after making this change I'm not seeing as much of a difference in runtimes as recorded by leetcode. Still, either way it makes the code cleaner and easier to read.

**Arrays instead of Maps**

Another modification, which is becoming quite the pattern in this section, is using arrays instead of maps when possible. Here, since the characters in the string are restricted to english lowercase letters only, we know there is a range of 26 characters possible in the strings. So, we could instead use an array of integers of length 26. I won't go into why it's more efficient (see previous problems for details there), but it should speed things up a bit!

> After these modifications, the runtime speed shot up dramatically to 19 ms! So clearly, the array modifications were definitely worth it.

The main change here is with `getFreq` again, where we now use a `[26]int` array to store the frequencies for each character:

```go
func getFreq(word string) [26]int {
    var freq [26]int
    for i := 0; i < len(word); i++ {
        freq[word[i] - 'a']++
    }
    return freq
}
```

How this works is, we can get the unicode value for these characters by subtracting `'a'` from them. This makes `'a'` the first value (index `0`), since the characters are limited to lowercase english letters and it's the first one. So, `'a'` evaluates to `0`, and `'z'` evaluates to `25`

> A note on fixed-size arrays:
>
> Here, we use a fixed size array of type `[26]int`. But why not just make a slice that is length `26` instead, using `make`?
> Well, it turns out that there is a small performance gain to be had by using fixed-size arrays when possible, instead of slices that can have their size increased or decreased.
> I actually didn't know about fixed-size arrays yet, and tend to use the term "array" instead of slice, which might be technically incorrect of me in terms of Go. So here's what Chat-GPT had to say about the benefits of fixed-size arrays in our use case:
>
> **Why Use [26]int in This Context?**
>
> In your getFreq function, you are working with a fixed number of lowercase English letters (26). Using a fixed-size array [26]int has some advantages:
>
> -   **Fixed Size and Simplicity:** The size is known at compile time, which makes it clear that you're dealing with exactly 26 elements. This can be simpler and more efficient in terms of memory allocation.
> -   **Performance:** Accessing elements in a fixed-size array is generally faster since there is no overhead of slice headers and bounds checks are straightforward.
> -   **Memory Allocation:** Arrays have a fixed size and are allocated once on the stack (if their size is known at compile time), which can be more efficient than dynamic memory allocation.

## Final Solution, Conclusion

Here's my final solution, after adding optimizations suggested by Chat-GPT:

```go
func closeStrings(word1 string, word2 string) bool {
    if len(word1) != len(word2) {
        return false
    }
    if word1 == word2 {
        return true
    }
    // get the frequencies of each character in each string
    freq1 := getFreq(word1)
    freq2 := getFreq(word2)
    // make sure the same characters are present in both strings
    for i := 0; i < 26; i++ {
        if freq1[i] > 0 && freq2[i] == 0 || freq1[i] == 0 && freq2[i] > 0 {
            return false
        }
    }
    // make sure there are the same number of characters for each frequency bucket
    bucket1 := getFreqBucket(freq1)
    bucket2 := getFreqBucket(freq2)
    if len(bucket1) != len(bucket2) {
        return false
    }
    for f, numChars := range bucket1 {
        numChars2, ok := bucket2[f]
        if !ok {
            return false
        }
        if numChars != numChars2 {
            return false
        }
    }
    return true
}

func getFreq(word string) [26]int {
    var freq [26]int
    for i := 0; i < len(word); i++ {
        freq[word[i] - 'a']++
    }
    return freq
}

func getFreqBucket(freq [26]int) map[int]int {
    bucket := map[int]int{}
    for _, f := range freq {
        if f != 0 {
            bucket[f]++
        }
    }
    return bucket
}
```

Time complexity: `O(n)`

Space complexity: `O(1)` (due to the constraints on characters to just the 26 letters of the alphabet, even our maps are constant space).

So, comparing to our first implementation, which at least according to leetcode metrics, was quite slow, the main change was to use arrays instead of maps for the frequency counting.

Since we are only dealing with lowercase english letters, we know there are exactly 26 possible characters we can encounter, and additionally they are all right next to each other in terms of their unicode codes. So it's pretty easy to just use a fixed-size array to count the frequency of the characters.

This problem was actually relatively hard for me at first. I didn't see an obvious technique for solving the problem at first, just based on the description. I had to look closer at the example cases to notice some patterns that we can use to solve this problem, so it was different a bit different from most of the other problems I've solved so far.

Once again, the big factor here ended up being using arrays instead of maps when possible. Apparently the difference is bigger than you'd think, at least judging by the performance metrics leetcode gives me. Moving from maps to arrays easily cut the average runtimes down in half, if not more, even though technically both solutions were `O(n)` time complexity.

Main takeaways: once again, try to find a way to use arrays instead of maps when possible (keep an eye on constraints, which may allow this to happen), and also if the problem seems like it calls for an overly complicated solution, take a closer look at examples and try to find some patterns.

Ben's difficulty rating: 7.2/10. Seemed scary at first, but we got to the bottom of it without too much trouble.
