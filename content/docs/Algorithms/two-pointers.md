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
