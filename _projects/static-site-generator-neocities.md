---
layout: post
title:  "Static Site Generator for Neocities"
date:   2026-08-07 00:00:00 +0000
categories:
---
<image src='../../assets/images/blog-index.png' width='350rem'></image>
<image src='../../assets/images/blog-post.png' width='350rem'></image>
[You can find the repo here.](https://github.com/kzrtn/simpler-SSG)
# The Purpose
I used to manually update neocities websites I made, copy pasting the HTML templates I made and editing it. It got nightmarish quick, and so I never bothered to maintain my websites for too long. Ever since I started using Jekyll to write this website, I realised that I could make a static site generator myself too. One where I have much more control over the complexity and system.
<br />
<br />

# The Set Up
I first started out with the concept of it. I wanted a site generator that takes markdown files and converts them into webpages with layouts and styles of my choosing. So naturally it should be a program that takes a markdown file, converts it into HTML, and then inserts them into another HTML file (a template) and saving the output as a new file in a folder called `dist`.

I could write my own markdown to HTML parser, but there are a lot of libraries that already do that, so I went with MarkdownIt.

Then, for the actual templating, I'm most familiar with Jinja2, so I went with a Node port called Nunjucks.

Setting the two up was not too difficult, I only ran into the problem where Nunjucks wasn't autoescaping the content, so my webpages were showing raw HTML like `<p>Hello world!</p>` within blog posts themselves. But that was easily fixed with the option:
```javascript
nunjucks.configure(path.join(ROOT, config.TEMPLATES_PATH), { autoescape: false })
```
It's safe to configure Nunjucks to not autoescape globally (all the time) because this is user generated content on a website that's not under my control. I don't need to worry about users doing weird injection attacks on their own websites.
<br />
<br />

# The Live Previews
I wanted a live preview of edits on localhost like Jekyll does. To do that, I had to use both [nodemon](https://www.npmjs.com/package/nodemon) and [Browsersync](https://browsersync.io/).

I needed nodemon so that each time a user saves a markdown file or makes a change to it, the script reruns and recompiles everything into HTML all over again. This means that they won't need to manually trigger `npm run build` to build their output.

But in order for users to actually see a live preview of their website, Browsersync comes into play. It's a good solution because this meant that users don't need to install a liveserver extension, and I can add a script (like `npm run serve`) that triggers Browsersync's preview.

I actually got a bit of a headache trying to figure out why Browsersync kept opening new tabs each time I made a change to my markdown files. But then I realised it was because:
```javascript
const bs = require('browser-sync').create()
bs.init({
  server: config.OUTPUT_PATH,
  startPath: `/${BLOG_INDEX_NAME}.html`,
  files: ["*/*"],
  notify: false,
  watchEvents: ["change", "add", "unlink", "addDir", "unlinkDir"]
})
```
was inside of the main `index.js` that gets restarted by nodemon each time.

In order to fix this, I had to pull this code out and separate it into it's own file (`server.js`). Users will have to call the website preview separately from the script calling nodemon. So:
* `npm run build:watch` runs nodemon on the markdown files
* `npm run serve` runs Browsersync that watches the `dist` (output) files

It's still not quite the `jekyll serve` that I was hoping for just yet. So, in order to get them to run at the same time, I used [concurrently](https://www.npmjs.com/package/concurrently). And added this to the scripts:
```json
"dev": "concurrently \"npm run build:watch\" \"npm run serve\""
```
Now `npm run dev` runs both scripts at the same time with nice coloured outputs!

Though I'm going to be honest here, I asked Claude in order to figure out how to get the live preview + updates working. Claude recommended me these packages and I put it together. In hindsight, concurrently is overkill for this project and a simple `"dev": "npm run build:watch && npm run serve"` is enough. I've been reflecting on my AI usage a lot ever since I started this project, and it's been proving both good but also not really good. It's hard to explain.
<br />
<br />

# Porting Images
When images went through MarkdownIt, input and outputs were formatted and linked correctly:

```md
Markdown: ![Image](../_media/landlady.png)
```

```html
HTML: <p><img src="../_media/landlady.png" alt="Image">
```

But images would break as the previous image location it was referring to in the Markdown file is completely changed the moment it's created in the `dist` folder. And not forgetting the fact that I want this make this so easy that the only thing the user needs to export are the contents in that output folder, I needed to make sure the images were copied over in the same style to the destination folder. If the images are in a subfolder, the script should also create that folder to copy the relative pathing that the original image referenced in the Markdown file followed.