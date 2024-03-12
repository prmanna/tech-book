---
title: "Breadth First Search"
bookCollapseSection: true
weight: 5
---

# Breadth First Search
---
## Intro
Hopefully, by this time, you've drunk enough DFS kool-aid to understand its immense power and seen enough visualization to create a call stack in your mind. Now let me introduce the companion spell Breadth First Search (BFS). The names are self-explanatory. While depth first search reaches for depth (child nodes) first before breadth (nodes in the same level/depth), breadth first search visits all nodes in a level before starting to visit the next level.
While DFS uses **recursion/stack** to keep track of progress, BFS uses a **queue (First In First Out)**. When we dequeue a node, we enqueue its children.
