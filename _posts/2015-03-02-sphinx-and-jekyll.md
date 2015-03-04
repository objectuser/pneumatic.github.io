---
layout: post
title:  "Using Sphinx and Jekyll on GitHub"
date:   2015-03-02 08:00:00
categories: pneumatic
---

I'm using Sphinx to create documentation for Pneumatic.IO. I'm hosting the docs on GitHub, here with the blog.

The blog is using Jekyll, so there's a bit of an impedance mismatach between how Jekyll wants to work and how Sphinx wants to work.

Recently I was trying to figure out how to add an image to a page. I used the Sphinx syntax:

```
.. image:: images/architecture.png
```

But the image wouldn't show up. It took an unfortunately long time to figure out the issue. My first problem is that my iterations were too slow. I was committing to GitHub and only then was I seeing the problem. This was silly, as I had already been using my local Jekyll server to test my blog. I just had to navigate through to the documentation to see the problem.

The problem was simple, and I remembered reading about it when I first started with Jekyll: Sphinx processed the images I was using to the `_images` directory. Jekyll ignores files beginning with underscores by default. So I needed to add that directory to my Jekyll configuration:

```Python
include: [".htaccess", "_sources", "_static", "_images"]
```

This was a bit frustrating, but there are always tradeoffs with any tool. Overall, I'm pretty happy with Jekyll and Sphinx. I'd like to use just one syntax (markdown vs restructured text), but I'll leave that until later.