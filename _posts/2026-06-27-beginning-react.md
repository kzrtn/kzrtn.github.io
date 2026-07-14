---
layout: post
title:  "Beginning React: What"
date:   2026-06-27 00:00:00 +0000
categories:
---
I have decided to quit CS50W and begin Full Stack Open instead. In my eyes it appears much more well rounded than CS50W as it covers RESTful, containers and more. It has all of the things that I wanted to delve into after CS50W. So, why not just drop CS50W and start this one now?

### Figuring how React works (The 'what')
React is so difficult to wrap my mind around. But in general, I feel this way towards any incredibly high level language. Tutorials simply tell you “when you declare/write ... etc etc, you MUST do this.” But why?

There are many more new things to follow when working with something like React. For example:
{% highlight jsx %}
function ChatMessage(props) {
  return (
    <div>
      {props.message}
      <img src="media/user.png" width="50px"/>
    </div>
  )
}

const App = (
  <>
  <ChatInput />
  <ChatMessage message="hello" />
  </>
{% endhighlight %}
In C, props in the function argument would be illegal, as the `ChatMessage` function call in `App` does not explicitly pass any variables/arguments into the function. Heck, the fact that there’s a `const` variable containing what looks like HTML that is also able to call other functions in the file is incomprehensible to me.

The answer is that JSX is syntax that gets translated into javascript by Babel afterwards. What is:
{% highlight jsx %}
<ChatMessage message="hello" />
{% endhighlight %}
is translated into:
{% highlight jsx %}
React.createElement(ChatMessage, { message: "hello" })
{% endhighlight %}

Which made me think… And realise that the way Babel works is not so foreign after all! In CS50W project 1, creating your own markdown to HTML translator with regex is an optional challenge. Babel follows the same principle. Apparently, Babel is what is called a source-to-source translator (aka transpiler). Though Babel’s own website calls itself a compiler. Is it a transpiler or compiler? [Stackoverflow users say that they’re both](https://stackoverflow.com/questions/43968748/is-babel-a-compiler-or-transpiler).

### Other notes
In any case, my arduous journey of learning Reacts syntax and their huge library officially begins...

```
Guard Operator (&&)
Const result = value1 && value2;
If value1 is true, the result will be value2. This works like an if-statement.
```

This is some Javascript syntax that’s useful to remember. I can use this for inserting if-statements directly in the JSX that renders HTML components. Like so:
{% highlight jsx %}
return (
  <div>
    {sender === 'robot' && <img src="media/robot.png" width="50"/>}
    {message}
    {sender === 'user' && <img src="media/user.png" width="50"/>}
  </div>
)
{% endhighlight %}
Note to self: I really need to properly look into arrow functions and what’s the deal with them.

<br>

[Previous Post](../../../2026/06/24/not-google.html) | [Next Post](../../../2026/07/01/react-states.html)