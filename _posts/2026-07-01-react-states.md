---
layout: post
title:  "React States and Derived Variables for Display of Data"
date:   2026-07-01 00:00:00 +0000
categories:
---
Today I finished part 1 of FSO.

![Image](/assets/images/image3.png)

I thought I understood how states worked, but turns out, I don’t. Now, my understanding of it was simply “any element that refreshes its data on the page due to user interaction should use state”. But I realised that I didn’t need to use state for the most votes calculation (and display). 
{% highlight jsx %}
const App = () => {
  const [anecdotesObj, setAnecdotesObj] = useState(anecdotes);
  const [selected, setSelected] = useState(0);
};

const Display = ({header, anecdotesObj, selected}) => {
  return (
    <div>
        <h1>{header}</h1>
        {anecdotesObj[selected].quote}
        <p>has {anecdotesObj[selected].votes} votes</p>
    </div>
  );
};

const MostVotes = (anecdotesObj) => {
  let highestVote = {
    index: 0,
    anecdote: "",
    votes: 0
  };
 
  anecdotesObj.forEach((element, index) => {
    if (highestVote.votes < element.votes) {
      highestVote = element;
      highestVote.index = index;
    }
  });
  return highestVote.index;
};
{% endhighlight %}

From my primitive understanding of React, it appears that I can directly calculate the most votes because it derives its data from calculating the `anecdotesObj`. And each time a new vote is cast (and `setAnecdotesObj` is used), it will refresh `anecdotesObj`, causing React to re-render `App`, re-calling the `MostVotes` function.
 
Also, I had no idea that in Javascript, primitive types are copied by value, while objects (including arrays and functions) are copied by reference. So it’s like... C? Except with support for objects, and much more, of course.

### Off-topic-ish Ramble
Those marathon length tutorial videos, turns out, are actually really useful and are the best thing ever! All this time I’ve been fumbling in the dark when all I needed was the foundational knowledge of computer science subjects, these videos and some documentation and I’d no longer be perpetually confused!

The people who yell “go build projects instead of following tutorials” are wrong. I tried doing that for years and would only get confused in my own search. Missing missing reasons is a massive obstacle for someone who doesn’t even understand the basics of x framework, library or language. People seem to forget that.

Or perhaps... I’m just too bad at this? I don’t think so.

<br>

[Previous Post](../../../2026/06/27/beginning-react.html) | [Next Post](../../../2026/07/07/useful-javascript.html)