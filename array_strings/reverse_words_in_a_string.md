# Reverse Words in a String

[leetcode link](https://leetcode.com/problems/reverse-words-in-a-string/)

4/24/2024

Difficulty: Medium

## Description

Given an input string `s`, reverse the order of the words.

A word is defined as a sequence of non-space characters. The words in `s` will be separated by at least one space.

Return _a string of the words in reverse order concatenated by a single space._

Note that `s` may contain leading or trailing spaces or multiple spaces between two words. The returned string should only have a single space separating the words. Do not include any extra spaces.

**Example 1:**

```
Input: s = "the sky is blue"
Output: "blue is sky the"
```

**Example 2:**

```
Input: s = "  hello world  "
Output: "world hello"
Explanation: Your reversed string should not contain leading or trailing spaces.
```

**Example 3:**

```
Input: s = "a good   example"
Output: "example good a"
Explanation: You need to reduce multiple spaces between two words to a single space in the reversed string.
```

**Constraints:**

-   `1 <= s.length <= 10^4`
-   `s` contains English letters (upper-case and lower-case), digits, and spaces `' '`.
-   There is at least one word in `s`.

**Follow-up**: If the string data type is mutable in your language, can you solve it in-place with O(1) extra space?

(Golang strings are immutable, so N/A for the follow up.)

## Initial Thoughts / Planning

I think Go has a built in function that will be quite useful for this: `strings.Fields()`. This function will split a string by spaces, handling for extra spaces and trailing whitespace.

So, using that function, I think it'll be as simple as getting all the `fields` of the string, and then building the string using the reverse order of the list of words produced by `strings.Fields(s)`.

Additionally, in order to make it more efficient, rather than concatenating the immutable strings using `+`, I can use `strings.Builder` to build a string using a bytes buffer.

## Code Implementations, Optimizations

Here's my code solution - this actually ended up being very quick, but I made a modification from the original plan:

```go
import (
    "strings"
    "slices"
)

func reverseWords(s string) string {
    words := strings.Fields(s)

    slices.Reverse(words)

    return strings.Join(words, " ")
}
```

Runtime: 2 ms (beats 74%)

Memory: 3.22 (beats 70%)

The change I made is, instead of using `strings.Builder`, I discovered that there is a built in `strings.Join` function which works well for our purpose of joining together all the strings. I also used `slices.Reverse` to reverse the elements.

The results seem to be pretty good for both runtime and memory. I think time complexity is O(n), since all of these operations probably just require iterating through the string or list of words. Space complexity is O(n), since I made a words array to hold all the words found in the string.

I'm really not sure where this can be optimized though. This solution is very straightforward and minimalistic, so if there is an optimization it probably means not using one or more of these built in functions and doing the work a different way manually. Perhaps it would be more efficient to use `strings.Builder` to form the string (rather than using `strings.Join`), and to just iterate through the words list backwards to eliminate the `slices.Reverse` call?

## Let's ask Chat-GPT

My conversation with Chat-GPT was somewhat fruitless. We went down a brief rabbit hole trying to make the solution more efficient by rewriting it to use one main loop that accomplishes everything, then that kind of broke down as we sought to fix the bugs in the code that it gave me. Eventually we wound up right back where we started with my initial solution there!

I believe there should be some sort of optimization, since apparently other go solutions have achieved better runtimes and memory usage than mine, but it's not clear what those optimizations are.

## Final Solution, Conclusion

The final solution remains as my first one:

```go
import (
    "strings"
    "slices"
)

func reverseWords(s string) string {
    words := strings.Fields(s)

    slices.Reverse(words)

    return strings.Join(words, " ")
}
```

I'm actually not sure if there is a good optimization for this. I've tried other peoples' solutions that are posted on leetcode, and all of them seem to have the same performance as this. Sometimes it would say the runtime was 0 ms (beating 100%), but rerunning it would show it's actually the same runtime as my own solution. Additionally, rerunning my own solution, sometimes it would say it took 0 ms too. So seems indecisive to me.

Anyway, I think my solution is the cleanest overall, with 3 lines of code, so I'll stick with it.

Main takeaway: **don't be afraid to use built in functions?**

Ben's difficulty rating: 3/10. Maybe Go is just well suited for this kind of problem, but this felt pretty quick and easy.
