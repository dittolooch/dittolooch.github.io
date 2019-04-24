---
layout: post
title: LinkedList Implementation with C++, malloc or not?
tags: [c++]
---

## linkedList

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
