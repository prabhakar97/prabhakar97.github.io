---
layout: post
title: "Elegant solutions to common interview problems - Part 1 - Linked Lists"
date: 2017-03-10 13:48:24 -0500
comments: true
tags: [computer-science]
---

> Define a linked list node for integers

{% highlight cpp %}
class Node {
    public:
    int data;
    Node* next;
};
{% endhighlight %}

> Reverse a singly linked list in-place iteratively

{% highlight cpp %}
Node* reverseIterative(Node* head) {
    // Initialize two pointers, current(ptr) and previous
    Node* ptr = head;
    Node* prev = NULL;
    // At each node, we point it's next to previous
    while (ptr) {
        Node* next = ptr->next;
        ptr->next = prev;
        // Advance both pointers
        prev = ptr;
        ptr = next;
    }
    // Point head to last node
    head = prev;
    return head;
}
{% endhighlight %}

> Reverse a singly linked list in-place recursively

{% highlight cpp %}
Node* reverseRecursive(Node* head) {
    if (!head || !head->next) {
        return head;
    }
    Node* rest = reverseRecursive(head->next);
    // Point head's next node towards head
    head->next->next = head;
    head->next = NULL;
    return rest;
}
{% endhighlight %}

> Traverse a linked list recursively

{% highlight cpp %}
void traverse(Node* head) {
    if (!head) {
        return;
    } else {
        cout<<head->data<<endl;
    }
    traverse(head->next);
}
{% endhighlight %}

> Traverse a linked list recursively in reverse order

{% highlight cpp %}
void traverseReverse(Node* head) {
    if (!head) {
        return;
    }
    traverseReverse(head->next);
    cout<<head->data<<endl;
}
{% endhighlight %}

> Print nth element of linked list (counted from 1)

{% highlight cpp %}
void printNthElement(Node* head, int n) {
    if (!head) {
        return;
    }
    if (n == 1) {
        cout<<head->data;
    }
    printNthElement(head->next, n-1);
}
{% endhighlight %}

> Print nth element from last in linked list 

{% highlight cpp %}
void printNthLastElement(Node* head, int* n) {
    if (!head) {
        return;
    }
    printNthLastElement(head->next, n);
    *n = *n - 1;
    if (*n == 0) {
        cout<<head->data;
        return;
    }
}
{% endhighlight %}
