---
title: Read more toggler
description: A simple pure JavaScript package to truncate paragraphs and add functionality to expand with read more and read less. Just add class="read-more" in your p element.
author: Pratik Kute
date: 2022-10-23T05:35:57.631Z
tags:
  - post
  - featured
image: /assets/blog/readmore.png
imageAlt: example
---
# Read more toggler ...

A simple pure JavaScript package to truncate paragraphs and add functionality to expand with read more and read less.
Just add `class="read-more"` in your p element.


Ref:

[npm package "read-more-toggler" by me](https://www.npmjs.com/package/read-more-toggler)

## Add using CDN

```html
import { readMoreToggler } from "https://unpkg.com/read-more-toggler@1.1.0/main.js";
```

## Install npm package

```bash
npm i read-more-toggler
```

```html
<p class="read-more">YOUR LONG PARA</p>

<script type="module">
  import { readMoreToggler } from "./main.js";
  readMoreToggler({
    charLimit: 250, // default is 100
    expandText: "show more", // default is 'Read more'
    collapseText: "collapse", // default is 'Show less'
  });
</script>
```