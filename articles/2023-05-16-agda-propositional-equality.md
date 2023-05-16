// title: Agda: What Even is Propositional Equality?
// date: 2023-05-16
// author: Ibzan

# Agda: What Even is Propositional Equality?

_I would like to apologise in advance for the inevitable semantic satiation you will experience when reading the word &ldquo;type&rdquo; in this post. Sorry._

If you ever write [Agda][Agda], you will come across the following import statement in practically every piece of code you read:

```agda
open import Relation.Binary.PropositionalEquality
```

Followed by things like:

```agda
1+1≡2 : 1 + 1 ≡ 2
1+1≡2 = refl
```

Equality in Agda is a fundamental part of its proof system, but the definition can be a bit mind-warping.
So, let&rsquo;s explore what it means for things to be equivalent in Agda.

## Function signatures and types

To understand what equality means and how it is written in Agda, we must first understand how type signatures work in functional languages.
If you are already familiar with them, I&rsquo;ll be talking about how they&rsquo;re different in a dependently-typed language, too.

Most programmers come from an proceduaral background, and proceduaral languages tend to use a syntax for functions descended from either C or Pascal.

```c
int add(int a, int b);
```

```pascal
function add(a: int, b: int): int;
```

>>> Sometimes the terms _imperative_ and _procedural_ are used interchangably.
They are, however, two distinct paradigms of programming, albeit with a large overlap.  
_Imperative_ programming refers to the use of sequential statements to change a program&rsquo;s state.
It focuses on describing exactly how the program works, step-by-step.  
_Procedural_ programming refers to languages that use procedures (a.k.a. functions (a.k.a. subroutines (a.k.a. methods (a.k.a. &hellip;))))
Each procedure contains a sequence of instructions to be executed.  
Many languages are both.

Let&rsquo;s compare and contrast quickly.

- Both styles have all the parameters within parentheses
- In C-style syntax, the types come first, followed by the name
- In Pascal-style syntax, the names come first, and the types are separated by a colon

It follows from those last two points that the return type of the function goes either before or after the name, the same as with parameters and variables.
Furthermore, when making function calls in these languages, one has to provide all the parameters at once.

In addition to function signatures, these languages also often have associated function _types_:

```
typedef int int_2adic(int, int);
```

```
type Int2Adic = function(a: int, b: int) : int;
```

These function types follow the same pattern as the signatures themselves.

The proceduaral way is not the only way
Within functional programming, there are also two main styles &mdash; Haskell-style, and ML-style.

```
add :: int -> int -> int
add a b = a + b
```

```
let add (a: int) (b: int): int = a + b
```

Again, let&rsquo;s compare these.

- In Haskell-style, the type signature and the names of the parameters are _separate_
- ML-style is very similar to Pascal-style
- These signatures need to have an accompanying body

Why is that last point relevant here?
Simply put, it&rsquo;s because the functions both also have types, similar to the procedural languages!
Both of them use the same syntax, derived from mathematics:

```
int -> int -> int
```

This is exactly what is on the right hand side of the `::` in Haskell-style syntax.
In fact, `::` in Haskell descendents just specifies the type of the term to its left.
The ML-based languages use this too, but do not display it as explicitly in every function definition.

There&rsquo;s a key difference with functions in functional languages compared to procedural ones, and it&rsquo;s hinted to by these arrows.
In the procedural languages, the parameters were all grouped together.
In the functional languages, the parameters are separated by arrows.

To explain further, let&rsquo;s consider a 1-adic function:

```
int -> int
```

This function takes one parameter and returns a result.
You may notice that it looks like the end of 2-adic function signature, too.

```
int -> int -> int
       int -> int
```

That&rsquo;s because it is &mdash; the 2-adic function is _curried_.
Curried functions take their parameters one at a time, yielding a new function that takes the next until finally all the parameters have been given and the function can return its result.

There are advantages to this style of function which I won&rsquo;t go into here (maybe another time?) but are very interesting in their own right.

Agda&rsquo;s signatures are very similar to 

Now that that&rsquo;s out of the way, we can move on to the interesting concepts that follow from it in Agda.

## Types and functions in Agda

