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
There are many appeals to this approach; sets are provided in virtually all modern standard libraries so it is easy to implement, and it is a mathematically sound way of solving the problem. But let’s pause for a moment and look at how much memory we are using. Because most big data applications live on the JVM we will use that as our platform. Java Strings contain multiple fields and so the size of a string with n characters can be calculated using the following equations. $$
sizeOfString(n) = sizeOfCharArray(n) + \underbrace{4}_\text{char array ref} + \underbrace{3*4}_\text{three int fields} + \underbrace{8}_\text{Java Object header} + \text{padding}
$$ 

$$
sizeOfCharArray(n) = \underbrace{2n}_\text{size of chars} + \underbrace{4}_\text{array length field} + \underbrace{8}_\text{Java Object header} + \text{padding}
$$

$$
_\text{The JVM allocates memory in 8 byte increments so padding must be added to make each object size is a multiple of 8}
$$

From this we can calculate that an empty string requires 40 bytes and a string with 5 characters, the average length of a word in the English language, requires 48 bytes. This means that for at least half of all English words we are using more memory for overhead than to store the actual characters! In most situations memory is not an issue and the above algorithm is sufficient. However, when your dataset becomes large enough that even the number of distinct items can not longer fit on one or even several machines this becomes unacceptable. To reduce memory usage, we can heavily optimize an algorithm towards doing one thing. This problem never called for us to determine what strings (or other objects) we have seen or retrieve them in the future. All we want to do is determine the number of distinct items encountered. HyperLogLog to the rescueHyperLogLog, instead of storing each item, makes use of the random uniform distribution of a hash function in order to estimate the number of items seen given a specific phenomenon.  If that made sense to you on first pass, then you are a lot smarter than I am and can stop reading. If not, we will break it apart to understand how HyperLogLog works. Hash functions are generally understood as a transformation of an object from a larger space to a smaller space. When most people hash an object they expect it to return a random looking number which only has a very small chance of being the same as the hash of another object of the same type. Instead, lets view each hash as a uniformly distributed bit stream. Put another way, each hash is a list of 1’s and 0’s where the probability of each bit being a specific value is independent of the value of every other bit. As stated above, we are looking for a specific phenomenon; the number of consecutive 0’s at the beginning of the stream. Because each bit is independent of every other the odds that the first bit is 0 is $$\frac{1}{2}$$, the odds that the nth bit is 0 is $$\frac{1}{2}$$, and the odds that the first consecutive n bits are 0 is $$\frac{1}{2}^n$$. To be clear, we are interested in the probability of different patterns not specific numbers. The odds of a function producing the hashes 00101101 and 00010110 are the same but the probability of 00xxxxx (two zeros followed by some numbers) is twice as likely as 000xxxx (three zeros followed by some numbers).  This means that if the largest number of consecutive 0’s seen is 32 we know that the odds of that happening are 1/4294967296 and we can expect that we have seen 4,294,967,296 distinct items. If we were to stop here, then we would have the algorithm LogLog. LogLog provides a very a very small memory footprint. The number 32 is the same as the binary number 100000b which is only 6 bits and with those 6 bits we can count up to  items. LogLog’s name comes from the fact that it only requires Log(Log(n)) bits to count up to n. The problem with LogLog is that it is incredibly susceptible to outliers. If the first item seen hashes to 000000000001 then LogLog will tell us it has seen 2048 distinct items when it has only seen one. HyperLogLog was born out of LogLog as a way to minimize the influence of outliers in a data set. To increase accuracy HyperLogLog maintains a group of buckets. For each item it sees it will calculate its hash and take the first k bits of the hash to use that to pick a bucket. It will then count the number of 0-bits after that point and insert the number found if it is greater than the number already in that bucket. $$
hash(n) = \underbrace{00100101}_\text{k=8}\underbrace{\overbrace{0000000}^\text{7 consecutive 0's}1110101}_\text{search area}
$$

$$
_\text{Given this hash function and k = 8 we would store the number 7 into bucket 37 if that is greater than the number already there}
$$

By storing multiple 0-bit sequence lengths, even if HyperLogLog does find one or two outliers it will have many more numbers which accurately represent the number of items seen. The buckets are combined by taking their harmonic mean which throws out outliers. The authors of the paper also present several constants for range and bias correction although that is outside the scope of this post.    The power of HyperLogLog comes not only from its small memory footprint but because of how easy it is to parallelize across many machines (**cough** map-reduce **cough**). Each machine first calculates its own group of HyperLogLog buckets from the content that it has locally. These individual results are then combined by takeing the max value of each corresponding bucket. After all the machines results have been reduced down to one bucket group, the buckets can again be combined to generate a final estimate.    p.s. This post contains 386 distinct words