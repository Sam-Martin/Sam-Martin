---
title: "Typescript Function Parameter Contravariance"
date: 2021-07-25T10:13:40Z
draft: false
---

# The question

While reading the excellent ["Programming TypeScript: Making Your JavaScript Applications Scale"](https://www.amazon.co.uk/Programming-TypeScript-Making-JavaScript-Applications/dp/149203765) by [@bcherny](https://twitter.com/bcherny) I managed to confuse myself utterly.

{{<tweet 1418896064434905092>}}

To sum it up, the question is: Why does the following not throw an error?

{{< highlight typescript >}}
function test(a: SuperType): void {}

test(new SubType)
{{</highlight>}}

But this does?

{{< highlight typescript >}}
function test(f: (a: SuperType) => void): void {}

test((a: SubType): void { })
//  Argument of type '(a: SubType) => void' is not assignable to parameter of type '(a: SuperType) => void'.
//    Types of parameters 'a' and 'a' are incompatible.
{{</highlight>}}

In other words, why is it okay to pass a subtype as an argument to a function, but not as a parameter for a function passed as an argument?

{{% note %}}
**Reminder:**
> A **parameter** is the variable listed inside the parentheses in the function definition.  
> An **argument** is the value that is sent to the function when it is called.  
> \- [StackOverflow](https://stackoverflow.com/a/59928588/336783)
{{% /note %}}

## Covariance and Contravariance

> In TypeScript, every complex type is covariant in its members—objects, classes, arrays, and function return types—with one exception: function parameter types, which are contravariant.  
> - [Programming Typescript, Chapter 6](https://www.amazon.co.uk/Programming-TypeScript-Making-JavaScript-Applications/dp/149203765)

So this is the _only_ example of type contravariance in TypeScript.  
You can always assign subtypes to their supertypes (e.g. pass in an object of a subtype as an argument to a parameter that expects a supertype) except when those types are **parameters in function types**.



## What is a function type?

A function type is a type definition for a function. This can be declared either as a type alias, e.g.

{{<highlight typescript>}}
type TestCallbackFunction = (a: SuperType) => void
function test(f: TestCallbackFunction) { }
{{</highlight>}}

or as a function type expression as we did in the original example:

{{<highlight typescript>}}
function test(f: (a: SuperType) => void): void { }
{{</highlight>}}

### Why are most types covariant?

The purpose of restricting type assignability is to ensure type safety (i.e. prevent type errors).
Restricting type assignment to either covariance (only allowing subtypes to be assigned to supertypes or equals) or contravariance (only allowing supertypes to be assigned to subtypes or equals) is intended to ensure that whatever code is being executed does not attempt to perform operations that the type does not support.

To put this in real terms, one of the most common scenarios in which we rely on type assignability is when passing an argument into a function.

{{<highlight typescript>}}
function shoutyGreeter(name: string): void {
    console.log("HELLO", name.toUpperCase())
}
shoutyGreeter("Sam")
// HELLO SAM
{{</highlight>}}

In our call to `shoutyGreeter` with the argument `"Sam"` Typescript is making sure that the string `"Sam"` is covariant (a subtype of or equal) to the type `string`.

Great! Simple. We don't want an object to be passed in that is a supertype, or an unrelated type of `string` as we're running `.toUpperCase()` against it and that method won't be present on say, a `number`.

### What are function type parameters?

When considering the type assignability of function **type** parameters, we're trying to figure out whether the parameter of one function type is assignable to the parameter of another function type.

The most common scenario for caring about that is when you're passing a function as an argument to another function, as we saw with our initial test case.

{{< highlight typescript >}}
function test(f: (a: SuperType) => void): void {}

test((a: SubType): void { })
// Argument of type '(a: SubType) => void' is not assignable to parameter of type '(a: SuperType) => void'.
//   Types of parameters 'a' and 'a' are incompatible.
{{</highlight>}}

TypeScript is telling us that the type of parameter `a` (`SubType`) in the callback function we're passing in as an argument to `test` is not assignable to the type of parameter `a` (`SuperType`) in the function type of `f` which is in turn defined as a parameter to the function `test`.

Let's unpack that a little. We are talking about a bunch of different types here, so let's give them all unique names:

{{<highlight typescript>}}
type TestCallbackType = (a: SuperType) => void
type TestFunctionType = (f: TestCallbackType) => void
let test: TestFunctionType = function (f) { }

type CallbackFunctionType = (b: SubType) => void
let callbackFunction: CallbackFunctionType = (b) => { }
test(callbackFunction)
//  Argument of type 'CallbackFunctionType' is not assignable to parameter of type 'TestCallbackType'.
//    Types of parameters 'b' and 'a' are incompatible.

{{</highlight>}}

The above is functionally equivalent to the previous snippet. 
The only differences are:

1. We've given all our types and parameters unique names
3. The `callbackFunction` parameter has been renamed to `b` to make it easier to talk about.

Typescript is comparing the type of parameter `b` of function type `CallbackFunctionType` with the type of parameter `a` of function type `TestCallbackType` and is complaining that `b` is not a supertype (or equal) to `a`. 

To restate it in the original terminology: TypeScript is complaining that function type parameter `b` is not contravariant to function type parameter `a`.

### Why are function type parameters contravariant?

In this scenario TypeScript is still trying to do the same thing it always does, make our type assignment as safe as possible.  
So the question is: "Why is it safer for function type parameter `b` to be contravarient to function type parameter `a`"?

It's safer because the function `test` which accepts the function parameter `f` of type `TestCallbackType` is responsible for defining what argument (and therefore what type) gets passed into `f`.  
This means that `test` is most likely passing a an object of type `SuperType` to `callbackFunction` as an argument, but `callbackFunction` is expecting a **`SubType`**, which will have methods/attributes that `SuperType` does not, and will therefore do all sorts of type unsafe things with our poor innocent `SuperType` object. 

In other words if we were to allow covariant function type parameters, the actual object passed into the function as an argument would most likely be contravariant and therefore unsafe.

### Hypothetical covariant function type parameter example

Okay that's a lot of words, what would the problem look like in practice?  
Let's go back to our simpler version of the problem.

{{< highlight typescript >}}
class SuperType {
    super: string = 'super'
}

class SubType extends SuperType {
    sub: string = 'sub'
}


function test(f: (a: SuperType) => void): void {
    f(new SuperType)
}

test((b: SubType): void => {
    console.log(b.sub)
 })
{{</highlight>}}

Our callback function is expecting a `SubType` as `b` and so is going to call `console.log` with `b.sub` as an argument, which should be fine as objects of type `SubType` always have the attribute `sub`.  
**BUT** the callback function is being called inside `test` which is passing a `SuperType` into `f` as that's what it said `f` should expect so it's damn well gonna give that to it.  
As `sub` doesn't exist on `SuperType`, our `console.log` would log `undefined` if TypeScript allowed it to get that far.

So that's it basically, TypeScript prevents you from using covariant function type parameters because if it didn't, you'd end up passing in contravariant arguments into the function when it was called, which is fundamentally unsafe.

# Further Reading

* [What are covariance and contravariance? - Stephan Boyer](https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)
* [More on Functions - TypescriptLang](https://www.typescriptlang.org/docs/handbook/2/functions.html)
