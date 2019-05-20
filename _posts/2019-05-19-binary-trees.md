---
layout: post
title: Binary Trees Explained
tags: [data-structure, c++]
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

#### Pre-order Tree Traversal

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

#### In-order Tree Traversal

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

#### Post-order Tree Traversal

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

#### Breadth-first Traversal

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


## Binary Search Tree

A binary search tree is a binary tree <i>T</i> with each position <i>p</i> storing a key-value pair (k,v) such that

* keys stored in the left subtree of <i>p</i> are less than <i>k</i>
* keys stored in the right subtree of <i>p</i> are greater than <i>k</i>

An graphical example of a BST:

![inorder](https://user-images.githubusercontent.com/44837996/57979233-81c8c600-7a4d-11e9-9b8a-152388bad907.png)

In a BST, an in-order traversal will visit the keys in ascending order; in this case it'll be A B C D E F G H I.

## Implementing BST

You can choose to implement the BST with an array or a linked-list-like structure.
[Here](https://www.geeksforgeeks.org/binary-tree-array-implementation/) is a link that introduces the implementation with an array. In this post I introduce the linked-list implementation, and therefore there's the struct node that we need to define first.

The node should store the key value of any type. Note that the key type has to support the operator larger than (>), smaller than(<) and equality (==) for this to work. Therefore if you want to use a custom made struct or class instance, you need to implement [operator overloading](https://en.cppreference.com/w/cpp/language/operators).

{% highlight c++ %}
template <typename K, typename V>
struct Node {
  K key;
  V value;
  Node* left = nullptr;
  Node* right = nullptr;
  Node* parent = nullptr;
};
{% endhighlight %}

You also need to have some private variables declared in private for the BST class.

```
Node<K, V>* root = nullptr;
int count = 0;
```
## Search

Searching is quite straightforward given the definition and properties of a BST. When given a key-value pair, we can start from the root node and compare the key of the visited node with the to-be-inserted key. If the key of the visited node is smaller than the to-be-inserted key, we next visit the right child; if the key of the visited node is larger than the to-be-inserted key, we next visit the left child; finally if the visited node's key is equal to the key to be inserted, we return the visited node.

Note that even if the search is not successful, we still gain valuable information as we would identify the supposed parent node of the to-be-inserted key. In the following implementation, the function return the supposed parent node of the to-be-inserted key if the search is unsuccessful, and it returns the target node of which key is the same as the to-be-inserted-key.

{% highlight c++ %}

Node<K, V>* searchNode(K key){
    Node<K,V>* walker = root;
    Node<K,V>* prevWalker = nullptr;
    while (walker != nullptr){
      prevWalker = walker;
      if (walker->key==key){
        return walker;
      } else if (walker->key< key){
        walker = walker->right;
      } else {
        walker = walker->left;
      }
    }
    return prevWalker;
  }

{% endhighlight %}

## Insert

With the implementation of search, insertion becomes a lot easier. When given a key-value pair, we call the search function to find the key.

There are 4 cases that the function needs to process:

1. If the key of the returned node is equal to the given key, it means that the key already exists, and we can replace the value of the node with the new value.

2. If the key of the returned node is larger than the given key, we create a new node, store the inserted key and value in it, and attach it to the left child of the returned node.

3. If the key of the returned node is smaller than the given key, we create a new node, store the inserted key and value in it, and attach it to the right child of the returned node.

4. If the node returned by the search function is nil, it means that there's nothing in the BST yet. In this case, we just need to create a new node, store the necessary values in it and point the root node pointer to this new node.

In the following function implementation, the ```createNewNode``` function is just a simple initiation of the ```struct Node```. You can write a custom constructor in the struct declaration and use the ```new``` keyword to achieve the same results.

{% highlight c++ %}
void insert(K key, V value){
    Node<K,V>* found = searchNode(key);
    if (found==nullptr){
      Node<K,V>* newNode = createNewNode(key, value);
      root = newNode;
    } else if (found->key < key){
      Node<K,V>* newNode = createNewNode(key, value);
      newNode->parent = found;
      found->right = newNode;
    } else if (found->key > key){
      Node<K,V>* newNode = createNewNode(key, value);
      newNode->parent = found;
      found->left = newNode;
    } else {
      found->value = value;
      return;
    }
    count ++;
  }
{% endhighlight %}

## Remove
Removal is a bit more complicated. Here we need to discuss two cases.

<strong>Case 1. the node to be removed has at 0 or 1 child but not 2.</strong>

![image](https://user-images.githubusercontent.com/44837996/58001715-de3fea00-7b0e-11e9-8819-f9fe4cfc399f.png)

If we want to remove the node with key -4 as show in the image, we simply use the node itself to find the parent, identify whether the target node is a left child or the right child, and make it null.


If the target node has one child, it's similar.

* identify whether the target node has left child or right child.
* identify whether the target node itself is a left child or left child.
* adjust the target node's parent to point its left or right child pointer to the child node of the target node

We can combine the two cases into one function called splice

{% highlight c++ %}
void splice(Node<K, V>* node){
    //input: target node to be removed, the node has to have 0 or 1 child.
    //result: the target node is spliced and its position is replaced by either its left or right child
    Node<K, V>* descendant;
    Node<K, V>* parent;
    //find the target node's single child, it may be nil.
    if (node->left != nullptr){
      descendant = node->left;
    } else {
      descendant = node->right;
    }
    //edge case: if target node itself is root, update root
    if (node==root){
      root = descendant;
      parent = nullptr;
    } else {
      parent = node->parent;
      //check if target node is left child or right child, splice the target node.
      if (parent->left == node){
        parent->left = descendant;
      } else {
        parent->right = descendant;
      }
    }
    //update the parent pointer of the child of the target node.
    if (descendant!=nullptr){
      descendant->parent = parent;
    }
  }
{% endhighlight %}

<strong>Case 2. the node to be removed has 2 children.</strong>

![image](https://user-images.githubusercontent.com/44837996/58002176-417e4c00-7b10-11e9-8a62-7c73795a9750.png)

In the example above, if we want to remove the node with key 12 (let it be target node <i>T</i>), we need to:

1. find the largest node in its left child subtree (let it be <i>L</i>).
2. swap the keys and values of <i>L</i> and <i>T</i>.
3. splice node <i>L</i>.

It works because <i>L</i> has the key that strictly precedes <i>T</i>'s key (this is based on the definition of a BST). Thus if you swap the keys and values of the aforementioned 2 nodes, all the keys in the right child subtree of <i>T</i> would still be larger than the key of <i>T</i>, and all the keys in the left child subtree of <i>T</i> would be smaller than the key of <i>T</i>

Splicing node <i>L</i> works because node <i>L</i> can at most has 1 child. Being the node with the largest key in a tree, it cannot possibly have a right child.

{% highlight c++ %}
bool remove(K key){
    Node<K,V>* found = searchNode(key);
    if (found==nullptr || (found->key)!=key){
      return false;
    } else {
      // target node has 2 children.
      if (found->left != nullptr && found->right != nullptr){
        //swap keys and values
        Node<K,V>* largestInLeftChild = findLargest(found->left);
        K kHolder = largestInLeftChild->key;
        V vHolder = largestInLeftChild->value;
        largestInLeftChild->key = found->key;
        largestInLeftChild->value = found->value;
        found->key = kHolder;
        found->value = vHolder;
        //splice
        splice(largestInLeftChild);
      } else {
        // target node has <= 1 child
        splice(found);
      }
      count--;
      return true;
    }
  }
{% endhighlight %}

The ```findLargest``` helper function can be easily implemented:

{% highlight c++ %}
Node<K, V>* findLargest(Node<K, V>* walker){
    while (walker->right != nullptr){
      walker = walker->right;
    }
    return walker;
  }
{% endhighlight %}

## Runtime Analysis

The ```search```, ```insert```, and ```remove``` methods all run in O(h) time where h equals the height of the BST. In the worst case it means O(n) where n is the number of items in the BST. Imagine inserting numbers in an ascending order (1, 2, 3, 4.....n) the items will always be added to the right child. This is not ideal and can be improved with [self-balanced BST](https://en.wikipedia.org/wiki/Self-balancing_binary_search_tree)

## The Gist

{% gist 1a32bf6312aae47dc119e19888c89524 %}

## Next...

In the next post I might talk about the self balancing BST that can run the aforementioned methods in $O(\log{n})$ time.

Cheers!
