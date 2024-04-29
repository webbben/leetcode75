# Merge Strings Alternately

[Leetcode link](https://leetcode.com/problems/merge-strings-alternately)

4/23/2024

Difficulty: Easy

## Description

You are given two strings `word1` and `word2`. Merge the strings by adding letters in alternating order, starting with `word1`. If a string is longer than the other, append the additional letters onto the end of the merged string.

Return the _merged string_.

**Example 1:**

```
Input: word1 = "abc", word2 = "pqr"
Output: "apbqcr"
Explanation: The merged string will be merged as so:
word1:  a   b   c
word2:    p   q   r
merged: a p b q c r
```

**Example 2:**

```
Input: word1 = "ab", word2 = "pqrs"
Output: "apbqrs"
Explanation: Notice that as word2 is longer, "rs" is appended to the end.
word1:  a   b
word2:    p   q   r   s
merged: a p b q   r   s
```

**Example 3:**

```
Input: word1 = "abcd", word2 = "pq"
Output: "apbqcd"
Explanation: Notice that as word1 is longer, "cd" is appended to the end.
word1:  a   b   c   d
word2:    p   q
merged: a p b q c   d
```

**Constraints:**

-   `1 <= word1.length, word2.length <= 100`
-   `word1` and `word2` consist of lowercase English letters.

## Initial Thoughts / Planning

This seems relatively simple; I'm envisioning just using a for-loop, incrementing an index up to the length of the smaller word (if one is smaller than the other), and then just adding the remaining slice of the longer word to the end.

I don't think I'll need to do anything fancy, and I also don't think this solution idea is too inefficient or anything.

## Code Implementations, Optimizations

### First implementation

This is the first solution I put together - just looping between the two words, and then adding on any remainder to the end if one word is longer than the other.

```go
func mergeAlternately(word1 string, word2 string) string {
    minSize := min(len(word1), len(word2))

    merged := ""

    for i := 0; i < minSize; i++ {
        merged += string(word1[i]) + string(word2[i])
    }

    if len(word1) > minSize {
        merged += word1[minSize:len(word1)]
    } else if len(word2) > minSize {
        merged += word2[minSize:len(word2)]
    }
    return merged
}
```

Runtime: 2ms (beats 37% of go users)

Memory: 2.47 MB (beats 55% of go users)

I don't know if there's a ton of room for optimizing this, but one thought I considered is: maybe instead of starting with `merged := ""`,
I could start by initializing `merged` to `word1` and then inserting the characters from `word2`. But perhaps that wouldn't be an improvement - it mostly depends on if an insertion is the same efficiency as adding a character to the end of a string. If it is, then it would be about half the number of string adding operations.

Another idea is; perhaps adding strings using the `+` sign is slightly inefficient; maybe there's a function in the built-in `strings` package that we could utilize, or maybe even `fmt.Sprintf()` would be slightly more efficient. I'd need to investigate these things more.

### Let's ask Chat-GPT

Here's what Chat-GPT has to say about my solution:

> Instead of repeatedly concatenating strings inside the loop, which can be inefficient due to string immutability in Go, you can use a bytes.Buffer for better performance. Here's how you can do it:

```go
import "bytes"

func mergeAlternately(word1 string, word2 string) string {
    minSize := min(len(word1), len(word2))

    var merged bytes.Buffer

    for i := 0; i < minSize; i++ {
        merged.WriteByte(word1[i])
        merged.WriteByte(word2[i])
    }

    if len(word1) > minSize {
        merged.WriteString(word1[minSize:])
    } else if len(word2) > minSize {
        merged.WriteString(word2[minSize:])
    }
    return merged.String()
}

```

> By using bytes.Buffer, you avoid the repeated allocation and copying of strings, leading to better performance, especially for larger inputs.

Here's a longer explanation it gives as to why writing to a buffer is more efficient than string concatenations:

> Writing to a buffer is more efficient than string concatenation because strings in Go are immutable. When you concatenate two strings using the + operator or += operation, a new string is created with the combined contents of the two original strings. This operation involves allocating memory for the new string, copying the contents of the original strings into the new string, and then appending the new content. If you're concatenating multiple strings in a loop, this process happens repeatedly, leading to a lot of unnecessary memory allocations and copying.

> In contrast, a bytes.Buffer provides a mutable buffer where you can efficiently append bytes (or strings) without the overhead of creating new strings each time. The WriteByte and WriteString methods of bytes.Buffer allow you to append bytes or strings to the buffer efficiently, without needing to allocate new memory or copy existing content unnecessarily. This makes it more efficient for string concatenation, especially in loops.

So - it sounds like one optimization idea is indeed replacing the inefficient string concatenation, but with a different approach of writing to a mutable buffer instead.

If I did insist on using string concatenation, it does sound like rather than using `+` concatenations, I could use `strings.Builder` to do something similar to the `bytes.Buffer` approach. It would essentially be the same thing. In fact, ChatGPT seems to think they might be about the same efficiency-wise.

## Final Code Implementation, Conclusion

Here's the final, optimized solution:

```go
import "bytes"

func mergeAlternately(word1 string, word2 string) string {
    minSize := min(len(word1), len(word2))

    var merged bytes.Buffer

    for i := 0; i < minSize; i++ {
        merged.WriteByte(word1[i])
        merged.WriteByte(word2[i])
    }

    if len(word1) > minSize {
        merged.WriteString(word1[minSize:])
    } else if len(word2) > minSize {
        merged.WriteString(word2[minSize:])
    }
    return merged.String()
}
```

Runtime: 0 ms (hm? - well, beats 100% of go users. I'll take it!)
Memory: 2.29 MB (beats 86% of go users)

I re-ran the submission a couple times to make sure it wasn't just a dud, but it does seem this new solution may be faster, and also seems to use less memory. So main lesson we learned is:

**don't concatenate immutable strings in big loops** - instead, use something like a bytes buffer that is mutable.

Ben's difficulty rating: 4.5/10. It's not too complicated of a concept, but you have to pay attention to make sure you do it efficiently.
