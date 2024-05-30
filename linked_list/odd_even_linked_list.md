# Odd Even Linked List

[leetcode link](https://leetcode.com/problems/odd-even-linked-list/)

5/29/2024

Difficulty: Medium

## Description

Given the `head` of a singly linked list, group all the nodes with odd indices together followed by the nodes with even indices, and return the reordered list.

the first node is considered odd, and the second node is even, and so on.

Note that the relative order inside both the even and odd groups should remain as it was in the input.

You must solve the problem in `O(1)` extra space complexity and `O(n)` time complexity.

**Example 1:**

![ex1](https://assets.leetcode.com/uploads/2021/03/10/oddeven-linked-list.jpg)

```
Input: head = [1,2,3,4,5]
Output: [1,3,5,2,4]
```

**Example 2:**

![ex2](https://assets.leetcode.com/uploads/2021/03/10/oddeven2-linked-list.jpg)

```
Input: head = [2,1,3,5,6,4,7]
Output: [2,3,6,7,1,5,4]
```

**Constraints:**

-   The number of nodes in the linked list is in the range `[0, 10^4]`.
-   `-10^6 <= Node.val <= 10^6`

### 日本語の説明

連結リストの先頭ノード`head`が与えられます。ノードのインデックスが奇数のものを先に、偶数のものを後にまとめて、並べ替えた連結リストを返してください。

最初のノードは奇数で、次のノードは偶数で、そのように順番が繰り返されます。

奇数のグループと偶数グループの中の相対的な順番は元の入力の順番と同じにしてください。

この問題には`O(1)`の空間計算量と`O(n)`の時間計算量の解決を使ってください。

## Initial Thoughts / Planning

Alright, so I think there are two things we need to do here:

-   Get all the nodes divided into two groups, by their index (the position they are found in the input linked list)
-   Put the two groups together, with the "odd" group going before the "even" group.

To do this, I think we might want to build up two linked lists: `odd` and `even` (these two represent the head node). As we traverse the input linked list, we build up `odd` and `even`, alternating between adding to those two lists. At the end, we then connect `even` to the last node in the `odd` linked list, and then we return `odd`.

But, **is this solution `O(1)` space?** Well, I'm actually not 100% sure, but I believe it is - hear me out: we are creating a constant amount of extra space, `odd` and `even`. Then, as we build these linked lists, we are simply setting pointers to existing nodes from the input linked list. If we did something different, like create two arrays and added the odd and even elements to them, then I think that would be `O(n)`, but due to how we are using linked lists that consist of pointer nodes, I believe we are just re-using the same space.

So, for now, I'm going to say **yes**, this solution concept here is constant space. And as for time complexity, it's linear time as well, since we are iterating through the linked list (size `n`) a constant number of times.

## My Code Implementations, Optimizations

Ok, so I finally got the solution to work!

```go
func oddEvenList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
	node := head
	var oddNode, evenNode, odd, even *ListNode
	i := 0
	for node != nil {
		if i%2 == 0 {
			if odd == nil {
				oddNode = node
				odd = oddNode
			} else {
				oddNode.Next = node
				oddNode = oddNode.Next
			}
		} else {
			if even == nil {
				evenNode = node
				even = evenNode
			} else {
				evenNode.Next = node
				evenNode = evenNode.Next
			}
		}
		i++
		node = node.Next
	}
	oddNode.Next = even
	evenNode.Next = nil
	return odd
}
```

I don't care to admit how long I've been messing with this solution; trying to get all the pointers to work correctly, trying to figure out what's causing endless loops, etc.

It turned out I was causing an endless loop for a while because the last node of the result linked list actually still had a `Next` value to an earlier node! So beware of that. At the end, make sure to set the last node in your solution to have a nil `Next`.

**However**, one thing I'm not entirely sure about is if this is **constant extra space** or not. We do technically maintain two extra linked lists to make this solution. However, my main defense is that the nodes in the "helper" linked lists are just pointers to the original nodes (of type `*ListNode`, not `ListNode`), so if I'm not mistaken, that implies there is no extra memory usage going on when adding these pointer nodes to the helper linked lists.

Chat-GPT seems unsure about if it's using constant extra memory or not. At first it said it isn't, but when I argued back and pointed out how the `odd` and `even` linked lists all use pointers to the original linked list to reuse the same memory, it suddenly changed its mind:

> Your original function does indeed reuse the original nodes and doesn't allocate additional memory for new nodes. Therefore, it should technically use constant extra memory.
>
> Upon closer inspection, it seems that the issue might not be related to the memory usage of your function per se, but rather to how it's being evaluated within the LeetCode platform. Sometimes, platforms like LeetCode may have specific constraints or requirements that aren't immediately obvious, and the error message about exceeding memory usage limits could be due to how the function is being assessed within their environment.

Ok, well I wonder if there's another solution that doesn't require maintaining extra helper linked lists?

> The next day...

I was a little busy and had to let this one sit for a day. But when I came back, I think I realized a solution that would be `O(1)` memory usage, and it sounds pretty simple now that I think about it:

-   first, find the `tail` node. we'll need to iterate over the linked list until we get to the end, then keep track of this node. Also, record the **length** of the list.
-   then, iterate over the linked list again:
    -   on `odd` nodes, do nothing; leave the node where it is
    -   on `even` nodes, pop the node out and append it to the end of the list. Re-link the two "odd" nodes that were surrounding the "even" node to each other.
    -   repeat this process until the end of the linked list - but only iterating over the **original** linked list size

If I'm not mistaken, at this point we should have all the nodes separated into their two groups, and also in the same relative order as they were in originally; and only using constant space and linear time.

```go
func oddEvenList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
		return head
	}
	// first, find the tail node, and also count the size of the linked list
	n := 1
	node := head
	for node.Next != nil {
		node = node.Next
		n++
	}
	tail := node
	tail.Next = nil

	// now, move all even nodes to the end
	// we check the next node, not the current one, so we can handle
	// re-linking nodes together
	node = head
	for i := 2; i <= n; i++ {
		if i%2 == 0 {
			tail.Next = node.Next
			tail = tail.Next
			node.Next = node.Next.Next
			i++
		}
		node = node.Next
	}
	tail.Next = nil
	return head
}
```

We've got another working solution here too. I do think this solution seems a little more light-weight, as it reuses the original linked list and modifies it in-line. But interestingly, the Leetcode performance metrics suggest that it's not faster, and might have the same memory usage. I think this calls for some benchmarking!

```shell
$ go test -bench=.
goos: darwin
goarch: arm64
pkg: perf_testing
BenchmarkSolutionA-8   	  848509	      1407 ns/op	       0 B/op	       0 allocs/op  # first solution
BenchmarkSolutionB-8   	  516871	      2324 ns/op	       0 B/op	       0 allocs/op  # second solution
PASS
ok  	perf_testing	3.431s
```

Well that's interesting: the second solution here appears to be slower! What's also interesting is both solutions apparently don't make any allocations or use extra memory.

> I was a bit confused by this, so I did some research: apparently these two memory-related metrics only track memory allocations to the **heap** - excluding any memory allocations happening on the stack level, such as local variables scoped within a function. So that explains that.

Anyway, I think it's fair to say that neither of these solutions use much extra memory, but also that our first solution is decidedly more performant.
Perhaps the first solution is faster since it only does one iteration through the linked list? Also, during the loop, there is less going on in terms of manipulating list nodes, moving them around, changing pointers, etc. I don't think those operations are particularly expensive though.

## Other Leetcode Solutions

I skimmed over several solutions, and it seems my two solutions here cover the different approaches people are using. The first approach I used is the more popular one, I think.

## Final Solution, Conclusion

My first solution:

```go
func oddEvenList(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
	node := head
	var oddNode, evenNode, odd, even *ListNode
	i := 0
	for node != nil {
		if i%2 == 0 {
			if odd == nil {
				oddNode = node
				odd = oddNode
			} else {
				oddNode.Next = node
				oddNode = oddNode.Next
			}
		} else {
			if even == nil {
				evenNode = node
				even = evenNode
			} else {
				evenNode.Next = node
				evenNode = evenNode.Next
			}
		}
		i++
		node = node.Next
	}
	oddNode.Next = even
	evenNode.Next = nil
	return odd
}
```

Time complexity: `O(n)`

Space complexity: `O(1)`

In this solution, we maintain two extra linked lists, `odd` and `even`, but due to the fact that these linked lists only contain pointers to the already existing list nodes, it is still considered using only constant extra space: the 4 pointers we create.

This solution builds up these two lists, containing the odd and even nodes respectively, and then puts them together and returns them at the end.

この解決方法は二つの連結リスト、`odd`と`even`を作り、最後に`odd`を先に、`even`を後に結びつける返します。`odd`と`even`は追加の連結リストなので空間計算量は`O(n)`と見えるかもしれませんが、その追加の連結リストのノードが全部ポインターですから、実は`O(1)`になると思います。

Ben's difficulty rating: 6.8/10. This one took me a while to think of a solution to, and even when it came time to code it up, I kept running into problems with infinite loops and pointer problems. In general I think linked lists are difficult for me, mainly due to the pointer management.
