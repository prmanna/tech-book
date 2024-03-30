---
title: "C++ Tips"
bookCollapseSection: true
weight: 5
---

# Easy Complexity
---

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
