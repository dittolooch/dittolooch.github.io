---
layout: post
title: Let's Talk About Hash Tables
tags: [c++, hashtable]
mathjax: true
---

> Wikipedia: In computing, a hash table (hash map) is a data structure that implements an associative array abstract data type, a structure that can map keys to values. A hash table uses a hash function to compute an index into an array of buckets or slots, from which the desired value can be found.

Its time complexity in big O notations are as follows:

|Algorithm|		Average|	Worst case|
|---|---|---|
|Space	|	O(n)|	O(n)|
|Search	|	O(1)|	O(n)|
|Insert	|	O(1)|	O(n)|
|Delete	|	O(1)|	O(n)|

The hash table as a abstract data type is the underlying concept of the ```map``` class in C++'s standard template library and of ```dict``` of Python.

An intuitive approach to implement the methods of the hash table is to use <strong>open addressing</strong>.

With this approach, upon given a key-value pair of (i, x), the hash table can simply put the value in the i-th slot of an array.

For example, when given a key-value pair of (5, 10), the hash table stores the data as in an array as shown below:

|0|1|2|3|4|5|6|7|8|9|10|
|-|-|-|-|-|-|-|-|-|-|-|-|
|-|-|-|-|-|10|-|-|-|-|-|

This method, despite guarantees constant time operations for insert and delete, is a giant memory hog if the key universe is big. If the key universe has 1 million different possibilities, an array of 1 million in size needs to be created.

Therefore in reality I really want to have the number of slots M to be smaller or equal to the number of elements (N) inserted.

$ M \leq N\\ $

...to be continued.