In Agda, types are defined by using the keyword `data`.
Being from a mathematical background, Agda refers to a type as `Set`.
If you prefer, you can just think of every instance of `Set` as saying `Type` instead.
For example, here&rsquo;s a definition of bools:

```agda
data Bool : Set where
  true false : Bool
```

This is definining two _constructors_ for `Bool`.
Each of these has type `Bool` &mdash; no additional information is needed to construct the type.
`true` and `false` can be used straight away as two values belonging to the type `Bool`.

>>> Note that we have not defined any behaviour for `Bool` yet!
That is up to us to do &mdash; things like Boolean algebra aren&rsquo;t magically granted to any type with constructors called `true` and `false`, but they do exist in the standard library.

Functions can be defined in a similar method to languages like Haskell:

```agda
_and_ : Bool → Bool → Bool
true and y = y
false and _ = false
```

Here, the underscores represent where the operands will go.
The function is defined by _pattern matching_ on the first operand, to cover all possible cases.

## Parametric types

Types in Agda can be more complex, too.
Just as functions in functional languages can take parameters, so can _types_ in Agda.
As a basic example, here&rsquo;s the popular `Maybe` (or `Option`) type:

```agda
data Maybe (A : Set) : Set where
  just : A → Maybe A
  nothing : Maybe A
```

Our types just got more advanced!
This one takes a parameter, `A`, which is another type, and produces a type.
The `nothing` constructor isn&rsquo;t too dissimilar from `true` or `false`; by this point, all the required information has been provided, so it is just a value.

`just` is more involved &mdash; it involves a value of type `A` in order to make the `Maybe A`.
In other words, `just` is a function from `A` to `Maybe A`, hence the signature.
We can create instances of this type by using the constructors, just as with `Bool`:

```agda
nothing-Bool : Maybe Bool
nothing-Bool = nothing

just-true : Maybe Bool
just-true = just true

just-false : Maybe Bool
just-false = just false
```

## Going even further beyond

So far, our types have been based purely on other types at most.
`Bool`, the simplest case, didn&rsquo;t depend on anything.
`Maybe` required another type to construct.

Types could depend on _values_, as well as types.
This is what it means to be dependently-typed.
As a dependently-typed language, Agda supports this kind of type.

Enter the definition of propositional equality (with some simplifications for the sake of introduction).
Equality is a type that represents two things being identical.
It is called _propositional_ because for Agda to be able to check the type, the very definitions of the two sides must be the same.

```agda
data _≡_ {A : Set} (x : A) : A → Set where
  refl : x ≡ x
```

This is a pretty complex type, so the rest of the article will be explaining the collection of concepts going on here all at once, and what they entail.

Starting on the left, equality is called `_≡_` in Agda; a triple equals symbol, with an operand either side.
Then, we get the first parameter, `{A : Set}`.
Compared to the parameters we have seen so far, this one is in braces!
Braces around a parameter mark it as _implicit_, i.e., the type checker will attempt to infer the relevant type.
There is almost always enough information present for this to happen when the type is being used in its intended fashion.

The next parameter is `(x : A)`.
`A` is a type, so that means that `x` is a member of `A` &mdash; it is a value, not another type.
This is what makes equality dependently-typed in Agda.

Given these two inputs, we are given the signature for the type itself, which is a **function**!
`A → Set` means that we must be given a _second_ value of type `A` to complete the construction of this type.
Just as providing one parameter to a function in a functional language produces a _new_ function that takes the rest of the parameters, giving a type `A` implicitly and a value `x` explicitly gives us a function to construct the type with.

Now read there is a single constructor, called `refl`.
The _type_ of `refl` is `x ≡ x`.
This means that in order to create an instance of this type, the type checker **must** prove that the value on the left-hand side of the `≡` is the same as the value on the right-hand side.
In return for proving this, we are given an instance, `refl`, which serves as _evidence_ that this is true.
This evidence can then be used in other proofs without repeating the whole chain of reasoning that derives it.

Logically, it makes sense that the definition of equality is that any value is identical to itself.
If we have two values that look different at first, but can be shown to be equivalent, then `refl` can be used to construct equality.
If they _can&rsquo;t_, then we can&rsquo;t use `refl` &mdash; so there is no evidence, and no proof!

