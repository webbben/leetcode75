# Dota2 Senate

[leetcode link](https://leetcode.com/problems/dota2-senate/)

5/21/2024

Difficulty: Medium

## Description

In the world of Dota2, there are two parties: the Radiant and the Dire.

The Dota2 senate consists of senators coming from two parties. Now the Senate wants to decide on a change in the Dota2 game. The voting for this change is a round-based procedure. In each round, each senator can exercise one of two rights:

-   Ban one senator's right: a senator can make another senator lose all his rights in this and all the following rounds.
-   Announce the victory: if this senator found the senators who still have rights to vote are all from the same party, he can announce the victory and decide on the change in the game.

The string `senate` represents each senator's party belonging. The characters `'R'` and `'D'` represent the Radiant and Dire parties respectively. If there are `n` senators, then the size of the string `senate` will be `n`.

The round-based procedure starts from the first senator to the last senator in the given order. This procedure will last until the end of voting. All the senators who have lost their rights will be skipped during the procedure.

Suppose every senator is smart enough and will play the best strategy for his own party. Predict which party will finally announce the victory and change the Dota2 game. The output should be `"Radiant"` or `"Dire"`.

**Example 1:**

```
Input: senate = "RD"
Output: "Radiant"
Explanation:
The first senator comes from Radiant and he can just ban the next senator's right in round 1.
And the second senator can't exercise any rights anymore since his right has been banned.
And in round 2, the first senator can just announce the victory since he is the only guy in the senate who can vote.
```

**Example 2:**

```
Input: senate = "RDD"
Output: "Dire"
Explanation:
The first senator comes from Radiant and he can just ban the next senator's right in round 1.
And the second senator can't exercise any rights anymore since his right has been banned.
And the third senator comes from Dire and he can ban the first senator's right in round 1.
And in round 2, the third senator can just announce the victory since he is the only guy in the senate who can vote.
```

**Constraints:**

-   `n == senate.length`
-   `1 <= n <= 10^4`
-   `senate[i]` is either `'R'` or `'D'`.

## Initial Thoughts / Planning

At first I was thinking "this seems pretty simple: the party with the most senators wins, right?". But after looking at this longer, I've realized it's definitely not going to be as simple as counting Rs and Ds.

...

Ok, after a lot of confusion, I think I've figured out a little more clearly how this works.

Maybe I'm just a little slow to figure this out, but this is a **round-based** game. I guess I knew that much before, but I misinterpreted what a round was. I think this problem description is just a bit clunky and could be explained a little clearer.

Each **round** consists of going over **all the senators** in `senators`. Before I was thinking a "round" was just a single senators turn to exercise his right.

So, here's a description on how it might go:

-   Round 1 begins; each senator starts blocking opposing senators' rights to vote. This leave us with a certain number of senators who are remaining, still able to vote.
-   Round 2 begins; If there is still any remaining opposition, those people start voting each other out. At some point, we expect that only a single party's senators remain with their full rights, and one of those can declare victory.

I'm not sure, but it's possible this process could go on for several rounds. But I kind of anticipate that it wouldn't for more than a couple rounds.

One of my first attempts at coding a solution failed on this case:

`"RRDDD"`

So let's walk through this one, round by round, to see what happens:

-   Round 1: 2 `D` senators have their rights stripped, and 1 `R` senator has his right stripped.

`"RD"`

-   Round 2: The last `R` senator blocks the rights of the last `D` senator.

`"R"`

-   Round 3: The `R` senator declares victory.

So, as we can see here, it's not just about the number of `R` vs `D`, the order in which they appear also plays a significant role in determining the outcome.
This means that we can't just iterate over `senators`, counting the number of Radiant and Dire senators. That's because once it's time for the next round (i.e. the next iteration), we won't be able to remember the relative order of the remaining senators, which is crucial.

Initially I was thinking a stack would work here. I imagine having a stack for each party, and when a block occurs, we simply don't add a senator to his party's stack. However, at that point, what's the difference between using a stack and a counter? Also, when it comes time to reconstruct the order of the remaining senators for the next round, we don't know where to start here either.

I think we need a single data structure that will maintain the order of the senators, but maybe it'll represent the "outcome" of a round. It will then be re-run successively until one party has won.

Alright, at this point let's just stop pretending: we know we need to use a queue! It's the section we are in.

Looking at the example problem above, let's imagine we use a queue to simulate the progress of a round:

Round 1:

`['R']` - the first `R` senator is added

`['R', 'R']` - the second `R` senator is added. We encounter two `D` senators, but both have their rights blocked by the preceeding 2 `R` senators, so they are not added to the queue.

`['R', 'D']` - a third `D` senator appears; there is not a third `R` senator to block his rights, so he is added to the queue, and he blocks the first `R` senator's rights, who leaves the queue from the top.

Alright, this seems to work pretty nice doesn't it?

But, what I'm kind of confused about is, what if there were another senator after the last `D` who is another `R`?

`"RRDDDR"`

Everything would go the same, according to the previous example, up to the last `D`:

`['R', 'D']` - The last `D` joins the queue, pushes out an older `R`.

But, next up is another `R`. And we would expect that this next `R` tries to block the rights of the preceeding `D`, right?

So, as we can see, we can't just shift out the leading part of the queue, since it would be friendly fire. Does that mean this incoming `R` just replaces the last `D` in its place? I suppose that would work, but it seems kind of like we are abusing this queue a bit. I think it might work though: I'm trying to picture a scenario where opposing senators would get "lost" in the middle of the queue, but I'm actually not sure if that is possible, if we implemented this extra rule here.

Ok, so summarizing our logic:

-   We encounter a senator
    -   If any prior opposing senators haven't cast a vote yet, this senator is skipped; we decrement a counter representing the opposing party's available votes
    -   If not, we check if the entry of the queue (last index) or exit of the queue (index 0) are opposing senators. One of these, in that order, is kicked out if so.
    -   If no opposing senators were there, then we just add this senator to the queue; we also add a voting point for his party.

This process is repeated each round, with the resultant queue being used for the next round. Eventually, only senators from one party will remain.

## My Code Implementations, Optimizations

After lots of battle, here's my solution so far, which I believe implements the above logic:

```go
func predictPartyVictory(senate string) string {
    q := make([]byte, 0)
    rVotes, dVotes := 0, 0

    // do the first round, since our queue starts as empty
    for i := 0; i < len(senate); i++ {
        q, rVotes, dVotes = handleRound(q, rVotes, dVotes, senate[i])
    }
    for {
        newQ := make([]byte, 0)
        rVotes, dVotes = 0, 0
        d, r := 0, 0
        for _, sen := range q {
            if sen == 'R' {
                r++
            } else {
                d++
            }
            newQ, rVotes, dVotes = handleRound(newQ, rVotes, dVotes, sen)
        }
        if d == 0 {
            return "Radiant"
        }
        if r == 0 {
            return "Dire"
        }
        q = newQ
    }
}

func handleRound(q []byte, rVotes, dVotes int, nextSen byte) ([]byte, int, int) {
    if nextSen == 'R' {
        if dVotes > 0 {
            dVotes--
            return q, rVotes, dVotes
        }
        // Check if we should replace an existing senator
        if len(q) > 0 {
            if q[len(q)-1] == 'D' {
                q[len(q)-1] = 'R'
                return q, rVotes, dVotes
            }
            if q[0] == 'D' {
                q = q[1:]
                q = append(q, 'R')
                return q, rVotes, dVotes
            }
        }
        // No replacement; add to the end and record an unused vote
        q = append(q, 'R')
        rVotes++
    } else {
        if rVotes > 0 {
            rVotes--
            return q, rVotes, dVotes
        }
        // Check if we should replace an existing senator
        if len(q) > 0 {
            if q[len(q)-1] == 'R' {
                q[len(q)-1] = 'D'
                return q, rVotes, dVotes
            }
            if q[0] == 'R' {
                q = q[1:]
                q = append(q, 'D')
                return q, rVotes, dVotes
            }
        }
        // No replacement; add to the end and record an unused vote
        q = append(q, 'D')
        dVotes++
    }
    return q, rVotes, dVotes
}
```

**_However..._** there must be something wrong with my my original strategy. For this case:

`"DRRDRDRDRDDRDRDR"`

The expected output is `"Radiant"`. However, when I carefully walk through it using my current strategy, I get `"Dire"` as the answer.
the first round turns this into `"DDDD"`:

```python
D R R D R D R D R D D R D R D R

D D D D
# we got here basically by each subsequent R and D blocking the last one over and over; which clearly ends in D's favor.
```

So, I guess the strategy is invalid, because I recall in the description it says we can assume the senators always make the optimal move:

> _"Suppose every senator is smart enough and will play the best strategy for his own party"_

So let's look at this example and figure out what the optimal strategy is for Radiant to win.
Before, I was assuming that a senator should always look backwards into the queue for an opposing senator to remove. Perhaps that's not the case?

```python
D R R D R D R D R D D R D R D R

# Round 1: if R chooses to block D's after it, then it actually ends up in a superior position
D R R R R D D D

# Round 2: after this, R now clearly has the advantage, and can choose to block the senator at the front.
D R R R

R R
```

So I think this demonstrates that perhaps, it's smarter to try and gain some "momentum" rather than just try to go back and get revenge from the last senator. If you can snowball a big group of senators all of the same party right next to each other, then it's hard to take them down.

The thing is, we need to know when it's appropriate to look back vs look forward. For example, if `R` never looked back to block an opposing senator, then it would end up losing since the first position is a `D`. Maybe the logic could be, if the next senator in line is the same party, you can look back to block a previous opposing senator?

```python
D R R D R D R D R D D R D R D R

# first round goes the same
D R R R R D D D

# second round goes slightly different; one of the Rs blocks a D behind instead of forward, since there was a line of multiple Rs and one D behind.
R R D

# in the final round, R still wins.
R R
```

Ok, here's the revised logic for a given senator at position `i`:

-   if the other party has votes they are waiting to use:
    -   he is eliminated before he can vote
-   else
    -   if the next position, `i + 1`, is the opposing party, block the next senator.
    -   if not, then look behind in the queue (either immediately behind or at the front of the queue?)
    -   if none of these positions have an opposing senator, save the vote for later in this round

> So at this point I've come back to this problem/solution I'm working on a few times, and it seems like I just keep finding new cases where my logic for determining the correct moves is flawed.
> I wonder if I'm just approaching this wrong? The next case which I'm failing is quite a long string, so I'm not really motivated to try and walk through it manually, and then ponder how I can adjust my strategy.
>
> A quick complaint: this problem is poorly defined in my opinion. They don't give enough details on the nuances of how a senator should choose other senators to block, or if there are any constraints in that process. It seems like I'm just having to discover the correct strategy of where and when a senator blocks another senator, and having to re-invent things as new cases fail. Also, the example cases are way too simple; only 2 examples, each only having a really basic case?
>
> This feels like giving up, but I think I'm going to look for help on this before proceeding.

## Other Leetcode Solutions

Looking at solutions on leetcode, I see there are a couple popular approaches out there.

1. The "index-based" approach.

This approach seems to require you to first make two queues, each tracking one of the parties, and holding the indices of each senator of that party.
Then, they iterate through the two queues, finding which senators of each party come before the other, and I guess banning each other. Once a senator bans someone, he is put at the end of the queue again.

Eventually, one of the queues will be empty, and this determines the winner.

2. The "float ban" approach - very similar to the one I came up with.

This one seems to be very similar to my solution, but a bit simpler I think. Basically, there is a count of the number of senators for each party, and a count of the number of "floated bans" - that is, a ban that hasn't been used yet, and is waiting for the next senator of the opposing party to be used. This one also loops until one of the parties senator count gets to 0, which determines the winner.

> Since my original attempt at a solution is similar to solution 2 here, I think I'm going to attempt to implement this solution idea first.

```go
func predictPartyVictory(senate string) string {
    r, d := 0, 0 // number of un-voted-out senators
    rVotes, dVotes := 0, 0 // number of votes a party has ready to use
    q := make([]byte, 0) // queue of the senators, in their voting order

    // first, fill the queue and get the count of senators from each party
    for i := 0; i < len(senate); i++ {
        q = append(q, senate[i])
        if senate[i] == 'R' {
            r++
        } else {
            d++
        }
    }

    // repeat the rounds process until only one party remains
    for r > 0 && d > 0 {
        sen := q[0] // shift off the next senator
        q = q[1:]
        if sen == 'R' {
            // is a ban already waiting to be used?
            if dVotes > 0 {
                dVotes--
                r--
                continue
            }
            rVotes++
        } else {
            if rVotes > 0 {
                rVotes--
                d--
                continue
            }
            dVotes++
        }
        q = append(q, sen) // put this senator back in line for next round
    }
    if r > 0 {
        return "Radiant"
    }
    return "Dire"
}
```

Well there we have it - this solution finally works!

I feel kind of dumb now, looking at how simple this solution is. It's actually a queue that just keeps running until it becomes empty. How was my previous solution failing I wonder?
There must have been some discrepancy in how the votes were being used. The main difference I notice is here, the queue continually is shifting senators off the front of the queue, and then if they aren't blocked, is putting them back on at the end. It then just repeats this loop of checking if a senator is blocked by another, and if not, re-enqueuing him.

I guess my solution before was failing probably because in a single round, the senators are only working with the knowledge accumulated so far - about who has gone previously, but not who is to come ahead. That's a slightly personifying way to put it - because the logic is dead simple in the end - but basically there must have been some gap between the different rounds, whereas this solution there is no gap, just a continuity.

So, I guess I was on the right track, but got bogged down with trying to make some elaborate logic that will fix all the edge cases I was running into. When in reality, I just needed a simpler, but slightly different, implementation of the queue. No need to "look back" at the previous senators or anything, just count up unused votes and apply them as soon as an opposing senator is encountered. I feel like one of my test implementations early on was closer to this idea honestly.

### Optimizations?

One idea I have, which is probably a pretty small optimization, is to initialize `q` to be the same length as senate, and then just insert all the senators in their correct places during the initialization process for the queue. No need to append a bunch there. This would only make a minor difference though, and only on a very large `senate` string. Let's see if there's a perceptible change or not in efficiency.

```shell
BenchmarkSolutionA-8   	 2719264	       425.6 ns/op	     392 B/op	       6 allocs/op  # with append
BenchmarkSolutionB-8   	 3424144	       348.6 ns/op	     320 B/op	       2 allocs/op  # pre-allocated q initial size
```

Sure enough - there's a tiny difference here. That's interesting at least, but not really a big deal.

I don't see any other minor optimization ideas in here for now, so what if we consider the other approach idea - the "index-based" solution?

Here, I'll just copy and paste a solution on Leetcode, courtesy of user `vibhushit`:

```go
func predictPartyVictory(senate string) string {
    radiant := []int{} // Slice to store the indices of Radiant senators
    dire := []int{}    // Slice to store the indices of Dire senators

    // Initialize the slices
    for i, party := range senate {
        if party == 'R' {
            radiant = append(radiant, i)
        } else {
            dire = append(dire, i)
        }
    }

    for len(radiant) > 0 && len(dire) > 0 {
        // Find the first Radiant senator to ban a Dire senator
        if radiant[0] < dire[0] {
            radiant = append(radiant, radiant[0]+len(senate))
        } else {
            dire = append(dire, dire[0]+len(senate))
        }

        radiant = radiant[1:] // Remove the Radiant senator
        dire = dire[1:]       // Remove the Dire senator
    }

    if len(radiant) > 0 {
        return "Radiant"
    } else {
        return "Dire"
    }
}
```

Here are the benchmark results, with `SolutionB` being this new approach:

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	 3288640	       349.8 ns/op	     320 B/op	       2 allocs/op  # "floating ban" approach
BenchmarkSolutionB-8   	 1635192	       726.2 ns/op	    3312 B/op	      16 allocs/op  # "index-based" approach
PASS
ok  	perf_testing	3.706s
```

At least in its current implementation, it's clear to see that the "floating ban" approach (the solution before) works better than this new one. Seems to be a lot more memory efficient and also faster in terms of run-time.

I'm not really sure what makes this solution noticeably slower though. Is having two queues adding more overhead? I would think that explains the memory usage, at least in part. Maybe the comparison happening inside the loop is slightly more expensive too, than just checking if an int value is greater than 0. I guess inside that main loop, there are also two arrays being shifted simultaneously, which may be the more expensive part of the algorithm in the first place.

## Final Solution, Conclusion

```go
func predictPartyVictory(senate string) string {
    r, d := 0, 0 // number of un-voted-out senators
    rVotes, dVotes := 0, 0 // number of votes a party has ready to use
    q := make([]byte, len(senate)) // queue of the senators, in their voting order

    // first, fill the queue and get the count of senators from each party
    for i := 0; i < len(senate); i++ {
        q[i] = senate[i]
        if senate[i] == 'R' {
            r++
        } else {
            d++
        }
    }

    // repeat the rounds process until only one party remains
    for r > 0 && d > 0 {
        sen := q[0] // shift off the next senator
        q = q[1:]
        if sen == 'R' {
            // is a ban already waiting to be used?
            if dVotes > 0 {
                dVotes--
                r--
                continue
            }
            rVotes++
        } else {
            if rVotes > 0 {
                rVotes--
                d--
                continue
            }
            dVotes++
        }
        q = append(q, sen) // put this senator back in line for next round
    }
    if r > 0 {
        return "Radiant"
    }
    return "Dire"
}
```

Time complexity: `O(n)` - where `n` is the length of `senate`

Space complexity: `O(n)` - due to the queue we are using.

Ben's difficulty rating: 9/10. I don't really know why this was so difficult, but this is the first problem that I couldn't figure out on my own in a reasonable amount of time, and where I had to resort to finding solutions to help solve this. Not to complain too much, but I think it would've been nice if the problem description elaborated on what the "optimal strategy" is for a given party. It turns out that it's probably nothing elaborate, but the fact they point out that senators need to act in the "smartest way" made me second guess and think perhaps there's a deeper strategy required.

Anyway, yeah I think this is only considered a Medium just because the data structures and overall high-level concepts used aren't too complicated; it's just a tricky problem to understand and organize a solution for. If I were asked this in an interview, I'm guessing I would've failed it unless the interviewer gave me some good hints.
