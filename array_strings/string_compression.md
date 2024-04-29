# String Compression

[leetcode link](https://leetcode.com/problems/string-compression/)

4/26/2024

Difficulty: Medium

## Description

Given an array of characters `chars`, compress it using the following algorithm:

Begin with an empty string `s`. For each group of consecutive repeating characters in `chars`:

-   If the group's length is 1, append the character to `s`.
-   Otherwise, append the character followed by the group's length.

The compressed string `s` **should not be returned separately, but instead, be stored in the input character array** `chars`. Note that group lengths that are 10 or longer will be split into multiple characters in `chars`.

After you are done modifying the input array, return the new length of the array.

You must write an algorithm that uses only constant extra space.

**Example 1:**

```
Input: chars = ["a","a","b","b","c","c","c"]
Output: Return 6, and the first 6 characters of the input array should be: ["a","2","b","2","c","3"]
Explanation: The groups are "aa", "bb", and "ccc". This compresses to "a2b2c3".
```

**Example 2:**

```
Input: chars = ["a"]
Output: Return 1, and the first character of the input array should be: ["a"]
Explanation: The only group is "a", which remains uncompressed since it's a single character.
```

**Example 3:**

```
Input: chars = ["a","b","b","b","b","b","b","b","b","b","b","b","b"]
Output: Return 4, and the first 4 characters of the input array should be: ["a","b","1","2"].
Explanation: The groups are "a" and "bbbbbbbbbbbb". This compresses to "ab12".
```

**Constraints:**

-   `1 <= chars.length <= 2000`
-   `chars[i]` is a lowercase English letter, uppercase English letter, digit, or symbol.

## Initial Thoughts / Planning

Ok, so the actual compression logic doesn't seem like it's too complicated. We are compressing an array of characters by listing the number of characters in a row, for each character we encounter.

The catch is that we can only use constant extra space, and the output should be returned in the input array directly.

The part I'm not really sure about for now is, how to convert an integer to its individual chars. For example, I'd need to be able to turn `1` into `'1'`, or `12` into `'1', '2'`. After a quick google search, it looks like you can use `strconv.Itoa` to convert an integer to a string, so I guess we could then get the chars/bytes from there.

So here's what I'm thinking:

-   iterate over the array of chars
-   count the number of times a char appears in a row
-   when a new char is encountered, put the compressed char info into the array and continue on.

Seems pretty straightforward.

## Code Implementations, Optimizations

Here's my first solution:

```go
import (
    "strconv"
)

func compress(chars []byte) int {
    insertIndex := 0
    curChar := chars[0]
    curLength := 1

    for i := 1; i < len(chars); i++ {
        if chars[i] == curChar {
            curLength++
            continue
        }
        // insert compressed char info into chars
        for _, c := range getGroupBytes(curChar, curLength) {
            chars[insertIndex] = c
            insertIndex++
        }
        curChar = chars[i]
        curLength = 1
    }
    for _, c := range getGroupBytes(curChar, curLength) {
        chars[insertIndex] = c
        insertIndex++
    }
    return insertIndex
}

func getGroupBytes(char byte, length int) []byte {
    out := []byte{}
    out = append(out, char)
    if length == 1 {
        return out
    }
    for _, c := range strconv.Itoa(length) {
        out = append(out, byte(c))
    }
    return out
}
```

Runtime: 5 ms (beats 71%?)

Memory: 3.15 MB (beats 13%)

I'm not really sure about runtime - it says that 62% of solutions also took 5 ms, so not sure why I'm considered "beating" them. Oh well, I'll take it!

In terms of memory, I'm not really sure why my solution uses more memory than most other solutions. I wonder if having another function contributes to it?

> After inlining my function calls and removing the function definition for getGroupBytes, the memory went down a bit, boosting me up to beating 58% of solutions in terms of memory usage. I guess defining a function does take some memory!

Beyond this though, I'm not seeing many opportunities for further optimization. I looked around at other solutions on leetcode, but they all seem to be more or less the same concept, with just slight variations in how the loop works.

## Let's ask Chat-GPT

Chat-GPT also doesn't seem to have any optimization ideas beyond what we've done here.

## Final Solution, Conclusion

Here's the final solution, which I've kept the same as my initial solution except I removed the helper function:

```go
import (
    "strconv"
)

func compress(chars []byte) int {
    insertIndex := 0
    curChar := chars[0]
    curLength := 0

    for i := 0; i < len(chars); i++ {
        if chars[i] == curChar {
            curLength++
            continue
        }
        // insert group into chars
        chars[insertIndex] = curChar
        insertIndex++
        if curLength > 1 {
            for _, c := range strconv.Itoa(curLength) {
                chars[insertIndex] = byte(c)
                insertIndex++
            }
        }
        // set next group char
        curChar = chars[i]
        curLength = 1
    }
    // add whatever the last group was
    chars[insertIndex] = curChar
    insertIndex++
    if curLength > 1 {
        for _, c := range strconv.Itoa(curLength) {
            chars[insertIndex] = byte(c)
            insertIndex++
        }
    }
    return insertIndex
}
```

Main takeaway: defining helper functions may increase the memory overhead for a solution. I think they are still good in terms of improving readability and reusability of code, but if my goal is to minimize memory usage altogether (not just in terms of space complexity), avoiding defining helper functions will help.

Ben's difficulty rating: 5.8/10. The concept of this compression algorithm is pretty simple, and really the only slight tricky thing I had to look up was figuring out how to handle the length values when converting them to chars. but that ended up being pretty simple.
