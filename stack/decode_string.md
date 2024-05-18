# Decode String

[leetcode link](https://leetcode.com/problems/decode-string)

5/18/2024

Difficulty: Medium

## Description

Given an encoded string, return its decoded string.

The encoding rule is: `k[encoded_string]`, where the `encoded_string` inside the square brackets is being repeated exactly `k` times. Note that `k` is guaranteed to be a positive integer.

You may assume that the input string is always valid; there are no extra white spaces, square brackets are well-formed, etc. Furthermore, you may assume that the original data does not contain any digits and that digits are only for those repeat numbers, `k`. For example, there will not be input like `3a` or `2[4]`.

The test cases are generated so that the length of the output will never exceed 10<sup>5</sup>.

**Example 1:**

```
Input: s = "3[a]2[bc]"
Output: "aaabcbc"
```

**Example 2:**

```
Input: s = "3[a2[c]]"
Output: "accaccacc"
```

**Example 3:**

```
Input: s = "2[abc]3[cd]ef"
Output: "abcabccdcdcdef"
```

**Constraints:**

-   `1 <= s.length <= 30`
-   `s` consists of lowercase English letters, digits, and square brackets `'[]'`.
-   `s` is guaranteed to be a valid input.
-   All the integers in `s` are in the range `[1, 300]`.

## Initial Thoughts / Planning

At first this problem seemed trivial: when you encounter a number, repeat whatever characters are found in the proceeding brackets that number of times. But, the trick here is that there can be **nested brackets**. See Example 2 - before doing the first repetition, you need to first evaluate the resulting string from the nested repitition.

So, I think it's fair to say we probably use a stack to solve this problem. Not only are we still in the "stack" section of the problem list, but I've noticed by now that stacks are useful for these kinds of problems, where you need to have a "memory" of the order in which you want to process things.

So maybe the first question is, what's the logic here for stacking these operations? How do we know when to stack a new operation, vs when to pop them?

This problem is somewhat similar to another problem I've done before, where you need to evaluate if all the brackets in a given string have their corresponding closing bracket. In that, we would add **opening brackets** to the stack, and as we encounter **closing brackets**, we pop them back off the stack. If you ever encountered closing brackets but there were no more open brackets on the stack, or if you reached the end of the string and there were still open brackets on the stack, it was an invalid/not-properly-closed string of brackets.

So similarly, I think we can handle stacking these operations by looking for when we encounter opening brackets. For each open bracket we meet in succession, we just keep stacking those "expansion" operations on the stack, and once we start meeting the closing brackets, we start putting the resultant string together, gradually expanding it.

But I'm still a little confused about how this is going to work exactly. Let's take Example 2, which has some nested brackets, and walk through it.

`"3[a2[c]]"`

So this will be:

`3` x `[a2[c]]`

which breaks down further as:

`3` x (`a` + (`2` x `c`))

We would start solving this by solving the inner-most parentheses. So this would become:

`3` x (`a` + `cc`)

`3` x `acc`

`accaccacc`

What I think is, we can have a stack of `numbers` where, everytime we encounter a number, we put it on the stack. Additionally, we can have a `strings` stack where, everytime we encounter a number, we put the current in-progress string onto the stack to save it for later.

The idea here is, as we are encountering characters, we can just start building the output string. But once we hit a number, we know that we are going to need to do a repeat operation on a different string first, which will later be added onto whatever our current progress string is.

So let's walk through the example again, and describe how this algorithm is going to work:

`"3[a2[c]]"`

-   we first read `3`, and we've hit a `'['` opening bracket, so we know the number is done. We put the number on the `numbers` stack

```
numbers = [3];
strings = [];
```

-   We read `'a'`, but then we read `'2'` after it, so we take our "in progress string" `a` and put it in `strings`

```
numbers = [3]
strings = ['a']
```

-   We encounter another opening bracket, so we put `2` in the numbers stack

```
numbers = [3, 2]
strings = ['a']
```

-   We read `'c'`, and then a closing bracket `']'` - this signals that we've hit the end of an expression, and need to do a repetition now for whatever the current string is. For this purpose, we `pop` off the last number in `numbers`, which is `2`. We repeat our current string `'c'` `2` times, resulting in `"cc"`. We also need to `pop` from `strings` and add this current string to whatever string we get from it. This is because every time an expression is done, it's expected to be added onto whatever came before it. So, this makes our current string now `"acc"`

```
`numbers = [3]`
`strings = []`
```

-   Now, we encounter another closing bracket (the last one), which prompts another `pop` from `numbers` - `3` - which is then used to multiply our current string `"acc"` into `"accaccacc"`. We also attempt to `pop` from strings, but we find nothing there, which is okay.

... and just like that, we've got our correct answer!

I think the logic can be summarized based on what characters we hit:

-   When you hit a `number`, you add whatever the current in-progress string is to the `strings` stack. You once again begin recording the characters as a string, but from scratch, since we are now getting a number
-   When you hit a `'['`, you add the current in-progress string to `numbers` - the in-progress string at this point should be a number, so you'd parse this to an integer.
-   When you hit a `']'`, you have hit the end of an expression: you `pop` from both `numbers` and `strings`, where you multiple the in-progress string by the `number` and then add it on the end of the `string`.

## My Code Implementations, Optimizations

One important thing I didn't realize is, there can be nested expressions that start with numbers too. I was assuming that they would always start with a letter, but for example, the following is a valid input:

`"3[z]2[2[y]pq4[2[jk]e1[f]]]ef"` (note things like `2[2[y]...]`)

So, that means even if there is no string yet, you should make sure to add an empty string to the `strs` stack so that when the expression is closed, it can be pulled from the stack without disrupting the correct order.

also, btw, I renamed the strings stack to `strs` since I realized I'd need to make use of the `strings` package. I also renamed `numbers` to `nums`, to follow suit.

```go
import (
    "unicode"
    "strings"
    "strconv"
)

func decodeString(s string) string {
    nums := make([]int, 0)
    strs := make([]string, 0)

    var currentStr strings.Builder
    trackNum := false
    for i := 0; i < len(s); i++ {
        c := s[i]
        // number - start building a number string
        if !trackNum && unicode.IsDigit(rune(c)) {
            strs = append(strs, currentStr.String())
            currentStr = strings.Builder{}
            currentStr.WriteByte(c)
            trackNum = true
            continue
        }
        // open bracket - add the number we've found to nums
        if c == '[' {
            num, _ := strconv.Atoi(currentStr.String())
            nums = append(nums, num)
            currentStr = strings.Builder{}
            trackNum = false
            continue
        }
        // close bracket - multiple the current string by last number, and add to last string
        if c == ']' {
            num := nums[len(nums) - 1]
            nums = nums[:len(nums) - 1]
            repeat := strings.Repeat(currentStr.String(), num)
            currentStr = strings.Builder{}
            if len(strs) > 0 {
                currentStr.WriteString(strs[len(strs) - 1])
                strs = strs[:len(strs) - 1]
            }
            currentStr.WriteString(repeat)
            continue
        }
        currentStr.WriteByte(c)
    }
    return currentStr.String()
}
```

Time complexity: `O(n)`, where `n` is the length of the string `s`.

Space complexity: `O(n)`. We have two stacks, but the memory usage of both will scale linearly with the size of `s`.

One early optimization I went ahead and used in this first implementation is `strings.Builder`, since I know from previous problems how inefficient string concatenations are.

Leetcode says that the performance of this solution is pretty good - supposedly taking 0 ms (beating 100%) and using 2.21 MB of memory (beating 84%). So I'm honestly feeling pretty satisfied with this solution, and I'm also not really able to think of any good optimizations here on my own.

## Let's ask Chat-GPT

Chat-GPT had a few suggestions to improve the solution, namely:

-   simplifying the logic a bit, so we don't need `trackNum` to know when we are building a number or string.
-   not reinitializing the `string.Builder` everytime we want to clear it; instead, we can use the function `currentStr.Reset()`, which can help re-use the previously allocated memory.

Let's see if/how these changes affect the runtime and memory usage of the solution.

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	 1890592	       619.7 ns/op	     504 B/op	      29 allocs/op  # my original solution
BenchmarkSolutionB-8   	 2214171	       541.4 ns/op	     456 B/op	      23 allocs/op  # Chat-GPT optimized
PASS
ok  	perf_testing	3.786s
```

So it looks like Chat-GPT's modifications do help a bit. But which changes are actually having the biggest impact here? Let's split the optimized solution into two pieces: one with the changed logic structure, but still reinitializing the string builder the old way, and one with the changed logic but using the `currentStr.Reset()` function.

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	 1914385	       617.7 ns/op	     504 B/op	      29 allocs/op  # my original solution
BenchmarkSolutionB-8   	 2212957	       540.0 ns/op	     456 B/op	      23 allocs/op  # updated logic, without the Reset() function
BenchmarkSolutionC-8   	 2221717	       540.4 ns/op	     456 B/op	      23 allocs/op  # updated logic, with the Reset() function
PASS
ok  	perf_testing	5.424s
```

So interestingly enough, it doesn't look like `currentStr.Reset()` makes the difference at all here - it's the updated logic! Let's take a look at what's different:

```go
func decodeString(s string) string {
    nums := []int{}
    strs := []string{}
    var currentStr strings.Builder

    for i := 0; i < len(s); i++ {
        c := s[i]
        if unicode.IsDigit(rune(c)) {
            numStart := i
            // 1) instead of a trackNum variable in our logic, we just find the entire number here
            for i+1 < len(s) && unicode.IsDigit(rune(s[i+1])) {
                i++
            }
            num, _ := strconv.Atoi(s[numStart:i+1])
            nums = append(nums, num)
            strs = append(strs, currentStr.String())
            currentStr.Reset()
        } else if c == '[' {
            // 2) due to (1), we no longer need to do anything here, just skip
            continue
        } else if c == ']' {
            num := nums[len(nums)-1]
            nums = nums[:len(nums)-1]
            repeat := strings.Repeat(currentStr.String(), num)
            currentStr.Reset()
            if len(strs) > 0 {
                currentStr.WriteString(strs[len(strs)-1])
                strs = strs[:len(strs)-1]
            }
            currentStr.WriteString(repeat)
        } else {
            currentStr.WriteByte(c)
        }
    }
    return currentStr.String()
}
```

See the comments in the code above that point out the logical changes - but also, it uses `else` statements instead of `continue` statements. It's not exactly clear to me why that might impact performance, but perhaps it's more efficient for some reason. Let's put that to the test in a new benchmark!

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	 1936065	       621.8 ns/op	     504 B/op	      29 allocs/op  # original
BenchmarkSolutionB-8   	 2220750	       540.8 ns/op	     456 B/op	      23 allocs/op  # uses continue
BenchmarkSolutionC-8   	 2224498	       539.5 ns/op	     456 B/op	      23 allocs/op  # uses else
PASS
ok  	perf_testing	5.435s
```

I'm not seeing a clear winner here - it kind of goes back and forth between one approach being slightly faster than the other. Maybe it's not really a big factor either.

This leads me to believe that the real improvement in performance (which mind you is only about `100 ns/op` faster and `50 B/op` less memory in our benchmarks) is the loop that finds the entire number.

```go
// ...
if unicode.IsDigit(rune(c)) {
    numStart := i
    for i+1 < len(s) && unicode.IsDigit(rune(s[i+1])) {
        i++
    }
    num, _ := strconv.Atoi(s[numStart:i+1])
    nums = append(nums, num)
    strs = append(strs, currentStr.String())
    currentStr.Reset()
}
// ...
```

It's still not very clear to me why this would make much of a difference. I guess it's just a little bit more efficient to find the number's endpoints in the string, and handle it wholesale rather than character by character maybe. Interesting; I think it does improve the solution overall, making it a little more readable too, but it probably still counts as a "micro-optimization".

## Final Solution, Conclusion

Here's our solution, touched up a bit by Chat-GPT:

```go
import (
    "strconv"
    "strings"
    "unicode"
)

func decodeString(s string) string {
    nums := []int{}
    strs := []string{}
    var currentStr strings.Builder

    for i := 0; i < len(s); i++ {
        c := s[i]
        if unicode.IsDigit(rune(c)) {
            numStart := i
            for i+1 < len(s) && unicode.IsDigit(rune(s[i+1])) {
                i++
            }
            num, _ := strconv.Atoi(s[numStart:i+1])
            nums = append(nums, num)
            strs = append(strs, currentStr.String())
            currentStr.Reset()
        } else if c == '[' {
            continue
        } else if c == ']' {
            num := nums[len(nums)-1]
            nums = nums[:len(nums)-1]
            repeat := strings.Repeat(currentStr.String(), num)
            currentStr.Reset()
            if len(strs) > 0 {
                currentStr.WriteString(strs[len(strs)-1])
                strs = strs[:len(strs)-1]
            }
            currentStr.WriteString(repeat)
        } else {
            currentStr.WriteByte(c)
        }
    }
    return currentStr.String()
}
```

I really got into the weeds of it here with the benchmarking, but mostly just for fun - I don't think we made any major optimizations beyond the original solution, just some logical improvements and a little code cleanup. I think this was just kind of fun to really test the minor differences in how code style can affect efficiency.

Ben's difficulty rating: 7.4/10. It actually took me a bit of thinking to figure out the solution to this, but I think walking through it step by step helped me figure it out.
