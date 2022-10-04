# Everyday Types

## Primitives

JavaScript has three very commonly used primitives: string, number and boolean. Each has a
corresponding type in TypeScript.

The type names String, Number, and Boolean are legal but refer to special built-in types.

## Arrays

To specify the type of an array you can use the syntax `number[]`. You may also see this written as
`Array<number>`, which means the same thing.


## any

TypeScript also has a special type, `any`, that you can use whenever you don't want a particular
value to cause typechecking errors.

When a value is of type `any`, you can access any properties of it, call it like a function,
assign to it (or from) a value of any type and pretty much anything else that is syntactically
legal.

```
let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```


### noImplicitAny

When you don't specify a type, and TypeScript can't infer it from context, the compiler will
typically default to `any`.

Use the compiler flag `noImplicitAny` to flag any implicit any as an error.


## Type Annotations on Variables

When you declare a variable using `const`, `var`, or `let`, you can optionally add a type
annotation to explicitly specify the type of the variable.

`let myName: string = "Alice";`


Whenever possible, TypeScript tries to automatically infer the types in your code.

## Functions

Functions are the primary means of passing data around in JavaScript. TypeScript allows you to
specify the types of both the input and output values of functions.

## Parameter Type Annotations

```
function greet(name: string) {
    console.log("Hello, " + name.toUpperCase() + "!");
}
```

Even if you don't have type annotations on your parameters, TypeScript will still check that you
passed the right number of arguments.


You can also add return type annotations:

```
function getFavoriteNumber(): number {
    return 7;
}
```


### Anonymous Functions

When a function appears in a place where TypeScript can determine how it's going to be called, the
parameters of that function are automatically given types.

The context in which the function occurs determines what type it should have.

## Object Types

Apart from primitives, the most common sort of type you'll encounter is an object type. This refers
to any JavaScript value with properties, which is almost all of them. To define an object type,
we simply list its properties and their types.

For example, here's a function that takes a point-like object:

```
function printCoord(pt: {x: number, y: number}) {
    console.log("The coordinate's x value is " + pt.x);
    console.log("The coordinate's y value is " + pt.y);
}

printCoord({x: 3, y: 7});
```

The type part of each property is optional. If it is not specified, it will be assumed to be any.


### Optional Properties

Object types can also specify that some or all of their properties are optional. To do this, add a
? after the property name:

```
function printName(obj: {first: string; last?: string} {
    // ...
}

// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last : "Alisson" });
```

In JavaScript, if you access a property that doesn't exist, you'll get the value `undefined` rather
than a runtime error. Because of this, when you read from an optional property, you'll have to
check for `undefined` before using it.

```
function printName(obj: { first: string; last?: string }) {
    if (obj.last !== undefined) {
        // OK
        console.log(obj.last.toUpperCase());
    }
 
    // A safe alternative using modern JavaScript syntax:
    console.log(obj.last?.toUpperCase());
}
```
## Union Types

TypeScript's type system allows you to build new types out of existing ones using a large variety
of operators.

### Defining a Union Type

function printId(id: number | string) {
    console.log("Your ID is: " + id);
}

// OK
printId(101);

// OK
printId("202");


TypeScript will only allow an operation if it is valid for every member of the union. For example,
if you have the union `string | number`, you can't use methods that are only available on `string`.

The solution is to narrow the union with code, the same as you would in JavaScript without type
annotations. Narrowing occurs when TypeScript can deduce a more specific type for a value based
on the structure of the code.

For example, TypeScript knows that only a string value will have a typeof value "string":

```
function printId(id: number | string) {
    if (typeof id === "string") {
        // In this branch, id is of type 'string'
        console.log(id.toUpperCase());
    } else {
        // Here, id is of type 'number'
        console.log(id);
    }
}
```

Another example is to use a function like `Array.isArray`:

```
function welcomePeople(x: string[] | string) {
    if (Array.isArray(x)) {
        // Here: 'x' is 'string[]'
        console.log("Hello, " + x.join(" and "));
    } else {
        // Here: 'x' is 'string'
        console.log("Welcome lone traveler " + x);
    }
}
```

## Type Aliases

A type alias is a name for any type. An example:

```
type Point = {
    x: number;
    y: number;
};

// Exactly the same as the earlier example
function printCoord(pt: Point) {
    console.log("The coordinate's x value is " + pt.x);
    console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

You can use a type alias to give a name to any type at all, for example:
```
type ID = number | string;
```

When you use the alias, it's exactly as if you had written the aliased type.


## Interfaces

An interface declaration is another way to name an object type:

```
interface Point {
    x: number;
    y: number;
}

