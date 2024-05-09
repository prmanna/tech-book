---
title: "C++ Tips"
bookCollapseSection: true
weight: 5
---

# C++ Tips
---

## Templates

A template is a simple yet very powerful tool in C++. The simple idea is to pass the data type as a parameter so that we don’t need to write the same code for different data types. For example, a software company may need to sort() for different data types. Rather than writing and maintaining multiple codes, we can write one sort() and pass the datatype as a parameter.

C++ adds two new keywords to support templates: ‘template’ and ‘typename’. The second keyword can always be replaced by the keyword ‘class’.

### Function Templates
We write a generic function that can be used for different data types.

```cpp
#include <iostream>
using namespace std;

template <typename T>
T myFunc (T x, T y)
{
    return (x>y)? x:y;
}
int main ()
{
    cout << myFunc<int>(3,7) << endl;
    std::cout << myFunc<char>('g', 'e') << std::endl;

    return 0;
}
```
### Class Templates
Class templates like function templates, class templates are useful when a class defines something that is independent of the data type. Can be useful for classes like LinkedList, BinaryTree, Stack, Queue, Array, etc.

```cpp
// C++ Program to implement
// template Array class
#include <iostream>
using namespace std;

template <typename T> class Array {
private:
    T* ptr;
    int size;

public:
    Array(T arr[], int s);
    void print();
};

template <typename T> Array<T>::Array(T arr[], int s)
{
    ptr = new T[s];
    size = s;
    for (int i = 0; i < size; i++)
        ptr[i] = arr[i];
}

template <typename T> void Array<T>::print()
{
    for (int i = 0; i < size; i++)
        cout << " " << *(ptr + i);
    cout << endl;
}

int main()
{
    int arr[5] = { 1, 2, 3, 4, 5 };
    Array<int> a(arr, 5);
    a.print();
    return 0;
}
```

