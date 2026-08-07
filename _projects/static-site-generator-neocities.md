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

I also needed a way for the app to easily tell what post titles and dates are. I decided to copy what Jekyll does and use a yaml front matter. In order to parse frontmatter, I decided to use the first yaml frontmatter parser that I could find, [gray-matter](https://www.npmjs.com/package/gray-matter).

It was very simple to use, the basic usage on the npm page was enough. I didn't need to change much.

Now, all users needed to create a blog post was write a file like so:
```md
---
title: Blog title here
date: 07 August 2026
---
# My first blog post
Hello, world!
```
And save it as a markdown file in the `_posts` folder.
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
HTML: <img src="../_media/landlady.png" alt="Image">
```

But images would break as the previous image location it was referring to in the Markdown file is completely changed the moment it's created in the `dist` folder. And not forgetting the fact that I want this make this so easy that the only thing the user needs to export are the contents in that output folder, I needed to make sure the images were copied over in the same style to the destination folder. If the images are in a subfolder, the script should also create that folder to copy the relative pathing that the original image referenced in the Markdown file followed.

I ended up spending an hour on [regexr.com](https://regexr.com/) learning basic regex and trying to figure out how to write a regex expression that selects everything in an image path up to the last slash.
```javascript
const imageHomePath = imagePath.match(/^\..*\//g)
```
After that it's simple, just check if that pathing exists in `dist/posts/`, and if it doesn't, create the path. Though, there are things to account for that I haven't checked yet:
* The regex expression only checks for forward slashes `/`, so `\` would break it
* Absolute file paths will also break this system
* The app also lets you use pathing to folders outside of the project, and hence would create folders outside of the project. Very confusing and unfriendly behaviour. Should warm/error if user tries to do this.

However I did consider the situation where `<` and `>` is used in image pathing because spaces are present in the file name or folder names:
```md
![Image with spaces in name/directory](<../_media/uwu girl.PNG>)
```
My program will simply check for these with a regex expression and remove the arrows before further manipulation:
```javascript
if (imagePath.includes('<') && imagePath.includes('>')) {
  imagePath = (imagePath.match(/(?<=<).*(?=>)/g))[0]
}
```
<br />

# Running npm packages from CLI with npx
Everything worked fine and dandy in my project. But it was still not quite what I imagined. Users will need to clone my project and then edit the files from there. I want users to be able to generate their own project files with my starter templates with a single command line. I don't want them actually downloading my project and using `require('simple-ssg')` on it.

Enter npx, a command already bundled inside of node that allows users to run an npm project without downloading it. It took me quite a bit, but I realised that in order to use npx I had to do these:
* Publish my project on npm
* Add `"bin": "index.js",` to my `package.json`
* Add the shebang line `#!/usr/bin/env node` to the top of `index.js` so the shell know to use node to execute the rest of the script
* Refactor my whole `index.js`

The last part was the biggest headache. Now that I could simply `npx simpler-ssg` to get it to run, my original app would run in whatever directory I'm in, expecting to already have the project structure already laid out (`_templates`, `_posts`, `_styles` folders etc). I didn't want to edit my original file too much, so I decided to make `index.js` the entry point that deals with the commands and rename the old `index.js` to `app.js`. It's much more fitting that the actual program logic lives in `app.js` as well.

The first part to my new `index.js` was getting it to do the command `npx simpler-ssg create (new-site)` just like [create-react-app](https://create-react-app.dev/docs/getting-started/). Honestly, I thought that I was doing something wrong, because I thought that there was a way to get npx to execute npm scripts within a node package. But no matter how much I looked I just couldn't find a straight answer to this. Like there has to be a way that apps like create-react-app was implementing different npx commands.

Like what, are they actually handwriting the `argv` logic? I don't see the `bin` referenced in their [github repo](https://github.com/react/create-react-app/blob/main/package.json).

Yeah. Turns out I was looking in the wrong place. This is apparently the repo root, and the actual repo is inside `/packages/create-react-app` <-- the same name?!

And, it also turns out that YES projects are actually using `argv` arguments to execute different commands. Projects that bundle different npx commands are either handrolling their `argv` or using packages like [commander](https://www.npmjs.com/package/commander).

This is the bit that makes me most frustrated about living without AI. I don't know how one can figure any of these out on their own! I knew how to use `argv`, but I was so caught up with trying to find a way to run npm scripts within a package via npx that I got completely sidetracked and sent on an impossible search quest (because what I was looking for is impossible in the first place??). I just never thought of writing `argv` arguments on my own because I thought that they were using something else to execute npm scripts. Node is so different from low level programs I've written in C after all! It's really tough when you're missing too much context to piece things together on your own or via documentation/searches. You need to know enough to be able to keyword search in the first place! "npx run npm script from cli" is an INSANE thing to search that definitely will not give anything useful at all!

/rant over

# npx simpler-ssg serve
Anyway, with all of that out of the way. All's left to the program after getting `npx simpler-ssg create (site-name)` was re-routing all of the directories for `npx simpler-ssg serve` to work. It was easy enough to get the app to copy over the template folders from the package into the user's working directory, but now it was time to get the live preview and build working.

This took a lot longer than I expected because I would keep accidentally getting the paths wrong and the app would crap itself out. But thankfully, after a lot of trial and error. I managed to get everything to work together. However, I feel this is all very... duck taped together. I am sure there are many bugs, I need to write some tests for this.

Getting nodemon and Browsersync to work for the `serve` command was also easier than I expected:
```javascript
else if (process.argv[2] === 'serve') {
  const config = require(path.join(process.cwd(), '/config/config.js'))

  nodemon({
    "watch": ["_posts", "_templates", "_styles", "_media", "config"],
    "ext": "md,html,css,js,gif,jpg,jpeg,png,svg,webp",
    "exec": `cd ${__dirname} && node app.js -- ${process.cwd()}`
  })

  nodemon
    .on('start', () => console.log('Nodemon started'))
    .on('quit', () => process.exit())

  const BLOG_INDEX_NAME = config.BLOG_INDEX.replace('.html', '')
  bs.init({
    server: config.OUTPUT_PATH,
    startPath: `/${BLOG_INDEX_NAME}.html`,
    files: ["*/*"],
    notify: false,
    watchEvents: ["change", "add", "unlink", "addDir", "unlinkDir"]
  })
  bs.reload(["*.html", "*.css"])
}
```
Nodemon is different here. This time it's spawned as a child process.

I could have them live together in `index.js` at the same time since nodemon was watching and executing `app.js`, so Browsersync is still living outside of whatever nodemon watches.

And that's all so far! I still need to add a few things, so I'm not done yet.
<br />
<br />

# Plans for the future
* Implement `npx simpler-ssg build` to build `dist` without live preview
* Abstract Browsersync and Nodemon away from `index.js`
* Add unit tests

