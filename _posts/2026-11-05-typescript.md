---
layout: post
title:  "So that's why it's called Typescript?"
date:   2026-11-5 00:00:00 +0000
categories:
---
It took me awhile, but I'm finally here at Typescript. Someone told me that people don't write in Javascript anymore, the industry standard is Typescript. I don't know how true that is though.

I didn't understand what the difference between Typescript and Javascript was, but now I sure do. God it's SO strictly typed!

Syntatically, it's very different from other strongly typed languages like C++ and Java.

<br />

## The Basics
Typescript is Javascript with types added on top. It can't run directly on browsers like Javascript can, so it's compiled to plain Javascript first. A little like how JSX gets transpiled to Javascript. The whole point of Typescript is to help catch mistakes while code is being written, and to also encourage self documenting code. The types are all removed before the code is run.

It looks just like Javascript, types are annotated with a colon `:`. For example:
```ts
const someString: string = "This is a string";
```

TypeScript can often infer the type on its own, so for certain declarations, annotations are not needed. However, they're most useful in function parameters and other places where typing is not so clear, where there's no value to infer from:

```ts
// Here, we're writing that the input argument message is a string
// So Typescript expects a string to be passed in
function shout(message: string) {
  return message.toUpperCase();
}

shout("hello");
shout(42); // Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

<br />

## The New Types
### any
For when you want to opt out of type checking completely. Assigning a variable to `any` makes it accept any kind of type.

It's also a type that Typescript implicitly assigns when no type that can be inferred and you don't give it a type. Unlike `unknown`, `any` variables will still allow you to access arbitrary properties, even ones that don't exist. These properties include functions and Typescript will not check their existence or type.

It's like an escape hatch to go back to vanilla Javascript.

### unknown
Sometimes, you don't know what type the variable is while you're writing the code. The variable values may be coming from elsewhere (like from users). In these cases, `unknown` tells the compiler that the variable can be anything.

`unknown` variables can be narrowed down by using `typeof` checks, comparison checks, or more advanced type guards like so:
```ts
declare const maybe: unknown;
const aNumber: number = maybe; //Error: Type 'unknown' is not assignable to type 'number'.
 
if (maybe === true) { // TypeScript knows that maybe is a boolean now
  const aBoolean: boolean = maybe;
  const aString: string = maybe; // Type 'boolean' is not assignable to type 'string'.
}
 
if (typeof maybe === "string") { // TypeScript knows that maybe is a string
  const aString: string = maybe;
  const aBoolean: boolean = maybe; // Type 'string' is not assignable to type 'boolean'.
}
```
A value with type `unknown` MUST be narrowed before it can be used in a type-specific way. `any` can be narrowed too, but it's not necessary as Typescript lets you do almost anything with it.

### never
`never` is a weird one. It's a type mainly used to denote that a function never returns, such as functions that always throw an error or loops forever. In Javascript, `void` functions that doesn't return any value will implicitly return the value `undefined`, `never` guards against that. Annotating a function that can finish normally with `never` will cause Typescript to report an error.
```ts
function greet(x:string): void {
  console.log(`Hello ${x}`);
}

const wow = greet("world"); // wow becomes undefined

function fail(message: string): never {
  throw new Error(message);
} // Never finishes normally, so it never returns
```

It's also used to check that there's nothing left in a union:
```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // do something
  } else if (typeof x === "number") {
    // do something else
  } else {
    const unreachable: never = x; // has type 'never'!
    // This assignment works as a safety check. If someone changes the
    // function to accept more types, Typescript will show an error since
    // the new type cannot be assigned to 'never' in the else branch
  }
}
```
(Work in progress)

<br />

## Objects and Interfaces
Doing something like:
```ts
function parsePerson(person: object) {
  isString(person.name);
  isNumber(person.age);
}
```
Is not good. Because annotating the `person` variable simply with just `object` doesn't tell Typescript anything about it.

<br />

[Previous Post](../../../2026/10/03/graphql.html) | Next Post