While this may not seem like a useful tool at first, the real power comes from how we manipulate the left-hand side to create this instance.
By applying properties known to the type checker, it can be guided through the steps to show that two different expressions are in fact the same.

### A simple pair of examples

Let&rsquo;s use our `Bool` type from earlier, and prove something evident from the definition of `_and_`.

```agda
and-false-identity : ∀ {b : Bool} → (false and b) ≡ false
and-false-identity = refl
```

`∀` means for all &mdash; the compiler _must_ show this property holds for every possible value.
Again, the braces mean that the value of `b` itself is implicit.
This pattern of providing the values of types implicitly in Agda is common &mdash; more of the attention is given to the _evidence_ that a property holds than the values themselves.
It also helps avoid syntactic noise in proofs.

The proof of this property should be fairly easy to follow; we start by asserting what we want to prove.
Then, we examine all the possible cases for the thing on the _left_, and check that they match the _right_.

Here, our left-hand side is `false and b`.
Consulting the definition of `_and_`, we get `false and _ = false`.
Hence, the left-hand side must be `false`, which is the same as `false` on the right, so we are allowed to use `refl`.

Similarly, we could show and identity for `true`:

```agda
and-true-identity : ∀ {b : Bool} → (true and b) ≡ b
and-true-identity = refl
```

This also follows straight from the definition of `_and_`.

## So what?

While we haven&rsquo;t done anything complicated logically yet, there&rsquo;s a lot of syntax to unpack here.
Using the dependent type system, we have managed to construct a type that represents two values being exactly equivalent.
Much of the art of Agda involves applying properties in some order to show that two things are identical.

The advantage of this is formal verification.

Lots of code is testable.
Lots of tests check for a few simple value, or use a table of outcomes.
We can assert that `1 + 1 == 2` and `1 + 2 == 3` and `1 + 3 == 4` and so on and so forth.

More sophisticated testing frameworks use something called _property-based testing_, which seeks to assert that some property holds for all inputs.
However, most languages cannot exhaustively test all possiblities; it would simply take too long to be sensible.
Instead, they generate random examples to try and cover a wide variety of cases.

Enter Agda.

Types in Agda are almost always built inductively &mdash; there is a base case, and a step.
If you can prove something holds for the base case, and it holds whenever you take that step, you have proved it holds for all possible values!
This concept, [induction][induction], is so powerful that it is used widely for proofs in mathematics.

Agda, and languages like it, bring that technique to computer science and programming.
Via the [Curry&ndash;Howard correspondence][Curry-Howard], mathmatial proofs and things shown by Agda&rsquo;s type system are isomorphic &mdash; they can be swapped between!
With this power, it is possible to prove that code will _always_ behave a certain way.
No randomness, no human-generated examples, no untested edge cases.
Just machine logic.

Unfortunately, with great power come a a great many problems.
First and foremost, interacting with the outside world is hard!
Without going into detail, some of the properties that Agda proves about programs makes them annoying to deal with input and output.
Secondly, the performance is not currently very good.
This is an area of active work, though, so there may be interesting developments soon!

## Conclusion

Briefly, we have explored the definition of equality in Agda.
This blog post serves as a primer to Agda materials that assume that you know what its version of equality is, but I also try to explain how it _works_.

If you&rsquo;re interested in more Agda writing, watch this space &mdash; I plan to write more soon.

>>> Some types in this post have been simplified slightly from their definitions in the standard library to avoid having to discuss the [sort system][Sort System] and [universe levels][Universe Levels].
If you are a set theorist who is worried about Russel&rsquo;s paradox, don&rsquo; lament &mdash; Agda has a solution!

[Agda]: https://wiki.portal.chalmers.se/agda/pmwiki.php
[Curry-Howard]: https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence
[induction]: https://en.wikipedia.org/wiki/Mathematical_induction
[Sort System]: https://agda.readthedocs.io/en/latest/language/sort-system.html
[Universe Levels]: https://agda.readthedocs.io/en/latest/language/universe-levels.html
