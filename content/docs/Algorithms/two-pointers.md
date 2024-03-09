---
title: "Two Pointers"
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
