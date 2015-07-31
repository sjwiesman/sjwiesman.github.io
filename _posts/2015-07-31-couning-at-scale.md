---
layout: post
title: Counting at Scale
---

Many simple algorithms and data structures do not scale well in big data environments. 
In this blog post I will explore the problem of counting distinct items, and how it can be solved using the HyperLogLog algorithm.


For several years in college I worked as a teaching assistant for an introductory computer science course. 
One assignment that I would use each semester was to have the students write a program to count the number of distinct words in a text file.  
  
Although most of the students taking this course had only been programming for a semester, almost all would quickly realize that this problem can easily be solved using a set and hurry to complete the assignment; more concerned about misplaced semicolons than their logic being correct.

Most software engineers, if tasked to solve this problem, would go for the same solution. For each item seen they would put it into a set and after all items are inserted they would check the sets size. 
There are many appeals to this approach; sets are provided in virtually all modern standard libraries so it is easy to implement, and it is a mathematically sound way of solving the problem. 
sizeOfString(n) = sizeOfCharArray(n) + \underbrace{4}_\text{char array ref} + \underbrace{3*4}_\text{three int fields} + \underbrace{8}_\text{Java Object header} + \text{padding}
$$ 

$$
sizeOfCharArray(n) = \underbrace{2n}_\text{size of chars} + \underbrace{4}_\text{array length field} + \underbrace{8}_\text{Java Object header} + \text{padding}
$$

From this we can calculate that an empty string requires 40 bytes and a string with 5 characters, the average length of a word in the English language, requires 48 bytes. This means that for at least half of all English words we are using more memory for overhead than to store the actual characters! 
hash(n) = \underbrace{00100101}_\text{k=8}\underbrace{00000001110101}_\text{search area}
$$

