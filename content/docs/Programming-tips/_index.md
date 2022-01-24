---
title: "Programming Tips"
bookFlatSection: false
bookCollapseSection: true
draft: false
---

# Programming Tips
---

#### Bit Manipulation

* XORing a bit with 1 always flips the bit, whereas XO Ring with O will never change it.

#### Miscellaneous

* Passing a 2D array to a C++ function

  There are three ways to pass a 2D array to a function:

  1. The parameter is a 2D array

     ```c++
     int array[10][10];
     void passFunc(int a[][10])
     {
         // ...
     }
     passFunc(array);
     ```

  2. The parameter is an array containing pointers

     ```c++
     int *array[10];
     for(int i = 0; i < 10; i++)
         array[i] = new int[10];
     
     void passFunc(int *a[10]) //Array containing pointers
     {
         // ...
     }
     passFunc(array);
     ```

  3. The parameter is a pointer to a pointer

     ```c++
     int **array;
     array = new int *[10];
     for(int i = 0; i <10; i++)
         array[i] = new int[10];
     void passFunc(int **a)
     {
         // ...
     }
     passFunc(array);
     ```

     

