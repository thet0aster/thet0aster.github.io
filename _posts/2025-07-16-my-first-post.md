---
layout: post
title: My first post
date: 2025-07-16 19:58 -0400
last_modified_at: 2025-07-16 20:17 -0400
comments: true
categories: [Testing, post_testing]
tags: [Test]
---

# Testing first post

This is a test of my very first post. History is being made here, ladies and
gentlemen. Here's some code from GeeksForGeeks:

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
{% if page.comments %}
<div id="disqus_thread"></div>
<script>
    /**
    *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
    *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
    /*
    var disqus_config = function () {
    this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = 'https://thet0aster.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}