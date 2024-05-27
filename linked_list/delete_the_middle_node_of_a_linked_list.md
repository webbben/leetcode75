# Delete the Middle Node of a Linked List

[leetcode link](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list)

5/27/2024

Difficulty: Medium

## Description

You are given the `head` of a linked list. Delete the middle node, and return the `head` of the modified linked list.

The middle node of a linked list of size `n` is the `[n / 2]`-th node from the start using 0-based indexing, where `[x]` denotes the largest integer less than or equal to `x`.

-   For `n = 1, 2, 3, 4, 5`, the middle nodes are `0, 1, 1, 2, 2`, respectively.

**Example 1:**

![ex1](https://assets.leetcode.com/uploads/2021/11/16/eg1drawio.png)

```
Input: head = [1,3,4,7,1,2,6]
Output: [1,3,4,1,2,6]
Explanation:
The above figure represents the given linked list. The indices of the nodes are written below.
Since n = 7, node 3 with value 7 is the middle node, which is marked in red.
We return the new list after removing this node.
```

**Example 2:**

![ex2](https://assets.leetcode.com/uploads/2021/11/16/eg2drawio.png)

```
Input: head = [1,2,3,4]
Output: [1,2,4]
Explanation:
The above figure represents the given linked list.
For n = 4, node 2 with value 3 is the middle node, which is marked in red.
```

**Example 3:**

![ex3](https://assets.leetcode.com/uploads/2021/11/16/eg3drawio.png)

```
Input: head = [2,1]
Output: [2]
Explanation:
The above figure represents the given linked list.
For n = 2, node 1 with value 1 is the middle node, which is marked in red.
Node 0 with value 2 is the only node remaining after removing node 1.
```

**Constraints:**

-   The number of nodes in the list is in the range `[1, 10^5]`
-   `1 <= Node.val <= 10^5`

### 日本語の説明

> Note: I'm going to start writing brief descriptions and summaries of the conclusion in Japanese. This is just to help me get more used to describing technical and algorithmic details in Japanese.

連結リストの`head`の先頭ノードが与えられます。中央のノードを削除して、変更された連結リストの`head`を返してください。

連結リストの長さは`n`としたら、中央ノードはリストの先頭から`n / 2`番目のノードになります。`n`は奇数で、`n / 2`の値は整数じゃない場合は、その値を切り捨てます。

## Initial Thoughts / Planning

So we want to delete the middle node of a linked list.

First, let's just simplify some of the description above:

When they are defining which node the middle is, I think it's safe to just put it as the `floor` of `n / 2`, where `n` is the length of the linked list. I'm not sure why they worded it in an overly complicated way.

So the first problem is, given the `head` node, how do we find the middle node?

I think the only way to do this is to get the size of the linked list - which requires traversing the linked list first, counting the number of nodes. The data structure doesn't seem to provide a quick way to get the length:

```go
type ListNode struct {
    Val int
    Next *ListNode
}
```

Once we have `n`, then we need to connect the nodes before it and after it, which effectively deletes the middle node. If the middle node is at position `i`, then I believe we would first traverse to node `i - 1`, keep a pointer to it, and then go forward to node `i + 1` and set `Next` for node `i - 1` to point to node `i + 1`.

## My Code Implementations, Optimizations

```go
func deleteMiddle(head *ListNode) *ListNode {
    // first, let's find the size of the linked list by counting each node
    n := 0
    curNode := head
    for curNode.Next != nil {
        n++
        curNode = curNode.Next
    }
    if n == 0 {
        return nil
    }

    // now, let's remove the node at n / 2
    middleIndex := int(math.Ceil(float64(n) / 2))
    lastNode := head
    curNode = head
    for i := 0; i < middleIndex; i++ {
        lastNode = curNode
        curNode = curNode.Next
    }
    // next, skip to the node after the middle node, and link it to lastNode
    curNode = curNode.Next
    lastNode.Next = curNode
    return head
}
```

Time complexity: `O(n)`

Space complexity: `O(1)`, since we only create a constant number of extra nodes/helper variables.

So as described before, this solution basically just makes a first pass to get the size of the linked list, and then goes to find what should be the middle index, to remove it. Removing the middle node just involves changing the `Next` pointer of the node before the middle to point to the node after the middle node, effectively cutting the middle node out.

One thing I'm not sure about is how I currently calculate the `middleIndex` value. Contrary to what I thought we'd do before, here we are doing `math.Ceil`, rounding up the value.
Why would we be doing that? Then, I realized that our size calculation is actually potentially 1 shorter than it's supposed to be. For example, suppose we are given a `head` node that doesn't have a `Next` node? `n` would be `0`, which is incorrect; it should be `1`. And so, if all the sizes are one less than they are supposed to be, maybe that's why we are rounding up instead of down to get the correct middle index. Let's see if I can fix that.

```go
func deleteMiddle(head *ListNode) *ListNode {
    // first, let's find the size of the linked list by counting each node
    n := 1
    curNode := head
    for curNode.Next != nil {
        n++
        curNode = curNode.Next
    }
    if n == 1 {
        return nil
    }

    // now, let's remove the node at n / 2
    middleIndex := n / 2
    lastNode := head
    curNode = head
    for i := 0; i < middleIndex; i++ {
        lastNode = curNode
        curNode = curNode.Next
    }
    // next, skip to the node after the middle node, and link it to lastNode
    curNode = curNode.Next
    lastNode.Next = curNode
    return head
}
```

Much better; now `middleIndex` is just calculated with a simple `n / 2`. Since `n` is an integer, the result is also an integer, which effectively rounds down any decimal value that could be produced too.

Considering optimizations, I'm not really sure what we can do here. My understanding is there are two problems we need to solve:

-   find the size of the linked list, so we can calculate the middle index
-   find the element at the middle of the linked list, so it can be unlinked

Maybe there are some micro-optimizations for space usage, but I can't think of any notable problems with our approach here.

Let's see what Leetcode solutions have to offer.

## Other Leetcode Solutions

Courtesy of user `krokop`, I found this solution:

```go
func deleteMiddle(head *ListNode) *ListNode {
	// count list length
	length := 0
	for node := head; node != nil; node = node.Next {
		length++
	}
	if length == 1 {
		return nil
	}
	middle := length / 2
	// delete middle node
	node := head
	for i := 0; i < middle-1; i++ {
		node = node.Next
	}
	node.Next = node.Next.Next
	return head
}
```

This solution is nearly identical to mine, but I noted that it actually is a little bit simpler. Once the middle index has been found, instead of recording the node before the middle, and then using that after to link it to the node after the middle, it just iterates to `middle - 1`. From there, it can just link the nodes up a little simpler. I like it, so I think I'll make a similar modification in my solution, since it'll remove the need for `lastNode` in my solution.

Here's our solution after this small modification:

```go
func deleteMiddle(head *ListNode) *ListNode {
    // first, let's find the size of the linked list by counting each node
    n := 1
    curNode := head
    for curNode.Next != nil {
        n++
        curNode = curNode.Next
    }
    if n == 1 {
        return nil
    }

    // now, let's remove the node at n / 2
    middleIndex := n / 2
    curNode = head
    for i := 0; i < middleIndex - 1; i++ {
        curNode = curNode.Next
    }
    curNode.Next = curNode.Next.Next
    return head
}
```

Courtesy of leetcode user `krex0r`, I also found this interesting solution. It's called the "fast and slow pointers algorithm" apparently, and it's a way to find the middle of the solution in one iteration - probably slightly faster than how we were doing it before. One pointer traverses the array twice as fast as the other, so when the "fast" pointer gets to the end, we know the "slow" pointer is currently at the middle.

```go
func deleteMiddle(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return nil
    }
    var previous, slow, fast *ListNode = nil, head, head

    for slow != nil && fast != nil && fast.Next != nil {
        previous, slow, fast = slow, slow.Next, fast.Next.Next
    }
    previous.Next = slow.Next
    return head
}
```

It seems possible that the memory usage could be barely more (since there are 3 node variables being used, instead of just 2?) but it does sound like it would require less iterations to find our middle node.

Let's do some benchmarking to settle this!

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	  874227	      1363 ns/op	       0 B/op	       0 allocs/op  # My solution
BenchmarkSolutionB-8   	 1277602	       937.9 ns/op	       0 B/op	       0 allocs/op  # fast & slow pointer solution
PASS
ok  	perf_testing	4.418s
```

Sure enough, it looks like it's decently faster with this optimization! For these tests, I generated linked lists with 1000 nodes in them, and the generation of the data shouldn't be included in the actual benchmark data. One interesting thing is how there is zero memory usage or allocations. Perhaps this is because we are only using pointers? Pretty cool.

> Note: to make these functions not mess up the input data, which was getting reused hundreds of thousands of times, I made a small modification to the two algorithms: at the point where they were to remove the middle index, I just made them increment the middle node's value by `0`. So no change happens to the input data, which preserves it for the next run, but the compiler is also kept happy since it thinks all the variables we are declaring are still being used.

## Final Solution, Conclusion

```go
func deleteMiddle(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return nil
    }
    var previous, slow, fast *ListNode = nil, head, head

    for slow != nil && fast != nil && fast.Next != nil {
        previous, slow, fast = slow, slow.Next, fast.Next.Next
    }
    previous.Next = slow.Next
    return head
}
```

Time complexity: `O(n)`

Space complexity: `O(1)`

It turns out the leetcode solution we found is indeed more efficient than my own; it uses a "fast and slow pointer" technique to quickly find the middle node of the linked list, which is the primary problem we need to solve for this code challenge.

このアルゴリズムは、ポインターの二つを使って速く中央ノードを見つけることご出来ます。ポインターの一つ目、`fast`、は二倍の速度で連結リストを反復して、二つ目のポインター、`slow`、は一つずつ反復します。`fast`はその連結の最後のノードに着いたら、`slow`は連結リストの中央ノードに着きます。そのため、二倍と反復しなくて早速その中央ノードの立場を知って削除することが出来ます。

Ben's difficulty rating: 2.8/10. I think this one was conceptually pretty simple, but I hadn't considered the optimization we discovered which lets us find the middle node in one iteration.
