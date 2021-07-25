---
title: "TypeScript Function Type Parameter Contravariance"
date: 2021-07-25T10:13:40Z
image: /images/2021/07/confusion_tweet.png
summary: |
    While reading the excellent ["Programming TypeScript: Making Your JavaScript Applications Scale"](https://www.amazon.co.uk/Programming-TypeScript-Making-JavaScript-Applications/dp/1492037656/) by [@bcherny](https://twitter.com/bcherny) I managed to confuse myself utterly.  
    What does it mean for for function type parameters to be contravariant in TypeScript?  
    Let's find out!

draft: false
---

**Table of Contents**

{{< toc >}}


## My Contravariance Rabbit Hole

{{% note %}}
**Tip:** Skip to "Understanding the Question" if this bit is intimidating/boring as I explain all the terms you need to know after this introduction. Yes it's like a recipe blog, in that respect, don't judge me too harshly.
{{% /note %}}

While reading the excellent ["Programming TypeScript: Making Your JavaScript Applications Scale"](https://www.amazon.co.uk/Programming-TypeScript-Making-JavaScript-Applications/dp/1492037656/) by [@bcherny](https://twitter.com/bcherny) I managed to confuse myself utterly.

> In TypeScript, every complex type is covariant in its members—objects, classes, arrays, and function return types—with one exception: function parameter types, which are contravariant.  
> - [Programming TypeScript, Chapter 6](https://www.amazon.co.uk/Programming-TypeScript-Making-JavaScript-Applications/dp/1492037656/)

This bit in particular:
>  function parameter types [...] are contravariant.

I read as meaning the type of an argument of a function should be a supertype of the parameter of the function.

If this were true, then this code snippet would be valid:

{{< highlight typescript >}}
class SuperType {
    super: string = 'super'
}

class SubType extends SuperType {
    sub: string = 'sub'
}

function nonsenseFunction(a: SubType): void {}

nonsenseFunction(new SuperType)
// Argument of type 'SuperType' is not assignable to parameter of type 'SubType'.
{{</highlight>}}

Of course, it's not. It's complete nonsense. So that lead me down a rabbit hole re-reading the previous pargraphs over and over thinking I'd misunderstood something until eventually...

{{<tweet 1418896064434905092>}}

I'd successfully identified that we were talking about function types, but my journey was far from over. It was only after writing the entirety of this blog post that I felt like I had a good grasp on what was really being discussed.

We are, very simply talking about where one function type can be assigned to another.

{{< highlight typescript >}}
class SuperType {
    super: string = 'super'
}

class SubType extends SuperType {
    sub: string = 'sub'
}

// Define the function types
type TestCallbackFunctionType = (a: SuperType) => void
type CallbackFunctionType = (b: SubType) => void

// Initialise the variables with those types
let testCallbackFunction: TestCallbackFunctionType
let callbackFunction: CallbackFunctionType = () => {}

// Try and assign a function of type CallbackFunctionType to a function of type TestCallbackFunctionType
testCallbackFunction = callbackFunction
// Type 'CallbackFunctionType' is not assignable to type 'TestCallbackFunctionType'.
//   Types of parameters 'b' and 'a' are incompatible.
{{</highlight>}}

The above example highlights what the sentence was actually talking about. 

> A function's parameter type is not assignable to another function's parameter type if it is a subtype of the other function's parameter type.

This is because function type parameters must be *contra* variant not *co* variant like other complex types.

Understanding what this means and why it is the case will be the subject of this blog post.

{{% note %}}
**Note:**   
I'm going to keep using the phrase "function type parameter" as opposed to "function parameter type" as to my untrained ear the second still sounds like we're talking about the types of the arguments that a function will accept. However, I recognise this confusion is all my own doing and both phrasings mean the same thing!
{{% /note %}}

## Understanding the Question

To sum it up, the question is: why does the following not throw an error?

{{< highlight typescript >}}
class SuperType {
    super: string = 'super'
}

class SubType extends SuperType {
    sub: string = 'sub'
}

function test(a: SuperType): void {}

// Call test with a subtype of what it asked for
test(new SubType)
{{</highlight>}}

But this does?

{{< highlight typescript >}}
function test(f: (a: SuperType) => void): void {}

// Call test with a subtype of what it asked for
test((a: SubType): void { })
//  Argument of type '(a: SubType) => void' is not assignable to parameter of type '(a: SuperType) => void'.
//    Types of parameters 'a' and 'a' are incompatible.
{{</highlight>}}

In other words, why is it okay to pass a subtype as an argument to a function, but not as a parameter for a function passed as an argument?

{{% note %}}
**Reminder:**  
A **parameter** is the variable listed inside the parentheses in the function definition.  
An **argument** is the value that is sent to the function when it is called.  
\- [StackOverflow](https://stackoverflow.com/a/59928588/336783)
{{% /note %}}


### Subtypes and Supertypes

Because TypeScript is structurally typed, `A` is a subtype of `B` if all of `B`'s members are found in `A`.  
This is perhaps more intuitive if you think of `A` as extending `B`, as `A` will then naturally have *at least* all the members `B` does.

A supertype is just the opposite relationship. `B` is a supertype of `A` if all of `B`'s members are found in `A`.
### Covariance and Contravariance

* **Covariance:** You want a `T` or a subtype of `T`
* **Contravariance:** You want a `T` or a supertype of `T`


### What is a function type?

A function type is a type definition for a function. This can be declared either as a type alias, e.g.

{{<highlight typescript>}}
type TestCallbackFunction = (a: SuperType) => void
function test(f: TestCallbackFunction) { }
{{</highlight>}}

or as a function type expression:

{{<highlight typescript>}}
function test(f: (a: SuperType) => void): void { }
{{</highlight>}}

They are a way to annotate the types of the arguments of and return of a function.

### Why are most types covariant?

The purpose of restricting type assignability is to ensure type safety (i.e. prevent type errors).
Restricting type assignment to either covariance (only allowing a type to be assigned to its supertypes or its structural equal) or contravariance (only allowing a type to be assigned to subtypes or its structural equal) is intended to ensure that whatever code is being executed does not attempt to perform operations that the type it's performing them on does not support.

To put this in real terms, one of the most common scenarios in which we rely on type assignability is when passing an argument into a function.

{{<highlight typescript>}}
function shoutyGreeter(name: string): void {
    console.log("HELLO", name.toUpperCase())
}
shoutyGreeter("Sam")
// HELLO SAM
{{</highlight>}}

In our call to `shoutyGreeter` with the argument `"Sam"` TypeScript is making sure that the string `"Sam"` is covariant (a subtype of or structurally equal) to the type `string`.

Great! Simple. We don't want an object to be passed in that is a supertype, or an unrelated type to `string` as we're calling `.toUpperCase()` on it and that method won't be present on say, a `number`.

### What are function type parameters?

When considering the type assignability of function type parameters, we're trying to figure out whether the parameter of one function type is assignable to the parameter of another function type.



The most common scenario for caring about function type parameter assignability is when you're passing a function as an argument to another function.

{{< highlight typescript >}}
function test(f: (a: SuperType) => void): void {}

test((a: SubType): void { })
// Argument of type '(a: SubType) => void' is not assignable to parameter of type '(a: SuperType) => void'.
//   Types of parameters 'a' and 'a' are incompatible.
{{</highlight>}}

TypeScript is telling us that the type of parameter `a` (`SubType`) in the callback function we're passing in as an argument to `test` is not assignable to the type of parameter `a` (`SuperType`) in the function type of parameter `f` we're passing our callback into.

Let's unpack that a little. We are talking about a bunch of different types here, so let's give them all unique names:

{{<highlight typescript>}}
type TestCallbackFunctionType = (a: SuperType) => void
type TestFunctionType = (f: TestCallbackFunctionType) => void
let test: TestFunctionType = function (f) { }

type CallbackFunctionType = (b: SubType) => void
let callbackFunction: CallbackFunctionType = (b) => { }
test(callbackFunction)
//  Argument of type 'CallbackFunctionType' is not assignable to parameter of type 'TestCallbackFunctionType'.
//    Types of parameters 'b' and 'a' are incompatible.

{{</highlight>}}

The above is functionally equivalent to the previous snippet. 
The only differences are:

1. We've given all our types, parameters, and functions unique names
3. `callbackFunction`'s parameter has been renamed to `b` to make it easier to talk about.

TypeScript is comparing the type of parameter `b` of function type `CallbackFunctionType` with the type of parameter `a` of function type `TestCallbackFunctionType` and is complaining that `b` is not a supertype (or equal) to `a`. 

To restate it in the original terminology: TypeScript is complaining that function type parameter `b` is not contravariant to function type parameter `a`.

## Why are function type parameters contravariant?

In this scenario TypeScript is still trying to do the same thing it always does, make our type assignment as safe as possible.  
So the question is: "Why is it safer for function type parameter `b` to be contravariant to function type parameter `a`"?

It's safer because the function `test` which accepts the function parameter `f` of type `TestCallbackFunctionType` is responsible for defining what argument (and therefore what type) gets passed into `f`.  
This means that `test` is most likely passing a an object of type `SuperType` to `callbackFunction` as an argument, but `callbackFunction` is expecting a **`SubType`**, which will have methods/attributes that `SuperType` does not, and will therefore do all sorts of type unsafe things with our poor innocent `SuperType` object. 

In other words if we were to allow covariant function type parameters, the actual object passed into the callback function as an argument would most likely be contravariant with the parameter type and therefore be unsafe.

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

// Define our test function with a parameter that is a function that expects a SuperType object
function test(f: (a: SuperType) => void): void {
    f(new SuperType)
}

// Call our test function, passing in a function that expects a SubType object
test((b: SubType): void => {
    console.log(b.sub)
 })
{{</highlight>}}

Our callback function is expecting a `SubType` called `b` and so is going to call `console.log` with `b.sub` as an argument, which it thinks should be fine as objects of type `SubType` always have the attribute `sub`.  
**BUT** the callback function is being called inside `test` which is passing a new `SuperType` object into our callback `f` as that's what it said `f` should expect so that's what it's damn well going to give it.    
As `sub` doesn't exist on `SuperType`, our `console.log` would log `undefined` if TypeScript allowed it to get that far.

Let's walk through the code one more time:

1. We call `test()` passing in our callback function that expects a `SubType` as `b`
2. Test calls our callback `f(new SuperType)` passing in a new object of type `SuperType`
3. Inside our callback function, it runs `console.log(b.sub)` expecting `b` to be a `SubType`
4. `b` does not have a `sub` member and so `console.log()` will log `undefined`

So that's it basically, TypeScript prevents you from using covariant function type parameters because if it didn't, you'd end up potentially passing contravariant arguments into the function when it was called, which is fundamentally unsafe.

I hope this didn't hurt your head as much to read as much as it hurt mine to write, but either way thank you for bearing with it as writing this has helped me cement my understanding immensely!

## Troubleshooting


If you're thinking: "Hey, this whole thing is bogus, this doesn't error on my machine at all!".
Oops, I should have mentioned that, this only applies if you're using `"strict": true` in your `tsconfig.json`, enable that and you should see the error!

{{<highlight json>}}
{
    "compilerOptions": {
        "strict": true,
    }
}
{{</highlight>}}

## Further Reading

* [What are covariance and contravariance? - Stephan Boyer](https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)
* [More on Functions - TypeScriptLang](https://www.typescriptlang.org/docs/handbook/2/functions.html)
* [Type Compatibility - TypeScriptLang](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)
* [TypeScript 2.6 - Strict function types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-6.html)
