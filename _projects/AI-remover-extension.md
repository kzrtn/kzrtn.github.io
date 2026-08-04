---
layout: post
title:  "Remove AI Overview from Google Search Extension"
date:   2026-08-04 00:00:00 +0000
categories:
---
# The Purpose
I'm learning to program properly. Getting answers spoiled by the AI overview has been stunting my ability to read through documentation and figure things out myself. So I decided to write a simple script to remove any AI answers from Google searches.
<br />
<br />

# The Journey
I first tried something incredibly simple, I went with: 
```javascript
function removeAIOverview(el) {
  el.remove()
}

removeAIOverview(document.querySelector('[jsname="ZLxsqf"]'))
```
Well, it immediately worked which was great but it also came with one caveat. You'd see the AI Overview area load for a split second before it got nuked out of existence.
<image src='../../assets/images/overview-bug.gif' style='border: 1px solid #e6e6e6;'></image>
<br>
But I was also getting this error:
`Uncaught TypeError: Cannot read properties of null (reading 'remove')`
Looks like line 3 (`el.remove()`) was triggering an error since it was grabbing an element that didn't exist. But the script worked, what's happening?

It looks like there's a race condition in here somewhere. If the script fired the element existed, the querySelector returns null and `.remove()` is not a method for null objects.

Adding a simple check solves the bug:
```javascript
if (el) {
  el.remove()
}
```

Unfortunately, the split second load bug still persists. And so I discovered that quickly mashing F8 in the sources tab on Google Chrome brings up the debugging menu. I spent a good hour going over the page loading step by step, to little results.

My first instinct was: "The element with the class appears on page but the script doesn't run. Is it not being detected?" I thought the script wasn't running before the element loaded, which would explain things, except that the bug earlier came from the script loading before the element existed. (or so I thought)

So maybe there was something wrong with my script. I continued to troubleshoot.

Selecting the element by the class key value pair seems pretty fragile (what if other elements have the same class in the future?), so I tried a few others:
```javascript
id="Odp5De" // This is when the AI overview is the first div, doesn't work in all cases
id="rcnt" // Turns out to be the whole search result body. Oops
class="bzXtMb M8OgIe dRpWwb" // The style for the AI Overview box. Still very fragile
```
<br>
Speaking of that last fragile selecting three classes at one go, I thought that maybe setting the CSS properties of those to `display: none` would work. Since the style for the classes can be added to the document before the document is loaded.
```javascript
let style = document.documentElement.appendChild(document.createElement('style'))
style.textContent = '.bzXtMb {display: none;}'
style.textContent = '.M8OgIe {display: none;}'
style.textContent = '.dRpWwb {display: none;}'
```
That didn't work either. Even weirder, I couldn't find the appended `<style>`.

Eventually I settled on a reliable enough ID:
```javascript
const element = document.querySelector('[id="eKIzJc"]')
```
<br>
Getting the right ID didn't work either, back to the drawing board. I noticed that the search page was dynamically updating the results with javascript scripts, so that must mean that the page is a Single Page Application (SPA)? Then in that case...

```javascript
const observer = new MutationObserver((mutationList, observer) => {
  const target = document.getElementById('eKIzJc')
  if (target) {
    console.log('found element', target)
    target.remove()
    observer.disconnect()
  }
})

const config = { childList: true, subtree: true };
observer.observe(document, config);
```
I used `MutationObserver` to observe changes made on the page, specifically to the AI Overview element. Hopefully this meant that the moment the page edits the space to show the loading dots in the AI Overview `div`, it'll remove it.

Aaaand I was wrong. Again.

I tried `console.log`'ing every single detected change:
```javascript
const observer = new MutationObserver((mutationList, observer) => {
  console.log(mutationList)
})
```
With the debugger, I noticed that the script just straight up wasn't firing at all while the page was loading in.

And that's when I realised that on default, chrome extensions don't run on document start, they run on document idle. God I already had an idea about this, but when I tried looking for 'run' on the Chrome Extensions Docs, I was looking for it in the References tab. Which only showed me results for `chrome.runtime`.

What's the difference between 'Develop' and 'Reference'? Why are the pages laid out this way? Why is the overall search on the Docs an AI search instead of a proper one like MDN? WTF.

So the answer was the [run_at](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts#run_time) field that's defined in the `manifest.json`. Bruh.
```json
"content_scripts": [
  {
    "js": ["scripts/content.js"],
    "matches": [
      "https://www.google.com/search?q=*"
    ],
    "run_at": "document_start"
  }
]
```
And it works perfectly now! I double checked with the debugger and the AI Overview only exists for one frame before it gets completely removed. I didn't expect a small simple script to need this much debugging. But TIL (again) that docs are confusing to navigate and it's a skill issue of mine :)

# Plans for the future
Of course, there's still a lot more to implement with this little extension.
* Remove the AI Answers in the 'People also ask' section
* Make sure this also works on mobile
* Add other Google regions
* Create an in-extension on/off toggle
I'll keep the rest of it on my Github.
