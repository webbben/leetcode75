# Is Subsequence

[leetcode link](https://leetcode.com/problems/is-subsequence)

5/1/2024

Difficulty: Easy

## Description

Given two strings `s` and `t`, return `true` if `s` is a subsequence of `t`, or `false` otherwise.

A subsequence of a string is a new string that is formed from the original string by deleting some (can be none) of the characters without disturbing the relative positions of the remaining characters. (i.e., `"ace"` is a subsequence of `"abcde"` while `"aec"` is not).

**Example 1:**

```
Input: s = "abc", t = "ahbgdc"
Output: true
```

**Example 2:**

```
Input: s = "axc", t = "ahbgdc"
Output: false
```

**Constraints:**

-   `0 <= s.length <= 100`
-   `0 <= t.length <= 10^4`
-   `s` and `t` consist only of lowercase English letters.

**Follow up:** Suppose there are lots of incoming s, say s1, s2, ..., sk where k >= 109, and you want to check one by one to see if t has its subsequence. In this scenario, how would you change your code?

## Initial Thoughts / Planning

I think this problem seems pretty simple: traverse through the characters of `t`, looking for each character in `s`. For example:

if `s = "ace"` and `t = "abcde"`,

We would start with character `'a'` (first character of `s`) and find the first occurence of it in `t`. Then, from that position in `t`, we would continue searching further, but for the next character `'c'`, and then `'e'`. If we reach the end of string `t` before having found all characters in `s`, then `s` is not a subsequence.

## Code Implementations, Optimizations

```go
func isSubsequence(s string, t string) bool {
    if s == "" {
        return true
    }
    if t == "" {
        return false
    }
    si := 0 // index of the substring's character we are searching for
    for i := 0; i < len(t); i++ {
        if s[si] == t[i] {
            // substring char found, move to next
            si++
            if si >= len(s) {
                return true
            }
        }
    }
    return false
}
```

Runtime: 1 ms (beats 84%)

Memory: 2.18 MB (beats 78%)

Well I think this solution works pretty well. It's O(n) time complexity since it only does one iteration over string `t` in the worst case, and it's constant space complexity since we don't create any arrays or anything like that.

I don't really see much potential for micro-optimizations or anything like that, and looking around on Leetcode, I don't see any solutions that take a different approach than this or improve upon it. I think I'm gonna skip asking Chat-GPT since this is a pretty straightforward problem.

## Final Solution, Conclusion

(same as my first solution)

```go
func isSubsequence(s string, t string) bool {
    if s == "" {
        return true
    }
    if t == "" {
        return false
    }
    si := 0 // index of the substring's character we are searching for
    for i := 0; i < len(t); i++ {
        if s[si] == t[i] {
            // substring char found, move to next
            si++
            if si >= len(s) {
                return true
            }
        }
    }
    return false
}
```

Runtime: 1 ms (beats 84%)

Memory: 2.18 MB (beats 78%)

Nice and easy - no big takeaways here!

Ben's difficulty rating: 2/10.
