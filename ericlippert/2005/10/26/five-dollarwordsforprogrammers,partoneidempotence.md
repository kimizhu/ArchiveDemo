# Five-Dollar Words for Programmers, Part One: Idempotence

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 10/26/2005 10:00:00 AM

-----

Programmers, particularly those with a mathematical background, often use words from mathematics when describing their systems. Unfortunately, they also often do so without consideration of the non-mathematical background of their colleagues. I thought I might talk today a bit about the word "idempotent". This is a very easy concept but lots of people don't know that there is a word for it. There are two closely related definitions for "idempotent. A **value** is "idempotent under function foo" if the result of doing foo to the value results in the value right back again. A **function** is "idempotent" if the result of doing it twice (feeding the output of the first call into the second call) is exactly the same as the result of doing it once. (Or, in other words, every output of the function is idempotent under it.) The most trivial mathematical example of the second kind is the constant function.  If f(x) = c, then clearly f(x) = f(f(x)) = f(f(f(x))) ... so f is idempotent (and the constant is idempotent under it). The identity function f(x) = x is also idempotent (and every value is idempotent under it). The function that takes a set of numbers and returns a set containing its largest element is idempotent (and every one-element set is idempotent under it). I'm sure you can think of lots of examples of idempotent functions. To get a handle on the other sense, pick an operation -- say, doubling.  The only value which is idempotent under that operation is zero. The operation "subtracting any non-zero value" has no idempotent values. Squaring has two idempotent values, zero and one. The second characterization of this concept comes up all the time in practical programming, particularly around caching logic. Usually when used in the computer science sense we don't mean that the effect of the function is invariant under composition, but rather that it is invariant over the number of calls. For example, I don't know how many times I've written: HRESULT GetTypeLibCreator(ICreateTypeLib2 \*\* ppctl) {  
  if (this-\>m\_pctl == NULL) {  
    HRESULT hr;  
    hr = CreateTypeLib2(SYS\_WIN32, pszName, \&this-\>m\_pctl);  
    if (FAILED(hr)) return hr;  
  }  
  \*ppctl = this-\>m\_pctl;  
  this-\>m\_pctl-\>AddRef();  
  return S\_OK;  
} A nice little idempotent function -- calling it two, three, n times has exactly the same result as calling it once. The place you see the other characterization of idempotence all the time is in C++ header files. Include a needed header zero times and you'll get "not defined" errors. Accidentally include it twice and you'll get "redefinition" errors. It's a major pain to make sure that every header file is included exactly once. Therefore, most headers use  some trick to make them idempotent under the inclusion operation: \#ifndef STUFF\_H\_INCLUDED  
\#define STUFF\_H\_INCLUDED // headers here \#endif // STUFF\_H\_INCLUDED or in more modern systems, the \#pragma once directive makes headers idempotent under inclusion.

