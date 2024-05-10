# Maximum Number of Vowels in a Substring of Given Length

[leetcode link](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/)

5/9/2024

Difficulty: Medium

## Description

Given a string `s` and an integer `k`, return the maximum number of vowel letters in any substring of `s` with length `k`.

Vowel letters in English are `a`, `e`, `i`, `o`, and `u`.

**Example 1:**

```
Input: s = "abciiidef", k = 3
Output: 3
Explanation: The substring "iii" contains 3 vowel letters.
```

**Example 2:**

```
Input: s = "aeiou", k = 2
Output: 2
Explanation: Any substring of length 2 contains 2 vowels.
```

**Example 3:**

```
Input: s = "leetcode", k = 3
Output: 2
Explanation: "lee", "eet" and "ode" contain 2 vowels.
```

**Constraints:**

-   `1 <= s.length <= 10^5`
-   `s` consists of lowercase English letters.
-   `1 <= k <= s.length`

## Initial Thoughts / Planning

I think this is going to be extremely similar to the last problem: sliding window, but this time over characters in a string, and whenever we encounter a vowel (or leave a vowel behind) we increment or decrement the vowel count, keeping track of the overall maximum. Let's get started.

## Code Implementations, Optimizations

```go
func maxVowels(s string, k int) int {
    left, right := 0, k - 1
    vowelCount := 0
    // first, get the vowel count for the first k characters
    for i := 0; i < k; i++ {
        if isVowel(s[i]) {
            vowelCount++
        }
    }
    maxVowelCount := vowelCount

    // now, slide the window over the rest of the string's characters
    for right < len(s) - 1 {
        right++
        if isVowel(s[right]) {
            vowelCount++
        }
        if isVowel(s[left]) {
            vowelCount--
        }
        left++
        maxVowelCount = max(maxVowelCount, vowelCount)
    }
    return maxVowelCount
}

func isVowel(c byte) bool {
    return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u'
}
```

Time Complexity: `O(n)`, since it only includes one iteration over the string, without any nested for loops. `isVowel` is also constant time.

Space Complexity: `O(1)`, since it only uses a constant number of extra variables.

Since this problem is almost identical to the last one, just dealing with strings and characters instead of numbers, I don't think there's much value in digging in deep for micro-optimizations here. I could save a bit of space by inlining `isVowel` probably, but firstly that just makes the code uglier, and secondly, in terms of actual optimization in the real world, the Go compiler should do that automatically for us. Since the leetcode performance stats says that my solution beats 71% in runtime and 84% in memory, I think that shows there aren't any big issues I should be fixing here too.

## Final Solution, Conclusion

Sticking with our first solution:

```go
func maxVowels(s string, k int) int {
    left, right := 0, k - 1
    vowelCount := 0
    // first, get the vowel count for the first k characters
    for i := 0; i < k; i++ {
        if isVowel(s[i]) {
            vowelCount++
        }
    }
    maxVowelCount := vowelCount

    // now, slide the window over the rest of the string's characters
    for right < len(s) - 1 {
        right++
        if isVowel(s[right]) {
            vowelCount++
        }
        if isVowel(s[left]) {
            vowelCount--
        }
        left++
        maxVowelCount = max(maxVowelCount, vowelCount)
    }
    return maxVowelCount
}

func isVowel(c byte) bool {
    return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u'
}
```

Main takeaway: nothing really, just another rendition of the sliding window problem.

Ben's difficulty rating: 3/10. I can't say it's any easier than the last problem which is almost identical, so we'll keep the rating the same. If you can figure out that last problem, I think this one will be just as simple.