function printCoord(pt: Point) {
    console.log("The coordinate's x value is " + pt.x);
    console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
```

Type aliases and interfaces are very similar, and in many cases you can choose between them freely.
Almost all features of an `interface` are available in `type`. The key distinction is that a type
cannot be re-opened to add new properties while an interface is always extendable.

Pripor to TypeScript version 4.2, type alias names may appear in error messages, sometimes in place
of the equivalent anonymous type. Interfaces will always be named in error messages.

Type aliases may not participate in declaration merging, but interfaces can.

Interfaces may only be used to declare the shapes of objects, not rename primitives.

Interface names will always appear in their original form in error messages, but only when they
are used by name.


Example of extending an interface:
```
interface Animal {
    name: string
}

interface Bear extends Animal {
    honey: boolean
}

const bear = getBear()
bear.name
bear.honey
```

Adding new fileds to an existing interface:
```
interface Window {
    title: string
}

interface Window {
    ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

Extending a type via intersections:
```
type Animal = {
    name: string
}

type Bear = Animal & {
    honey: boolean
}

const bear = getBear();
bear.name;
bear.honey;
```
A type cannot be changed after being created:

```
type Window = {
    title: string
}

type Window = {
    ts: TypeScriptAPI
}

// Error: Duplicate identifier 'Window'.
```

## Type Assertions

You can use type assertions to specify a more specific type:
```
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

You can also use the angle-bracket syntax (except if the code is in a `.tsx` file), which is
equivalent:

```
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

Because type assertions are removed at compile-time, there is no runtime checking associated with
a type assertion. There won't be an exception or null generated if the type assertion is wrong.

TypeScript only allows type assertions which convert to more specific or less specific version
of a type. This rule prevents "impossible" coercions like:
```
const x = "hello" as number;
```

Sometimes this rule can be too conservative and may disallow more complex coercions that might be
valid. If this happens, you can use two assertions, first to `any` (or `unknown`) then to the
desired type:
```
const a = (expr as any) as T;
```

# Literal Types

In addition to general types string and number, we can refer to specific strings and numbers in
type positions.

```
let changingString = "Hello World"; // (let changingString: string)
changingString = "Olá Mundo";

const constantString = "Hello World"; // (const constantString: "Hello World")
```

By themselves, literal types aren't very valuable:

Literal types are very useful when you combine them into unions. You can express much more useful
concepts, for example, functions that only accept a certain set of known values:

```
function printText(s: string, alignment: "left" | "right" | "center") {
    // ...
}

printText("Hello, world", "left");
```

You can combine literal types and non-literal types:

```
interface Options {
    width: number;
}

function configure(x: Options | "auto") {
    // ...
}

configure({ width: 100 });
configure("auto");
```

The type `boolean` itself is just an alias for the union `true | false`.


### Literal Inference

When you initialize a variable with an object, TypeScript assumes that the properties of that object migh change values later. For example, if you wrote code like this:
```
const obj = { counter : 0 };

if (someCondition) {
    obj.counter = 1;
}
```

TypeScript does not assume the assignment of 1 to a field which previously had 0 to be an error. Another way of saying this is that `obj.counter` must have the type `number` not `0`, because types are used to determine both reading and writing behavior.


The same applies to strings:
```
const req = { url: "https://example.com", method: "GET"};
handleRequest(req.url, req.method);
```

Because code can be evaluated between the creation of `req` and the call to `handleRequest` which could assign a new string to `req.method`, TypeScript considers this code to have an error.

There are two ways to work around this:

1. You can change the inference by adding a type assertion in either location:
```
const req = { url: "https://example.com", method: "GET" as "GET"}
handleRequest(req.url, req.method as "GET")
```

2. You can use `as const` to convert the entire object to be type literals:
```
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

The `as const` suffix acts like `const` but for the type system, ensuring that all properties are assigned the literal type instead of a more general version like `string` or `number`.

### null and undefined

JavaScript has two primitive values used to signal absent or uninitialized value: `null` and `undefined`.

TypeScript has two corresponding types by the same names. How these types behave depends on whether you have the `strictNullChecks` option on.

#### strictNullChecks off

Values that might be null or undefined can still be accessed normally, and the values `null` and `undefined` can be assigned to a property of any type. This is similar to how languages without null checks (e.g. Java) behave.
The lack of checking for these alues tends to be a major source of bugs. It is always recommended to turn `strictNullChecks` on if it's practical to do so in the codebase.

#### strictNullChecks on

With `strictNullChecks` on, when a value is `null` or `undefined`, you will need to test for those values before using methods or properties on that value. Just like checking for `undefined`  before using an optional property, we can use narrowing to
check for values that might be `null`:

```
function doSomething(x: string | null) {
    if (x === null) {
        // do nothing
    } else {
        console.log("Hello, " + x.toUpperCase());
    }
}
```

#### Non-null Assertion Operator (Postfix !)

TypeScript also has a special syntax for removing `null` and `undefined` from a type without doing any explicit checking. Writing `!` after any expression is effectively a type assertion that the value isn't `null` or `undefined`:

```
function liveDangerously(x? : number | null) {
    // No error
    console.log(x!.toFixed());
}
```

Just like other type assertions, this doesn't change the runtime behavior of your code, so it's important to only use `!`  when you know that the value can't be `null` or `undefined` 


### Enums

Enums are a feature added to JavaScript by TypeScript which allows for describing a value which could be one of a set of possible named constants. Unlike most TypeScript features, this is not a type-level addition to JavaScript but something added to the language and runtime.

### Less Common Primitives

Here we briefly mention the rest of the primitives in JavaScript which are represented in the type system.

#### bigint

From ES2020 onwards, there is a primitive in JavaScript used for very large integers, BigInt:

```
const oneHundred: bigint = BigInt(100);  // creating a bigint via the BigInt function
const anotherHundred: bigint = 100n;  // creating a BigInt via the literal syntax
```

#### symbol

There is a primitive in JavaScript used to create a globally unique reference via the function `Symbol()`:

```
const firstName = Symbol("name");
const secondName = Symbol("name");

// the following condition will always return false since the types 'typeof firstName' and 'typeof secondName' have no overlap
if (firstName === secondName) {
    // Can't ever happen
}
```

