---
title: "Two Pointers & Sliding Window"
bookCollapseSection: true
weight: 5
---

# Two Pointers
---

## Valid Palindrome
Determine whether a string is a palindrome, ignoring non-alphanumeric characters and case. Examples:

Input: Do geese see God? Output: True

Input: Was it a car or a cat I saw? Output: True

Input: A brown fox jumping over Output: False


```c++
    #include <cctype> // isalnum, tolower
    #include <iostream> // boolalpha, cin, cout
    #include <string> // getline

    bool is_palindrome(std::string s) {
        int l = 0, r = s.size() - 1;
        while (l < r) {
            // Note 1, 2
            while (l < r && !std::isalnum(s[l])) {
                l++;
            }
            while (l < r && !std::isalnum(s[r])) {
                r--;
            }
            // compare characters ignoring case
            if (std::tolower(s[l]) != std::tolower(s[r])) return false;
            l++;
            r--;
        }
        return true;
    }

    int main() {
        std::string s;
        std::getline(std::cin, s);
        bool res = is_palindrome(s);
        std::cout << std::boolalpha << res << '\n';
    }
```
## Remove Duplicates
Given a sorted list of numbers, remove duplicates and return the new length. You must do this in-place and without using extra memory.

Input: [0, 0, 1, 1, 1, 2, 2].

Output: 3.

Your function should modify the list in place so the first 3 elements becomes 0, 1, 2. Return 3 because the new length is 3.

```c++
    #include <algorithm> // copy
    #include <iostream> // boolalpha, cin, cout
    #include <iterator> // back_inserter, istream_iterator, ostream_iterator, prev
    #include <sstream> // istringstream
    #include <string> // getline, string
    #include <vector> // vector

    int remove_duplicates(std::vector<int>& arr) {
        int slow = 0;
        for (int fast = 0; fast < arr.size(); fast++) {
            if (arr.at(fast) != arr.at(slow)) {
                slow++;
                arr.at(slow) = arr.at(fast);
            }
        }
        return slow + 1;
    }

    template<typename T>
    std::vector<T> get_words() {
        std::string line;
        std::getline(std::cin, line);
        std::istringstream ss{line};
        ss >> std::boolalpha;
        std::vector<T> v;
        std::copy(std::istream_iterator<T>{ss}, std::istream_iterator<T>{}, std::back_inserter(v));
        return v;
    }

    template<typename T>
    void put_words(const std::vector<T>& v) {
        if (!v.empty()) {
            std::copy(v.begin(), std::prev(v.end()), std::ostream_iterator<T>{std::cout, " "});
            std::cout << v.back();
        }
        std::cout << '\n';
    }

    int main() {
        std::vector<int> arr = get_words<int>();
        int res = remove_duplicates(arr);
        arr.resize(res);
        put_words(arr);
    }
```

# Sliding Window
---

Sliding window problems is a variant of the same direction two pointers problems. The function performs on the entire interval between the two pointers instead of only at the two positions. Usually, we keep track of the overall result of the window, and when we "slide" the window (insert/remove an item), we simply manipulate the result to accomodate the changes to the window. Time complexity wise, this is much more efficient as we do not recalculate the overlapping intervals between two windows over and over again. We try to reduce a nested loop into two passes on the input (one pass with each pointer).

## Fixed Size Sliding Window
Given an array (list) nums consisted of only non-negative integers, find the largest sum among all subarrays of length k in nums.
For example, if the input is nums = [1, 2, 3, 7, 4, 1], k = 3, then the output would be 14 as the largest length 3 subarray sum is given by [3, 7, 4] which sums to 14.

```c++
    #include <algorithm> // copy
    #include <iostream> // boolalpha, cin, cout, streamsize
    #include <iterator> // back_inserter, istream_iterator
    #include <limits> // numeric_limits
    #include <sstream> // istringstream
    #include <string> // getline, string
    #include <vector> // vector

    int subarray_sum_fixed(std::vector<int> nums, int k) {
        int window_sum = 0;
        for (int i = 0; i < k; ++i) {
          window_sum = window_sum + nums[i];
        }
        int largest = window_sum;
        for (int right = k; right < nums.size(); ++right) {
            int left = right - k;
            window_sum = window_sum - nums[left];
            window_sum = window_sum + nums[right];
            largest = std::max(largest, window_sum);
        
        }
        return largest;
    }

    template<typename T>
    std::vector<T> get_words() {
        std::string line;
        std::getline(std::cin, line);
        std::istringstream ss{line};
        ss >> std::boolalpha;
        std::vector<T> v;
        std::copy(std::istream_iterator<T>{ss}, std::istream_iterator<T>{}, std::back_inserter(v));
        return v;
    }

    void ignore_line() {
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    }

    int main() {
        std::vector<int> nums = get_words<int>();
        int k;
        std::cin >> k;
        ignore_line();
        int res = subarray_sum_fixed(nums, k);
        std::cout << res << '\n';
    }

```

