# Asteroid Collision

[leetcode link](https://leetcode.com/problems/asteroid-collision)

5/16/2024

Difficulty: Medium

## Description

We are given an array `asteroids` of integers representing asteroids in a row.

For each asteroid, the absolute value represents its size, and the sign represents its direction (positive means right, negative means left). Each asteroid moves at the same speed.

Find out the state of the asteroids after all collisions. If two asteroids meet, the smaller one will explode. If both are the same size, both will explode. Two asteroids moving in the same direction will never meet.

**Example 1:**

```
Input: asteroids = [5,10,-5]
Output: [5,10]
Explanation: The 10 and -5 collide resulting in 10. The 5 and 10 never collide.
```

**Example 2:**

```
Input: asteroids = [8,-8]
Output: []
Explanation: The 8 and -8 collide exploding each other.
```

**Example 3:**

```
Input: asteroids = [10,2,-5]
Output: [10]
Explanation: The 2 and -5 collide resulting in -5. The 10 and -5 collide resulting in 10.
```

**Constraints:**

-   `2 <= asteroids.length <= 10^4`
-   `-1000 <= asteroids[i] <= 1000`
-   `asteroids[i] != 0`

## Initial Thoughts / Planning

So I think the key thing we need to figure out how to handle is collisions (_duh!_). Ok, that's kind of obvious, but what I mean is, before tackling the entire problem, I want to just take a minute to think about the collisions more.

All asteroids are going either left or right: left is a negative value, and right is positive. No asteroids are sitting still (at value 0). Asteroids moving in the same directon will never hit each other.

So I guess we need to be able to find the points where a positive and negative asteroid are adjacent in the `asteroids` array, and then handle those collisions as they happen. But the trick is, after a collision, it could mean a new collision is waiting to happen. For example, if there are 3 asteroids moving right, and one large asteroid moving left, that large left-moving asteroid might destroy the first one it encounters and keep moving towards another collision.

> One thing I realize at this point is, if there is a maximum size asteroid, it will destroy all potential collisions it will encounter. Maybe that idea will be useful later on when considering optimization ideas?

Well, if we do as we often do, and consider tackling this problem moving left to right across the `asteroids` array, let's see what happens then:

-   We traverse the `asteroids` array, waiting until we find a "collision site"; i.e. a position where a positive and negative asteroid are adjacent.
-   a collision occurs: the larger asteroid destroys the smaller one. So, if our asteroids were `1, 2, 3, -2, -3, -4`, this first collision happens with `3` and `-2`, and `3` survives the collision.
-   now, our asteroids are like this: `1, 2, 3, -3, -4`, and we are currently at `3`, looking to move forward
-   uh oh, another collision! this time it's between `3` and `-3`. this results in **both** asteroids exploding. We now have: `1, 2, -4`
-   at this point, our position "moves back"? since the value `3` was destroyed, we are now looking at `2` and `-4`. Of course, `-4` wins this collision too, and will win the next one.

So, this all sounds fine and dandy, but we need to figure out what data structures to use here. I don't think traversing the array and making modifications in place is a good idea, because obviously the logic gets super messy, and would require some weird trickery with 2 pointers probably. It just sounds like a bad time in general.

At this point I'll stop playing dumb and pretending I don't know which section we are in - of course, we are in the **stack** section still, and I think using a stack sounds like a decent idea.

If we use a stack, we can envision placing values on the stack as we traverse `asteroids`, waiting to find a collision. If the next value to place on the stack collides with the last "stacked" asteroid, then a collision occurs and we follow the given logic. If the incoming asteroid is destroyed, then **don't add it to the stack**, just move on. If the incoming asteroid destroys the last-stacked asteroid, **pop the asteroid off the stack** and continue checking the next collision. This continued checking would repeat until either all elements in the stack were popped off (destroyed), or until this incoming asteroid were itself destroyed.

## My Code Implementations, Optimizations

Oops, one important thing I forgot about while planning my solution: a collision doesn't just occur for asteroids going in different directions (a negative is adjacent to a positive).
It happens if the two asteroids are moving **towards each other**. The asteroid at position `i` must be positive (moving right) and the asteroid at position `i + 1` must be negative (moving left).
Asteroids `[-1, 1]` will never meet.

So, with that in mind, here's my first solution:

```go
func asteroidCollision(asteroids []int) []int {
    stack := []int{asteroids[0]}
    for i := 1; i < len(asteroids); i++ {
        if len(stack) == 0 {
            stack = append(stack, asteroids[i])
            continue
        }
        lastAsteroid := stack[len(stack) - 1]
        if isCollision(lastAsteroid, asteroids[i]) {
            // handle collision
            addNew := true
            for len(stack) > 0 && isCollision(stack[len(stack) - 1], asteroids[i]) {
                if abs(stack[len(stack) - 1]) > abs(asteroids[i]) {
                    // incoming asteroid is destroyed; nothing changes
                    addNew = false
                    break
                } else {
                    if abs(stack[len(stack) - 1]) == abs(asteroids[i]) {
                        // size match; incoming asteroid is destroyed too
                        addNew = false
                        stack = stack[:len(stack) - 1]
                        break
                    }
                    // stacked asteroid is destroyed
                    stack = stack[:len(stack) - 1]
                }
            }
            if addNew {
                stack = append(stack, asteroids[i])
            }
        } else {
            // continue stacking asteroids
            stack = append(stack, asteroids[i])
        }
    }
    return stack
}

// ast1 must be left of ast2, index wise
func isCollision(ast1, ast2 int) bool {
    return ast1 > 0 && ast2 < 0
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```

Time complexity: `O(n)`. I wasn't sure about this actually, so I had to look into this in more detail. There is a nested for loop here, so at first I feared it might be `O(n^2)`. But in reality, the nested for loop is just for handling collisions, and each asteroid can only be added and removed from the stack once. So, therefore, each asteroid is only processed a **constant** number of times.

Space complexity: `O(n)`. Since we create a stack that will end up having all the asteroids in it, we use linear space.

The logic is the same as described above for the most part; once a collision is found, the incoming asteroid (from the right side) plows through all the asteroids on the stack until it meets a match or gets outright destroyed itself. This process is repeated until the end of `asteroids`. One tricky thing was just making sure I wasn't accessing invalid indices of `stack`; since we are deleting and adding to the stack in various places, I had to make sure to be careful with when I'm accessing an index, and to check if it's still valid (e.g. is stack empty now?).

But, this wasn't too difficult to figure out once I knew we were using a stack.

### Optimization Ideas

So, now is the part where I look at my solution and try to figure out ways it could be optimized.

One thing I always like to avoid, when possible is `append` with arrays. This causes the array to be resized, which can be a little inefficient. In a previous problem I was convinced that appends were _very_ inefficient, but had some uncertain results from leetcode's performance metrics, so I'm actually no longer as confident in this optimization. But, still, from that last problem I believe I concluded that avoiding `append` had _some_ benefit.

Another thing I wondered about is, maybe there's a way to do this without needing to have the nested for loop, that does the backtracking where we find out how far into the stack an incoming asteroid destroys. I was envisioning something where we approach from the right instead, keeping track of the "most negative" asteroid, and using that value to know how far into the right-moving asteroids it would destroy in. But then I realized that I would end up basically making a reverse solution of what I currently have, because I'd need a way to "remember" the values before or after the big asteroid value. Anyway, I don't think switching the direction in which we process the asteroids will change much.

### Optimizing away append - a good idea, or not?

Let's see if we can avoid using appends in my solution. I think I will initialize `stack` to be the length of `asteroids`, and then use a pointer to keep track of the "real length".

Here's my code after removing the appends, and instead using a pointer to keep track of the length of the stack:

```go
func asteroidCollision(asteroids []int) []int {
    stack := make([]int, len(asteroids))
    stack[0] = asteroids[0]
    si := 1 // pointer to the insertion index of the stack
    for i := 1; i < len(asteroids); i++ {
        if si == 0 {
            stack[si] = asteroids[i]
            si++
            continue
        }
        lastAsteroid := stack[si - 1]
        if isCollision(lastAsteroid, asteroids[i]) {
            // handle collision
            addNew := true
            for si > 0 && isCollision(stack[si - 1], asteroids[i]) {
                if abs(stack[si - 1]) > abs(asteroids[i]) {
                    // incoming asteroid is destroyed; nothing changes
                    addNew = false
                    break
                } else {
                    if abs(stack[si - 1]) == abs(asteroids[i]) {
                        // size match; incoming asteroid is destroyed too
                        addNew = false
                        si--
                        stack[si] = 0
                        break
                    }
                    // stacked asteroid is destroyed
                    si--
                    stack[si] = 0
                }
            }
            if addNew {
                stack[si] = asteroids[i]
                si++
            }
        } else {
            // continue stacking asteroids
            stack[si] = asteroids[i]
            si++
        }
    }
    return stack[:si]
}
```

**However**, after running benchmarks to compare this solution to the previous one, the results are unexpected. Here's the console output:

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	     550	   1996428 ns/op
BenchmarkSolutionB-8   	     586	   2040177 ns/op
PASS
ok  	perf_testing	2.938s
```

> Note added later: the `ns/op` here includes the data generation as well. Oops! Didn't consider removing that from the benchmarks until later in this write-up.

This is after running the solutions against a randomly generated slice of input data of length `100000`.

`SolutionA` is my original solution, using a regular `[]int` slice as a stack, without pre-allocating the size.

`SolutionB` is the "optimized" solution, using a `[]int` slice of pre-allocated size, and a pointer to track the size of the stack.

As we can see here, `SolutionB` might be just slightly **slower** than `SolutionA`!
This is honestly pretty surprising news. I tried changing the input size of the data from 1000, to 10000, to 100000, but in each case the result was the same: `SolutionA` performed slightly better than `SolutionB`. Running these benchmarks over and over return the same result.

Running this by Chat-GPT and trying his suggestions, I still find the same result. My attempt at optimizing this by using pre-allocation and removing all `append` commands seems to be fruitless.
Chat-GPT has been telling me about how Go's `append` command is quite optimized though, and that when an array resize occurs, it's unlikely another resize will need to happen (under the hood) due to how it handles it. So it may be just as efficient as if no resizing were occurring, depending on the situation. It also had other various ideas, like maybe the "optimized" solution makes branch prediction more difficult to do. Branch prediction is some pretty low level stuff concerning how code is compiled and how well a CPU can predict the path of the code. If code is easily predicted in this way, it is further optimized since the CPU can efficiently predict it. I don't know if there's any merit to those random assumptions though.

> I guess at this point I have to face the fact: `append` must not be such a problem as I thought it was. Either that, or my "optimization" must include some unknown overhead that ends up making it perform worse. In the future, unless I find myself in a dire situation where I'm desperate for optimizations, I will no longer view `append` with so much skepticism. Perhaps I will only consider this optimization when the algorithm using the slice is itself not too complicated, and not requiring extra pointers.
>
> Here's a stackoverflow post that compares `append` with regular assignments to indices: https://stackoverflow.com/questions/38654729/golang-slice-append-vs-assign-performance
> I think it gives some good insight into what happens. It sounds like basically `append` is "amortized" O(1). Nearly O(1), but due to some overhead and the possibility of sometimes needing to resize the underlying array structure, it could be more than constant time in some cases. That's why technically it's better to pre-allocate space when you know how much space you need, but in our case I think there's just a little too much overhead involved with the pre-allocation method, which renders it not an improvement but maybe a slight degradation in runtime.

## Other Leetcode Solutions

Looking around at other Leetcode solutions, I found pretty much all of them take the same approach, but with varying logic. The logic itself gets to the same goal, but some solutions looked a bit simpler or more straightforward than mine. I tried running some benchmarks again, comparing my solution with others on leetcode, and interestingly even with logic that appears a little bit simpler, the average `ns/op` was still largely the same, sometimes mine beating theirs or vice-versa by a slim margin. I guess my logic must still perform alright.

## Let's ask Chat-GPT

I talked to Chat-GPT, and he mainly suggests that I can simplify my logic a bit. I used the modified solution he gave me, and benchmarked it alongside my two other solutions (the one with a regular stack, and the one with the (not so) "optimized" stack). Here are the results:

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	    2034	    590093 ns/op	   60024 B/op	      14 allocs/op  # Ben's original solution/logic
BenchmarkSolutionB-8   	    1843	    650488 ns/op	  802817 B/op	       1 allocs/op  # "append-less" stack version
BenchmarkSolutionC-8   	    1920	    619734 ns/op	   60024 B/op	      14 allocs/op  # Chat-GPT's modified logic
PASS
ok  	perf_testing	4.966s
```

Honestly surprising, but so far, my original solution apparently performs the best. Not sure why, but I'll take it!
It does appear - now that I've added in memory allocation to the benchmarking - that the append-less stack version might be using more memory. But at the same time, it apparently only does one allocation.
Maybe that shines a light into what's going on with append: the other solutions end up doing allocations 14 times (out of the 100,000 asteroids its iterating over) to resize the array under the hood. But since we allocate the array to the max size it would need, it uses up more memory, but only does 1 allocation.

> You may also notice `ns/op` is considerably lower here. That's because I removed the code that was generating randomized asteroid data from the benchmarks. Seems like a no-brainer to do that in hindsight lol. Now they all share the same asteroid data, and the generation is excluded from the performance capture.

## Final Solution, Conclusion

Even after all the attempts to optimize my first solution, interestingly I seem to have failed. At least when it comes down to the numbers.

Final solution - same as my first one:

```go
func asteroidCollision(asteroids []int) []int {
    stack := []int{asteroids[0]}
    for i := 1; i < len(asteroids); i++ {
        if len(stack) == 0 {
            stack = append(stack, asteroids[i])
            continue
        }
        lastAsteroid := stack[len(stack) - 1]
        if isCollision(lastAsteroid, asteroids[i]) {
            // handle collision
            addNew := true
            for len(stack) > 0 && isCollision(stack[len(stack) - 1], asteroids[i]) {
                if abs(stack[len(stack) - 1]) > abs(asteroids[i]) {
                    // incoming asteroid is destroyed; nothing changes
                    addNew = false
                    break
                } else {
                    if abs(stack[len(stack) - 1]) == abs(asteroids[i]) {
                        // size match; incoming asteroid is destroyed too
                        addNew = false
                        stack = stack[:len(stack) - 1]
                        break
                    }
                    // stacked asteroid is destroyed
                    stack = stack[:len(stack) - 1]
                }
            }
            if addNew {
                stack = append(stack, asteroids[i])
            }
        } else {
            // continue stacking asteroids
            stack = append(stack, asteroids[i])
        }
    }
    return stack
}

// ast1 must be left of ast2, index wise
func isCollision(ast1, ast2 int) bool {
    return ast1 > 0 && ast2 < 0
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```

The main lesson I learned here is that `append` is not necessarily a thing to be avoided at all costs. In fact, the overhead of trying to avoid append, even if it's really minor, could end up making the solution less efficient. I think avoiding appends and pre-allocating array size is really only something you should do if you already know the (approximate) size of the array in the beginning, and if doing so doesn't require any extra work or complexity added on. It may only make a difference when handling really large amounts of data, and where almost all that data will need to be appended. Go already handles appends very efficiently, and when a resize occurs to the underlying array, it adds plenty of extra space to where it's unlikely you'll need many more additional resize/allocations.

Ben's difficulty rating: 6.8/10. Wasn't super simple, and was a bit more complicated than the last stack implementation I did.
