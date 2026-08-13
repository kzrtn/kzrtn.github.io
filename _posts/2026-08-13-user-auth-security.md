---
layout: post
title:  "Referencing across collections with Mongoose and User Authentication"
date:   2026-08-13 00:00:00 +0000
categories:
---
## Referencing Collections With Mongoose
MongoDB can use the objectID type to reference documents in other collections, similar to using foreign keys in relational databases (like sqlite). We can establish connections between collections within a database that way.

In relational databases, the `JOIN` query links two tables together. But for a noSQL database like MongoDB, there isn't a way to join queries likes that. Mongoose can take care of joining and aggregating data (which gives the appearance of a join query), however it is actually making multiple queries to the database in the background.

So, in order to reference across collections, a *reference key* is needed in the schema:
```javascript
const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true, // ensures uniqueness of username
  },
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})
```
The `ref: 'Note'` references the name of the collection that you wish to reference. Now querying for users will include an array of `Note` IDs that belongs to that user.

However, what if we want that array of `Note` IDs to show the contents of the notes as well? To do that, we need to join the collections. While document databases do not properly support join queries between collections, the Mongoose library can do some of these joins for us. The Mongoose join is done with the [populate](https://mongoosejs.com/docs/populate.html) method.
<br />
<br />

## User Authentication
Creating users with Express and Mongoose is not too difficult. Users are simply another form of data. However, the important distinguishing bit is the passwords.

**Passwords should never be stored as plaintext in the database**.

A few rules I've learned as I go through FSO:
1. Passwords should be validated by both front and backend before getting hashed
2. The plaintext password only lives for a moment on the backend as the backend handles logic (validation, hashing)
3. The backend only passes the hashed password value to the database.
4. If the backend needs to respond with user data, it is necessary to delete the `passwordHash` from the response

The fourth one is the most interesting. The backend fetching data from the database will always retrieve the password hash as MongoDB stores data indiscriminately (it does not differentiate between any data nor validate/modify/transform any of them). However, it's practical to remove the `passwordHash` field with Mongoose the moment the data reaches the backend and from responses.

FSO handles removing it from responses by transforming the schema:
```javascript
const userSchema = new mongoose.Schema({
  username: String,
  name: String,
  passwordHash: String,
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})

userSchema.set('toJSON', {
  transform: (document, returnedObject) => {
    returnedObject.id = returnedObject._id.toString()
    delete returnedObject._id
    delete returnedObject.__v
    // the passwordHash should not be revealed
    delete returnedObject.passwordHash
  }
})

const User = mongoose.model('User', userSchema)
```
Whenever `User` is called to construct a new user, all fields are available. `transform` is applied to documents of the `User` class that gets called `toJSON()`, such as:
```javascript
const users = await User.find({})
// console.log(users) will not include
// passwordHash on any of the returned objects
```
`JSON.stringify()` is called within `res.json(user)` by Express, so the `transform` fires. `console.log` also transforms documents, because Mongoose has a handler that calls `toJSON()` internally.

Basically, FSO's method strips sensitive data from data on its way out of the backend.

Alternatively, the other method is to remove it during querying time:
```javascript
const userSchema = new mongoose.Schema({
  username: String,
  name: String,
  passwordHash: {
    type: String,
    select: false
  }
  notes: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Note'
    }
  ],
})
```
`select: false` unselects the data. It only affects queries (`find`, `findOne`, `findById`, etc.), so the `passwordHash` gets stripped entirely on the way into the database. It has no effect on document creation.

In order to get it back during querying, `.select('+passwordHash')` needs to be used:
```javascript
const user = await User.findOne({ username }).select('+passwordHash')
```

The two methods deal with the two halves of the problem: stripping the data coming in and stripping the data going out of the backend. Using both is not uncommon, as they do not conflict with each other.
<br />
<br />

## Hashing and Salt Rounds
FSO uses [bcrypt](https://github.com/kelektiv/node.bcrypt.js) to hash passwords.

FSO also links[ this resource](https://bytebytego.com/guides/how-to-store-passwords-in-the-database/) as a comprehensive guide on storing passwords safely.

What's a salt round used by bcrypt? According to a stack overflow answer, it's more like a cost factor. It means how many times bcrypt's algorithm is used to calculate a single bcrypt hash.

The algorithm version, the salt and the number of salt rounds is included in the final hashed result in readable form, so storing the hashed result stores all of that data as well.

The salt is closer to a Minecraft seed, the salt rounds is the number of times bcrypt repeats its hashing algo for the final output.

So when bcrypt hashes an inputted password (such as a user logging in) to compare to the hashed password in the database (their hashed password saved in the database referencing their account), it reads the hashed password's algorithm version, salt, the salt rounds and uses that to recompute the inputted password and compare if they match.

<br />

[Previous Post](../../../2026/08/09/testing-envp-await.html) | Next Post