---
title: Eleventy website with local netlify-cms
description: Create blogs locally and host them on Github pages using eleventy
  and netlify-cms
author: Pratik Kute
date: 2021-07-20T05:35:57.631Z
tags:
  - post
  - featured
image: /assets/blog/11ty.png
imageAlt: A simpler static site generator. An alternative to Jekyll. Written in
  JavaScript. Transforms a directory of templates (of varying types) into HTML.
---
# Eleventy static website

Ref:

[Eleventy Documentation](https://www.11ty.dev/docs/)

<https://www.npmjs.com/package/netlify-cms-proxy-server/>

<tps://www.netlifycms.org/docs/beta-features/>

[kevin powell: Turn static HTML/CSS into a blog with CMS using the JAMStack](https://www.youtube.com/watch?v=4wD00RT6d-g)

## Install eleventy

```js
npm install @11ty/eleventy --save-dev
```
## update package.json scripts and install devDependencies
```json

  "scripts": {
    "start": "eleventy --serve",
    "build": "eleventy",
    "deploy": "git subtree push --prefix public origin gh-pages"
  },
  "devDependencies": {
    "@11ty/eleventy": "^0.12.1",
    "@11ty/eleventy-plugin-syntaxhighlight": "^3.1.1",
    "html-minifier": "^4.0.0",
    "netlify-cms-proxy-server": "^1.3.19"
  },
```

## Update .eleventy.js

```js
const { DateTime } = require("luxon");
const syntaxHighlight = require("@11ty/eleventy-plugin-syntaxhighlight");
const htmlmin = require("html-minifier");



module.exports = function(eleventyConfig) {

  eleventyConfig.addPlugin(syntaxHighlight);

  eleventyConfig.addPassthroughCopy("./src/style.css");
  eleventyConfig.addPassthroughCopy("./src/assets");
  eleventyConfig.addPassthroughCopy("./src/admin");

  eleventyConfig.addFilter("postDate", (dateObj) => {
    return DateTime.fromJSDate(dateObj).toLocaleString(DateTime.DATE_MED);
  });


  // Minify HTML
  eleventyConfig.addTransform("htmlmin", function (content, outputPath) {
    // Eleventy 1.0+: use this.inputPath and this.outputPath instead
    if (outputPath.endsWith(".html")) {
      let minified = htmlmin.minify(content, {
        useShortDoctype: true,
        removeComments: true,
        collapseWhitespace: true,
      });
      return minified;
    }

    return content;
  });

  return {
    dir: {
      input: "src",
      output: "public",
    },
  };
}
```

## Netlify CMS settings

Add admin folder to your src folder

add two files to admin folder config.yml and index.html

### config.yml

```yml
backend:
  name: proxy
  proxy_url: http://localhost:8081/api/v1
  branch: main

local_backend: true
media_folder: "public/assets/blog"
public_folder: "/assets/blog"
collections:
  - name: "blog"
    label: "Blog"
    folder: "src/blog"
    create: true
    slug: "{{year}}-{{month}}-{{day}}-{{slug}}"
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Description", name: "description", widget: "string" }
      - { label: "Author", name: "author", widget: "string" ,default: "Pratik Kute"}
      - { label: "Date", name: "date", widget: "datetime" }
      - { label: "Tags", name: "tags", widget: "list", default: ["post"] }
      - { label: "Featured Image", name: "image", widget: "image" }
      - { label: "Image caption", name: "imageAlt", widget: "string" }
      - { label: "Body", name: "body", widget: "markdown" }


```

### index.html

```html

<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>


</head>
<body>
  <!-- Include the script that builds the page and powers Netlify CMS -->
  <script src="https://unpkg.com/netlify-cms@^2.10.147/dist/netlify-cms.js"></script>

</body>
</html>

```