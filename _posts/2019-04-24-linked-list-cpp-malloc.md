---
layout: post
title: LinkedList Implementation With C++, malloc Or Not?
tags: [c++]
---

## The Problem

```linkedList``` is a linear data structure, in which the elements are not stored at contiguous memory locations, but in "nodes".

When implementing a linked list with C++, I encountered a curious error when initiating a new ```node``` struct that holds the data.

Here's the implementation of the ```push``` function of a linked list based stack.

{% highlight c++ %}

void push(T value){
    struct Node<T> node;
    node.value = value;
    node.next = head;
    head = &node;
    elementCount++;
  }

{% endhighlight %}

If the function is implemented this way, I would get an error message upon calling the push function.

<strong>SIGSEGV - Segmentation violation signal</strong>

However, when I change the struct initiation to malloc as shown in the snipper below, it works!

{% highlight c++ %}

void push(T value){
    struct Node<T>* node;
    node = (struct Node<T>*)malloc(sizeof(struct Node<T>));
    node->value = value;
    node->next = head;
    head = node;
    elementCount++;
  }
{% endhighlight %}


## Why?

My first guess is that initiating an instance of a struct with

```struct Node node;```

creates the variable ```node``` of the size ```struct Node``` in the stack frame created when the ```push``` function is called. That means when the execution of the push function is completed and the stack frame is popped, that address space would be dis-allocated, and therefore reading into the node address causes a segmentation fault.

Using ```malloc```, on the other hand, dynamically allocates memory space of the size ```struct Node``` in the heap. The heap address space persists until the end of the program, and thus reading into the heap address space after push function is finished does not lead to a segmentation fault.

To prove it, I printed the address of the node created by

```
struct Node node;
cout << &node << endl;
```

and

```
struct Node<T>* node;
node = (struct Node<T>*)malloc(sizeof(struct Node<T>));
cout << $(*node) << endl;
```

Here's the result:

```
use struct Node<T>* node
0x7ffee0b33110
use malloc
0x7fe651402df0
use struct Node<T>* node
0x7ffee0b33110
use malloc
0x7fe651402f90
use struct Node<T>* node
0x7ffee0b33110
use malloc
0x7fe651402fa0
use struct Node<T>* node
0x7ffee0b33110
use malloc
0x7fe651402fb0
use struct Node<T>* node
0x7ffee0b33110
use malloc
0x7fe651402ff0
```

In a for loop that calls the ```push``` function for 5 times, a stack frame is pushed on the stack every time ```push``` is called, and then is popped when the execution finishes. The node variable created in that stack frame is always stored in that same memory space, as seen in the output above (0x7ffee0b33110).


This proves my theory, and it explains why initiating the node ```struct Node node;``` does not work.

## Final Words

I chose C++ to practice implementing data structure and algorithms because the class I am taking only allows C++ and Java, and I know nothing about Java.

Quite often I encounter errors related to memory allocation and pointers, so I jot down this note as a reminder for myself, and for anyone who might find useful.
