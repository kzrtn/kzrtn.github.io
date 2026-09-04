---
layout: post
title:  "React Router lied to me."
date:   2026-08-30 00:00:01 +0000
categories:
---
### Multiple Pages aren't multiple
I've been pretty excitedly waiting for this part of FSO. I put [an entire project on pause](../../../projects/delivery-order/) because of a React-induced state hell and the need for multiple pages.

One of the big issues I ran into during that project was discovering that in a simple SPA React app, the way to create the illusion of multiple pages was the have a master state that keeps track of what page is currently active. This added onto the many, MANY states that I needed to pass down per page. I had hoped that with React Router, the prop drilling could be mitigated slightly.

I had envisioned that multiple pages in React to work like *actual* different pages would in a traditional HTML style. That each page had it's own Javascript files and whatnot alongside of it. Which, in my mind, meant that I could cap the state hell per page, each page works on its own set of API calls to the backend and such.

But no. I was wrong. I can't believe React is THIS convoluted.

There are no different pages. React Router only makes it easier by that master page state, but it also updates the URL of the window to make it *look* like there's multiple pages. It's still only a single page.

WTF.

But I guess it does help with loading specific documents via ID fetching from the backend via useParams. Which would simplify utils usage for loading new orders, saving orders and marking orders as fulfilled. Which will undercut some of the prop drilling.

I suppose the final piece to this puzzle is React's useContext, which I will cross that bridge when I need to.

And yes, since React is an abstraction of traditional website methods, it is important to remember to use `<Link>` to link to external websites instead of using the old `<a href="..."></a>`

### Styling Components with libraries
Pretty straightforward if we're to use a React Component Library like [Material UI](https://mui.com/). We simply `npm install` it as always and then import the components we want to use. As always, there's a lot of documentation reading. As component libraries aren't styled with inline React CSS methods (usually) and the components have their own props for styling.

FSO also goes through [styled-components](https://styled-components.com/), which has an interesting way to define styles:
```javascript
import styled from 'styled-components'

const Button = styled.button`
  background: Bisque;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid Chocolate;
  border-radius: 3px;
`

const Input = styled.input`
  margin: 0.25em;
  width: 300px;  
`
```
styled-components is stronger than inline CSS, because it now supports:
* psuedo-classes (`:hover`, `:focus`)
* keyframe animations
* actual CSS classes

And since they're scoped within the component, there's no need to worry about global collisions like plain CSS files.

### Accessible Rich Internet Applications (ARIA)
When I was working on my Gemini blocker extension, I spent hours looking through Google Search's source code, I saw a lot of components with attributes that started with the word 'aria' like `aria-label` and `aria-hidden`.

Turns out, they're a set of HTML attributes that give screen readers information about the component (much like the `<label>` tag) since custom component are just *really* heavily styled `<div>`s that act as other things (like buttons). Screen readers have no way of knowing what they are, actual button elements would tell them that it's a button and is clickable, but not a `<div>`

Component libraries like MUI have these already have these inside each component by default, so there's no need to include them. However, if we're creating our own heavily styled divs with styled-components, then it's on us to add the right ARIA for accessibility.
<br />
<br />

[Previous Post](../../../2026/08/30/react-testing.html) | [Next Post](../../../2026/09/04/flux-zustand.html)