---
layout: post
title: Binary Trees
tags: [data-structure]
mathjax: true
---

## Getting Started...
A <strong>binary tree</strong> is an ordered tree with the following properties:

1. Every node has at most two children
2. Each child node is labeled as being either a left child or a right child.
3. A left child precedes a right child in the order of children of a node.

Here's a graphical example of a binary tree.

![binarytree](https://user-images.githubusercontent.com/44837996/57978804-92297280-7a46-11e9-9399-2eea8c68fb38.png)


There is a certain terminology associated with (binary) trees:

1. <strong>depth</strong> of a node: is the length of path from the node to the root of the tree.

2. If a node A is in on the path from a node B to root node R, then A is called an <strong>ancestor</strong> of B and B a <strong>descendant</strong> of A.

3. The <strong>height</strong> of a node is the length of the longest path from the node to one of its descendants. The height of a tree is the height of its root node.

4. A node is a <strong>leaf</strong> it it has no children.

## Operations

A binary tree should support several operations listed below:

1. int size(Node n)
2. int height(Node n)

Both methods can be easily implemented recursively and run at O(n) time.

{% highlight c++ %}

int size(Node n){
    if (n == nullptr){
      return 0;
    }
    return 1 + size(n.left) + size(n.right);
  }

int height(Node n){
  if (n==nullptr){
    return -1;
  }
  return max(height(n.left), height(n.right));
}

{% endhighlight %}

## Traversal Algorithm

There are 2 types of traversal algorithms for trees, namely breadth-first traversal and depth-first traversal. Within depth-first traversal algorithms, it can be further divided into 3 different methods: pre-order, in-order, and post-order traversal.

### Pre-order Tree Traversal

In this traversal algorithm, the root of the tree is visited first and then the sub trees rooted at its children are traversed recursively.

![preorder](https://user-images.githubusercontent.com/44837996/57979169-5f827880-7a4c-11e9-843a-e7d51eb29cf7.png)


{% highlight c++ %}

void preOrderTraversal(Node n){
    if (n == nullptr){
      return;
    }
    cout << n.value << "\t";
    preOrderTraversal(n.left);
    preOrderTraversal(n.right);
  }

{% endhighlight %}

With the example in the image above, the output result would be F B A D C E G I H.

### In-order Tree Traversal

In this traversal algorithm, the tree is visited from left to right. That is, for every position p, the in-order traversal visits p:
* after all the positions in the left sub-tree of p
* before all the positions in the right sub-tree of p

![inorder](https://user-images.githubusercontent.com/44837996/57979233-81c8c600-7a4d-11e9-9b8a-152388bad907.png)

{% highlight c++ %}

void inOrderTraversal(Node n){
    if (n == nullptr){
      return;
    }
    if (n.left != nullptr){
      inOrderTraversal(n.left);
    }
    cout << n.value << "\t";
    if (n.right!=nullptr){
      inOrderTraversal(n.right);
    }
  }

{% endhighlight %}


With the example in the image above, the output result would be A B C D E F G H I

### Post-order Tree Traversal

In this traversal algorithm, it recursively traverses the children of the root first and then visit the root.

![postorder](https://user-images.githubusercontent.com/44837996/57979306-a7a29a80-7a4e-11e9-84eb-d322b0660b24.png)


{% highlight c++ %}

void postOrderTraversal(Node n){
    if (n == nullptr){
      return;
    }
    if (n.left != nullptr){
      postOrderTraversal(n.left);
    }
    if (n.right!=nullptr){
      postOrderTraversal(n.right);
    }
    cout << n.value << "\t";
  }

{% endhighlight %}


With the example in the image above, the output result would be A C E D B H I G F.

### Breadth-first Traversal

In this traversal algorithm, the positions at the same depth <i>d</i> before it visits the positions at depth <i>d+1</i>.

![breadthfirst](https://user-images.githubusercontent.com/44837996/57979452-6f03c080-7a50-11e9-9716-b314c4ff09e1.png)

To achieve this, we utilise a FIFO queue to give us the ability to traverse in the order in which we visit the nodes.

{% highlight c++ %}

void breadthFirstTraversal(Node n){
    Queue q;
    q.enqueue(n);
    while (q.size()!=0){
      n = q.dequeue();
      cout << n.value << '\t';
      if (n.left != nullptr) q.enqueue(n.left);
      if (n.right != nullptr) q.enqueue(n.right);
    }
  }

{% endhighlight %}

With the example in the image above, the output result would be F B G A D I C E H.
