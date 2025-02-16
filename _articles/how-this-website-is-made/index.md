---
title: How This Website is Made
date: 2024-02-13
description: Are you interested in how this website is built and hosted?
image: /assets/img/Logo-White-Background-Circle-4K-Small.png
tags: [Jekyll, GitHub Pages, MarkDown, SSG]
---

This website is hosted using GitHub pages, which is free for public repositories.  GitHub Pages uses Jekyll under the hood, which is a very flexible Static Site Generator (SSG).

I didn't want to tie myself to any particular technology, and having worked with projects like Strapi in the past, I felt keeping my content stored in MarkDown would allow me the flexibility to move in a number of different directions if I felt Jekyll wasn't up to the task. Time will tell.

Almost all the code here was generated using Claude AI, under my guidance. I started with one of the default themes (minimal) and then rewrote the CSS/HTML to create a responsive design. I'm not a big fan of the left-side column approach these days, but I also wanted to tip my hat to the previous designs I had used for Spode's Abode for a little nostalgia.

The [source code](https://github.com/andrewspode/spodeme) is visible for all if you wanted to see how elegant GitHub Pages can be.

Below is a bunch of MarkDown that I asked Claude to generate, so that I could easily see the impact of any style changes in a single place.

---

## Second Level Heading
### Third Level Heading

Regular paragraph with *italic*, **bold**, and ***bold italic*** text. Here's a [link to somewhere](#) and some `inline code`.

> This is a blockquote
> It can span multiple lines
> And can contain **formatting**

Here's a list:
* Unordered item
* Another item
  * Nested item
  * Another nested item
* Back to main level

Numbered list:
1. First item
2. Second item
   1. Nested numbered
   2. Another nested
3. Back to main

Here's a code block:
```python
def hello_world():
    print("Hello from Python!")
    for i in range(5):
        print(f"Count: {i}")
```

Here's an image:

![Logo Test](/assets/img/Logo-White-Background-Circle-4K.png)