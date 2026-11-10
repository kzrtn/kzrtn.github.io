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

Typescript can often infer the type on its own, so for certain declarations, annotations are not needed. However, they're most useful in function parameters and other places where typing is not so clear, where there's no value to infer from:

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
 
if (maybe === true) { // Typescript knows that maybe is a boolean now
  const aBoolean: boolean = maybe;
  const aString: string = maybe; // Type 'boolean' is not assignable to type 'string'.
}
 
if (typeof maybe === "string") { // Typescript knows that maybe is a string
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

### unions
Unions don't exist in plain JavaScript. I've also only ever encountered them in a C way, where a union looks like a struct, but instead of each member having its own memory space and location, all of them share the same memory address. This means that it can only hold one member at a time. It's not something a programmer would normally touch unless they worked on low-level devices.

Unions exist as a type in Typescript. A union doesn't work like C memory-wise, but it does behave similarly in concept: it's a type with several options, and a value can only be one of them at a time.

```ts
type UserId = string | number;

let id: UserId = "42"; // OK
id = 42; // OK
id = true; // Error: Type 'boolean' is not assignable to type 'UserId'
```

They're normally combined with the literal type.

### literal
Strings and numbers can have a literal type. Where Typescript ensures that the type only has a specific string or number, like so:
```ts
type Hello = "HelloWorld";

let hello: Hello = "HelloWorld";
hello = "Hi!"; // Error: Type '"Hi!"' is not assignable to type Hello
```
Literals are commonly used together with unions:
```ts
type TrafficLight = "red" | "green" | "yellow";


let light: TrafficLight = "red"; // OK
light = "blue"; // Error: Type '"blue"' is not assignable to type 'TrafficLight'
```

<br />

## Object Types and Interfaces
Doing something like this is not good:

```ts
function parsePerson(person: object) {
  isString(person.name);
  isNumber(person.age);
}
```

Annotating the `person` parameter with just `object` only tells Typescript that it isn't a primitive value (string, number, etc). It doesn't know anything about the object's properties.

### Object Types
We can expand the description of types like so:

```ts
type PersonObject = {
  name: string;
  age: number;
  description?: string;
};
```

The **?** after a property denotes that it is optional, so the two declarations below are both valid:

```ts
const alice: PersonObject = {
  name: "Alice",
  age: 20,
  description: "Likes reading",
};

const bob: PersonObject = {
  name: "Bob",
  age: 12,
};
```

### Interface
`interface` is another way to declare a type of object. Interfaces can build on other interfaces using `extends`, which reports an error if properties conflict. They also support declaration merging, where declaring the same interface twice combines the two.

```ts
interface Person {
  name: string;
  age: number;
} // So far, very similar to an object type

interface Student extends Person {
  studentId: number;
}

// Student becomes an interface type with:
// Student: {
//   name: string;
//   age: number;
//   studentId: number;
// }

interface MarathonEntry {
  marathonId: number;
  timeInSeconds: number;
  location: string;
}
```

<br />
## Other Commonly Used Manipulations and Types
### Intersections
Types can be intersected by using `&`. Following our example from earlier:

```ts
type MarathonStudent = Student & MarathonEntry;

// MarathonStudent becomes an object type with:
// MarathonStudent: {
//   name: string;
//   age: number;
//   studentId: number;
//   marathonId: number;
//   timeInSeconds: number;
//   location: string;
// }
```

### Generics
The hardest part for me to wrap my head around honestly. If I'm not wrong, this is a concept that also appears in C++, but I'm not familiar with C++ myself.

Generic types are like templates for types. You write out what the type looks like, but leave some of the types as placeholders. Later on, those placeholders are either filled in by you or inferred automatically (in generic functions). For example:

```ts
// Our generic Box type
interface Box<Type> {
  contents: Type;
}

// Our variable box is a Box of strings
let box: Box<string>;
```

When Typescript sees `Box<string>`, it replaces every instance of `Type` in `Box<Type>` with `string`, so it ends up working with something conceptually like this:

```ts
interface Box {
  contents: string;
}
```

This makes `Box` reusable, since `Type` can be substituted with anything:

```ts
interface Apple {
  color: string;
}

// Same as { contents: Apple }
type AppleBox = Box<Apple>;
```
Box and Apple examples are from the [Typescript docs](https://www.typescriptlang.org/docs/handbook/2/objects.html#the-array-type)

Functions can be generic too. Here, Typescript infers `Type` from whatever you pass in:

```ts
function first<Type>(items: Type[]): Type {
  return items[0];
}

const word = first(["a", "b"]); // Type is inferred as string
const num = first([1, 2, 3]);   // Type is inferred as number
```

Typescript sees whatever passed into the function as a whole, so passing an array of mixed types results in so:
```ts
const mixed = first(["a", 123]); // mixed is string | number
```

This would require further narrowing down the line like so:
```ts
mixed.toUpperCase(); // Error: Property 'toUpperCase' does not exist on type 'number'

if (typeof mixed === "string") {
  mixed.toUpperCase(); // OK
}
```

If `T` is set manually, Typescript checks every element against it:

```ts
first<string>(["a", 123]); // Error: Type 'number' is not assignable to type 'string'
```
### Arrays
The Array Type itself is a generic type (`string[]` is shorthand for `Array<string>`). Taken from the [Typescript docs](https://www.typescriptlang.org/docs/handbook/2/objects.html#the-array-type):
```ts
interface Array<Type> {
  /**
   * Gets or sets the length of the array.
   */
  length: number;
 
  /**
   * Removes the last element from an array and returns it.
   */
  pop(): Type | undefined;
 
  /**
   * Appends new elements to an array, and returns the new length of the array.
   */
  push(...items: Type[]): number;
 
  // ...
}
```

### Tuples
A tuple type is type of Array type that knows exactly how many elements it contains, and exactly which types it contains at specific positions.
```ts
type StringNumberPair = [string, number];
```

<br />

[Previous Post](../../../2026/10/03/graphql.html) | Next Post