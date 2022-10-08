# Narrowing

Imagine we have a function called `padLeft`:

```
function padLeft(padding: number | string, input : string): string {
    return " ".repeat(padding) + input;  // ERROR
}
```
The example produces an error. We can use type narrowing to fix the error:
```
function padLeft(padding: number | string, input : string): string {
    if (typeof padding === "number") {
        return " ".repeat(padding) + input;
    } else {
        return padding + input;
    }
}
```

TypeScript overlays type analysis on JavaScript's runtime control flow constructs like if/else, conditional ternaries, loops, truthiness checks, etc., which can all affect those types.

The `typeof padding === "number"` in the example is called a type guard.

TypeScript follows possible paths of execution that our program can take and analyses the most specific possible type of a value at a given position. It looks at these special checks (called type guards) and assignments, and the process of refining types to more specific types than declared is called narrowing.

There are a couple different constructs TypeScript understands for narrowing.

## typeof Type Guards

As we've seen, JavaScript supports a `typeof` operator which can give very basic information about the type of values we have at runtime. TypeScript expects this to return a certain set of strings:
- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

Like we saw with `padLeft`, this operator comes up pretty often in a number of JavaScript libraries, and TypeScript can understand it to narrow types in different branches.

In TypeScript, checking against the value returned by `typeof` is a type guard. Because TypeScript encodes how `typeof` operates on different values, it knows about some of its quirks in JavaScript.

```
function printAll(strs: string | string[] | null) {
    if (typeof strs === "object") {
        for (const s of strs) {
            // Object is possibly 'null'.
            console.log(s);
        }
    } else if (typeof strs === "string") {
        console.log(strs);
    } else {
        // do nothing
    }
}
```

In JavaScript, `typeof null` is actually `object`.

## Truthiness Narrowing

In JavaScript, constructs like `if` first "coerce" their conditions to booleans and then choose their branches depending on whether the result is `true` or `false`. Values like:
- `0`
- `NaN`
- `""`
- `0n`
- `null`
- `undefined`

All coerce to `false`, and other values get coerced `true`. You can always coerce values to `boolean` by running them through the `Boolean` function, or by using the shorter double-Boolean negation:
```
// both of these result in `true`
Boolean("hello");
!!"world";  // type: true, value: true
```

It's fairly popular to leverage this behavior, especially for guarding against values like `null` or `undefined`. As an example, let's try using it for our `printAll` function:

```
function printAll(strs: string | string[] | null) {
    if (strs && typeof strs === "object") {
        for (const s of strs) {
            console.log(s);
        }
    } else if (typeof strs === "string") {
        console.log(strs);
    }
}
```

## Equality Narrowing

TypeScript also uses `switch` statements and equality checks to narrow types. For example:

```
function example(x: string | number, y: string | boolean) {
    if (x === y) {
        // we can now call any 'string' method on 'x' and 'y'.
        x.toUpperCase();
        y.toLowerCase();
    } else {
        console.log(x);
        console.log(y);
    }
}
```

When we checked that `x` and `y` are both equal in the above example, TypeScript knew their types also had to be equal. Since `string` is the only common type that both `x` and `y` could take on, TypeScript knows that `x` and `y` must be a string in the first branch.

Checking against specific literal values (as opposed to variables) works also:

```
function printAll(strs: string | string[] | null) {
    if (strs !== null) {
        if (typeof strs === "object") {
            for (const s of strs) {
                console.log(s);
            }
        } else if (typeof strs === "string") {
            console.log(strs);
        }
    }
}
```

Checking whether something `== null` actually not only checks whether it is specifically the value `null` - it also checks whether it's potentially `undefined`. The same applies to `== undefined`: it checks whether a value is either `null` or `undefined`.

```
interface Container {
    value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
    // Remove both 'null' and 'undefined' from the type.
    if (container.value != null) {
        console.log(container.value);

        // now we can safely multiply 'container.value'.
        container.value *= factor;
    }
}
```

## The in Operator Narrowing

JavaScript has an operator for determining if an object has a property with a name: the `in` operator. TypeScript takes this into account as a way to narrow down potential types.

For example, with the code `"value" in x`. Where `"value"` is a string literal and `x` is a union type. The "true" branch narrows `x`'s types which have either an optional or required property `value`, and the false branch narrows the types which have an optional or missing property `value`.

```
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
    if ("swim" in animal) {
        return animal.swim();
    }

    return animal.fly();
}
```

Optional properties will exist in both sides for narrowing, for example a human could both swim and fly (with the right equipment) and thus should show up in both sides of the `in` check:
```
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
    if ("swim" in animal) {
        animal;  // (parameter) animal: Fish | Human
    } else {
        animal;  // (parameter) animal: Bird | Human
    }
}
```

## instanceof Narrowing

JavaScript has an operator for checking whether or not a value is an "instance" of another value. More specifically, in JavaScript `x instanceof Foo` checks whether the prototype chain of `x` contains `Foo.prototype`.

`instanceof` is also a type guard, and TypeScript narrows in branches guarded by `instanceof`.

```
function logValue(x: Date | string) {
    if (x instanceof Date) {
        console.log(x.toUTCString());
    } else {
        console.log(x.toUpperCase());
    }
}
```

## Assignments

As we mentioned earlier, when we assign to any variable, TypeScript looks at the right side of the assignment and narrows the left side appropriately.
```
let x = Math.random() < 0.5 ? 10 : "hello world!";
// let x: string | number

x = 1;
console.log(x);
// let x: number

x = "goodbye!";
console.log(x);
// let x: string
```

