---
layout: post
title:  "Electronic Delivery Order Service"
date:   2026-07-23 00:00:00 +0000
categories:
---
# The Purpose
Currently at my workplace, we use physical delivery order (DO) books to write DOs. It’s pretty cumbersome and extra work doing manual data entry whenever I get the signed copies back for invoicing. Creating an electronic version of this system would make my life so much easier, I could essentially invoice the moment a DO is signed because everything is already digital.
<br />
<br />

# Timeline of Events
###  The Start
I managed to put together the order creation page, with the list of already created DOs at the bottom of the page. Clicking the previous orders loads the order into the template above. When you sign an order and save it, the next time you open the order, it’ll show the image of the signature instead of the signature box.

<image src='../../assets/images/initial_look.gif' height='500px' style='border: 1px solid #e6e6e6; padding: 20px'></image>

I’m glad I managed to get this proof of concept working, but it’s only the frontend (completely static data as well, no JSON-server or whatnot) and I’ve accidentally coded myself into a corner.

Realistically, if I wanted to view an order that has been created but not signed yet, why would it show up in this editable format? Same with orders that have been completely signed. I was going about this project in the wrong manner. I've pretty much created an order creation page and tried squeezing in the rest of the features into the form I created.
<br />
<br />

###  Figuring the Rough Page Layout
I need multiple pages, which means I need to figure out how the component tree is supposed to look like. I need to plan around them. I could just hop onto using [React Router](https://reactrouter.com/) for the multiple pages though. It looks like the best solution.
<image src='../../assets/images/page_layout.png' width='500px' style='border: 1px solid #e6e6e6; padding: 20px'></image>

The bottom three pages are just copies of each other, so they could just be one page with components swapped in and out based on order status and passed in argument variables.

I decided to keep pressing forward just to see how the app would look in its most basic form:
<image src='../../assets/images/newer_look.gif' height='600px' style='border: 1px solid #e6e6e6; padding: 20px'></image>

I’m much more used to the way other stacks like django use templating to quickly add navbars and footers to every page. Leaving the backend to be the main driver in filling in the templates and pages is way better than React’s frontend way. I kind of dislike how React works.
<br />
<br />

###  A Surprise Problem
My codebase very quickly became very very messy. Many states and functions passed over everywhere and anywhere. I drew this component tree to illustrate:
<image src='../../assets/images/do_component_tree.png' style='border: 1px solid #e6e6e6;'></image>
[Click here to open image in a new tab for a closer look](../../assets/images/do_component_tree.png)

“There’s no way this can be normal”, and then I found out about [prop drilling and createContext](https://react.dev/learn/passing-data-deeply-with-context).

Ahh... Maybe instead of following tutorials like FSO, I should just be reading through all of the pages in React learn and the docs for everything connected? It truly is... a ‘read the docs’ moment 😭

Though... All of this seems ridiculous, is it me or this seems like a super dumb solution to a really dumb situation that is only possible in React because of React's horrifying component passing state ways?

**PAUSED: I’ll come back to this and refactor it with React Router and React Context later. I shall get back with FSO first.**