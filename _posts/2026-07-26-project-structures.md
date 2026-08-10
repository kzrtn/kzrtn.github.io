---
layout: post
title:  "Clean Express Backend Project Structuring and More Javascript Operators"
date:   2026-07-26 00:00:00 +0000
categories:
---
Roughly a week off of FSO makes me feel a bit nervous, I’ve only finished the first 3 parts! Although, I’ve already learned so much. I went back to look at the [start up example for the discord bots](https://docs.discord.com/developers/quick-start/getting-started) and I can actually follow what’s going on now. Since the backend used is node express, I’m pretty sure I can actually link a discord bot to applications I create in FSO. It is only a matter of login and authentication now (to which I’m not certain on how to implement just yet).

<br>
## More Javascript Operators
I found out about even more question mark operators in Javascript that I had no idea about. Looks like Javascript overloads `?` for a few different things, let’s compile them here:

**1. Ternary operator `?`:**

The classic conditional expression has been around since C.
```js
const status = condition ? true : false
```

**2. Optional chaining `?`.**

This was a new one for me. It’s a way for an object to safely access a property/method only if the property/method before it isn’t `null`/`undefined`. Stops evaluation and returns `undefined` instead of throwing. A great way to test if a property exists in an object without using a try catch, which means you can further chain it with other operators.
```js
user?.profile?.name  // property access
user?.getName?.()  // method call that only calls if getName exists
arr?.[0] // array/computed access
```

**3. Nullish coalescing `??`**

It’s like `||`, it returns the right value only if the left value is `null` or `undefined`. `||` triggers the right value if the left is falsy. This version keeps it strict to `null` or `undefined`.
```js
const value = value1 ?? value2
//if value1 is not null/undefined, set to value1. Else set to value2
```

**4. Nullish coalescing assignment `??=`**

Basically an assignment version of nullish coalescing:
```js
options.timeout ??= 5000 //assigns only if the variable isn’t currently null or undefined
```

Speaking of the nullish coalescing assignment, there’s also the other assignments for the guard operator and OR operator: `&&=` and `||=`. They basically shorten things like:
```js
value &&= newValue //sets to newValue if value is truthy
value ||= newValue //value is set to newValue if value is falsy
```

<br>
## Express Project Structures
The first focus of FSO part 4A is on project structuring. Best practices when working with node/express is to split the app and server files. Splitting them will make it easier to maintain, re-use and test separately. This concept is called [separation of concerns](https://www.geeksforgeeks.org/software-engineering/separation-of-concerns-soc/).

In order to follow best practices, moving route handlers to a dedicated module is required. The router object on express is a middleware, something like a ‘mini-app’, it’s only able to perform middleware and routing functions. It’s self contained and has no server of its own. Its pathing is also relative to where it was mounted. E.g.
```js
const notesRouter = require('./controllers/notes.js')
app.use('/api/notes', notesRouter)
```
So instead of `.delete('/api/notes/:id', (request, response, next) => {`, it’s now just `.delete('/:id', (request, response, next) => {` inside the `notesRouter` itself.

Also,
```js
const app = express()
```
This creates a top level application object. It’s the actual server instance that listens to the port on the machine. Which means that backend software has no awareness of domains, DNS, or URLs at all. It’s the thing above the backend that handles the translation of (URL -> DNS -> IP -> backend url) like Heroku/Render/etc.

What exactly handles the translation on PaAS is a reverse proxy (nginx, a cloud load balancer, etc). It is the thing that listens on the public-facing port (442), terminates HTTPS, and then forwards the request internally to whatever the app is actually running. [This video](https://youtu.be/L0jMBrCEQNQ?si=tyoUGPt1AYDSB5ZX) is a fantastic resource that explains how nginx works internally.

I looked more into express project structuring and [this article](https://www.dhiwise.com/post/express-js-folder-structure-best-practices-for-clean-code) lays it out in more detail than FSO. Also, FSO’s note project structure is missing a services folder. Some of the files in FSO are combined together (E.g. controller file is combined with the service file, the route handler functions include logic that manipulates the data)

<br>
Project structure should be as follows:

### controllers
* For files that deal with HTTP requests
* Validates/parses inputs
* Calls appropriate service/model
* const noteService = require('../services/noteService')

```js
const getAllNotes = async (req, res) => {
  const notes = await noteService.getAll()
  res.json(notes)
}

const createNote = async (req, res) => {
  const savedNote = await noteService.createNote(req.body)
  res.status(201).json(savedNote)
}

module.exports = { getAllNotes, createNote }
```

### models
* Database stuff. Data schema definitions

```js
const noteSchema = new mongoose.Schema({
  content: String,
  important: Boolean
})

module.exports = mongoose.model('Note', noteSchema)
```

### services
* For program logic, like calculations, data handling etc
* The stuff handles the manipulation of data, the stuff the controller will call for their HTTP requests
* Doesn’t contain raw connection/query code for the database (that should be in model instead)

```js
const Note = require('../models/note')

const createNote = (body) => {
  const note = new Note({
    content: body.content,
    important: body.important || false,
  })
  return note.save()
}

module.exports = { createNote }
```

### utils
* Small pure functions like date formatting, currency conversion etc
* Type of tools that can be copy pasted into completely different projects without any modification whatsoever

```js
exports.formatDate = (date) =>
  new Intl.DateTimeFormat('en-US').format(date);
```

### routes
* Maps the URL/methods to controller functions
* Should not contain any logic

```js
const notesRouter = require('express').Router()
const notesController = require('../controllers/notes')

notesRouter.get('/', notesController.getAllNotes)
notesRouter.post('/', notesController.createNote)

module.exports = notesRouter
```

### config
* For configuration files (e.g. dotenv etc)

It's going to take a lot of practice before I can instinctively separate things out like so. That's many files and folders to traverse.

<br>

[Previous Post](../../../2026/07/16/cors-deployment.html) | [Next Post](../../../2026/08/09/testing-envp-await.html)