Notice that each of these assignments is valid. Even though the observed type of `x` changed to `number` after our first assignment, we were still able to assign a `string` to `x`

This is because the declared type of `x` - the type that `x` started with - is a `string | number`, and assignability is always checked against the declared type.

If we had assigned a `boolean` to `x`, we would have seen an error since that was not a part of the declared type.

```
let x = Math.random() < 0.5 ? 10 : "hello world!";
x = 1;
console.log(x);
// let x: number

x = true;
// ERROR
// type 'boolean' is not assignable to type 'string | number'

console.log(x);
// let x: string | number
```

## Control Flow Analysis

Up until this point, we've gone through some basic examples of how TypeScript narrows within specific branches.

```
function padLeft(padding: number | string, input: string) {
    if (typeof padding === "number") {
        return " ".repeat(padding) + input;
    }
    return padding + input;
}
```

TypeScript was able to analyze this code and see that the rest of the body (`return padding + input`) is unreachable in the case where `padding` is a `number`. As a result, it was able to remove `number` from the type of `padding` (narrowing from `string | number` to `string`) for the rest of the function.

This analysis of code based on reachability is called control flow analysis, and TypeScript uses this flow analysis to narrow types as it encounters type guards and assignments. When a variable is analyzed, control flow can split off and re-merge over and over again, and that variable can be observed to have a different type at each point.

```
function example() {
    let x: string | number | boolean;
    
    x = Math.random() < 0.5;

    console.log(x);
    // let x: boolean

    if (Math.random() < 0.5) {
        x = "hello";
        console.log(x);
        // let x: string
    } else {
        x = 100;
        console.log(x);
        // let x: number
    }
    return x;
    // let x: string | number
}
```

## Using Type Predicates

We've worked with existing JavaScript constructs to handle narrowing so far.

To define a user-defined type guard, we simply need to define a function whose return type is a type predicate.

```
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
```

`pet is Fish` is our type predicate in this example. A predicate takes the form `parameterName is Type`, where `parameterName` must be the name of a parameter from the current function signature.

Any time `isFish` is called with some variable, TypeScript will narrow that variable to that specific type if the original type is compatible.

```
let pet = getSmallPet();

if (isFish(pet)) {
    pet.swim();
} else {
    pet.fly();
}
```

TypeScript  not only knows that `pet` is a `Fish` in the `if` branch; it also knows that in the `else` branch, you don't have a `Fish`, so you must have a `Bird`.

You may use the type guard `isFish` to filet an array of `Fish | Bird` and obtain an array of `Fish`:

```
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()]
const underWater1: Fish[] = zoo.filter(isFish);
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];  // equivalent

// The predicate may need repeating for more complex examples

const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
    if (pet.name === "sharkey") return false;
    return isFish(pet);
})
```

In addition, classes can use `this is Type` to narrow their type.

## Discriminated Unions

Let's imagine we're trying to encode shapes like circles and squares. Circles keep track of their radiuses and squares keep track of their side lengths.

```
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}
```

We can write a `getArea` function that applies the right logic based on if it's dealing with a circle or square. We'll first try dealing with circles:

```
function getArea(shape: Shape) {
    return Math.PI * shape.radius ** 2;
}
```

Under `strictNullCehcks` that gives us an error as `radius` might not be defined.

What if we perform the appropriate checks on the `kind` property?

```
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius ** 2;
        // Object is possibly 'undefined'.
    }
}
```

We could use a non-null assertion (`!`) to say that `radius` is definitely present.

A better approach:

```
interface Circle {
    kind: "circle";
    radius: number;
}

interface Square {
    kind: "square";
    sideLength: number;
}

type Shape = Circle | Square;
```

Here, we've properly separated `Shape` out into two types with different values for the `kind` property, but `radius` and `sideLength` are declared as required properties in their respective types.

```
function getArea(shape: Shape) {
    if (shape.kind === "circle") {
        return Math.PI * shape.radius ** 2;
    }
}
```

In this case `kind` is the common property (discriminant property of `Shape`). Checking whether the `kind` property was `"circle"` got rid of every type in `Shape` that didn't have a `kind` property with the type `"circle"`. That narrowed `"shape"` down to the type `Circle`.

The same checking works with `switch` statements as well. Now we can try to write our complete `getArea` without any pesky `!` non-null assertions.

```
function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
    }
}
```

Discriminated unions are useful for representing any sort of messaging scheme in JavaScript, like when sending messages over the network or encoding mutations in a state management framework.

## The never Type

When narrowing, you can reduce the options of a union to a point where you have removed all possibilities and have nothing left. In those cases, TypeScript will use a `never` type to represent a state which shouldn't exist.

## Exhaustiveness Checking

The `never` type is assignable to every type: however, no type is assignable to `never` (except `never` itself). This means you can use narrowing and rely on `never` turning up to do exhaustive checking in a switch statement.

For example, adding a `default` to our `getArea` function which tries to assign the shape to `never` will raise when every possible case has not been handled.

```
type Shape = Circle | Square;

function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
        default:
            const _exhaustiveCheck: never = shape;
            return _exhaustiveCheck;
    }
}
```

Adding a new member to the `Shape` union, will cause a TypeScript error:

```
interface Triangle {
    kind: "triangle";
    sideLength: number;
}

type shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
    switch (shape.kind) {
        case "circle":
            return Math.PI * shape.radius ** 2;
        case "square":
            return shape.sideLength ** 2;
        default:
            const _exhaustiveCheck: never = shape;
            return _exhaustiveCheck;
    }
}
```