## Flexible Size Sliding Window - Longest
Recall finding the largest size k subarray sum of an integer array in Largest Subarray Sum. What if we dont need the largest sum among all subarrays of fixed size k, but instead, we want to find the length of the longest subarray with sum smaller than or equal to a target?

Given input nums = [1, 6, 3, 1, 2, 4, 5] and target = 10, then the longest subarray that does not exceed 10 is [3, 1, 2, 4], so the output is 4 (length of [3, 1, 2, 4]).

```c++
    #include <algorithm> // copy
    #include <iostream> // boolalpha, cin, cout, streamsize
    #include <iterator> // back_inserter, istream_iterator
    #include <limits> // numeric_limits
    #include <sstream> // istringstream
    #include <string> // getline, string
    #include <vector> // vector

    int subarray_sum_longest(std::vector<int> nums, int target) {
        int windowSum = 0, length = 0;
        int left = 0;
        for (int right = 0; right < nums.size(); ++right) {
            windowSum = windowSum + nums[right];
            while (windowSum > target) {
                windowSum = windowSum - nums[left];
                ++left;
            }
            length = std::max(length, right-left+1);
        }
        return length;
    }

    template<typename T>
    std::vector<T> get_words() {
        std::string line;
        std::getline(std::cin, line);
        std::istringstream ss{line};
        ss >> std::boolalpha;
        std::vector<T> v;
        std::copy(std::istream_iterator<T>{ss}, std::istream_iterator<T>{}, std::back_inserter(v));
        return v;
    }

    void ignore_line() {
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    }

    int main() {
        std::vector<int> nums = get_words<int>();
        int target;
        std::cin >> target;
        ignore_line();
        int res = subarray_sum_longest(nums, target);
        std::cout << res << '\n';
    }
```

## Flexible Size Sliding Window - Shortest
Let's continue on finding the sum of subarrays. This time given a positive integer array nums, we want to find the length of the shortest subarray such that the subarray sum is at least target. Recall the same example with input nums = [1, 4, 1, 7, 3, 0, 2, 5] and target = 10, then the smallest window with the sum >= 10 is [7, 3] with length 2. So the output is 2.

We'll assume for this problem that it's guaranteed target will not exceed the sum of all elements in nums.


```c++
    #include <algorithm> // copy
    #include <iostream> // boolalpha, cin, cout, streamsize
    #include <iterator> // back_inserter, istream_iterator
    #include <limits> // numeric_limits
    #include <sstream> // istringstream
    #include <string> // getline, string
    #include <vector> // vector

    int subarray_sum_shortest(std::vector<int> nums, int target) {
        int windowSum = 0, length = nums.size();
        int left = 0;
        for (int right = 0; right < nums.size(); ++right) {
            windowSum = windowSum + nums[right];
            while (windowSum >= target) {
                length = std::min(length, right-left+1);
                windowSum = windowSum - nums[left];
                ++left;
            }
        }
        return length;
    }

    template<typename T>
    std::vector<T> get_words() {
        std::string line;
        std::getline(std::cin, line);
        std::istringstream ss{line};
        ss >> std::boolalpha;
        std::vector<T> v;
        std::copy(std::istream_iterator<T>{ss}, std::istream_iterator<T>{}, std::back_inserter(v));
        return v;
    }

    void ignore_line() {
        std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
    }

    int main() {
        std::vector<int> nums = get_words<int>();
        int target;
        std::cin >> target;
        ignore_line();
        int res = subarray_sum_shortest(nums, target);
        std::cout << res << '\n';
    }
```
## Linked List Cycle
Given a linked list with potentially a loop, determine whether the linked list from the first node contains a cycle in it. For bonus points, do this with constant space.

```c++
    bool has_cycle(Node<int>* nodes) {
        Node<int>* tortoise = next_node(nodes);
        Node<int>* hare = next_node(next_node(nodes));
        while (tortoise != hare && hare->next != NULL) {
            tortoise = next_node(tortoise);
            hare = next_node(next_node(hare));
        }
        return hare->next != NULL;
    }
```
