# Equal Row and Column Pairs

[leetcode link](https://leetcode.com/problems/equal-row-and-column-pairs/)

5/14/2024

Difficulty: Medium

## Description

Given a 0-indexed `n x n` integer matrix `grid`, return the number of pairs `r_i, c_j` such that row `r_i` and column `c_j` are equal.

(Note on the above syntax: `r_i` means variable `r` with subscript `i`)

A row and column pair is considered equal if they contain the same elements in the same order (i.e. an equal array).

**Example 1:**

![example 1](https://assets.leetcode.com/uploads/2022/06/01/ex1.jpg)

```
Input: grid = [[3,2,1],[1,7,6],[2,7,7]]
Output: 1
Explanation: There is 1 equal row and column pair:
- (Row 2, Column 1): [2,7,7]
```

**Example 2:**

![example 2](https://assets.leetcode.com/uploads/2022/06/01/ex2.jpg)

```
Input: grid = [[3,1,2,2],[1,4,4,5],[2,4,2,2],[2,4,2,2]]
Output: 3
Explanation: There are 3 equal row and column pairs:
- (Row 0, Column 0): [3,1,2,2]
- (Row 2, Column 2): [2,4,2,2]
- (Row 3, Column 2): [2,4,2,2]
```

**Constraints:**

-   `n == grid.length == grid[i].length`
-   `1 <= n <= 200`
-   `1 <= grid[i][j] <= 10^5`

## Initial Thoughts / Planning

My first matrix problem so far! I have to say I'm a little unsure what to do here first. Let's start by re-iterating the problem to see if I have any ideas come up.

So, given a matrix, we need to count how many rows and columns match. That is, how many rows and columns have the same values in the same order.

I think a brute force method would be to do something like iterate over each row, and then check each column to see if it's a match. This would, as usual for brute force, be a lot slower than other solution options though.

So my immediate next thought is, is there a way we could store information on what row or columns exist, and check for their existence as needed, instead of needing to repeatedly iterate over and over to find matches? We are still in the hash map and set section, so maybe one solution is to use a map to accomplish something like this.

**Using trees?**

At first I was thinking a tree would work for this type of thing, and perhaps it would, but I'm a little unsure. I guess you would have to have multiple trees, not just one, and each would have a root node that would be the first number in the column or row. Then, if you were testing a specific column for example, you'd go down each number in the column, checking that there's a valid path in the tree corresponding to it. If not, then that column doesn't have a pair.

If we went that route, I guess we would need a map of integers (representing the root node values) to trees. If I'm not mistaken though, that sort of thing would actually be a space complexity of `O(n^2)`, since not only would we have a set of map keys corresponding to all columns' root values, but then each of those root values would have an array of possible paths to go (representing all the different columns that start with the root value). There may be a more efficient way to do this though, like making some kind of struct that would better represent this kind of tree. In fact, I've never done anything with trees in Go (or really at all since college) so maybe it would be smart to do some research first.

Or, we can just jump into it and try to figure it out first! Yes, I like the sound of that better.

Ok, so now I'm envisioning a `map[int]SomeStruct` where `SomeStruct` is some struct we make to represent possible paths in a tree.

```go
type NodeStruct struct {
    val int // the value of this node
    children map[int]NodeStruct // mapping the next value in the path to its tree branch
}
```

There, let's go with that.

Now, we will first build all the possible column paths using this style. We will keep a `map[int]NodeStruct` around, and go through all the columns, building this out.

Once we have the trees fully built, we will then go through each row, and traverse the trees until either it's determined to have a match, or there's a mismatch/invalid path along the way. We will count up all the matches, and return that number.

## My Code Implementations, Optimizations

Here's my first implementation:

```go
type NodeStruct struct {
    val int // integer at this node
    children map[int]*NodeStruct // possible paths forward from this node
    count int // number of paths that lead to this (terminal) node
}

func NewNode(val int) *NodeStruct {
    return &NodeStruct{
        val: val,
        children: make(map[int]*NodeStruct),
        count: 0,
    }
}

func equalPairs(grid [][]int) int {
    root := NewNode(0) // all column paths will technically start from here (0 is a dummy val)

    // traverse all the columns, building a tree of all possible column routes
    for c := 0; c < len(grid[0]); c++ {
        current := root
        for r := 0; r < len(grid); r++ {
            val := grid[r][c]
            if _, ok := current.children[val]; !ok {
                // this path doesn't exist yet; create a branch
                current.children[val] = NewNode(val)
            }
            current = current.children[val]
        }
        // there may be multiple identical columns, so count how many times this path is found
        current.count++
    }

    // traverse all the rows, checking if they are in one of the valid tree paths.
    pairs := 0
    for r := 0; r < len(grid); r++ {
        current := root
        match := true
        for c := 0; c < len(grid[0]); c++ {
            val := grid[r][c]
            if _, ok := current.children[val]; !ok {
                match = false
                break
            }
            current = current.children[val]
        }
        if match {
            // handle for multiple identical columns
            pairs += current.count
        }
    }
    return pairs
}
```

Time complexity: `O(n^2)` - since this is an `n x n` matrix, I believe technically, in order to traverse the entire matrix, we would still need to use `n x n` operations. So there's not much better we can do than this I'd assume.

Space complexity: `O(n^2)` - similar to the above, we are working with 2-dimensional data here. Unless we were able to only capture a single dimension of the data (e.g. a single row or column), then technically we will be using `O(n^2)` space too I believe. I think that's reasonable though: we aren't adding any extra dimensions, so should be good enough.

I have to admit, I had to look up how to make trees in Go, since I couldn't really figure it out. Turned out it was pretty simple: just make a `struct` that has a field for the `children` nodes. It could be a map or an array, but here I chose to use a map so I wouldn't need to iterate through an array of children elements to find the right one.

So, it turns out my idea about using trees works! One thing I was overlooking though is the possibility of multiple columns being the same, in which case we need to know _how many_ columns of a given path there are. This was pretty easy to fix in the end though: I just added an extra `count` field to `NodeStruct`, which I incremented in the "tree building" code everytime we hit a "terminal node".

Judging by the leetcode performance metrics, my solution's efficiency is overall good enough, but there should be some room for improvement in both runtime and memory usage.

**Memory usage**

This seems to be the area I could improve on the most, judging by leetcode's performance stats. However, when I look around, I'm not certain where the optimizations would be.
I know there are some ways to minimize the memory usage in sort of "hacky" ways, like not declaring as many helper variables, or not declaring helper functions, etc. But those methods are kind of dumb in my opinion and just make the solution harder to read. If I were writing this code for a real application, I definitely wouldn't be refusing to declare helper functions or helper variables for the sole purpose of micro-optimizing memory usage. Besides, that's the job of the Go compiler, right?

> Interestingly, inlining my helper function doesn't seem to change memory usage at all. So maybe my notion from before that helper functions increase memory usage is actually not true? Huh, oh well.

**Runtime speed**

So far in the hash map and sets section, the theme has been that switching to using arrays instead of maps improves runtime speed a lot. However, in this case I'm not sure that I can actually make that optimization. Since the constraints don't limit the numbers to a specific, small range, I don't think there's a way to switch to arrays. If the numbers were limited to, say, 1 to 5, then sure, we could just make an array of size 5 and put nodes in the index corresponding to the value of the node. But honestly even if the range was something like 1 to 1000, I'm not sure if the speed optimization of using arrays would be worth the memory overhead, since every single node in every single path would need to have an array of size 1000, most of which would probably be unused.

Otherwise, I'm not sure what would be slowing down my solution. All the operations happening are just messing with maps.

I considered early return options too, but I actually don't know if that's possible here. We need to traverse every row of the matrix to make sure all row/column pairs are considered. We already break early from rows when no valid path is found, too.

> At this point, I'm out of ideas for what optimizations I could make here. Let's see if I can find any optimization ideas with some outside help.

## Let's ask Chat-GPT

Talking to Chat-GPT got me some interesting results! It turns out my tree-based solution may not be the optimal one, as it can use a lot of memory. This checks out, as that's the main problem with my solution judging from leetcode's performance feedback.

### Trees using too much memory

Per Chat-GPT:

> The tree structure is memory-intensive, especially with the `children` map at each node. Since `n` can be up to 200 and the values can be as large as 10<sup>5</sup>, this could use a significant amount of memory.

So, given the constraints (or lack thereof), mapping all the nodes of all possible paths can take a lot of memory. This is especially true for solutions that have a lot of disjoint columns that don't share many values in common. We could end up with 200 sub-trees that all could approach up to 100,000.

Now, the solution Chat-GPT suggests is that instead of using a tree structure that contains a bunch of nodes, each with their own maps of children nodes, we can just store the column as its string representation in a map, and count the frequencies of each column string.

I think this will still result in the same number of disjoint mappings, but I assume the reason it improves memory is it has a lot less memory overhead involved for storing strings, rather than a bunch of recursively linked maps of nodes. Creating a map just to hold potentially one or two key/value pairs, and each one in turn has its own map, probably uses up a lot of extra memory as opposed to just storing those paths as individual strings. Let's see what Chat-GPt has to say about this too:

> 1. **String Representation:**
>
> Each row or column is converted to a string representation, which is then used as a key in the map.
> A string in Go is a slice of bytes with a pointer to the underlying array and length, which is generally more memory-efficient compared to a full-fledged struct with multiple fields.
>
> 2. **Hash Map Efficiency:**
>
> Hash maps (used for map[string]int) are optimized for efficient storage and lookup.
> Storing strings as keys avoids the overhead of managing multiple nested maps and pointers.
>
> 3. **Reduced Redundancy:**
>
> If two rows (or columns) are identical, they share the same string key, and we simply increment the count. This avoids redundant storage of identical structures.
> Since we only store the unique rows and columns once, the memory usage is directly proportional to the number of unique rows and columns, which can be lower than the total number of nodes in a tree structure.

In the end, it's clear enough that this string-based map approach is both simpler in terms of code complexity, as well as more efficient overall.

Let's give this a shot and see if we can see some improvements in the code's performance.

```go
import (
    "fmt"
)

func equalPairs(grid [][]int) int {
    rowMap := make(map[string]int)

    // map the frequency of every row, as its string representation
    for r := 0; r < len(grid); r++ {
        rowMap[fmt.Sprint(grid[r])]++
    }

    // traverse all the columns, checking if a corresponding row exists
    pairs := 0
    for c := 0; c < len(grid[0]); c++ {
        col := make([]int, len(grid))
        for r := 0; r < len(grid); r++ {
            col[r] = grid[r][c]
        }
        if freq, ok := rowMap[fmt.Sprint(col)]; ok {
            pairs += freq
        }
    }
    return pairs
}
```

Time complexity: `O(n^2)` - since we are traversing a matrix which is `n x n`, the fastest we can do is `n x n` time I figure.

Space complexity: `O(n^2)` - same as above; using space that scales up the same as the entire matrix is technically going to be `O(n^2)` I think, not "linear". But this is a matrix problem, so I think that's acceptable.

Yep - so it's immediately obvious that this solution looks _way_ cleaner. All that messy struct business has been done away with, and we can simply use `fmt.Sprint()` to convert an array to a string.

According to leetcode's metrics, my runtime speed is still pretty similar, but the memory usage is a lot lower which is nice.

One further idea I have is using `strings.Builder` to further optimize making the strings. I think it might be fine to keep the string formation the same for rows, since we already have the arrays readily available, but since we are building the column arrays, only to then convert them to strings, maybe it would be a little more efficient to just build the strings as we go.

```go
import (
    "strings"
    "strconv"
)

func equalPairs(grid [][]int) int {
    rowMap := make(map[string]int)
    n := len(grid)

    // map the frequency of every row, as its string representation
    for r := 0; r < n; r++ {
        var sb strings.Builder
        for c := 0; c < n; c++ {
            sb.WriteString(strconv.Itoa(grid[r][c]))
            if c < n-1 {
                sb.WriteString(",")
            }
        }
        rowMap[sb.String()]++
    }

    // traverse all the columns, checking if a corresponding row exists
    pairs := 0
    for c := 0; c < n; c++ {
        var sb strings.Builder
        for r := 0; r < n; r++ {
            sb.WriteString(strconv.Itoa(grid[r][c]))
            if r < n-1 {
                sb.WriteString(",")
            }
        }
        if count, exists := rowMap[sb.String()]; exists {
            pairs += count
        }
    }

    return pairs
}
```

It appears this does further improve the runtime speed! I ended up using string builder for the rows too, since I realized I'll need to make sure the format of the strings is the same in both places. I think when you use `fmt.Sprint` on a slice of ints, it comes out something like `[ 0 1 2 3 4 ...]`. So rather than making sure I built the strings in this same format, I just made it separate each integer with a simple comma.

> At this point, we're beating about 70% of solutions in terms of time and memory usage, so I think this is good enough.

## Final Solution, Conclusion

Here's my final solution, after making a couple optimizations as advised by Chat-GPT:

```go
import (
    "strings"
    "strconv"
)

func equalPairs(grid [][]int) int {
    rowMap := make(map[string]int)
    n := len(grid)

    // map the frequency of every row, as its string representation
    for r := 0; r < n; r++ {
        var sb strings.Builder
        for c := 0; c < n; c++ {
            sb.WriteString(strconv.Itoa(grid[r][c]))
            if c < n-1 {
                sb.WriteString(",")
            }
        }
        rowMap[sb.String()]++
    }

    // traverse all the columns, checking if a corresponding row exists
    pairs := 0
    for c := 0; c < n; c++ {
        var sb strings.Builder
        for r := 0; r < n; r++ {
            sb.WriteString(strconv.Itoa(grid[r][c]))
            if r < n-1 {
                sb.WriteString(",")
            }
        }
        if count, exists := rowMap[sb.String()]; exists {
            pairs += count
        }
    }

    return pairs
}
```

There's an even shorter and slightly less complex version of this where instead of `string.Builder`, we use `fmt.Sprint` to convert arrays to strings, but this version is slightly more efficient in terms of runtime speed. But, if I were to actually use this code in a production situation where I didn't need to micro-optimize everything like we are doing here, I might opt for the more compact and readable solution, since it just looks cleaner and is more straight-forward.

_Where `n` is a single dimension of the matrix..._

Time: `O(n^2)`

Space: `O(n^2)`

So from the first solution to here, I don't think time or space complexity changed at all, but we were able to make meaningful changes to reduce memory usage overhead and also just simplify the approach in general. I'm much happier with this final solution when compared to my first solution, which wasn't as readable or intuitive.

Main takeaway: Try to stick with simple data types, rather than structs, when possible. While custom struct types can make the code a lot more flexible and creative, they can consume more memory due to their overhead, especially if you get too fancy with it, or if you are going to need to form a big data structure consisting of a ton of these node structures like we had initially.

Ben's difficulty rating: 7.5/10. This one was a bit difficult actually. I initially struggled with getting my tree implementation to work, since I needed to rely on a lot of references and pointers, which was confusing. In hindsight, with this final approach, it probably shouldn't have been as hard as it was though.
