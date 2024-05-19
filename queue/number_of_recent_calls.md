# Number of Recent Calls

[leetcode link](https://leetcode.com/problems/number-of-recent-calls)

5/18/2024

Difficulty: Easy

## Description

You have a `RecentCounter` class which counts the number of recent requests within a certain time frame.

Implement the `RecentCounter` class:

-   `RecentCounter()` Initializes the counter with zero recent requests
-   `int ping(int t)` Adds a new request at time `t`, where `t` represents some time in milliseconds, and returns the number of requests that has happened in the past `3000` milliseconds (including the new request). Specifically, return the number of requests that have happened in the inclusive range `[t - 3000, t]`.

It is **guaranteed** that every call to `ping` uses a strictly larger value of `t` than the previous call.

**Example 1:**

```
Input
["RecentCounter", "ping", "ping", "ping", "ping"]
[[], [1], [100], [3001], [3002]]

Output
[null, 1, 2, 3, 3]

Explanation
RecentCounter recentCounter = new RecentCounter();
recentCounter.ping(1);     // requests = [1], range is [-2999,1], return 1
recentCounter.ping(100);   // requests = [1, 100], range is [-2900,100], return 2
recentCounter.ping(3001);  // requests = [1, 100, 3001], range is [1,3001], return 3
recentCounter.ping(3002);  // requests = [1, 100, 3001, 3002], range is [2,3002], return 3
```

**Constraints:**

-   `1 <= t <= 10^9`
-   Each test case will call ping with strictly increasing values of `t`.
-   At most `10^4` calls will be made to `ping`.

## Initial Thoughts / Planning

For this problem, we need to keep track of all pings that have come in, and when a ping comes in, return the number of pings that have come in in the past 3 seconds (3000 milliseconds).

I think the trick here is that we want an efficient way to know the number of pings in the past 3000 milliseconds. I also think we want to avoid storing too much history of pings, because of course once a ping is older than 3000 milliseconds, it's no longer needed.

All pings come in in chronological order (strictly increasing values of `t`), so at least we have our data sorted for us by default. So I think we could simply just store all the pings in an array as they come in, and then when each ping comes in, we could add it to the array and also check if any values can be removed from the front of the array (the front would be the first added, and therefore the oldest pings).

Here's a little visual of how it might work:

```
[]

ping: 50
[50]

ping: 260
[50, 260]
50 is still within 3000 milliseconds of 260

ping: 1900
[50, 260, 1900]
Once again, 50 is still within range.

ping: 3100
[50, 260, 1900, 3100]
At this point, 50 is too old. So as we add 3100, and we check the value (50) at the first index, we realize it's not within 3000 ms of the latest ping,
So we also remove 50.
[260, 1900, 3100]
```

So, with this algorithm, we will be able to track all the pings that come in and record them, but we also have a mechanism to clear out old pings as needed.
We will need to iterate through the array to the index where the values are no longer out of date, to clean out those out of date values, and then return the new size of the array.

## My Code Implementations, Optimizations

```go
type RecentCounter struct {
    History []int
}

func Constructor() RecentCounter {
    return RecentCounter{
        History: []int{},
    }
}

func (this *RecentCounter) Ping(t int) int {
    this.History = append(this.History, t)
    i := 0
    if this.History[0] < t - 3000 {
        for this.History[i] < t - 3000 {
            i++
        }
        this.History = this.History[i:]
    }
    return len(this.History)
}
```

(Pertaining to `Ping`)

Time complexity: `O(n)`, where `n` is the number of pings coming in

Space complexity: `O(n)`, since in the worst case, all `n` pings could be stored (if they are all within 3000 ms of each other)

I think this solution is pretty reasonable. Arrays are how queues are normally implemented in Go I believe - at least that's how they are done with stacks, and in my mind the only difference between a stack and a queue is the queue takes the elements off from the opposite side of where they are added (first in first out) where stacks take off from the same side they add (first in last out).

## Other Leetcode Solutions

So one thing I was wondering is, can we optimize this solution by better finding the point in `History` where the pings are too old. Right now we just iterate starting from index 0, and continue until we have found all the expired pings.

I then opened some solutions on Leetcode and found that one user made a solution where they use **binary search** to do this! This totally makes sense, because the `History` array is already sorted, which is the main requirement for using binary search. I believe the idea would be, you start the search in the middle of the array, and then choose to go to the lower half or upper half based on if the value we are looking for is less than or greater than the middle value. We repeat this process until we find the value we are looking for or determine it doesn't exist. In our case, we are looking for `t - 3000`, which is the breakpoint for when a ping in `History` would be considered expired. Let's see if binary search improves this algorithm.

```go
type RecentCounter struct {
    requests []int
}

func Constructor() RecentCounter {
    return RecentCounter{requests: make([]int, 0)}
}

func (this *RecentCounter) Ping(t int) int {
    this.requests = append(this.requests, t)
    min := t - 3000
    i := binarySearch(this.requests, min)
    return len(this.requests) - i
}

func binarySearch(nums []int, target int) int {
    l, r := 0, len(nums)-1
    for l <= r {
        mid := (l + r) / 2
        if nums[mid] == target {
            return mid
        } else if nums[mid] < target {
            l = mid + 1
        } else {
            r = mid - 1
        }
    }
    return l
}
```

One thing I don't like about this algorithm is that it doesn't bother to remove old, expired pings. Instead, it just keeps adding them on. I wonder why? Maybe the removal of the old values causes a slight runtime speed decrease, and they excluded it for that reason? Regardless, I think it's better to have it. Firstly, if you don't have any cleanup, then eventually the history array is going to get bloated with a ton of data, eventually overloading the memory and potentially even causing a crash. Secondly, the longer the data is that we need to search through, the longer in theory it will take to find what we are looking for. Let's try some benchmarking:

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	 3749209	       317.7 ns/op	    1016 B/op	       7 allocs/op  # my solution
BenchmarkSolutionB-8   	 2815012	       432.7 ns/op	    1016 B/op	       7 allocs/op  # leetcode solution
PASS
ok  	perf_testing	3.269s
```

For these benchmarks, I have the solutions pinging in a for loop from 0 ms to 6000 ms, incrementing 100 ms each ping. This means that there will be 60 total pings, and halfway through it will start needing to find the "expiration point" in the list of pings since there will be old pings more than 3000 ms old.

So far, it seems that my solution is just a little bit more efficient. Let's see what happens if we increase the number of pings overall to a much larger; here, we will run the same loop, but only incrementing by 1 ms each ping, resulting in 6000 total pings:

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	   38728	     28862 ns/op	  185592 B/op	      18 allocs/op  # my solution
BenchmarkSolutionB-8   	    8002	    148014 ns/op	  185593 B/op	      17 allocs/op  # leetcode solution
PASS
ok  	perf_testing	2.874s
```

We start to see my solution perform even better when it scales up; I think the reason for this might be that each ping is only 1 ms later than the last, so my solution is pretty effective then at finding the expired ping: it's just the value in the first index of the array, which is where my solution searches from. However the other solution uses binary search, which while still being a pretty fast way to search a sorted array, has a lot more overhead and is probably performing in the worst case.

Maybe if we had a bit more complicated search scenario, like having to find the value in the middle of the array somewhere, we would see different results?

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	   78925	     13282 ns/op	   87288 B/op	      15 allocs/op  # my solution
BenchmarkSolutionB-8   	   33715	     37268 ns/op	   87288 B/op	      15 allocs/op  # leetcode solution
PASS
ok  	perf_testing	3.075s
```

Still, it seems my solution performs a little better.

I'm now wondering if we modified the leetcode solution to also get rid of expired pings, perhaps it would perform better?

> Actually, we see a **degrade** in ns/op after making it get rid of the expired pings. I guess the main conclusion here is that, at least for the test cases we are using here, binary search is kind of overkill.
>
> Additionally, removing that last modification and using the leetcode solution as it was originally, and simply pinging from 1 to 1000000, we get these results:
>
> ```shell
> BenchmarkSolutionA-8   	     255	   4557312 ns/op	29938975 B/op	     926 allocs/op  # my solution
> BenchmarkSolutionB-8   	      54	  21498660 ns/op	41678136 B/op	      38 allocs/op  # leetcode solution
> ```
>
> Here we see the weakness of not cleaning out the history cache: not only is the speed slower (since it needs to search through 100s of thousands of elements every time), but it also appears to be using more memory, probably since the array isn't emptying. The interesting thing is that my solution has a lot more allocations though.
>
> If you think about it it makes sense though: on average, the expired pings will be much closer to the beginning than anywhere else. In many cases, there may only be one expired ping too - the very first element in the array. Having to use a more complex algorithm to search through the whole array - even if it's in a very efficient manner - may on average end up using more operations than just starting from the beginning.

I think the binary search technique is an interesting idea, but maybe it's just a little overly complex for this use case.

## Let's ask Chat-GPT

I talked to Chat-GPT, but I think it just started hallucinating an improvement to make. It didn't end up bearing any fruit, as its optimization ideas performed the same or worse in benchmarking, so decided to leave all that discussion out.

## Final Solution, Conclusion

My original solution:

```go
type RecentCounter struct {
    History []int
}

func Constructor() RecentCounter {
    return RecentCounter{
        History: []int{},
    }
}

func (this *RecentCounter) Ping(t int) int {
    this.History = append(this.History, t)
    i := 0
    if this.History[0] < t - 3000 {
        for this.History[i] < t - 3000 {
            i++
        }
        this.History = this.History[i:]
    }
    return len(this.History)
}
```

I don't think I have any big takeaways here. This was just a basic queue implementation, more or less.

Ben's difficulty rating: 3/10
