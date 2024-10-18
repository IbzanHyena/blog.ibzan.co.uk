// title: A Proof of Cantor’s Diagonal Argument in Agda
// date: 2024-10-18
// author: Ibzan

# A Proof of Cantor’s Diagonal Argument in Agda

One of my favourite proofs in mathematics is [Cantor’s diagonal argument][CDA].
It’s an elegant way of proving that there are sets with a greater cardinality than the naturals.
I remember [reading][cabinet] about it when I was around eight or nine in the context of [Hilbert’s hotel][Hilbert's hotel].
The problem was built up, starting from the single new guest on their own, then the (countably) infinitely many new guests in a coach, and after that infinitely many coaches each with infinitely many new guests.

The real killer was when a single countably infinitely long coach turned up with seats that were numbered with the reals.
At first, the hotel manager tries the same trick as with a single natural-numbered coach, but, by the diagonal argument, not everyone is accommodated.
When I grasped the proof, I was struck with the simplicity of the mechanism involved, and the relative power of the result.
Even better, it’s a _constructive_ proof, providing a way of actually _finding_ an example missing element!
That means you can verify it (if you have time to check the infinitely-long list of reals).

-----

A while back, I got into a “discussion” about [Turing completeness][Turing completeness] (or, perhaps more pedantically, the [_Entscheidungsproblem_][Decision problem]) and whether it had actually been proven or not—spoiler alert, it has—but that sparked a thought: is it possible to write a proof for the diagonal argument using [Agda][Agda]?

Given the simplicity, it seemed fairly straightforward.
Happily, it was!

To start, there’s just the front matter of declaring a module and importing things.

```agda
module Cantor where
<wbr>
open import Data.Bool using (Bool; not)
open import Data.Bool.Properties using (not-¬)
open import Data.Nat using (ℕ)
open import Data.Product using (_×_; _,_; ∃; ∃-syntax)
open import Function using (_$_)
open import Relation.Nullary using (¬_)
open import Relation.Binary.PropositionalEquality using (_≡_; refl)
```

Definitions for these terms can be found in the [standard library][agda-stdlib], but I won’t go into detail here.

To simplify the implementation of the problem, I chose to work in binary rather than decimal.
A sequence of ones and zeroes can be thought of as a base-2 real in the [0,1) interval, just as with the 0–9 digits in base-10.
This makes writing a later function a bit easier without sacrificing any important generality.

To encode these binary numbers in Agda, I use a bitstring—essentially, an ordered sequence of bits with the cardinality of the naturals.
Instead of using an explicit collection like a list, I use a function.
That just means I don't have to evaluate the whole string to reach a given element.
The input to the function is the index into the bitstring that we want to retrieve; members of this set are functions that fit this signature.

```agda
tfʷ : Set
tfʷ = ℕ → Bool
```

The mnenomic for the name here is “true—false to the power of omega”, where omega is the cardinality of the naturals.

Now that we have a single real, we need an infinite number of them!
We can do this in much the same way as before:

```agda
Ω : Set
Ω = ℕ → tfʷ
```

where instead of using an explicit collection, a function will suffice.
The input this time is the index of the _bitstring_ we want to retrieve, and again we are defining a set whose members are functions.

Now that we have something that should represent an uncountable set, we need to build the machinery to prove that.
The first thing we will need is a type that contains evidence that two bitstrings are not identical.
(Recall that types in Agda represent proofs or properties that we have, and their values are the evidence.)
For this, we define a new set:

```agda
data _≠_ (a₁ a₂ : tfʷ) : Set where
  different : (n : ℕ) → ¬ (a₁ n ≡ a₂ n) → a₁ ≠ a₂
```

To construct this type, we provide two bits of evidence: an index, and proof that the bits in the two strings are different from each other at that index.
Keep in mind that `a₁` and `a₂` here are both functions that return bits, so `a₁ n` is the nth bit in that bitstring.

Next, we need to encode a way of showing that a bitstring is not contained in an instance of `Ω`, our countably infinite set of bitstrings.
To construct a member of this type, we will need both an instance of `Ω`, an individual bitstring not present in it, and the evidence of that difference in at least one place from our earlier `_≠_` definition.

```agda
data _∉_ (r : tfʷ) (ω : Ω) : Set where
  notin : ((n : ℕ) → ((ω n) ≠ r)) → r ∉ ω
```

This is the most complex signature yet.
To break it down, we will take just the sole input (in parentheses): `(n : ℕ) → ((ω n) ≠ r)`.
This is a function, which takes an index, and produces the evidence that the nth item of our countably infinite collection of bitstrings is not equal to the bitstring we are testing for membership of.
In particular, this function is the constructive part of the proof—it takes an arbitrary index and returns a counterexample.

We’re nearly done now.
To use `_∉_`, we must have a function that generates counterexample.
That is to say, given an `Ω`, it must return a bitstring that will not be present in our `Ω`.
Fortunately, with `Bool`s, this is quite simple: we just have to invert the diagonal!

```agda
constructMissing : Ω → tfʷ
constructMissing ω = λ n → not $ ω n n
```

`ω n n` is just the nth bit of the nth bitstring—the diagonal—and then we flip it with `not`.
To demonstrate briefly with an example:

```
1: 000000…
2: 111111…
3: 010101…
4: 101010…
5: 001001…
6: 100100…
…
<wbr>
diagonal: 010000…
inverted: 101111…
```

Since this is different to each member of `ω` in at least one place, it can’t be in `ω`.
To tie it all together, we just have to combine everything into a statement of that fact.

```agda
cantor : ∀ {ω} → ∃[ r ] (r ∉ ω)
cantor {ω} = constructMissing ω , notin λ n → different n (not-¬ refl)
```

The input to our proof function is a specific `Ω` that we wish to test.
By using curly brackets, we can have it be provided implicitly by Agda; this would matter at a potential call site, but it isn’t especially impactful here.

The “there exists” type has two parts in its signature: an object, and a property of that object.
For us, the object is a bitstring that will not be present in `ω`, and the property is that proof.
Constructing the type is just providing instances of those things; a bitstring, and the evidence that it is not contained in `ω`.

We can make the bitstring with our `constructMissing` function, and then we need only create the evidence.
To do that, we need a new function, here just a lambda which takes any given index and constructs our `_≠_` by using the definition of `constructMissing`.

And that’s all it takes!
The Agda compiler will verify this for us.
If you wanted to run code, you could get the language to print out an n-by-n square of bits and construct the missing item for you, but from a correctness perspective that isn’t necessary, thanks to the type system.

## Further work

The main thing I would change in this code is extending the proof to work with arbitrary types, not just bitstrings.
To do this, I would need to adjust my definitions of `tfʷ` and `Ω`.
`Ω` is simple—it just needs to be parametric over the actual type it returns—but the `tfʷ` would have to be a record that contains its own `constructMissing` function.
I believe that after doing that the rest of the code should be pretty straightforward, just stitching things together.

## Conclusion

I hope this has been an interesting brief read!
When I set out to write this, I wasn’t certain how hard it would be to prove such an interesting argument, but fortunately the verbal simplicity is matched by this formulation, needing only twelve lines of my own definitions to make.

Given how I enjoyed writing this code, I hope in future to work on some more (hopefully?) small proofs in Agda, such as the infinity of primes or Fermat’s little theorem.
I can highly recommend playing around with Agda (or another proof assistant language) if you are interested in learning about different kinds of programming languages and their applications.

[Agda]: https://wiki.portal.chalmers.se/agda/pmwiki.php
[agda-stdlib]: https://agda.github.io/agda-stdlib/v2.0/
[cabinet]: https://books.google.co.uk/books?id=yhu6PAAACAAJ
[CDA]: https://en.wikipedia.org/wiki/Cantor%27s_diagonal_argument
[Decision problem]: https://en.wikipedia.org/wiki/Entscheidungsproblem
[Hilbert's hotel]: https://en.wikipedia.org/wiki/Hilbert%27s_paradox_of_the_Grand_Hotel
[Turing completeness]: https://en.wikipedia.org/wiki/Turing_completeness
