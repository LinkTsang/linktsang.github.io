---
layout: post
title: Reading Effective C++ 3rd
date: 2016-07-03 18:09:08
tags: "Notes"
---
最近重温了下Effective C++ 3rd，抄下Items提纲摘录一些内容方便日后检索
<!-- more -->
# Accustoming Yourself to C++
## Item 1: View C++ as a federation of languages

## Item 2: Prefer consts, enums, and inlines to #defines
- For simple constants, prefer const objects or enums to #defines.
    - "the enum hack"
    ```cpp
        class GamePlayer {
        private:
            enum{NumTurns = 5};
            int scores[NumTurns];
        };
    ```
- For function-like macros, prefer inline functions to #defines.

## Item 3: Use const whenever possible.
### STL iterators
    ```
        std::vector<int> vec;
        ...
        const std::vector<int>::iterator iter = vec.begin(); // iter acts like a T* const
        *iter = 10; // OK, changes what iter points to
        ++iter; // error! iter is const
        std::vector<int>::const_iterator cIter = vec.begin(); // cIter acts like a const T*
        *cIter = 10; // error! *cIter is const
        ++cIter; // fine, changes cIter
    ```


- Declaring something const helps compilers detect usage errors. const can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
- Compilers enforce bitwise constness, but you should program using logical constness.
  - mutable
- When const and non-const member functions have essentially identical implementations, code duplication can be avoided by having the non-const version call the const version.
  ```CPP
    class TextBlock {
    public:
    ...
    const char& operator[](std::size_t position) const // same as before
    {
        ...
        ...
        ...
        return text[position];
    }
    char& operator[](std::size_t position) // now just calls const op[]
    {
        return
            const_cast<char&>(static_cast<const TextBlock&>(*this)[position]);
    }
    ...
    };
  ```
## Items 4 Make sure that objects are initialized before they're used
- Manually initialize objects of built-in type, because C++ only sometimes initializes them itself.
- In a constructor, prefer use of the member initialization list to assignment inside the body of the constructor. List data members in the initialization list in **the same order they’re declared in the class**.