Reference: [Templates in C++ with Examples](https://www.geeksforgeeks.org/templates-cpp/)

## Array

An array in C++ holds a fixed number of elements of a single type in contiguous memory locations.

```cpp
std::string clothes[3];     // declare a string array of size 3 named clothes
int numbers[5];       // declare an int array of size 5 named numbers
```

By default, the values in C++ arrays are undetermined at the time of declaration (meaning that the values can be random). An array can be initialized to specific values by enclosing the values in curly braces. The number of values in `{}` should not exceed the size of the array. If the number of values inside the braces is less than the size of the array, the rest of the array will be set to the default value for their type, such as `0` for `int` and `false` for `bool`. If an array is initialized to empty curly braces, all elements in the array is set to their default value.

```cpp
int a[3] = { 3, 4, 5 };   // int array a = [3, 4, 5]
int b[5] = { 1 };         // int array b = [1, 0, 0, 0, 0]
int c[2] = { };           // int array c = [0, 0]
```

An element in an array is accessed by its index in `O(1)` time with square brackets.

```cpp
std::string clothes[5];   // declare an array of size 5
clothes[0] = "shirt";     // initialize first element
clothes[1] = "dress";     // initialize second element
std::cout << clothes[0];  // print out "shirt"
```

C++ arrays do not have a `length` attribute that lets you access its size easily. The common way to get the size of an array is by:

```cpp
int arr[5] = { 7, 3, 4, 6, 8 }
std::cout << sizeof(arr)/sizeof(arr[0]) << '\n';    // print out 5
```

Since C++17, we can also retrieve the size of an array with `std::size()`.

```cpp
int arr[] = { 6, 7, 8 }
std::cout << std::size(arr) << '\n';    // print out 3
```

## Vector

When you need to create a dynamically resizable array, C++ provides a useful sequence container `vector`. Like arrays, it provides efficient constant time element access.
```cpp
vector<data_type> vectorName;
```
**Commonly used Vector Functions**
+ push_back() – It is used to insert the elements at the end of the vector.
+ pop_back() – It is used to pop or remove elements from the end of the vector.
+ clear() – It is used to remove all the elements of the vector.
+ empty() – It is used to check if the vector is empty.
+ at(i) – It is used to access the element at the specified index ‘i’.
+ front() – It is used to access the first element of the vector.
+ back() – It is used to access the last element of the vector.
+ erase() – It is used to remove an element at a specified position.
+ size()

- `vector_name[i]` gets or sets item stored at index `i`, `O(1)`
- `emplace_back(item)` adds item to the end of the vector, [amortized](https://stackoverflow.com/questions/200384/constant-amortized-time) `O(1)`
- `size()` returns the size of the vector, `O(1)`

The `emplace_back()` operation, which appends a new element to the end of the vector, runs in amortized constant time. A vector will resize itself when needed. A typical resizing implementation is that when the array is full, the array doubles in size, and the old content is copied over to the newer, larger array. The doubling takes `O(n)` time, but it happens rarely that the overall insertion still takes `O(1)` time.

In general, to iterate through an array, a `for` loop is easier to reason than `while` since there's no condition to manage that could skip elements. In C++, there are two types of for loops: the simple `for` loop and the `for-each` loop. Use `for-each` loop if you don't need to access the index since it avoids the possible error of messsing up the index.

```cpp
std::vector<int> numbers{20, 6, 13, 5};
// simple for loop goes through indices so we fetch elements using indices
for (int i = 0; i < numbers.size(); i++) {
    int number = numbers[i];
    std::cout << number << std::endl;
}

// for-each loop directly fetches elements
for (int number : numbers) {
    std::cout << number << std::endl;
}
```
## Linked List

C++ doesn't have a built-in singly linked list. Normally at an interview, you'll be given a definition like this:

```cpp
struct LinkedListNode {
  int val;
  LinkedListNode* next;
}
```

- append to end is `O(1)`
- finding an element is `O(N)`

Besides problems that specifically ask for linked lists, you don't normally define and use linked list. If you need a list with `O(1)` append you can use a `vector`, and if you want `O(1)` for both prepend and append you can use `deque`.

## Stack

C++ provides a container adaptor `std::stack` that works on the basis of LIFO (last-in-first-out).

- push: `push(item)` adds item to end of stack, `O(1)`
- pop: `pop()` removes item at the end of stack, `O(1)`
- size: `size()`, `O(1)`
- top: `top()` returns but doesn't remove item at the end of stack, `O(1)`

Stack is arguably the most magical data structure in computer science. In fact, with [two stacks and a finite-state machine](https://algomonster.s3.us-east-2.amazonaws.com/documents/A+Turing+Machine+Is+Just+a+Finite+Automaton+with+Two+Stacks+A+Co.pdf) you can build a Turing Machine that can solve any problem a computer can solve.

Recursion and function calls are implemented with stack behind the scene. We'll discuss this in the [recursion review](https://algo.monster/problems/recursion_intro) module.

## Queue

We normally use `std::deque` when we need a queue.

- enqueue: `push_back(item)` inserts item at the end of the queue, `O(1)`
- dequeue: `pop_front()` removes and return item from the head of the queue, `O(1)`
- size: `size()`, `O(1)`
- peek: `front()` returns but don't remove item at the head of the queue, `O(1)`

In coding interviews, we see queues most often in breadth-first search. We'll also cover monotonic deque where elements are sorted inside the deque that is useful in solving some advanced coding problems.

## Hash Table

`std::unordered_map` in C++ implements hash table.

- `at(key)` gets item mapped to key if present, `O(1)`
- `map[key]` gets item mapped to key if present (returns `0` if not present), and inserts the key if not, `O(N)`
- `map[key] = item` sets item mapped to key, `O(1)`
- `count(key)` finds out if key is present or not, `O(1)`
- `erase(key)` removes item mapped to key if present, `O(1)`

It's worth mentioning that these are average case time complexity. A hash table's worst time complexity is actually `O(N)` due to [hash collision and other things](https://en.wikipedia.org/wiki/Hash_table#Collision_resolution). For the vast majority of the cases and certainly most coding interviews, the assumption of constant time lookup/insert/delete is valid.

Use a hash table if you want to create a mapping from A to B. Many starter interview problems can be solved with hash tables.

## Hash Set

`std::unordered_set` in C++ is useful in answering existence queries in constant time.

- `count(item)` checks if item is in a set, `O(1)`
- `insert(item)` adds item to a set if it's not already present, `O(1)`
- `erase(item)` removes item from a set if it's present, `O(1)`
- `size()` gets the size of the set, `O(1)`

Hash set is useful when you only need to know existence of a key. Example use cases include DFS and BFS on graphs.

## Tree

Normally at an interview, you'd be given the following implementation for a binary tree:

```cpp
struct Node {
    int val;
    Node* left;
    Node* right;

    Node(T val, Node* left = nullptr, Node* right = nullptr)
        : val{val}, left{left}, right{right} {}
};
```

For n-nary trees:

```cpp
#include <vector>

struct Node {
    int val;
    std::vector<Node*> children;

    Node(int val, std::vector<Node*> children = {})
        : val{val}, children{children} {}
};
```
## Infinity

Infinity is useful when you want to initialize a variable that is greater or smaller than any value that your algorithm may want to compare with. `std::numeric_limits<T>` provides the maximum and minimum values of fundamental arithmatic types in C++.

```cpp
#include <limits>

int max = std::numeric_limits<int>::max();    // 2147483648
int min = std::numeric_limits<int>::min();    // -2147483648
```

