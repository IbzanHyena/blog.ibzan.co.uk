// title: Thoughts on J
// author: Ibzan
// date: 2023-05-15

# Thoughts on J

I&rsquo;ve been writing [the J programming language][J] regularly since roughly June 2022, so I thought I&rsquo;d write a blog post about my experience with it so far.
Originally, I wrote this post all the way back in July 2022.
It is now&hellip; May 2023.
Oops.

Better late than never!

As a heads-up warning, this post has ended up being quite long.
It might take a long time to read (say, 60&ndash;75 minutes) and digest (depends how much you grok the array language concepts).

For the most part, J has been a great language with a lot of cool innovations compared to paradigms I&rsquo;ve worked with before.
I do have some gripes with the language as well, which I&rsquo;ll talk about too.
Finally, I&rsquo;ll cover some of the changes I would make to an array language like J if I were designing my own.

This article contains some moderately complex J code.
It isn&rsquo;t too important to understand to get the points in the article &mdash; feel free to skim over it as it comes up.
I&rsquo;ve provided surface-level explanations but refrained from digressing too far to fully explore the way the code works.

If you want to follow along with any of the examples, you may find the following useful:

- [NuVoc][nuvoc], the glossary of J vocabulary
- [J download][J-download], for installing yourself
- [J online][J-online], to just try out small snippets &mdash; follow along!

## Why did I start writing J?

Learning an array language had been on my mind since roughly late 2020 to early 2021.
The novelty of the paradigm was intriguing, and the idea of having a large number of primitives to perform what would be standard library functions or left to the programmer to implement was interesting.

The first array language I met was [APL][APL].
Without going into a detailed history, APL is the ancestor of modern array languages (and libraries, like [NumPy][NumPy]).  
APL is alien.  
It&rsquo;s exotic.  
It looks like this: `{↑1 ⍵∨.∧3 4=+/,¯1 0 1∘.⊖¯1 0 1∘.⌽⊂⍵}` (which implements a single step of [Conway&rsquo;s game of life, written by John Scholes][APL game of life]!).

The allure of the strange character set and array operations was undeniable.
I also found a lot of the example programs very elegant with explanations.
However, I didn&rsquo;t really want to go through the relative pain of learning an alternate keymap or shortcuts.
J&rsquo;s ASCII ended up being simpler for me to learn while still offering a good experience with array languages.
At some point, I may learn an array language that uses an atypical character set (most probably [BQN][BQN]).
Until then, J will remain my array language of choice.

J has served well as a calculator/spreadsheet drop-in.
A lot of numerical questions can be answered quickly with the J terminal open.
It&rsquo;s also a great quick distraction when I want to think about solving some tiny problem for a bit when I need a break.
Problem sites like [LeetCode][LeetCode] or [Project Euler][Euler] have no shortage of quick programming puzzles to solve.
For longer problems, [Advent of Code][aoc] also offers a good selection of challenges.
I tweet the random snippets of J I write fairly frequently; those are examples of the kind of quick code I write.

## A note on vocabulary

The vocabulary used by J is quite different to most languages.
The concepts are generally fairly analogous to more common terms, but I will explain the J-specific terminology here.

Things in J are named after parts of speech.
This is because there are similarities between their linguistic counterparts and how the J constructs behave.
Rather than invent new terms, existing ones are borrowed.

Data in J is called a **noun**.
Other languages might use a term like &ldquo;value&rdquo; or &ldquo;variable&rdquo; to refer to a similar concept.
Just as in linguistics, nouns in J are &ldquo;things&rdquo;.

A noun can be either an **atom** (single value) or an **array** (a collection of atoms).
All nouns also have a **noun rank** (number of dimensions it has), a **shape** (the length of each dimension), and a **datatype**.
There are various datatypes, such as integral, boolean, floating point, ASCII character, Unicode character, and so forth.

Some example nouns:

```j
   NB. The literal number 1 - an atom
   1
%%% 1 %%%
   NB. The array of 1, 2, and 3 - an array
   1 2 3
%%% 1 2 3 %%%
   NB. A string
   'Ibzan'
%%% 'Ibzan' %%%
```

**Verbs** are similar to functions in other languages, or in mathematics.
They are the words that manipulate nouns by doing something to them.

Verbs have a **valence**, which is how many arguments they take.
In J, there are only two options: **monadic** (unary, one argument) and **dyadic** (binary, two arguments).
This is because the syntax only allows for a verb to take up to two arguments &mdash; one to the left, and one to the right.
J does not have function calls with brackets like other languages.
Everything is either prefix or infix, based on its valence.

Verbs also have a **verb rank**, which influences how they operate on nouns, and I will talk about at various points of the post.

Some example verbs:

```
   NB. Addition
   NB. Addition in action
   1 + 2
%%% 1 + 2 %%%
   NB. Unary negation - monadic -
   - 1
%%% -1 %%%
   NB. J uses the underscore for the minus sign in numeric literals
   NB. Subtraction - dyadic -
   5 - 3
%%% 5 - 3 %%%
   NB. As mentioned, J has more primitives than most languages
   NB. *: squares a number
   *: 4
%%% *: 4 %%%
```

**Modifiers** in J are similar to higher-order functions in other languages.
They take either one or two operands and *produce* a verb.
Modifiers that take one operand are called **adverbs**, and modifiers that take two are called **conjunctions**.

Some example modifiers:

```j
   NB. / insert, which puts the verb it modifies between elements of an array
   +/ 1 2 3
%%% +/ 1 2 3 %%%
   NB. This is equivalent to:
   1 + 2 + 3
%%% 1 + 2 + 3 %%%
   NB. @: at, which is function composition
   NB. add then square
   5 *: @: + 3
%%% 5 *: @: + 3 %%%
```

Finally, to round out the six parts of speech J uses, there are two relatively uninteresting parts.
**Copula** is the name given to the assignment operators in J, [`=.`&nbsp;is&nbsp;(local)][nuvoc-is-local] and [`=:`&nbsp;is&nbsp;(global)][nuvoc-is-global].
**Control words** are used for program flow in more typical imperative fashion, such as [`if.&nbsp;do.&nbsp;end.`][nuvoc-if].
I won&rsquo;t be covering control words here, since they aren&rsquo;t relevant to most J code I write.

Not usually classed as a part of speech, J refers to quotes, brackets (parentheses), and comment markers as **punctuation**.
Quotes demarcate strings, brackets control execution order, and comment markers indicate the start of a comment.

## The things I like

These features are presented in roughly decreasing order of relevance, but it&rsquo;s not a hard-and-fast ranking.
I&rsquo;ve listed things that I think are unique to J from a non-array perspective, or perhaps even to J within other array languages.

### Diverse collection of primitives

Most language have a few basic operations that the user can construct more advanced expressions from.
Commonly available operators include:

- `+`, `-`, `*`, `/` elementary arithmetic: addition, subtraction, multiplication, and division
- `%` modular arithmetic
- `<`, `>`, `<=`, `>=`, `==`, `!=` comparison operators
- `&&`, `||`, `!` logical AND, OR, and NOT
- `&`, `|`, `^`, `<<`, `>>`, `~` bitwise AND, OR, XOR, left shift, right shift, and complement

This is obviously not an exhaustive list.

J, however, has a far larger complement of primitives.
By overloading a given verb based on whether it is used [monadically or dyadically][nuvoc-valence], J allows for a high density of operations in a relatively small space.
These primitives can be viewed on [the wiki&rsquo;s NuVoc page][nuvoc].
Primitives are typically made of one or two characters (although some have three), of which there is a &ldquo;root&rdquo; character and an &ldquo;inflection&rdquo; in the form of a dot `.` or colon `:`.

While the inflection is purely cosmetic, and the primitive doesn&rsquo;t have to be related to the root, a lot of J primitives are.
For example, `+` is [plus][nuvoc-plus] and `+:` is [double][nuvoc-double].
Similarly, `-` is [minus][nuvoc-minus] and `-:` is [halve][nuvoc-halve], `*` is [times][nuvoc-times] and `*:` is [square][nuvoc-square], `%` is [divide][nuvoc-divide] and `%:` is [square root][nuvoc-sqrt].
This helps learning primitives by making related effects have related representations.

