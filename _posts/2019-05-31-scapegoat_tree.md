---
layout: post
title: Understanding Scapegoat Trees
tags: [data_structure, c++, scapegoat_tree, binary_tree]
mathjax: true
---

## What is a scapegoat tree?


[Wikipedia](https://en.wikipedia.org/wiki/Scapegoat_tree): In computer science, a scapegoat tree is a self-balancing binary search tree. It provides worst-case O(log n) lookup time, and O(log n) amortised insertion and deletion time.

This post is a note that attempts to make the [original paper](http://people.csail.mit.edu/rivest/pubs/GR93.pdf) by Igal Galperin and Ronald L. Rivest easier to understand.


## Notation

<strong>$\alpha$-weight-balanced</strong>

A binary tree node x is $\alpha$-weight-balanced if, for $\frac{1}{2} \le \alpha < 1$...

$$
\forall \alpha \in [0.5, 1) \\
size(x.left) \le \alpha \times size(x)\\
size(x.right) \le \alpha \times size(x)\\
$$

<strong>$\alpha$-height-balanced</strong>

Given a binary tree T, let $h_{\alpha}(size(T)) = \lfloor{\log_{\frac{1}{\alpha}}{size(T)}}\rfloor$.

As an example, if $\alpha = 0.5$ and size(T) = 8, then $h_{0.5}(8) = \lfloor{\log_{2}{8}}\rfloor = 3$,

A binary tree node x is $\alpha$-height-balanced if, for $\frac{1}{2} \le \alpha < 1$...

$$
h(T) \le h_{\alpha}(size(T))
$$

>Lemma: if T is $\alpha$-height-balanced, then T is $\alpha$-weight-balanced

<strong>Deep node</strong>

A node's depth, d(N), is the length of the path from the root to the node.

Given a binary tree T and a node N, we define a node N as deep if $d(N) > h_{\alpha}(T)$. The detection of a deep node would trigger a restructuring (or rebalancing) in the scapegoat tree.


<strong>Attributes</strong>

The node in scapegoat trees only need to store ```left``` and ```right``` pointers to its child and the ```key```, and the tree itself stores ```size``` and ```max_size``` attributes. The ```max_size``` attribute is the maximal value of the size of the tree since the last time the tree was completely rebuilt. It is used for the deletion method explained later.

## Operation - Insert

To insert a node into a scapegoat tree, we start with inserting with the normal BST insertion algorithm and then increment ```size``` and ```max_size``` by one.

Then, if the inserted node $X_{0}$ is deep, we climb the tree toward the root until we find a node $X_{i}$ that is not $\alpha$-weight-balanced. $X_{i}$ is then our scapegoat node.

We rebuild the subtree rooted at $X_{i}$ by replacing it with a 0.5-weight-balanced subtree.

First we flatten the subtree by add the nodes into an auxiliary array using in-order traversal, and then we rebuild the tree with 'divide and conquer', i.e. recursively finding the mid of the array as the root of the subtree and recursively build the left child and right child.

```
BUILD-TREE(nodeList, start, end):
if end<start:
  return nil

mid = (start+end) /2

nodeList[mid].left = BUILT-TREE(nodeList, start, mid-1)
if nodeList[mid].left != nil
  nodeList[mid].left.parent = nodeList[mid]

nodeList[mid].right = BUILD-TREE(nodeList, mid+1, end)
if nodeList[mid].right != nil
  nodeList[mid].right.parent = nodeList[mid]

return nodeList[mid]

```

<strong>Can we always find a scapegoat node?</strong>

Lemma:

$$
\text{let }  \alpha = 0.5, \beta = \text{a deep node.}\\
\text{i.e. } d(\beta)  >  h_{\alpha}(T) = \lfloor{\log_{2}{size(T)}}\rfloor\\
\exists W \in \text{ancestors of } \beta \text{ such that } W \text{ is not
 }\alpha\text{-weight-balanced: }\\
size(W.left) \ge \alpha \times size(x) \vee size(W.right) \ge \alpha \times size(x)\\
$$


Proof by contradiction:

$$
\text{let } K = \{K_0,K_1, K_2...K_{d(\beta)}=\beta\}\text{ be the nodes in the path from root}\text{ to } \beta ,\\
\text{suppose } \forall k \in K, k \text{ is }\alpha\text{-weight-balanced}\\
\longrightarrow size(K_1) \le \alpha \times size(K_0) = \alpha \times size(T)\\
\longrightarrow size(K_2) \le \alpha \times size(K_1) \le \alpha^2\times size(T)\\
\longrightarrow size(K_3) \le \alpha \times size(K_2) \le \alpha^3\times size(T)\\
...\\
\longrightarrow size(K_{d(\beta)}) \le \alpha \times size(K_{d(\beta)-1}) \le \alpha^{d(\beta)}\times size(T)\\
$$

Since $K_{d(\beta)} = \beta$ is a deep node and d($\beta$)>$h_{\alpha}(T)$ = $\lfloor{\log_{2}{size(T)}}\rfloor$. Also note that $\beta$ is the newly inserted node so the size of the tree rooted at $\beta$ is 1. It follows that...
$$
\longrightarrow 1 = size(\beta) \le \alpha^{d(\beta)}\times size(T) < 0.5^{ \lfloor{\log_{2}{size(T)}}\rfloor}\times size(T) \\
\longrightarrow 1 = size(\beta) \le \alpha^{d(\beta)}\times size(T) < \frac{1}{size(T)}\times size(T) \\
\longrightarrow 1 = size(\beta) <1 \longrightarrow contradiction\\
\longrightarrow \neg (\forall k \in K, k \text{ is }\alpha\text{-weight-balanced})\\
\longrightarrow \exists k \in K, k \text{ is NOT }\alpha\text{-weight-balanced}\\
$$
