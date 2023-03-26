# Effective C++
55 Specific Ways to Improve Your Programs and Designs


## CHAPTER 1: Accustoming Yourself to C++

### Item 1: View C++ as a federation of languages
* C
* Object-Oriented C++
* Template C++
* The STL

### Item 2: Prefer consts, enums, and inlines to #defines
prefer the *compiler* to the *preprocessor*

* For simple constants, prefer *const* objects or enums to #defines.
* For function-like macros, prefer inline functions to #defines.

> Note: inline is just a suggestion for compiler, whether the funtions are finally inlined depends on compiler, so I'm not sure whether we should use inline nowadays.

### Item 3: Use const whenever possible
Example:
```cpp
char greeting[] = "Hello"; char *p = greeting; 
// non-const pointer, non-const data

const char *p = greeting;
// non-const pointer, const data

char * const p = greeting; 
// const pointer, non-const data

const char * const p = greeting;
// const pointer, const data
```
* If the word *const* appears to the left of the asterisk, what's pointed to is constant
* If the word *const* appears to the right of the asterisk, the pointer itself is constant
* If *const* appears on both sides, both are constant.