Having access to all these primitives allows for terse code.
Sometimes terseness is viewed as a negative, but I believe there is an appropriate level of it &mdash; verbosity can be equally painful to follow.
It also aids writing in a point-free style (typically called _tacit_ within array languages).
This is in itself especially useful when dealing with [modifier trains](#modifier trains).

Compared to [APL][APL] or [BQN][BQN], J&rsquo;s ASCII primitives can be harder to read, since many of them are digraphs consisting of a character and either `.` or `:`.
In languages with distinct glyphs for each operator is more visually distinctive, which aids recognition.
It also makes easier to write by hand, for the few people who do.

However, I think this makes it vastly easier to write on a keyboard.
Each primitive is self-evident in how it can be typed on any keyboard layout that supports ASCII characters.

An interesting compromise may be to create a font that uses ligatures to combine the J di- and trigraphs into a single displayed character, while still treating them as separate objects for the interpreter.
An issue with this in J would be dealing with the difference between monadic and dyadic uses of a primitive, but I suspect they would just have to share the visual representation.
I do not currently use this approach, instead leaving the visual representation of the primitives alone.

When reading J (or any array language), there is definitely an ideal length for terse fragments.
Common patterns like (arithmetic) mean `+/%#` (which is made of [`+`&nbsp;plus][nuvoc-plus], [`/`&nbsp;insert][nuvoc-insert], [`%`&nbsp;divide][nuvoc-divide], and [`#`&nbsp;tally][nuvoc-tally] in a [fork][nuvoc-fork]) are perfectly reasonable to read.

On the other hand,  
``]`(97&([ + 26 | 13 + -~))`(65&([ + 26 | 13 + -~))@.((([ ([ >: [: a.&i. ]) [: {. ]) *. [ ([ <: [: a.&i. ]) [: {: ])&'az' + 2 * (([ ([ >: [: a.&i. ]) [: {. ]) *. [ ([ <: [: a.&i. ]) [: {: ])&'AZ')&.:(a.&i.)``  
is just too long.  
A later version of this was shorter:  
``]`(97&([ + 26 | 13 + -~))`(65&([ + 26 | 13 + -~))@.(64 91 96 123 >.@:-:@:I. a. i. ])&.:(a.&i.)``  
but it still too long to easily parse.

When separated out into parts that each perform a specific role, it becomes more manageable.
I shan&rsquo;t go too deep into explaining this implementation since it isn&rsquo;t the purpose of the article, but a brief explanation is included.

```j
   NB. Determine whether we are mapping an upper case letter,
   NB. lower case letter, or nothing at all.
   NB. 0 = nothing
   NB. 1 = upper case
   NB. 2 = lower case
   regime =. (((_1 1 $~ #) + a. i. ]) 'AZaz') >.@:-:@:I. ]
   
   NB. Perform the fundamental rot13 operation: subtract an offset,
   NB. add 13, modulo 26, then add the offset back.
   NB. The offset is necessary to account for treating ASCII characters as numbers
   NB. ranging from 0-255.
   rot13lim =. [ + 26 | 13 + -~
   
   NB. Versions of the above for lower and upper case
   mapLower =. (a.i.'a')&rot13lim
   mapUpper =. (a.i.'A')&rot13lim
   
   NB. Gerund + agenda to dispatch the appropriate verb
   NB. Using ] same to leave non-letters alone
   map =. ]`mapUpper`mapLower@.regime
   
   NB. Convert to/from characters using &. and map the numbers
   rot13 =: map&.(a.&i.)
```

NuVoc links: [`$`&nbsp;shape][nuvoc-shape], [`~`&nbsp;passive][nuvoc-passive], [`#`&nbsp;tally][nuvoc-tally], [`+`&nbsp;plus][nuvoc-plus], [`a.`&nbsp;alphabet][nuvoc-alphabet], [`i.`&nbsp;index of][nuvoc-index-of], [`]`&nbsp;same][nuvoc-samer], [`>.`&nbsp;ceiling][nuvoc-ceiling], [`@:`&nbsp;at][nuvoc-at], [`-:`&nbsp;halve][nuvoc-halve], [`I.`&nbsp;indices][nuvoc-indices], [`[`&nbsp;left][nuvoc-left], [`|`&nbsp;residue][nuvoc-residue], [`&`&nbsp;bond][nuvoc-bond], [`` ` ``&nbsp;tie][nuvoc-tie], [`@.`&nbsp;agenda][nuvoc-agenda], and [`&.`&nbsp;under (dual)][nuvoc-under-dual].

I consider this to be a decent compromise between brevity and verbosity.
The individual verbs are simple enough to be understood separately, and the names communicate the overall function of each verb to the later parts.
However, it is also a demonstration of the difficulty of reading J from a beginner perspective: even this short example is extremely dense.

### JQt as a notepad

One of the IDEs that comes from J&rsquo;s package manager is JQt.
The feature I like best about JQt is its terminal, which I have found great for interactive development.

The main feature of JQt that most other REPLs don&rsquo;t have is a mutable history.
JQt allows the user to treat the entire page as editable.
You can go to a previous line and edit or delete it if desired.
Deletion is the main thing I use this for &mdash; removing a lengthy output or failed solution from the page to keep it clean.
I also often prepare gists in the terminal, since it allows for writing text in comments alongside actually running code easily.

The ability to edit history is a double-edged sword.
It is possible for the user to edit the output or a prompt in a way that is misleading later.
So far this hasn&rsquo;t bitten me, and J does keep a separate log of inputs, but it is a potential issue.

An improvement here could be to basically copy the way a [Jupyter notebook][jupyter] handles it &mdash; by having cells.
In fact, there is a way to run [J inside a Jupyter notebook][J-jupyter].
I have played around with it a bit, but I haven&rsquo;t switched to using it full-time.
Ultimately, by allowing the user to remove or replace bits of history that aren&rsquo;t useful, but not letting them edit the output directly, the most sensible balance is probably struck.

JQt also comes with in-built support for debugging, running snippets, project management, as one would expect from an IDE, although it is a bit less clear than other IDEs.
This is all useful stuff one would (and should, from a serious language) expect nowadays, and the best part is definitely the interactive terminal &mdash; especially the deletable history &mdash; in my opinion.

### `^:` Power of verb conjunction

Sometimes you come across a language feature that is truly magical.
`^:` is one of those features.
It is the [power of verb][nuvoc-power-of-verb] conjunction, which has the ability to raise a verb to some power.
What does that mean?

A basic example is repeating an operation for a fixed number of times.

```j
   i. 5
%%% i.5 %%%
   NB. *: is square
   *: i. 5
%%% *: i. 5 %%%
   NB. Square twice using *: *:
   *: *: i. 5
%%% *: *: i. 5 %%%
   NB. Square twice using ^:
   *:^:2 i. 5
%%% *:^:2 i. 5 %%%
```

NuVoc links: [`*:`&nbsp;square][nuvoc-square], [`i.`&nbsp;integers][nuvoc-integers].

Zero and one are also both numbers, which can be used to create a construct similar to `if` in other languages.
A verb to the power of zero has the analogy of a number to the power of zero equalling one &mdash; in this case, doing nothing.

```j
   n =: %%% n =: 2%%%
   *:^:(n >: 5) n
%%% *:^:(n >: 5) n %%%
   n =: %%% n =: 10%%%
   *:^:(n >: 5) n
%%% *:^:(n >: 5) n %%%
```

NuVoc links: [`*:`&nbsp;square][nuvoc-square], [`>:`&nbsp;greater than or equal to][nuvoc-gte].

That&rsquo;s pretty cool!
This is only scratching the surface of `^:`.
It has another feature: it can perform the [inverse][nuvoc-inverse] operation of a given verb, too!

```j
   NB. %: is square root
   %: i. 5
%%% %: i. 5 %%%
   NB. Square root using ^:
   *:^:_1 i. 5
%%% *:^:_1 i. 5 %%%
```

NuVoc links: [`%:`&nbsp;square root][nuvoc-sqrt], [`i.`&nbsp;integers][nuvoc-integers].

The inverse is also worked out for verbs that are made out of existing verbs with inverses.

```j
   NB. The running total
   ] xs =: +/\ 2 1 3 5 3 1 1
%%% ] xs =: +/\ 2 1 3 5 3 1 1 %%%
   NB. The inverse of the running totals
   +/\^:_1 xs
%%% +/\^:_1 xs %%%
   NB. The inverse can be inspected using b. _1
   +/\ b. _1
%%% +/\ b. _1 %%%
```

>>> Strictly speaking, J calls this operation the _obverse_, not the inverse.
The difference is that the obverse can be manually assigned to be something else (but it is the inverse by default for supported primitives).

NuVoc links: [`+`&nbsp;plus][nuvoc-plus], [`/`&nbsp;insert][nuvoc-insert], [`\`&nbsp;prefix][nuvoc-prefix]. [`-`&nbsp;minus][nuvoc-minus], [`b.`&nbsp;verb info][nuvoc-verb-info] [`|.!.n`&nbsp;shift][nuvoc-shift], [`:.`&nbsp;assign adverse][nuvoc-adverse].

`^:` can be used with [`_`&nbsp;infinity][nuvoc-infinity] to repeat a verb until it converges.
Extremely useful when you want to find an end state of a convergent operation.

I&rsquo;ve frequently used `^:_` for implementing algorithms like [flood fill][flood fill], which stop when reaching a convergent state.
A simpler example is the [Collatz conjecture][Collatz]:

```j
   NB. Perform one step of the Collatz conjecture iteration.
   NB. Don't worry too much about the implementation, but the primitives are linked below.
   collatz =: -:`(>:@(3&*))`1: @. (1&= + 2&|)%%%collatz =: -:`(>:@(3&*))`1: @. (1&= + 2&|)%%%
   collatz^:_ (19)
%%% collatz^:_ (19) %%%
   NB. Yes, it converges (to 1).
```

Furthermore, J can collect the intermediate results into an array by using [`a:`&nbsp;ace (boxed empty)][nuvoc-ace]:

```j
   collatz^:a: 19
%%% collatz^:a: 19 %%%
```

NuVoc links: [`-:`&nbsp;halve][nuvoc-halve], [`>:`&nbsp;increment][nuvoc-increment], [`@`&nbsp;atop][nuvoc-atop], [`&`&nbsp;bond][nuvoc-bond], [`*`&nbsp;times][nuvoc-times], [`1:`&nbsp;constant functions][nuvoc-const], [`` ` ``&nbsp;gerunds][nuvoc-tie], [`@.`&nbsp;agenda][nuvoc-agenda], [`=`&nbsp;equals][nuvoc-equal], [`+`&nbsp;plus][nuvoc-plus], [`|`&nbsp;residue][nuvoc-residue].

If the exponent is an array, then `^:` will raise the verb to each value in the array.
This is useful for repeating an operation a set number of times when you want to retain the intermediate results.

```j
   NB. Tetration, using J's right-to-left order of operation
   2^^:(i. 4) 2
%%% 2^^:(i. 4) 2 %%%
   NB. To help understand tetration, this could alternately be calculated like this:
   |. ^/\. 4 $ 2
%%% |. ^/\. 4 $ 2 %%%
```

NuVoc links: [`^`&nbsp;power][nuvoc-power], [`i.`&nbsp;integers][nuvoc-integers], [`|.`&nbsp;reverse][nuvoc-reverse], [`/`&nbsp;insert][nuvoc-insert], [`\.`&nbsp;suffix][nuvoc-suffix], [`$`&nbsp;shape][nuvoc-shape].

So far, `^:` has only been used with a fixed value for the power &mdash; either a number, or another noun like `_` or `a:`.

`^:` also supports a dynamic application, where the power is determined by a verb.
Instead of the &ldquo;power&rdquo; being a noun (value), it can also be a verb (function).
This has two main applications: a dynamic `if`, and a `do&nbsp;while` construct, which are familiar from procedural languages.

```j
   NB. Dynamic if - ^: with a function
   NB. Double if greater than 5
   +:^:(5&<)"0 i. 10
%%% +:^:(5&<)"0 i. 10 %%%
   NB. Do-while - ^: with a function and also ^:_
   NB. Double while less than 1024
   +:^:(<&1024)^:_ (23)
%%% +:^:(<&1024)^:_ (23) %%%
```

NuVoc links: [`+:`&nbsp;double][nuvoc-double], [`&`&nbsp;bond][nuvoc-bond], [`<`&nbsp;less than][nuvoc-lt], [`"`&nbsp;rank][nuvoc-rank], [`i.`&nbsp;integers][nuvoc-integers].

### Modifier trains

Modifier trains are one of J&rsquo;s approaches to combining functions.
J does have a more typical approach with a family of conjunctions ([`@`&nbsp;atop][nuvoc-atop], [`@:`&nbsp;at][nuvoc-at], [`&`&nbsp;compose][nuvoc-compose], [`&:`&nbsp;appose][nuvoc-appose]) with varying semantics involving rank.
For certain classes of problems the two main [modifier trains][nuvoc-modifier-trains], [hooks][nuvoc-hook] and [forks][nuvoc-fork], which deal with combining verbs to make another verb.
Forks and hooks can be difficult to read at first since they are effectively invisible, but they become easier to read with experience.

When used monadically, forks may be familiar as the S&prime; combinator, and hooks as the S combinator.

A fork is made of two monads (or dyads), and a dyad.
The two verbs either side process the input arguments, and then the middle dyad combines their results.
In other words, `(f g h) y` is equivalent to `(f y) g (h y)`, and `x (f g h) y` is equivalent to `(x f y) g (x h y)`.
The wiki uses this image (on the [fork page][nuvoc-fork]) to explain a fork, with square brackets represented the optional `x` argument:

```plaintext
    result
       |
       g
      / \
     /   \
    /     \
   f       h
 [/]\    [/]\
[x]  y  [x]  y
```

My favourite example of a fork is the (arithmetic) mean, and can be developed bit-by-bit as follows:

```j
   NB. An array of random numbers (using a fixed seed):
   5 ?. 10
%%% 5 ?. 10 %%%
   NB. The mean is defined as the sum of the elements divided by the count.
   NB. Summing can be done using +/
   +/ 5 ?. 10
%%% +/ 5 ?. 10 %%%
   NB. And the count is given by #
   # 5 ?. 10
%%% # 5 ?. 10 %%%
   NB. J uses % for division.
   NB. Here's an explicit definition of the mean
   mean =: {{ (+/ y) % (# y) }}%%% mean =: {{ (+/ y) % (# y) }} %%%
   mean 5 ?. 10
%%% mean 5 ?. 10 %%%
   NB. Feel free to check this on a calculator.
   NB. Note that in the definition, y appears on the right of both +/ and #
   NB. +/ and # are both monadic verbs acting on y
   NB. % is a dyadic verb acting on the outputs of +/ and #
   NB. A fork can be used to replace this construct:
   mean =: {{ (+/ % #) y }}%%% mean =: {{ (+/ y) % (# y) }} %%%
   mean 5 ?. 10
%%% mean 5 ?. 10 %%%
   NB. Since y only appears once and on the right, deleted to make the verb tacit:
   mean =: (+/%#)%%% mean =: (+/%#)%%%
   mean 5 ?. 10
%%% mean 5 ?. 10 %%%
```

![A diagram of mean as a fork](assets/thoughts-on-j/mean.png)

NuVoc links: [`?.`&nbsp;roll][nuvoc-roll-fixed], [`+`&nbsp;plus][nuvoc-plus], [`/`&nbsp;insert][nuvoc-insert], [`#`&nbsp;tally][nuvoc-tally], [`%`&nbsp;divide][nuvoc-divide].

This is, of course, a rather simple example.
A lot of code follows the basic pattern of having two monads with a dyad to combine the result.
Forks within forks are also useful &mdash; one of the monadic verbs of a (monadic) fork can itself be a fork.

Hooks are made of a pair of verbs: a monad, and a dyad.
The monad &ldquo;pre-processes&rdquo; one argument, and then the dyad combines its result with either the other input argument (in the dyadic case), or the unmodified input (in the monadic case).
That is to say, `(v u) y` is equivalent to `y v u y`, and `x (v u) y` is equivalent to `x v u y`.
As with the fork, the [hook page][nuvoc-hook] has a diagram:

```plaintext
    result
       |
       u
      / \
x or y   v
         |
         y
```

A very simple example of a hook is a filter made of the verb `#~` and a verb to convert the input into a mask of 0s and 1s.
For example, to extract odd numbers from an array:

```j
   i. 10
%%% i. 10 %%%
   2 | i. 10
%%% 2 | i. 10 %%% 
   (2 | i. 10) # i. 10
%%% (2 | i. 10) # i. 10 %%%
   (i. 10) #~ 2 | i. 10
%%% (i. 10) #~ 2 | i. 10 %%%
   (#~ 2&|) i. 10
%%% (#~ 2&|) i. 10 %%%
```

![A diagram of the filter to odd numbers code as a hook](assets/thoughts-on-j/filter-odd.png)

NuVoc links: [`i.`&nbsp;integers][nuvoc-integers], [`|`&nbsp;residue][nuvoc-residue], [`#`&nbsp;copy][nuvoc-copy], [`~`&nbsp;passive][nuvoc-passive], [`&`&nbsp;bond][nuvoc-bond].

Here, `(#~ 2&|)` is the hook, made of two composite verbs.
`#~` acts as a filter, with the right-hand argument being the mask and the left-hand argument being the thing to filter.
`2&|` finds the remainder modulo 2 &mdash; 1 for odd numbers, 0 for evens.

It is possible to write longer modifier trains, too.
In my experience they get quite hard to read though.
A 4-train is not too hard to read though, such as this solution to [LeetCode problem 42][leetcode-42]:

```j
   +/ @: (-~ >./\ <. >./\.)
```

NuVoc links: [`+`&nbsp;plus][nuvoc-plus], [`/`&nbsp;insert][nuvoc-insert], [`@:`&nbsp;at][nuvoc-at], [`-`&nbsp;minus][nuvoc-minus], [`~`&nbsp;passive][nuvoc-passive], [`>.`&nbsp;max][nuvoc-max], [`\`&nbsp;prefix][nuvoc-prefix], [`<.`&nbsp;min][nuvoc-min], [`\.`&nbsp;suffix][nuvoc-suffix].

Here, `(-~ >./\ <. >./\.)` is a 4-train made of a fork `>./\ <. >./\.` and then that fork is the right-hand verb in a hook, with `-~`.
While useful in this case, the longer and longer a train gets the harder it can be for humans to parse.
The rule of thumb is that everything in J comes in groups of two or three.

```j
   height =: %%%height =: 0 1 0 2 1 0 1 3 2 1 2 1%%%
   +/ @: (-~ >./\ <. >./\.) height
%%% +/ @: (-~ >./\ <. >./\.) height %%%
```

![A diagram showing how the solution to Leetcode 42 transfers data around](assets/thoughts-on-j/leetcode-42.png)

>>> These diagrams were made by [dissect](#dissect), which I talk about later.


### J&rsquo;s adverbs

Modifiers are equivalent to higher-order functions in other languages.
They are primitives that come in two forms: adverbs and conjunctions.

Conjunctions take two inputs, such as a verb and a noun or two verbs and produce a new verb.
Useful conjunctions include:

- the six composition conjunctions [`@`&nbsp;atop][nuvoc-atop], [`@:`&nbsp;at][nuvoc-at], [`&`&nbsp;compose][nuvoc-compose], [`&:`&nbsp;appose][nuvoc-appose], [`&.`&nbsp;under (dual)][nuvoc-under-dual], and [`&.:`&nbsp;under][nuvoc-under]
- [`&`&nbsp;bond][nuvoc-bond]
- [`` ` ``&nbsp;tie][nuvoc-tie] and [`@.`&nbsp;agenda][nuvoc-agenda] for manipulating gerunds
- perhaps most of all, [`"`&nbsp;rank][nuvoc-rank]

These conjunctions allow for combining verbs or modifying them in a variety of ways.
However, I find that J&rsquo;s adverbs are especially expressive.

Adverbs are what J calls modifiers that act on a single verb.
While there aren&rsquo;t many of them, they are extremely useful.

Perhaps the most common adverb is `~` as both [reflex][nuvoc-reflex] and [passive][nuvoc-passive].
Reflex is the monadic case, and passive is the dyadic one.
Reflex copies its input to both arguments of the verb it modifies, also known as the W combinator (elementary duplicator).

```j
   NB. An implementation of double
   +~ 5
%%% +~ 5 %%%
```

NuVoc links: [`+`&nbsp;plus][nuvoc-plus].

Passive is the more common of the two and flips the arguments of the verb (the F combinator).
It is frequently used with verbs whose arguments can be swapped to make a tacit definition easier to write.

```j
   i. 10
%%% i. 10 %%%
   2 | i. 10
%%% 2 | i. 10 %%%
   (2 | i. 10) { i. 10
%%% (2 | i. 10) { i. 10 %%%
   ({~ 2&|) i. 10
%%% ({~ 2&|) i. 10 %%%
```

NuVoc links: [`i.`&nbsp;integers][nuvoc-integers], [`|`&nbsp;residue][nuvoc-residue], [`{`&nbsp;from][nuvoc-from], [`&`&nbsp;bond][nuvoc-bond].

This particular usage (`{~`) is extremely common in [hooks][nuvoc-hook] like this one, since the modified value is passed on the right-hand side.

The other adverbs that are constantly cropping up are `/`&nbsp;[insert][nuvoc-insert]&nbsp;and&nbsp;[table][nuvoc-table], `\`&nbsp;[prefix][nuvoc-prefix]&nbsp;and&nbsp;[infix][nuvoc-infix], and [`\.`&nbsp;suffix][nuvoc-suffix].
All of these adverbs take the verb they modify and apply it to an array in some way or another.

```j
   NB. Insert inserts the verb between elements.
   1 + 2 + 3 + 4 + 5
%%% 1 + 2 + 3 + 4 + 5 %%%
   +/ 1 2 3 4 5
%%% +/ 1 2 3 4 5 %%%
   NB. Prefix applies the verb to successive prefixes.
   NB. < creates boxes, letting us see the prefixes
   <\ 1 2 3 4 5
%%% <\ 1 2 3 4 5 %%%
   NB. +/ is itself a verb and can be used in an adverb.
   +/\ 1 2 3 4 5
%%% +/\ 1 2 3 4 5 %%%
   NB. This is the running total from earlier.
   NB. Similarly, \. applies the verb to successive suffixes.
   <\. 1 2 3 4 5
%%% <\. 1 2 3 4 5 %%%
   +/\. 1 2 3 4 5
%%% +/\. 1 2 3 4 5 %%%
   NB. Infix applies the verb to sliding windows of the given size.
   2 <\ 1 2 3 4 5
%%% 2 <\ 1 2 3 4 5 %%%
   2 +/\ 1 2 3 4 5
%%% 2 +/\ 1 2 3 4 5 %%%
   NB. Table applies the verb to all pairs.
   NB. Using ~ reflex here to provide the same argument to both inputs.
   +/~ 1 2 3 4 5
%%% +/~ 1 2 3 4 5 %%%
```

NuVoc links: [`+`&nbsp;plus][nuvoc-plus], [`<`&nbsp;box][nuvoc-box].

These adverbs allow you to slice up an input array in many useful ways.
The others that I haven&rsquo;t gone into detail about are `/.`&nbsp;[oblique][nuvoc-oblique]&nbsp;and&nbsp;[key][nuvoc-key], and [`\.`&nbsp;outfix][nuvoc-outfix]
Oblique operates on diagonals of a table, key groups identical items (so `#/.` counts unique items via [`#`&nbsp;tally][nuvoc-tally]), and `\.`&nbsp;outfix removes a sliding window from the array.

If these adverbs don&rsquo;t offer sufficient ability to divide up the array, [`;.`&nbsp;cut][nuvoc-cut] almost certainly does.
While it technically a conjunction, the moment it gets a numeric code it becomes an adverb.
Cut is a collection of three adverbs.
Between them they can extract subarrays, split up an array into groups specified by a mask, and apply a verb to tiles of an array.
They also have the added benefit of being fairly efficient in terms of performance, although I have not yet had any issues with that in general with J.

### Dissect

One of the addons in the J package manager is called `debug/dissect`.
It is one of the best debugging and explanation tools I have encountered in a language.
When writing a language as dense as J, it can be hard to fully understand how data is being passed around between verbs.

This is where dissect can help.
After loading the addon into your J session (`require&nbsp;'debug/dissect'`), you are given access to a new verb `dissect` that takes a single argument &mdash; a string of a J sentence that produces a noun result.
If the sentence does _not_ produce a noun, dissect cannot display it; it only tracks how data flows throughout the snippet from starting inputs to a final output.

Dissect was used to generate the diagrams in earlier sections &mdash; but it also allows you to interrogate the steps further.

![A diagram showing the intermediate steps of summing the numbers 1, 2, 3, 4, and 5.](assets/thoughts-on-j/partial-sum.png)

For example, in this dissection of `+/&nbsp;1&nbsp;2&nbsp;3&nbsp;4&nbsp;5`, dissect can show the intermediate results of `+/`.
The cell with a solid border and highlight (`14`) shows the value being probed, and the dashed borders of `12` and `2` (in the input) show the operands used to produce it.

I have found dissect to be an easy way to explain longer phrases to other people.
The visual approach can really help communicate things like forks and hooks easier than quoting the definition.

### Cyclic gerunds

In J, a [gerund][nuvoc-gerunds] is a collection of verbs (sort of &mdash; it&rsquo;s more complicated, but for now this explanation will suffice).
Gerunds have several different uses within J, such as allowing for dispatching a different verb based on some condition (see [`@.`&nbsp;agenda][nuvoc-agenda]).
One of these uses is to alternately apply the different verbs it represents.

For example, to alternate addition and subtraction over a list:

```j
   i. 10
%%% i. 10 %%%
   +`-/ i. 10
%%% +`-/ i. 10 %%%
   0 + 1 - 2 + 3 - 4 + 5 - 6 + 7 - 8 + 9
%%% 0 + 1 - 2 + 3 - 4 + 5 - 6 + 7 - 8 + 9 %%%
```

Another example is when combined with the [`"`&nbsp;rank][nuvoc-rank] conjunction to instruct the gerund to act as a verb:

```j
   (+:`-:)"0 i. 10
%%% (+:`-:)"0 i. 10 %%%
```

NuVoc links: [`+:`&nbsp;double][nuvoc-double], [`-:`&nbsp;halve][nuvoc-halve].

Here, the verb is acting on rank 0 &mdash; the individual [atoms][nuvoc-atom] of the input array.
Then, the gerund cycles through its elements to apply the given verb to each of those cells.
This works for any given rank:

```j
   i. 2 2
%%% i. 2 2 %%%
   (+:`-:)"1 i. 2 2
%%% (+:`-:)"1 i. 2 2 %%%
   i. 2 2 2 2
%%% i. 2 2 2 2 %%%
   (+:`-:)"2 i. 2 2 2 2
%%% (+:`-:)"2 i. 2 2 2 2 %%%
```

The main application I have found of cyclic gerunds is when dealing with heterogenous data in a boxed array.
In this sense, heterogenous may mean different in what it represents as opposed to what datatype it is represented by.
In the following example, each string is alternately converted to upper or lower case.

```j
   ] words =. ;: 'Some example text. Some more example text.'
%%% ] words =. ;: 'Some example text. Some more example text.' %%%
   (toupper each)`(tolower each)"0 words
%%% (toupper each)`(tolower each)"0 words %%%
```

NuVoc links: [`;:`&nbsp;words][nuvoc-words], [each][nuvoc-each].

This is a relatively simple example but shows the versatility of a cyclic gerund for dealing with textual data.
For an array of boxed strings, each atom is a boxed string.
A more realistic use case would be using [`rxcut`][nuvoc-rxcut] to split some input text into matching and non-matching boxes.
A cyclic gerund can then apply one verb to each match, and a different verb to the non-matches _without_ being restricted to solely textual manipulations as using [`rxapply`][nuvoc-rxapply] would be.

### Leading axis model

The leading axis model (also called leading axis theory) is made of two main parts.

The first is that arrays are manipulated by turning them into _cells_ of the necessary rank and then giving those to the verbs.
For example, [(dyadic)&nbsp;`+`&nbsp;plus][nuvoc-plus] has rank 0 0.
This means that to add two arrays together, they must first both be reduced all the way to 0-cells (atoms) and then added together.
Many other operators have rank `_` (infinity), which means they work on the array as a whole.
On the other hand, some verbs don&rsquo;t change with different rank.
For example [`+:`&nbsp;double][nuvoc-double] has to double each element in its input regardless of what rank it is modified to work on.

The second is that verbs act on the first axis of an array implicitly, or can be configured to use a later axis using the [`"` rank][nuvoc-rank] operator.
This adds a level of consistency to the language, which makes it easier to read and write.
The rank operator also provides the motivation for making verbs act on the first axis: it becomes trivial to have the verb act on any later axis in the array.
The ability to specify what cells of an array a verb acts on are how J allows the programmer to avoid writing explicit loops and still get fine-grained control over effects.

J was designed from the ground up with the leading axis model in mind (unlike both SHARP and Dyalog APL, which had it retrofitted on).
As a result, all of J&rsquo;s primitives are designed with the leading axis model in mind &mdash; rank one functions all operate on the first axis, for example.
This leads to many array operations being intuitive in J (at least in my experience).

```j
   i. 3 6
%%% i. 3 6 %%%
   NB. This has shape 3 6, as specified
   +/ i. 3 6
%%% +/ i. 3 6 %%%
   NB. This has summed up the columns, giving a result of shape 6
   NB. which the same as the last axis from before
   NB. If needed, we can sum rows instead:
   +/"1 i. 3 6
%%% +/"1 i. 3 6 %%%
   NB. Here the rank modifier creates a new verb that
   NB. acts on the 1-cells (rows) of the array.
   NB. The result is of shape 3 - the first axis.
```

NuVoc links: [`+`&nbsp;plus][nuvoc-plus], [`/`&nbsp;insert][nuvoc-insert].

### Mathematical primitives and verbs

These are great for small puzzles.
Want prime numbers?
[`p:`&nbsp;primes][nuvoc-primes] will give them to you.
J also gives you access to [complex numbers][essay-complex], [polynomials][nuvoc-polynomial] and [their roots][nuvoc-roots], [prime factors][nuvoc-prime-factors], and [hypergeometric][nuvoc-hypergeometric] [trigonometric, and hyperbolic][nuvoc-circle-fns] functions as primitives.
The standard library also has calculus facilities &mdash; J can find the derivative and integral of a lot of verbs, including user-defined ones.
Very handy to have.

Obviously other languages have a lot of these features too, but things like having easy access to primes and polynomials from primitives is invaluable for some kinds of problems.

## The things I don&rsquo;t like

Some parts of J are a huge pain.
While not dealbreakers for me, they add more of a faff than should be present in a modern programming language.
As with above, these are listed in a vaguely decreasing order with the most annoying parts first.  
In my opinion the more serious of these should be fixed outright.

### Importing files

This is hands down the worst part of writing J code with a multi-file project.
J imports files from the current working directory.
This means that if you have a script that loads another in it, the working directory that script is run in will cause it to either work or crash.

J does have a workaround for this: `jpath`.
It provides a way of referring to certain system paths that you might use a lot.
For example, `~Projects/md/md.ijs` might resolve to `/home/ibzan/j903-user/projects/md/md.ijs`.
The problem with this approach is it still relies on an absolute path effectively &mdash; just trading `/home/ibzan/j903-user/projects` for `~Projects`.
Requiring all J code to be installed in a specific location is the behaviour I want to avoid &mdash; while less bad for referring to separate projects, it means code within the _same_ project must know where it is installed!

I strongly believe that file imports should be relative to the directory the script is in.
For the most part, this is what you actually want as a user!

I ended up having to write a project to work around this issue.
You can find it [here][relimport].
Although only a bit of work, it is vastly more obtuse than should be necessary.

### Exceptions

J has exceptions.
I do not like exceptions in the first place, but within J it feels like the best compromise for signalling errors.
The type system is not rich enough to support option or result types, and errors are generally rare at &ldquo;runtime&rdquo; &mdash; more often than not they are effectively programming errors.
Many exceptions are closer to compiler errors than runtime exceptions, since J is an interpreted language.

There are still potential runtime errors, such as incorrect datatypes, invalid inputs, memory errors and such, but they are typically not an issue for the scale I work at.

On of my main gripes with these exceptions is how vague they are, which doesn&rsquo;t help working out what the problem is.
J assumes that the programmer is already familiar with the rules of J and can determine what is wrong with a phrase.
For a new J programmer, the errors offer practically nothing of value to aid understanding.
Some of the simpler ones are alright: a domain error means that the domain (input) is incorrect.
This is most frequently an incorrect type, but could also be an incorrect number of arguments.

```j
   'a' + 'b'
|domain error
|    'a'    +'b'
   1 *: 2
|domain error
|   1    *:2
```

However, when this is embedded within a larger phrase, it can be difficult to ascertain the issue.

J also uses spacing to indicate where the problem occurred rather than leaving the phrase formatted as the programmer did and annotating it.
I believe this is because the original format is discarded during the tokenisation step of interpreting.

J&rsquo;s dynamic scoping means that it searches for a name whenever a private scope is executed.
If J cannot find the corresponding object (whether it is a noun, verb, conjunction, etc.), it signals a value error:

```j
   f =. {{ g y }}
   f ''
|value error: g
|       g y   
```

In this case, J was able to provide the programmer with some additional information: `g` is the missing value.
However, many other errors don&rsquo;t include any such explanation.
This is mostly a pain when dealing with swapping between interactive development and development within a script.
It can be easy to overlook a particular definition and have to search back through the input log to find how some verb was written.

It may be clear from these last two examples that J exceptions aren&rsquo;t hugely informative.
Without enumerating other classes of errors that are signalled at runtime, these 
Considering the state of compiler error reporting in languages like Rust I think there&rsquo;s little excuse for not improving J&rsquo;s description of errors.

It isn&rsquo;t too hard to imagine some of the improvements that could be made to some of these errors.
Returning to the domain error, it would be trivial for J to report that the number of operands was incorrect:

```j
   1 *: 2
| domain error
| incorrect number of operands
| *: requires one operand
| 1 *: 2
|   ^^
   E. 'foobar'
| domain error
| incorrect number of operands
| E. requires two operands
| E. 'foobar'
| ^^
```

When operating on incompatible types, an improved error could also be easily reported:

```j
   'a' + 'b'
| domain error
| incompatible noun types
| + requires numeric nouns
| 'a' + 'b'
|     ^
```

How about a length error, when two array have incompatible shapes?

```j
   1 2 3 + 1 2
| length error
| x shape: 3
| y shape: 2
| 1 2 3 + 1 2
|       ^
   1 2 3 4 + i. 3 4
| length error
| x shape: 4
| y shape: 3 4
| 1 2 3 4 + i. 3 4
|         ^
```

The error could even go further as to explain what agreement is.

This quick experiment in working out a better format shows, in my opinion, how easy it could be to fix this glaring issue.

### Documentation

J documentation is often sufficient or very good &mdash; especially with primitives and core language concepts &mdash; but sometimes lacking.
In particular, descriptions of addons are frequently missing or equally terse as the language itself, which doesn&rsquo;t aid readability when looking for a detailed description.
The lack of an easy way to search the contents of addons makes them hard to adopt.

A recent example &mdash; I needed to copy a directory to a new location.
I expected that this would be a foreign, which is where file copying is.  
Nope.  
I tried implementing my own, but it caused the interpreter to segfault.
Not wanting to look into it too much further (it was really a small part of the project), I settled for invoking `cp -r` on the command line.  
When writing this article, I found out about the [`general/dirtrees` addon][addon-dirtrees], which has this feature.
I was surprised that this particular functionality wasn&rsquo;t included in the standard library, but at least it is an addon.

The lack of clear documentation for addons severely hinders their usage.
I think a real priority ought to be completing the wiki with missing contents and explanations for each addon.

### String operations

This is an artifact of using the [flat array model][flat array model] to represent strings as rank 1 arrays.
In the flat array model, an array cannot contain an array.
Since strings in J are actually just arrays, they can either be a higher rank (a rank 2 string is a rectangle of text, probably with padding characters) or boxing each string, which converts them to atoms.

To avoid padding strings with unwanted spaces, it&rsquo;s necessary to work on boxed strings instead.
These can be different lengths without adding padding characters.

The downside, however, is that boxed data is more awkward to interact with.
Primitives do not go into boxes; they operate on the box itself if applicable.
This means that when dealing with an array of boxed strings there are two options.

The first option is to use [`"`&nbsp;rank][nuvoc-rank] to run a verb over each individual box, and start by unboxing with [`>`&nbsp;open][nuvoc-open].
This approach is only viable when the final results can be combined into an array with either acceptable padding or no padding.
For example, to convert an array of boxed strings to their lengths, as an array: `#@>"0`, using [`@`&nbsp;atop][nuvoc-atop] and [`#`&nbsp;tally][nuvoc-tally].

When the desired operation does not reduce the string to a suitable shape, or when padding characters are to be avoided (as is often the case), instead one must use [`&.`&nbsp;under (dual)][nuvoc-under-dual] with [`>`&nbsp;open][nuvoc-open] to open each box, apply a verb, and then box the result.
To convert each string to upper case, one might write `toupper&.>`, using [`toupper`][nuvoc-toupper] from the standard library.
The adverb `&.>` is so common that it is also included in the standard library: `each`.

The problem with each of these approaches is the added complexity in what would hopefully be a simple operation.
Given the constraints of the array model, it is understandable why this is the case, but it does complicate text processing in my experience.

### Numerical codes

This is a minor one.
J does not like to use reserved words _at all_.
Most things that other language would use as keywords are implemented as standard verbs in J.
Instead, many common functions are implemented behind a primitive with some numerical codes.
The few words that J does use are all suffixed by a `.` to indicate at the parsing stage that they are keywords.

Take trigonometric functions, accessed via [`o.`&nbsp;circle&nbsp;functions][nuvoc-circle-fns].
Instead of names, they are accessed by a numeric code as the `x` argument to `o`:

```j
sin    =:  1 & o.
cos    =:  2 & o.
tan    =:  3 & o.
arcsin =: _1 & o.
arccos =: _2 & o.
arctan =: _3 & o.
```

The workaround is exactly as above: create aliases to the verbs.
There is also the [`math/misc`&nbsp;addon][math/misc] which contains a script (`trig.ijs`), which defines these aliases.

This is most prevalent with J&rsquo;s [foreigns][nuvoc-foreigns].
Foreigns are what J calls access to system functions, like reading a file or getting the current time, and some miscellaneous functions, like debugging controls and some things that don&rsquo;t fit into primitives.

All foreigns are addressed as a pair of numbers and the [`!:`&nbsp;foreign][nuvoc-foreign] conjunction.
This ends up being quite hard to remember in my experience &mdash; I prefer having names to numeric combinations.
As with above, assigning names myself or finding an appropriate addon will help, but is a bit tedious to repeat.

The once cases where I have found numeric codes acceptable is within the [`;:`&nbsp;sequential machine][nuvoc-seq-mac] primitive.
The state transition table uses a numeric code to determine what action to take in each state, given a certain input.
Since the state transition table is often quite large, the shorthand of a number instead of something longer is beneficial.

One of the other changes I would make ([namespaces](#namespaces)) would allow for more elegant solutions to this issue, by storing all of these values in sensibly-named namespaces.
Locales could also work here.
Not a huge problem, all in all.

## What I would change

These are the changes I would make to J but are probably too big to be changes to the current language.
They would likely require a _large_ breaking version, or a successor language that is inspired by J.

### Lexical (instead of dynamic) scoping

Most contemporary programming languages use lexical scoping.
J uses dynamic scoping.
Dynamic scoping is where the value of a name is determined when the code is executed, rather than when it is defined.
In other words, the &ldquo;most recent&rdquo; definition is the one used.
For example:

```j
   a =: %%% a =: 5 %%%
   b =: {{ +: a }}%%% b =: {{ +: a }} %%%
   b ''
%%% b '' %%%
   a =: %%% a =: 10 %%%
   b ''
%%% b '' %%%
```

On the REPL, this is great.  
It allows for fast iteration by constructing a verb out of two or three others, testing, and then tweaking one of the constituent parts and having that affect the final verb without having to redefine it.
As a result, you can spend far less time copy-pasting existing definitions without changing anything to just update the contents to their new versions.

However, in scripts this is often more of a pain than not.
If you want a verb to be accessible outside the script, it has to be globally accessible, but _so do the verbs it calls_.
This is because a private binding in J is only retained for the duration of the definition of its namespace.
After a script that defines a private name is finished, its private definitions are dropped.
Hence, a complicated script can export far more than needs to be accessible just so that it functions.
The current workaround is to use [`f.`&nbsp;fix][nuvoc-fix] to recursively expand names, which prevents private definitions from ceasing to exist at the time the longer name is used.

The impact of lexical scoping on the REPL can be mitigated by changing the workflow.
Instead of writing J code directly into the REPL at all times, the programmer instead writes to a script file which is then loaded on the REPL.
After developing the next piece of the script interactively, it&rsquo;s copied into the file which is then reloaded.

#### Special Combinations

This issue arises directly from dynamic scoping.

Each verb in J is executed right-to-left, without knowing anything about the verbs that executed before or after it.
This is often useful, because it means the code being written is inherently pure.
However, it can have performance issues, especially when some things could be terminated early, or intermediate products are not needed.

To tackle this, J has a lot of &ldquo;[special combinations][nuvoc-scs]&rdquo;, which are basically special compound verbs the interpreter recognises to perform and performs a specialised function optimised for the example.
As an example, take [`?`&nbsp;roll][nuvoc-roll] and [`$`&nbsp;shape][nuvoc-shape] being used together to create an array of random numbers, with [`@`&nbsp;atop][nuvoc-atop] for the special combination.

```j
   NB. The standard library verb timespacex records the time and space used in
   NB. a particular phrase.
   NB. Generate a 3000-by-3000 matrix of random numbers from 0 to 99999:
   timespacex '? 3000 3000 $ 100000'
%%% timespacex '? 3000 3000 $ 100000' %%%
   NB. Now let's try using a special combination: ?@$
timespacex '3000 3000 ?@$ 100000'
%%% timespacex '3000 3000 ?@$ 100000' %%%
```

The memory usage (second number) is better by a factor of two, because there is no need to create a copy of the initial array (`3000&nbsp;3000&nbsp;$&nbsp;100000`) in the first example.

Special combinations are recognised by the interpreter at the moment of verb creation.
Since they are built out of a conjunction to combine different verbs together, the interpreter must recognise the precise combination at the moment it is created.
A conjunction creates a new verb as soon as it is given both arguments.
In order to do this, the contents must match exactly.
And so:

```j
   roll =: ?%%% roll =: ? %%%
   shape =: $%%% shape =: $ %%%
   timespacex '3000 3000 roll@shape 100000'
%%% timespacex '3000 3000 roll@shape 100000' %%%
```

This fragment has to create a copy of initial array because the interpreter has not recognised the special combination.
[`f.`&nbsp;fix][nuvoc-fix] can be used here to convert a phrase to its primitive form, which allows the interpreter to detect the special combination.

```j
   timespacex '3000 3000 (roll@shape f.) 100000'
%%% timespacex '3000 3000 (roll@shape f.) 100000' %%%
```

In a more ideal setting though, the names assigned to verbs would not interfere with this optimisation.
This is because the values of the names cannot be determined until the code is actually run, due to dynamic scoping.  
Consider:

```j
   roll =: ?%%% roll =: ? %%%
   shape =: $%%% shape =: $ %%%
   randoms =: roll@shape%%% randoms =: roll@shape %%%
   roll =: 0: NB. Constant 0 function%%% roll =: 0: %%%
   3 3 randoms 100
%%% 3 3 randoms 100 %%%
```

#### Lexical Scoping and Special Combinations

The change I would make here is allowing for lexical scoping.
This would handle the definitions of those internal names lexically and then the user would be free to keep them as local definitions.

As I mentioned above, however, I really like dynamic scoping on the REPL.
Potentially the best solution is to allow a mix of both, but this comes across as nasty.
I think the added complexity of having mixed dynamic/lexical scoping may be too great for what it adds to the language.

After all, I am strong believer of the language _not allowing_ bad things to happen in the first place.
If something is _possible_ to do within a language, however niche or not designed to be used, you can practically guarantee someone will use it.
Usually, I&rsquo;d say this is their own fault, but this one raises a lot of questions about how it would interact across scripts. If one script specifies it uses dynamic scoping and another lexical, how do they interact?

Can there be a compromise to allow both scoping approaches?  
Possibly.  
Scoping style could be configured when the interpreter starts and not allow it to be changed from within a session (although one might expect a foreign to do this).
This would allow a REPL to run with dynamic scoping, and a file to run with lexical scoping.
The difference between developing interactively in the REPL and within a file may prove unintuitive, though.

To conclude this section, I don&rsquo;t think there&rsquo;s an obviously good way to support both dynamic and lexical scoping.
This is almost certainly why no language I am familiar with do both and settle on supporting only a single one.

### First-class verbs

Verbs in J are not first-class values; they are special objects that cannot be passed around as data.
This leads to the distinction between verbs, adverbs, and conjunctions.
In a language with higher-order functions and first-class functions, J&rsquo;s adverbs are simply higher-order functions.

This change would also remove the need for gerunds in the language for dispatching separate verbs.
Instead, the programmer could construct a list of verbs and select the appropriate one (with say [`{`&nbsp;from][nuvoc-from]) and then apply a noun to it.

Why is this change needed?  
In my opinion there are a few motivating reasons.

#### Messy Representation

When a verb in J is given a name, it is not actually stored as an instance of the verb (which would be possible if they were first-class).
Instead, it is passed around as the name and turned back into the value as necessary.
The adverb [`f.`&nbsp;fix][nuvoc-fix] can be used to recursively expand names to convert a definition into one made of primitives only.

Furthermore, a gerund created with [`` ` ``&nbsp;tie][nuvoc-tie] is actually stored as an array of boxed strings.

The representation of a verb with names in it in the meantime is not ideal &mdash; although unlikely, something that is not a verb could be recognised as one if used in the wrong scenario.
Ideally, I think this should be cleaned up to prevent it from being possible in the first place.

#### Rank Mishaps

Currently, the behaviour of verbs and their names can cause unexpected behaviour for even experienced programmers.
This is from a property of verbs and a property of names.

In J, verbs have a rank.
This fact is so important that every page on NuVoc includes &ldquo;[WHY IS THIS IMPORTANT?][nuvoc-rank-info]&rdquo; which explains why the rank of a verb matters.
The rank of a verb determines the maximum rank that a verb can operate on.
A higher rank input array is subdivided into arrays of a rank the verb can handle.
As covered in the [leading axis model](#leading-axis-model) section, this has advantages.

For example, the verb [`+`&nbsp;plus][nuvoc-plus] has rank `0 0`.
This means that both the `x` and `y` input arrays are broken down into atoms, added individually, and then collected to the original shape.
Note that the leading axes of the operands must still agree &mdash; you cannot add incompatible array shapes.

By comparison, consider the verb [`#`&nbsp;tally][nuvoc-tally].
It has rank `_` &mdash; infinity.
That means that it can take an input array of any rank whatsoever and produce an output without subdividing it.
This is because tally counts the number of elements in the first axis, so it can work on an array of any rank.

However, how J treats the rank of a verb is what leads to the weird behaviour.
When a verb is assigned to a name, the name is also given a rank that matches the verb.
However, if the verb is reassigned then the rank _does not change_.
This is deeply strange.
For example:

```j
   NB. Assign a rank 0 verb to f:
   f =. +%%%f =. +%%%
   f b. 0
%%% f b. 0 %%%
   NB. Assign f to g - g is rank 0:
   g =. f%%%g =. f%%%
   g b. 0
%%% g b. 0 %%%
   NB. Reassign f to something else:
   f =. ,%%%f =. ,%%%
   f b. 0
%%% f b. 0 %%%
   g b. 0
%%% g b. 0 %%%
   NB. Now g is rank 0, but f has infinite rank.
   NB. However, used monadically, g will resolve to , ravel - not + conjugate.
   NB. This is because f is now , instead of +
   NB. To demonstrate, let's use the close composition @ atop:
   i. 3 3
%%% i. 3 3 %%%
   <@f i. 3 3
%%% <@f i. 3 3 %%%
   <@g i. 3 3
%%% <@g i. 3 3 %%%
```

### Namespaces

Currently, J supports two kinds of namespaces: private namespaces, for definitions, and public namespaces called locales.
Private namespaces are for all intents and purposes inaccessible to the programmer.
They serve only to bundle together the contents of an explicit definition.

To me, locales feel very hacky to use &mdash; they are accessed by appending `_localename_` to the name of something you want to access, where `localename` is the name of the locale.
This form is called a _locative_.
For example, after loading `math/calculus`, one must use `deriv_jcalculus_` to access the `deriv` name it contains.

I think there are a few main issues with locales.

Firstly, J uses a hierarchy of locales to search through when resolving a name.
This can quickly become convoluted with multiple scripts involved, which can load more locales into the workspace.

Secondly, locales are not first-class values (this is a common theme in J for things that are more complex than raw data).
This causes the usual issues that arise from not being first-class: difficult or impossible to pass around between different parts of the program and requiring special syntax to construct or access.

Thirdly, executing a locative actually changes the implied locale to execute the verb.
That is to say, instead of accessing the value from the locale directly, J instead changes the search path and then looks up the name.
This behaviour can introduce subtle bugs into code that are hard to track down.
The only place I found this mentioned was on the page for [`f.`&nbsp;fix][nuvoc-fix] which covers the problem and a solution.

I would change this by replacing locales with a new datatype: namespaces.
My namespace is a first class value that contains names, accessible via some primitive &mdash; while appearing similar from the outside, requiring explicit access into a namespace would prevent the above issues.
Precisely what symbol would be used for access I am not sure &mdash; something not already used would have to be picked, which rules out something common like `.` or `::`, and J&rsquo;s syntax doesn&rsquo;t support `->`.
Of course, in a hypothetical successor language this would be designed from the ground up instead of being retrofitted into the existing specification.

I think this would be an overall benefit, allowing for a more sensible approach to grouping names together.
It also prevents namespaces from having to be manually freed by the programmer, which is something J does not expect you to do with most data.

### 2-train for function composition

This is a change that was [eventually adopted][J-2train] by Roger Hui, one of the main developers of J.
In short, this change replaces the meaning of the 2-train `(v&nbsp;u)` with [`@:`&nbsp;at][nuvoc-at] instead of [hook][nuvoc-hook].
I think this would generally lead to code that is easier to read, and a conjunction isn&rsquo;t a huge price to pay for still having access to the hook, which _is_ a very useful combinator at times.

Within array languages, the 2-train as hook is an oddity of J&rsquo;s.
Most other array languages treat a 2-train as composition of some form (J has four composition conjunctions).

In the meantime, the composition conjunctions and a capped fork (`[:&nbsp;v&nbsp;u`, with [`[:`&nbsp;cap][nuvoc-cap]) are both available for composition.
I use both of these forms from time to time &mdash; in fact [md][md] uses both capped forks and composition conjunctions, based on the situation.

The hook is then moved to its own conjunction &mdash; such as `h.` in J, as Hui suggests &mdash; and used as a normal modifier.
This would also allow for both a &ldquo;forward hook&rdquo; and &ldquo;backward hook&rdquo;, an idea shamelessly stolen from BQN.
For example, `f&nbsp;h.&nbsp;g&nbsp;y` could be equivalent to `y&nbsp;f&nbsp;g&nbsp;y`, and `f&nbsp;H.&nbsp;g&nbsp;y` could be equivalent to `y&nbsp;g&nbsp;f&nbsp;y`, with similar for the dyadic case.
Any backward hook could be written as a forward hook, however I believe there are scenarios where one would be clearer than the other based on surrounding context.

Given the ubiquity of function composition however, I believe that the terser syntax leveraging the 2-train is superior and would help to reduce the noise present in a lot of tacit definitions.

### Unicode

By default, J treats strings as a sequence of bytes (the same way C does, which the J interpreter is written in).
However, a large amount of text these days is represented as Unicode.
J does have support for Unicode &mdash; the [`u:`&nbsp;Unicode][nuvoc-unicode] primitive provides conversions to and from Unicode.
J could be changed to support Unicode by default for strings, which would simplify dealing with modern text.
This isn&rsquo;t a huge problem (IO verbs can be created that handle UTF-8 instead of ASCII), but a bit of a faff to have to handle manually.
Mostly, this is an artifact of the age of the language.

### Mutation

As a preface to this section, I love immutability.
It prevents classes of bugs from existing and makes data easier to reason about.
It&rsquo;s a good fit for a language like J that is &ldquo;data-oriented&rdquo;.  
Sometimes, however, mutability can be important for performance.
There are certain algorithms that just perform significantly better when written in a mutable fashion.

J does not provide easy ways to mutate a data structure.
This is because J is built on the observation that when modifying an array, most of the work is in calculating the new data, and little of the work is in writing it into memory.
This works very well for large arrays and when performing a whole-array modification, such as arithmetic or calculating some rolling value.
It also prevents race conditions, since a shared array will not be mutated by another region of code.
When modifying a small number of values in a large array, however, this is very inefficient.
Another case where mutation would be suitable is when working with a temporary array that is only used to work out the final result.
In this case, it is probably not shared and does not need to have its first value retained.

J does support a limited form of mutability in the form of assignment in place.
This is possible when using a particular adverb ([`}`&nbsp;amend][nuvoc-amend]), and only of the following particular form:  
`name&nbsp;=.&nbsp;x&nbsp;i}&nbsp;name`  
or slight variation on this.
Using this special combination avoids creating a copy of `name` and instead mutates the array, and it is implemented as a special case in the interpreter that recognises this pattern.  
In addition, appending another array is possible in-place via `name1&nbsp;=.&nbsp;name1&nbsp;,&nbsp;name2`.

Modifying a single element of an array is less prevalent in J than other languages, since every operation that the user cares about can be broadcast across an array to act on it as a whole.
Verbs can also be modified with [`"`&nbsp;rank][nuvoc-rank] to affect different rank subarrays, which makes manual iteration for modifying an array pointless.
This combination of factors means that the ability to edit single elements is unneeded.
In those cases it would be nice to specify that the interpreter is allowed to mutate by reassigning to the same name.

However, some data structures do not fit into arrays.
Trees, for example, are often not represented well in an array (you could represent a tree with an adjacency matrix).
J does not support mutation of boxed data structures like this, and each change requires the whole structure to be copied.

Adding a limited form of mutability would help this.
The place where it makes the most sense is within the private namespace of a definition.
Here, multithreading aside, the interpreter can be sure that no other bit of code will have access to a given value and rewrite its contents in place as much as it likes.
To demonstrate, consider the following:

```j
u =: {{
  NB. This has to copy, since y starts as being external to u's namespace
  y =. +: y
  NB. Since we made a copy, this namespace now owns y and can mutate it
  y =. *: y
  NB. Any operation that doesn't change the datatype or shape of y can mutate in place
  y =. -: y
  NB. When y is returned from the verb, it is given to the parent namespace and treated normally
  y
}}
```

NuVoc links: [`+:`&nbsp;double][nuvoc-double], [`*:`&nbsp;square][nuvoc-square], [`-:`&nbsp;halve][nuvoc-halve].

In this example, the interpreter would have to read ahead in the direct definition to know that it is safe to mutate `y`.
I will admit it is a slightly contrived example: this could be expressed as a single tacit verb `-:&nbsp;@:&nbsp;*:&nbsp;@:&nbsp;+:`, using [`@:`&nbsp;at][nuvoc-at] to compose the individual verbs, however the general point holds true for a more complex direct definition.
The fundamental idea is one of _ownership_: values are passed by _reference_, but any copies made are _owned_ until they are used in more than one place.
This allows for the interpreter to manage its memory far more precisely.

Another option could be to create a data structure that explicitly allows mutation, like [Clojure&rsquo;s&nbsp;transients][transients].

### Affine character arithmetic

This feature is directly lifted [from BQN][BQN-affine].
It&rsquo;s simple, but brilliant.

Treat characters as an _affine space_.
An affine space is a space that allows three operations:

- Find the difference between two elements
- Add a quantity to an element
- Remove a quantity from an element

Crucially, the ability to directly _add_ two elements is missing.
This prevents C-style `'a'&nbsp;+&nbsp;'b'` (which is `195`, by the way), but allows for `'a'&nbsp;+&nbsp;1&nbsp;==&nbsp;'b'`, `'b'&nbsp;-&nbsp;1&nbsp;==&nbsp;'a'`, and `'z'&nbsp;-&nbsp;'a'&nbsp;== 25`.

Want to parse a grid of digits?
You can write `grid&nbsp;-&nbsp;'0'`.
Rotating characters?
You can just add and subtract as necessary, handling boundary conditions.

I find this far superior to J&rsquo;s approach.
It avoids the extra conversion needed to and from a numeric datatype.
For J, you have to use `a.&i.` which finds the [`i.`&nbsp;index of][nuvoc-index-of] the given character within the [`a.`&nbsp;alphabet][nuvoc-alphabet].
Under some circumstances, you can avoid converting the characters to integers and keep them as bytes with the special combination `&.(a.&i.)` with [`&.`&nbsp;under&nbsp;(dual)][nuvoc-under-dual].

For Unicode, it can be converted to a number using `3&nbsp;u:&nbsp;7&nbsp;u:&nbsp;]` ([`u:`&nbsp;Unicode][nuvoc-unicode]) and manipulated the same way, using the codepoint as a number, but the conversion cannot be avoiding owing to the nature of the UTF-8 encoding.
While both of these work well, the extra work required to convert back and forth between the two representations can be noisy.
This would also discourage using things like `97` to represent `a.i.'a'` (the byte value of the character `a`), which I believe is a benefit.

## Conclusion

First things first, congratulations are in order.
Congratulations for making it this far.
Thank you, dear reader, for indulging me for as long as you have.
I hope you have found this post interesting.

J, and array languages in general, offer a very interesting paradigm for programming in.
I highly recommend learning an array language such as J, APL, BQN, or K, if you are interested in learning more about different styles of programming and ways of tackling problems.
As a tool for quickly developing algorithms, I find J to be an excellent tool to have at my disposal.
It provides the tools necessary to write code quickly and is efficient for getting ideas from concepts to a working example.
Understanding concepts like rank can really help formulating solutions to array-like problems in general, and array languages will guide you towards that as you use them.

The syntax adopted by J has strength in its brevity (albeit this can be a double-edged sword), although it has its flaws.
Lack of operator precedence makes well-written code easy to follow, but there is a large initial learning curve involved when encountering new primitives.
In addition, _poorly-written_ code can be very difficult to read for newcomers.
Invisible conjunctions do not aid this, either.

The advantages of a minimalist type system bring drawbacks with them.
Chiefly, composite data types like records and structs are missing.
This makes sense when you view the equivalent data structure in J as a collection of _columns_, not _rows_, but can be annoying when data processing needs to think about rows.
Furthermore, more advanced types, like unions, are also missing.
They really open the door to more advanced programming in areas like error handling, in my opinion, so J is stuck with exceptions for now.
On the other hand, for its core purposes, advanced types aren&rsquo;t necessary.
When it comes to crunching numbers, the underlying representation is sufficient.

J is missing some of the nicer features common in more recently-developed languages, and especially some of the rough edges can be annoying to have to deal with.
In particular, scoping and file importing are pretty nasty in my opinion.
There&rsquo;s not much of a chance of J changing fundamental parts of it like this, though, so the onus will be on newer array languages like BQN to improve on these shortcomings.

Ultimately, J was a language that was 100% worth me learning, and I encourage anyone looking to learn new approaches to programming to learn _an_ array language.

[addon-dirtrees]: https://code2.jsoftware.com/wiki/Addons/general/dirtrees
[aoc]: https://adventofcode.com/
[APL]: https://aplwiki.com/wiki/Main_Page
[APL game of life]: https://aplwiki.com/wiki/John_Scholes%27_Conway%27s_Game_of_Life
[BQN]: https://mlochbaum.github.io/BQN/
[BQN-affine]: https://mlochbaum.github.io/BQN/tutorial/expression.html#character-arithmetic
[Collatz]: https://en.wikipedia.org/wiki/Collatz_conjecture
[essay-complex]: https://code2.jsoftware.com/wiki/Essays/Complex_Operations
[Euler]: https://projecteuler.net/
[flat array model]: https://aplwiki.com/wiki/Array_model#Flat_array_theory
[flood fill]: https://en.wikipedia.org/wiki/Flood_fill
[J]: https://www.jsoftware.com
[J-2train]: https://code2.jsoftware.com/wiki/Essays/Hook_Conjunction%3F
[J-download]: https://code2.jsoftware.com/wiki/System/Installation
[J-online]: https://jsoftware.github.io/j-playground/bin/html/index.html
[jupyter]: https://jupyter.org/
[J-jupyter]: https://code2.jsoftware.com/wiki/Guides/Jupyter
[LeetCode]: https://leetcode.com/
[leetcode-42]: https://leetcode.com/problems/trapping-rain-water/
[math/misc]: https://code2.jsoftware.com/wiki/Addons/math/misc#trig
[md]: https://github.com/IbzanHyena/md
[NumPy]: https://numpy.org/
[nuvoc]: https://code2.jsoftware.com/wiki/NuVoc
[nuvoc-ace]: https://code2.jsoftware.com/wiki/Vocabulary/aco
[nuvoc-adverse]: https://code2.jsoftware.com/wiki/Vocabulary/codot
[nuvoc-agenda]: https://code2.jsoftware.com/wiki/Vocabulary/atdot
[nuvoc-alphabet]: https://code2.jsoftware.com/wiki/Vocabulary/adot
[nuvoc-amend]: https://code2.jsoftware.com/wiki/Vocabulary/curlyrt#dyadic
[nuvoc-appose]: https://code2.jsoftware.com/wiki/Vocabulary/ampco
[nuvoc-at]: https://code2.jsoftware.com/wiki/Vocabulary/atco
[nuvoc-atom]: https://code2.jsoftware.com/wiki/Vocabulary/Glossary#Atom
[nuvoc-atop]: https://code2.jsoftware.com/wiki/Vocabulary/at
[nuvoc-bond]: https://code2.jsoftware.com/wiki/Vocabulary/ampm
[nuvoc-box]: https://code2.jsoftware.com/wiki/Vocabulary/lt
[nuvoc-cap]: https://code2.jsoftware.com/wiki/Vocabulary/squarelfco
[nuvoc-ceiling]: https://code2.jsoftware.com/wiki/Vocabulary/gtdot
[nuvoc-circle-fns]: https://code2.jsoftware.com/wiki/Vocabulary/odot#dyadic
[nuvoc-compose]: https://code2.jsoftware.com/wiki/Vocabulary/ampv
[nuvoc-const]: https://code2.jsoftware.com/wiki/Vocabulary/zeroco
[nuvoc-copy]: https://code2.jsoftware.com/wiki/Vocabulary/number#dyadic
[nuvoc-cut]: https://code2.jsoftware.com/wiki/Vocabulary/semidot
[nuvoc-each]: https://code2.jsoftware.com/wiki/Standard_Library/stdlib#each
[nuvoc-divide]: https://code2.jsoftware.com/wiki/Vocabulary/percent#dyadic
[nuvoc-double]: https://code2.jsoftware.com/wiki/Vocabulary/plusco
[nuvoc-equal]: https://code2.jsoftware.com/wiki/Vocabulary/eq#dyadic
[nuvoc-fix]: https://code2.jsoftware.com/wiki/Vocabulary/fdot
[nuvoc-foreign]: https://code2.jsoftware.com/wiki/Vocabulary/bangco
[nuvoc-foreigns]: https://code2.jsoftware.com/wiki/Vocabulary/Foreigns
[nuvoc-fork]: https://code2.jsoftware.com/wiki/Vocabulary/fork
[nuvoc-from]: https://code2.jsoftware.com/wiki/Vocabulary/curlylf#dyadic
[nuvoc-gerunds]: https://code2.jsoftware.com/wiki/Vocabulary/GerundsAndAtomicRepresentation
[nuvoc-gte]: https://code2.jsoftware.com/wiki/Vocabulary/gtco#dyadic
[nuvoc-halve]: https://code2.jsoftware.com/wiki/Vocabulary/minusco
[nuvoc-hook]: https://code2.jsoftware.com/wiki/Vocabulary/hook
[nuvoc-hypergeometric]: https://code2.jsoftware.com/wiki/Vocabulary/hcapdot
[nuvoc-if]: https://code2.jsoftware.com/wiki/Vocabulary/ifdot
[nuvoc-increment]: https://code2.jsoftware.com/wiki/Vocabulary/gtco
[nuvoc-index-of]: https://code2.jsoftware.com/wiki/Vocabulary/idot#dyadic
[nuvoc-indices]: https://code2.jsoftware.com/wiki/Vocabulary/icapdot
[nuvoc-infinity]: https://code2.jsoftware.com/wiki/Vocabulary/under
[nuvoc-infix]: https://code2.jsoftware.com/wiki/Vocabulary/bslash#dyadic
[nuvoc-insert]: https://code2.jsoftware.com/wiki/Vocabulary/slash
[nuvoc-integers]: https://code2.jsoftware.com/wiki/Vocabulary/idot
[nuvoc-inverse]: https://code2.jsoftware.com/wiki/Vocabulary/Inverses
[nuvoc-is-global]: https://code2.jsoftware.com/wiki/Vocabulary/eqco
[nuvoc-is-local]: https://code2.jsoftware.com/wiki/Vocabulary/eqdot
[nuvoc-key]: https://code2.jsoftware.com/wiki/Vocabulary/slashdot#dyadic
[nuvoc-left]: https://code2.jsoftware.com/wiki/Vocabulary/squarelf#dyadic
[nuvoc-lt]: https://code2.jsoftware.com/wiki/Vocabulary/lt#dyadic
[nuvoc-max]: https://code2.jsoftware.com/wiki/Vocabulary/gtdot#dyadic
[nuvoc-min]: https://code2.jsoftware.com/wiki/Vocabulary/ltdot#dyadic
[nuvoc-minus]: https://code2.jsoftware.com/wiki/Vocabulary/minus#dyadic
[nuvoc-modifier-trains]: https://code2.jsoftware.com/wiki/Vocabulary/fork#invisiblemodifiers
[nuvoc-oblique]: https://code2.jsoftware.com/wiki/Vocabulary/slashdot
[nuvoc-open]: https://code2.jsoftware.com/wiki/Vocabulary/gt
[nuvoc-outfix]: https://code2.jsoftware.com/wiki/Vocabulary/bslashdot#dyadic
[nuvoc-passive]: https://code2.jsoftware.com/wiki/Vocabulary/tilde#dyadic
[nuvoc-plus]: https://code2.jsoftware.com/wiki/Vocabulary/plus#dyadic
[nuvoc-polynomial]: https://code2.jsoftware.com/wiki/Vocabulary/pdot#dyadic
[nuvoc-power]: https://code2.jsoftware.com/wiki/Vocabulary/hat#dyadic
[nuvoc-power-of-verb]: https://code2.jsoftware.com/wiki/Vocabulary/hatco
[nuvoc-prime-factors]: https://code2.jsoftware.com/wiki/Vocabulary/qco
[nuvoc-primes]: https://code2.jsoftware.com/wiki/Vocabulary/pco
[nuvoc-prefix]: https://code2.jsoftware.com/wiki/Vocabulary/bslash
[nuvoc-rank]: https://code2.jsoftware.com/wiki/Vocabulary/quote
[nuvoc-rank-info]: https://code2.jsoftware.com/wiki/Vocabulary/RankInfoIsImportant
[nuvoc-reflex]: https://code2.jsoftware.com/wiki/Vocabulary/tilde
[nuvoc-residue]: https://code2.jsoftware.com/wiki/Vocabulary/bar#dyadic
[nuvoc-reverse]: https://code2.jsoftware.com/wiki/Vocabulary/bardot
[nuvoc-roll]: https://code2.jsoftware.com/wiki/Vocabulary/query
[nuvoc-roll-fixed]: https://code2.jsoftware.com/wiki/Vocabulary/querydot
[nuvoc-roots]: https://code2.jsoftware.com/wiki/Vocabulary/pdot
[nuvoc-rxapply]: https://code2.jsoftware.com/wiki/Studio/Regular_Expressions#rxapply
[nuvoc-rxcut]: https://code2.jsoftware.com/wiki/Studio/Regular_Expressions#rxcut
[nuvoc-samer]: https://code2.jsoftware.com/wiki/Vocabulary/squarerf#dyadic
[nuvoc-scs]: https://code.jsoftware.com/wiki/Vocabulary/SpecialCombinations
[nuvoc-seq-mac]: https://code2.jsoftware.com/wiki/Vocabulary/semico#dyadic
[nuvoc-shape]: https://code2.jsoftware.com/wiki/Vocabulary/dollar#dyadic
[nuvoc-shift]: https://code2.jsoftware.com/wiki/Vocabulary/bardot#monadicfit
[nuvoc-sqrt]: https://code2.jsoftware.com/wiki/Vocabulary/percentco
[nuvoc-square]: https://code2.jsoftware.com/wiki/Vocabulary/starco
[nuvoc-suffix]: https://code2.jsoftware.com/wiki/Vocabulary/bslashdot
[nuvoc-table]: https://code2.jsoftware.com/wiki/Vocabulary/slash#dyadic
[nuvoc-tally]: https://code2.jsoftware.com/wiki/Vocabulary/number
[nuvoc-tie]: https://code2.jsoftware.com/wiki/Vocabulary/grave
[nuvoc-times]: https://code2.jsoftware.com/wiki/Vocabulary/star#dyadic
[nuvoc-toupper]: https://code2.jsoftware.com/wiki/Standard_Library/stdlib#toupper
[nuvoc-under]: https://code2.jsoftware.com/wiki/Vocabulary/ampdotco
[nuvoc-under-dual]: https://code2.jsoftware.com/wiki/Vocabulary/ampdot
[nuvoc-unicode]: https://code2.jsoftware.com/wiki/Vocabulary/uco
[nuvoc-valence]: https://code2.jsoftware.com/wiki/Vocabulary/Valence
[nuvoc-verb-info]: https://code2.jsoftware.com/wiki/Vocabulary/bdotu
[nuvoc-words]: https://code2.jsoftware.com/wiki/Vocabulary/semico
[relimport]: https://github.com/IbzanHyena/relimport
[transients]: https://clojure.org/reference/transients
