# Reverse Vowels of a String

[leetcode link](https://leetcode.com/problems/reverse-vowels-of-a-string)

4/24/2024

Difficulty: Easy

## Description

Given a string `s`, reverse only all the vowels in the string and return it.

The vowels are `a`, `e`, `i`, `o`, and `u`, and they can appear in both lower and upper cases, more than once.

**Example 1:**

```
Input: s = "hello"
Output: "holle"
```

**Example 2:**

```
Input: s = "leetcode"
Output: "leotcede"
```

**Constraints:**

-   `1 <= s.length <= 3 * 10^5`
-   `s` consists of printable ASCII characters.

## Initial Thoughts / Planning

This is an interesting one, and I have a feeling there are a few different ways this can be solved.

My first hunch is to try something like a 2-pointer solution, where I approach the string from both sides, looking for vowels. as each one is found, it's replaced with one corresponding to the other end of the string. Although, perhaps that's a little bit overly complex; I would have to add some logic to have one side wait until the other side also has a vowel, and that could get a little messy maybe.

Another idea I have is that we could first iterate over the string, and record all the vowels that appear in it, in order from left to right, in an array. Then, we could iterate over it again, and everytime we encounter a vowel, we replace it with a vowel at the **end** of the vowel list (and pop it from the vowel list). This idea seems more straight-forward, but it would require an extra iteration over the string. But, it would still be O(n) I believe.

Between these two, I think I like the second idea best. It'll probably look a little cleaner, and seems a little less likely to be error-prone. Let's give it a shot.

## Code Implementations, Optimizations

This is my first viable solution, which passes 479/480 solutions:

```go
func reverseVowels(s string) string {
    // first, find all the vowels (in order) that appear in s
    vowels := []string{}
    for _, c := range s {
        str := string(c)
        if isVowel(str) {
            vowels = append(vowels, str)
        }
    }
    // now, reverse the vowels
    out := ""
    for i := 0; i < len(s); i++ {
        if isVowel(string(s[i])) {
            c := vowels[len(vowels) - 1]
            out += c
            vowels = vowels[:len(vowels) - 1]
        } else {
            out += string(s[i])
        }
    }
    return out
}

func isVowel(c string) bool {
    c = strings.ToLower(c)
    return c == "a" || c == "e" || c == "i" || c == "o" || c == "u"
}
```

... but, it fails the last one as it exceeds the time limit. So that means we have some efficiency issues to work out. I was actually thinking as I wrote this though that there should be room for improvement, so no surprise there.

I think the first thing that struck me as bad was how I'm getting the vowels back to re-add them in in reverse order: the `vowels` array is continuously having its last element removed in kind of a clunky manner. One idea to possibly improve this would be to, instead of removing the last vowel each time, just record an index or count to know which vowel in `vowels` to use each time we are making a swap.

> with this change, we now pass all cases, but still some cases took too long. so maybe a slight improvement?

My next suspicion is how we are building up the `out` string. We are messing with an immutable string, so we are constantly making inefficient concatenations. Maybe I can using `strings.Builder` to make this more efficient?

> with this change, we finally pass all the cases without taking too much time!

However, the solution still isn't great. Here's the code after the two preceding changes:

```go
import (
    "strings"
)

func reverseVowels(s string) string {
    // first, find all the vowels (in order) that appear in s
    vowels := []string{}
    for _, c := range s {
        str := string(c)
        if isVowel(str) {
            vowels = append(vowels, str)
        }
    }
    // now, reverse the vowels
    var out strings.Builder
    swapCount := 0
    for i := 0; i < len(s); i++ {
        if isVowel(string(s[i])) {
            out.WriteString(vowels[len(vowels) - swapCount - 1])
            swapCount++
        } else {
            out.WriteByte(s[i])
        }
    }
    return out.String()
}

func isVowel(c string) bool {
    c = strings.ToLower(c)
    return c == "a" || c == "e" || c == "i" || c == "o" || c == "u"
}
```

Runtime: 8ms (beats 33%)

Memory: 6.36 MB (beats 23%)

Looking it over once more, I don't really see any glaring issues, at least if I'm to keep the current approach of two loops, recording the vowels first and then re-inserting them after.

One idea we could return to would be my first idea - doing some kind of two pointer solution that finds vowels on both sides of the strings, and swaps them as it goes. that would use less memory since we wouldn't need to keep the `vowels` array around, and might be a little more efficient too since it eliminates one of the for loops.

## Let's ask Chat-GPT

Here's what it has to say:

> Optimization: Your solution is efficient and straightforward, but there's a slight optimization that can be made. Currently, you're iterating over the entire string twice: once to find the vowels and once to reconstruct the string with reversed vowels. You can achieve the same result with a single pass through the string.
>
> String to Rune Conversion: When iterating over the string using range, each iteration yields a rune, not a string. Instead of converting the rune to a string using string(c), you can directly compare the rune to the vowels. This approach avoids unnecessary string allocations and conversions.
>
> Avoiding Case Conversion: Instead of converting each character to lowercase to check if it's a vowel, you can directly compare it to both lowercase and uppercase vowels. This simplifies the isVowel function.

These all look like valid optimizations to me. I admittedly don't know much about `rune` datatypes yet, so I didn't realize that instead of converting to strings, I can just compare the characters as runes. I guess a `rune` is go's version of a `char`?

It also looks like Chat-GPT prefers the single loop approach, which makes sense. In its optimized version, it ends up doing a similar implementation to what I described in my first idea in the planning section, but probably little more gracefully than I was envisioning.

## Final Solution, Conclusion

Here's the final, optimized solution:

```go
import (
    "strings"
)

func reverseVowels(s string) string {
    var out strings.Builder
    end := len(s) - 1
    for i := 0; i < len(s); i++ {
        if isVowel(s[i]) {
            for !isVowel(s[end]) {
                end--
            }
            out.WriteByte(s[end])
            end--
        } else {
            out.WriteByte(s[i])
        }
    }
    return out.String()
}

func isVowel(c byte) bool {
    return c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u' || c == 'A' || c == 'E' || c == 'I' || c == 'O' || c == 'U'
}
```

Runtime: 3 ms (beats 80%)

Memory: 4.55 MB (beats 60%)

The main changes that occurred is we simplified the rune comparisons, by not converting to strings and allocating extra variables there. We also switched to a single loop, although the single loop does have a nested loop for finding the next vowel on the end of the string.

I think there might even be room for more optimization the memory, as we could perhaps replace the vowels in string `s` in-line, rather than creating a new buffer to write to. But that also might sacrifice some efficiency, since I think writing to buffers is pretty efficient.

In general, I'm happy with this solution.

Main takeaway: **Don't convert runes to strings unless necessary**.

Ben's difficulty rating: 5.5/10. It took a couple iterations to get the optimal solution, but conceptually it wasn't too hard to figure out.
