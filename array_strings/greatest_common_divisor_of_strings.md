# Greatest Common Divisor of Strings

[Leetcode link](https://leetcode.com/problems/greatest-common-divisor-of-strings)

4/23/2024

Difficulty: Easy

## Description

For two strings, `s` and `t`, we say "`t` divides `s`" if and only if `s = t + t + t + ... + t` (i.e., `t` is concatenated with itself one or more times).

Given two strings `str1` and `str2`, return _the largest string `x` such that `x` divides both `str1` and `str2`._

**Example 1:**

```
Input: str1 = "ABCABC", str2 = "ABC"
Output: "ABC"
```

**Example 2:**

```
Input: str1 = "ABABAB", str2 = "ABAB"
Output: "AB"
```

**Example 3:**

```
Input: str1 = "LEET", str2 = "CODE"
Output: ""
```

**Constraints:**

-   `1 <= str1.length, str2.length <= 1000`
-   `str1` and `str2` consist of English uppercase letters.

## Initial Thoughts / Planning

So, we want to find the largest string that "divides" both input strings. "Divides" means that the "divisor" string is repeated over and over again to make up the entire string. So right off the bat, that means:

-   both strings must start with the same letter, and consistently have the same letters. if there is ever a single mismatch at a given position `i` in both strings, then there is no mutual divisor.

-   if there are no mismatches (same position index, different character), then we just need to figure out how big of a slice consistutes a mutual divisor. this is probably calculable just based on the string lengths, rather than having to test each individual possible divisor substring.

So, I think I'll go into this with this idea for the algorithm:

1. iterate through the strings, comparing the characters that are in each string at the same position. If there is a mismatch, return "" (no common divisor). If one string is longer, no problem.

2. compare the lengths of the strings; if they are the same size, then of course the common divisor is the entire string. if one is longer, then I _believe_ that means the extra piece of the longer string is the common divisor substring. If you think about it, it needs to be, or else there's no way to have a divisor between the two.
    - if one is longer than the other, test if that extra piece is indeed the substring; multiply it up to be the same size of the other string, and compare them. If it's not, then return "" as there is no common divisor.

## Code Implementations, Optimizations

### First Implementation

I started by implementing what is described above, but ran into a case I hadn't anticipated:

str1: `"TAUXXTAUXXTAUXXTAUXXTAUXX"`
str2: `"TAUXXTAUXXTAUXXTAUXXTAUXXTAUXXTAUXXTAUXXTAUXX"`

My code was assuming that the extra chunk on the end of `str2`, `"TAUXXTAUXXTAUXXTAUXX"`, must be the common divisor, but it's actually not; it's ~~one too many `TAUXX`'s~~ actually, it's 3 too many - because it needs to be the divisor of **both** strings. So just something to be careful of.

Ok, here's my code solution:

```go
import (
    "strings"
)

func gcdOfStrings(str1 string, str2 string) string {
    var subStr string
    var longStr string
    var shortStr string
    // first, make sure the strings have the same characters in the same positions
    if len(str1) == len(str2) {
        // they must be identical, or there is no common divisor
        if str1 == str2 {
            return str1
        } else {
            return ""
        }
    } else if len(str1) > len(str2) {
        // if they don't share the same "stem", then they can't have a common divisor
        if str1[:len(str2)] != str2 {
            return ""
        }
        longStr = str1
        shortStr = str2
        subStr = str1[len(str2):]
    } else {
        if str1 != str2[:len(str1)] {
            return ""
        }
        longStr = str2
        shortStr = str1
        subStr = str2[len(str1):]
    }
    // check if subStr or a portion of it is the common divisor
    for i := 0; i < len(subStr); i++ {
        subSubStr := subStr[:len(subStr)-i]
        if isDivisor(subSubStr, longStr) && isDivisor(subSubStr, shortStr) {
            return subSubStr
        }
    }
    return ""
}

func isDivisor(sub string, full string) bool {
    if len(full) % len(sub) != 0 {
        return false
    }
    return strings.Repeat(sub, len(full) / len(sub)) == full
}
```

Runtime: 2 ms (beats 71% for go)

Memory: 2.76 MB (beats 33% for go)

This solution seems to work pretty well overall - it's apparently faster than most go solutions, however it does use more memory than most go solutions. For the memory, I have a feeling some of it is just how I'm allocating some extra variables to make things more readable or easier to follow. But it might also be caused by how my isDivisor code works - perhaps there's a better way to do that?

Also, I actually realized that my original idea - looping through the strings and comparing one character at a time - seemed like an inefficient idea. So I opted to get slices of the strings and compare the entire slices wholesale. I'm assuming that one slice operation on a string and one string comparison with 2 large strings is better than n character extractions and n character comparisons.

### Let's ask Chat-GPT

Here's Chat-GPT's full analysis:

> 1. Handling Edge Cases: You're handling the case when the lengths of str1 and str2 are equal and when one is longer than the other. However, you can also optimize by handling some edge cases upfront:
>
> -   If either str1 or str2 is empty, there can't be a common divisor. You can add a check for this at the beginning.
> -   If one of the strings is a prefix of the other, then the shorter string is the common divisor. You can add a check for this condition as well.
>
> 2. Avoiding Substring Creation: In your loop, you're creating substrings subSubStr in each iteration. This can be avoided by manipulating indices directly on the original strings.
>
> 3. Optimizing isDivisor Function: Instead of using strings.Repeat to check if sub is a divisor of full, you can directly compare substrings of full to sub. This avoids creating a new string using strings.Repeat.

So it sounds like one of the big ones would be to use `strings.HasPrefix`, rather than using `strings.Repeat` to check if a string is a divisor. This not only is probably a lot more efficient, but also makes the code a lot cleaner which I like. There are a couple other smaller optimizations it suggests, like not creating as many extra variables, or adding an extra couple base cases.

## Final Solution, Conclusion

Here's the finalized, optimized solution:

```go
func gcdOfStrings(str1 string, str2 string) string {
    if len(str1) == 0 || len(str2) == 0 {
        return ""
    }
    if str1 == str2 {
        return str1
    }

    if strings.HasPrefix(str1, str2) {
        return gcdOfStrings(str1[len(str2):], str2)
    }
    if strings.HasPrefix(str2, str1) {
        return gcdOfStrings(str1, str2[len(str1):])
    }

    return ""
}
```

Runtime: 0 ms (beats 100%!)

Memory: 2.54 MB (beats 66%)

So it looks like Chat-GPT's optimizations indeed paid off! The solution it gives is a lot shorter than the one I dreamed up, since it uses recursion. But I do think shorter and simpler like this is inherently better, whether there is a performance improvement with it or not.

I think this was a problem that appeared easy, and overall wasn't too difficult to get a working solution, but in hindsight my first solution looks pretty bulky and had a lot of room to improve.

My main takeaway: **don't forget to look into built-in `strings` functions.** I checked for divisors in kind of a round-about way initially.

Ben's difficulty rating: 5.9/10. I think this one probably gets a little more into the "medium" level difficulty rather than "easy", but still manageable.
