---
title: "Priority Queue and Heap"
bookCollapseSection: true
weight: 5
---

# Easy Complexity
---
Priority Queue is an Abstract Data Type, and Heap is the concrete data structure we use to implement a priority queue.

## Priority Queue

A priority queue is a data structure that consists of a collection of items and supports the following operations:
- insert: insert an item with a key.
- delete\_min/delete\_max: remove the item with the smallest/largest key and return it.

Note that
- we only allow getting and deleting the element with the min/max key and NOT any arbitrary key.

## Implement Priority Queue using an array

To do this, we could try using

- an unsorted array, insert would be `O(1)` as we just have to put it at the end, but finding and deleting min value would be `O(N)` since we have to loop through the entire array to find it
- a sorted array, finding min value would be easy `O(1)`, but it would be `O(N)` to insert since we have to loop through to find the correct position of the value and move elements after the position to make space and insert into the space

There must be a better way! "Patience you must have, my young padawan." Enter Heap.

## Heap

Heaps are special tree based data structures. Usually when we say heap, we refer to the _binary_ heap that uses the binary tree structure. However, the tree isn't necessarily always binary, in particular, a `k`\-ary heap (A.K.A. `k`\-heap) is a tree where the nodes have `k` children. As long as the nodes follow the 2 heap properties, it is a valid heap.

## Max Heap and Min Heap

There are two kinds of heaps - Min Heap and Max Heap. A Min Heap is a tree that has _two properties_:

1. almost complete, i.e. every level is filled except possibly the last(deepest) level. The filled items in the last level are left-justified.
2. for any node, its key (priority) is greater than its parent's key (Min Heap).

A Max Heap has the same property #1 and opposite property #2, i.e. for any node, its key is less than its parent's key.

## Operations

### insert

To insert a key into a heap,

- place the new key at the first free leaf
- if property #2 is violated, perform a _bubble-up_

```python
def bubble_up(node):
    while node.parent exist and node.parent.key > node.key:
        swap node and node.parent
        node = node.parent
```

As the name of the algorithm suggests, it "bubbles up" the new node by swapping it with its parent until the order is correct

Since the height of a heap is `O(log(N))`, the complexity of bubble-up is `O(log(N))`.

### delete\_min

What this operation does is:

- delete a node with min key and return it
- reorganize the heap so the two properties still hold

To do that, we:

- remove and return the root since the node with the minimum key is always at the root
- replace the root with the last node (the rightmost node at the bottom) of the heap
- if property #2 is violated, perform a _bubble-down_

```python
def bubble_down(node):
    while node is not a leaf:
        smallest_child = child of node with smallest key
        if smallest_child < node:
            swap node and smallest_child
            node = smallest_child
        else:
            break
```

## Implementing Heap

Being a complete tree makes an array a natural choice to implement a heap since it can be stored compactly and no space is wasted. Pointers are not needed. The parent and children of each node can be calculated with index arithmetic

![](https://algomonster.s3.us-east-2.amazonaws.com/heap_intro/heap_intro_5.png)

For node `i`, its children are stored at `2i+1` and `2i+2`, and its parent is at `floor((i-1)/2)`. So instead of `node.left` we'd do `2*i+1`. (Note that if we are implementing a `k`\-ary heap, then the childrens are at `ki+1` to `ki+k`, and its parent is at `floor((i-1)/k)`.)


