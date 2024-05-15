# Removing Stars from a String

[leetcode link](https://leetcode.com/problems/removing-stars-from-a-string/)

5/15/2024

Difficulty: Medium

## Description

You are given a string `s`, which contains stars `*`.

In one operation, you can:

-   Choose a star in `s`
-   Remove the closest **non-star** character to its **left**, as well as remove the star itself.

Return the string after all stars have been removed.

**Note:**

-   The input will be generated such that the operation is always possible.
-   It can be shown that the resulting string will always be unique.

**Example 1:**

```
Input: s = "leet**cod*e"
Output: "lecoe"
Explanation: Performing the removals from left to right:
- The closest character to the 1st star is 't' in "leet**cod*e". s becomes "lee*cod*e".
- The closest character to the 2nd star is 'e' in "lee*cod*e". s becomes "lecod*e".
- The closest character to the 3rd star is 'd' in "lecod*e". s becomes "lecoe".
There are no more stars, so we return "lecoe".
```

**Example 2:**

```
Input: s = "erase*****"
Output: ""
Explanation: The entire string is removed, so we return an empty string.
```

**Constraints:**

-   `1 <= s.length <= 10^5`
-   `s` consists of lowercase English letters and stars `*`.
-   The operation above can be performed on `s`.

## Initial Thoughts / Planning

As I was reading the problem description, the first thought that popped into my head was that you essentially just find bodies of stars, count how many stars are in a given body, and then delete the same number of characters to its left. This seems _simple enough_, at least conceptually. It might get a little funky bouncing around the indices and all that, but maybe you could accomplish it with some kind of two pointer solution.

But then I remembered - this is the **stack** section. So what if we used a stack here? It kind of feels like cheating, since in a real interview I wouldn't have the benefit of being told what data structure I'm supposed to use. Oh well lol.

So, if we just begin by imagining putting each character onto a stack as we progress, I think the solution becomes clear right away:

-   loop through the string, putting each character on the stack
-   if we encounter a `*` character, pop the last element off the stack, and **skip** the `*` character (don't add it to the stack)
-   repeat this process to the end of the string. We end up with a slice of characters that represents the output string.

I'm going to go ahead and get started building this first, without going into any pre-mature optimization ideas.

## My Code Implementations, Optimizations

Ok! So after throwing together a solution in a few minutes, this is what I've got:

```go
func removeStars(s string) string {
    stack := []byte{} // our stack is just a slice of bytes
    for i := 0; i < len(s); i++ {
        if s[i] == '*' {
            // pop from stack
            stack = stack[:len(stack) - 1]
        } else {
            // add to stack
            stack = append(stack, s[i])
        }
    }
    output := ""
    for _, c := range stack {
        output += string(c)
    }
    return output
}
```

Time complexity: `O(n)`

Space complexity: `O(n)`

So, right away and as I was making this, I noticed a few problems with my solution's efficiency. The leetcode numbers back it up, since this solution apparently is less efficient in terms of runtime and memory usage than 95% of other solutions!

Firstly, the dreaded _`append`_. We already know from prior problems and as a general rule of thumb in programming, that appending to arrays is inefficient. Arrays are dynamically resizeable, which is nice, but resizing an array doesn't come without it's costs, and can end up being quite costly when you're doing this a lot in really "high traffic" code over a lot of iterations. Anytime it's possible, it's better to pre-allocate the array's size so we can just insert values into it directly and avoid resizing.

Second, the way we build the output string at the end is quite inefficient as well. Strings are also immutable (perhaps since under the hood in go, they are just `byte` slices I believe?), so each time we are adding a new character to the string, it's re-creating a new string which is inefficient. If we want to build a string, we already know to use `string.Builder` instead.

> The first thing I realized though is: what's the proper way to make a stack in Go anyway? I just used a `slice` since in other languages like `javascript`, they tend to use arrays as stacks.
>
> After looking around, I do believe the usual, idiomatic way to make a stack is to use a slice, since it's a dynamic sized array. Huh, so I guess stacks are just not ultra-efficient, at least in cases like this where we often seek to micro-optimize everything lol.

Ok, well the first thing I'm going to do is optimize the string building, using `string.Builder` instead of the current `+=` operations. Let's see how much of an improvement that gets us.

...

```go
// code for building the stack above...

var sb strings.Builder
for _, c := range stack {
    sb.WriteString(string(c))
}
return sb.String()
```

> _Wow_ - so just that one optimization brought our runtime speed down from a staggering 1300 ms to just **15 ms**. Even though the time complexity here _should be_ the same (I should confirm this actually), this optimization has a dramatic effect. Let's not forget to mention that this also brought down the memory usage from a massive 150 MB to just 8 MB.

Well honestly, according to how this solution compares to others, this feels like a pretty much finished, optimized implementation. But since that didn't take too long, I'm going to go ahead and explore other ideas, like somehow not using a slice, or avoiding appending the slice. For example, what if we pre-allocated the size of the slice to be the same length as the input array? and then used a pointer to track our position, rather than repeatedly popping and appending?

```go
import (
    "strings"
)

func removeStars(s string) string {
    stack := make([]byte, len(s))
    stackPos := 0
    for i := 0; i < len(s); i++ {
        if s[i] == '*' {
            // pop from stack
            stackPos--
            stack[stackPos] = 0
        } else {
            // add to stack
            stack[stackPos] = s[i]
            stackPos++
        }
    }
    var sb strings.Builder
    for i := 0; i < stackPos; i++ {
        sb.WriteByte(stack[i])
    }
    return sb.String()
}
```

This is my solution after pre-allocating the size of `stack`, and using a pointer to keep track of where the stack is at. To remove values from the stack, we just reset it to the zero-value for `byte`, which is just a `0`.

So, the surprising thing here is, contrary to my expectations, this solution doesn't seem to be any faster, and actually could be slightly... slower?
I'm not doing any formal benchmarking here, just basing this off what leetcode tells me, but now my submissions are taking around 30 to 40 ms, when before they were taking just around 20 ms on average. So what gives? Could it be that these so-called "dreaded" append operations aren't actually as inefficient as I thought they were? I'm almost 100% certain that we've done this type of optimization in the past and have seen great results.

Just as a sanity check, I ran my previous solution (which still uses `strings.Builder`) and now I'm getting some inconsistent reads from leetcode. Maybe I'm once again giving leetcode's metrics a little too much credit...

Doing a few more runs of both of these solutions, I actually do think this "pre-allocation" optimization may improve things a little, especially for run time, but not in a major way. So maybe my instinct was still correct - but it's just not as much of a major game-changer as I thought it was.

> In the future, maybe I ought to just start benchmarking these solutions on my own local machine instead of giving too much credence to leetcode's performance measurements. They seem to vary too much to give high level accuracy on how efficient a given solution is compared to other ones. After all, Go has it's own built-in support for benchmarking, so it wouldn't be too much more effort.

## Final Solution, Conclusion

I'm going to give two solutions here. One that uses a stack in it's usual manner, which consequently is a little more readable, and one which is possibly overly optimized just to win a few extra ms of runtime speed, which has a slightly more complex implementation of a stack.

**Using a regular slice as a stack:**

```go
func removeStars(s string) string {
    stack := []byte{}
    for i := 0; i < len(s); i++ {
        if s[i] == '*' {
            // pop from stack
            stack = stack[:len(stack) - 1]
        } else {
            // add to stack
            stack = append(stack, s[i])
        }
    }
    var sb strings.Builder
    for _, c := range stack {
        sb.WriteByte(c)
    }
    return sb.String()
}
```

**Using Ben's slightly overkill stack:**

```go
import (
    "strings"
)

func removeStars(s string) string {
    stack := make([]byte, len(s))
    stackPos := 0
    for i := 0; i < len(s); i++ {
        if s[i] == '*' {
            // pop from stack
            stackPos--
            stack[stackPos] = 0
        } else {
            // add to stack
            stack[stackPos] = s[i]
            stackPos++
        }
    }
    var sb strings.Builder
    for i := 0; i < stackPos; i++ {
        sb.WriteByte(stack[i])
    }
    return sb.String()
}
```

(One additional change I made is I realized I can use `WriteByte` instead of `WriteString`, avoiding the need to convert the bytes with `string()` first.)

Time complexity: `O(n)`

Space complexity: `O(n)`

I think the real big improvement was just using string builder. The jury is still out on whether or not the modification to the stack is worth it. I think logically it makes sense that avoiding appends and resizing the array should help, but since I haven't done actual benchmarking, it's hard to tell.

Main takeaway: string builders, as we already knew, are way more efficient than just using addition operations on strings. But this time it really drove that home. An interesting takeaway for me is that maybe array appends aren't _quite_ as big of a deal as I thought. Maybe. I think I'm still going to continue to look at them with skepticism though, just because. I think I should really consider investing the extra effort into doing some formalized benchmarking though to make a solid conclusion here.

Ben's difficulty rating: 3.2/10. I actually thought this one was pretty easy, but it might just be my prior experience with using stacks that made this one more intuitive than other problems.
