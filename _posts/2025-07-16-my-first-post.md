---
layout: post
title: My first post
date: 2025-07-16 19:58 -0400
categories: [Testing, post_testing]
tags: [Test]
---

# Testing first post

This is a test of my very first post. History is being made here, ladies and
gentlemen. Here's some code:

```c++
// C++ program for the implementation of Bubble sort
#include <bits/stdc++.h>
using namespace std;

void bubbleSort(vector<int>& v) {
    int n = v.size();

    // Outer loop that corresponds to the number of
    // elements to be sorted
    for (int i = 0; i < n - 1; i++) {

        // Last i elements are already
        // in place
        for (int j = 0; j < n - i - 1; j++) {
          
          	// Comparing adjacent elements
            if (v[j] > v[j + 1])
              
              	// Swapping if in the wrong order
                swap(v[j], v[j + 1]);
        }
    }
}

int main() {
    vector<int> v = {5, 1, 4, 2, 8};

    // Sorting the vector v
    bubbleSort(v);
    for (auto i : v)
        cout << i << " ";
    return 0;
}
```
