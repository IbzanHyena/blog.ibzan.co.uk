// title: Why I Love J
// author: Ibzan
// date: 2026-05-02

I’ve been writing code in the [J programming language][jsoftware] for nearly four years now.
Amongst languages I write (of which there are many), it’s one of only two I have used with daily consistency for so long—and the other is Python, which is part of my day job.
In this post, I want to write a bit about why I keep coming back to J, and what makes me love it so much.

## Practical expressiveness

When it comes to syntax, J, like most array languages, is ruthlessly efficient.

Operator precedence doesn’t exist—everything is evaluated right-to-left.
There is no need for brackets to invoke functions, or even for extensive function signatures to exist.
Combined with the standard array language ability to perform practically all operations on entire arrays of values at once, rather than individual elements, J is a language of incredible brevity.

Tacit programming, where arguments are omitted as data can just flow between code instead, promotes the ability to read _how_ data is being transformed, without needed to include temporary or repeated names, and this is something that like many array languages J excels in.
[Modifier trains][trains] are one such manfestation of this, allowing for “invisible” composition of functions together without need for an explicit invocation of some higher-order helper function (“invisible”-with-quotes only in the sense that you have to know to look for them).
However, the influence of (mostly-)tacit programming runs deeper.
Applying a sequence of functions to a single input can be expressed just by chaining the verbs together.
While learning, it can be one of the harder bits of the language to get used to at first, but afterwards it’s an essential tool for writing good J.

The chief benefit of brevity in this case is that the code takes up less space and is quicker to type.
Algorithms which take fifteen or so lines in most languages often fit in one or two in array languages.
When working on something new, and when combined with the RTL precedence, this hones the iteration cycle into a tight loop, aided by the malleability of the REPL environment.
Each intermediate version of a function while building an algorithm can immediately be extended into the next by simply adding more code immediately to the left of it.
Subsets of a function definition can be extracted as-is and turned into a new function with little effort.

In short, it makes it easy to prototype and refine one’s work: there is only the algorithm and the data.
I frequently find myself writing a few characters of J, looking at the result, and then typing just a few more to perform the next step.
Few languages give me such a lack of mental friction.

More generally, fundamental programing skills like how to structure a program and break down a program into constituent parts haven’t gone away, but rather have different building blocks to work with.
When an algorithm fits on one line, the functions you build can be bigger.

Naturally, none of this would work without the incredible range of primitives J contains.
While the theoretical expressive power of J is no better than any other Turing-complete language, the practicality of accessing it is far superior.
You only need to scroll through [NuVoc][nuvoc] to see just how many operations are made immediately available to the programmer—and would typically be a library function in most languages.

Compared to other array languages, the sheer number of primitives is something that sets J apart.
I believe this has emerged because J opted to use ASCII for its primitives, rather than the more typical symbology found in APL, BQN, uiua, and others.
Simply put, there aren’t enough good Unicode glyphs to make some of the primitives J uses; most array languages try to pick some consistent symbology to serve as a mnemonic.
While many of them are basic and could be built out of other ones already provided, providing the more complete set J does can make writing code faster, in my experience.

The way that J tackles the symbol problems is by allowing its glyphs to be multiple characters.
This is a bit of a necessity for an ASCII-based array language with lots of primitives.
In the vast majority of cases, they’re built out of a single root symbol and then an inflection (either `.` or `:`).
Similar operations are grouped by the radical and the inflection.
Thus, `+` is add and `+:` is double, `-` is subtract and `-:` is halve, `*` is multiply and `*:` is square, and `%` is divide and `%:` is square root.
This second axis provides the memorability of related symbols without leaving ASCII.

## Pragmatism

The breadth of primitives segues nicely to what I consider to be another J asset—a willingness to carry more complexity in the language in a sensible way.

Some array languages, such as BQN and uiua, have only one or two numeric types in the form of double-precision floating-point numbers.
This is fine for most use cases.
Hell, doubles can fit integers up to 2^53-1 just fine.

But it isn’t fine for _all_ use cases.

There are times when having access to arbitrary-size integers is useful—or rationals, or even long floats for maximum numerical precision.
J doesn’t shy away from this nuance by making it unavailable to the programmer.
Instead, it exposes tools to work with it, and rules for how operations can transform datatypes.
By giving the programmer the ability to interact with more types, J trades away some of the pure simplicty of a single numeric type for enhanced versatility.

Another example of J picking a nuanced middle ground is its approach to code optimisations.
The interpreter performs certain optimisations when it can detect them; this is fairly typical for any language.
However, you can also choose to write code using what J calls [“special combinations”][special-combinations].
A special combination is a short snippet of code, typically two or three operations long, which the interpreter has special-case handling for.
When they are used, the interpreter recognises the pattern and uses a dedicated code path to handle it rather than the standard implementations of the operators involved.
At first, this explicit opt-in behaviour by changing the source code may seem unwieldly or opaque, but it’s a side effect of the simplicity of the interpeter’s control loop, which has benefits in other areas when considered on the whole.
Special combinations are a sensible trade-off between the effort the interpreter has to put into understanding code and the effort the programmer has to put into writing efficient code.

However, J is still open to implementing relevant languages features developed elsewhere.
A nice example of this is ”structural under”—in short, this is a modifier (higher-order function) that allows you to take an array, reshape it somehow, perform some manipulation on the selected elements, and then put them back into the source array.
To my knowledge, this concept originated in variants of APL that eventually morphed into BQN, and has since spread across other array languages—but not all.
While work is still ongoing, it’s good to see that J can support adopting newer ideas developed from other languages and incorporate them into its own style.

Finally, I would describe J as stable, not stagnant.
Development proceeds on a yearly release cycle, with things being added, deprecated, removed, and proposed.
However, the main bulk of the language hasn’t changed for years.
Things are generally backwards- and forwards-compatible, and there isn’t a constant churn of features.
For uiua, this was unfortunately what put me off of it for a time, even though I knew what I was getting into.

## Utility beyond J

It’s all well and good to talk about why I like J for the sake of J, but I do also find uses for it beyond that.

One of the chief ones comes in its descdents, numpy and torch, both of which are heavily based on the J (and APL) array models.
While these languages apply some of the concepts differently to J, and have much more syntax to work around in their host languages, the fundamentals of array thinking and programming are identical.
Using J, I’ve managed to efficiently prototype a number of algorithms for my job as a data scientist and then translated them into torch or numpy to be actually used.

Similarly, for two other tasks I hold dear (plotting graphs and calculating statistics), J has all the tools I need to work on something quickly.
I’ve been able to write short snippets of code on my phone to calculate quiz results, determine odds of rolling dice and plot them, calculate ratios for baking bread on the fly, and run Monte Carlo simulations of more complex systems.
The code I have to write for these kinds of tasks is far easier to handle than in something like Python.
Having learnt J as a tool to use, it’s become a very natural one to reach for to help solve many problems in other areas for me, which is perhaps the ultimate achievement.

It’s fair to say that there are limits to this utility; I still use other languages for many tasks that are not well-reduced to processing data algorithmically.
In particular, J really struggles when functions start requiring a large number of parameters and with processing nested data.
It’s possible, just not nearly as pleasant.
That said, I still chose to write the markdown parser and blog generator for this site in J—it’s entirely possible to write typical software in the language.

The real Achilles heel for typical software is the lack of an ecosystem for certain areas.
There’s no Django or Rails, no Entity Framework Core.
You’re on your own if you want to tread these paths, which leaves a lot of work to be done.

## Conclusion

I love J.

I find it fun to write.
I enjoy the programs it produces.
I learnt a lot of new ways (the array paradigm and mindset, tacit/point-free programing, and less reliance on thorough typing, to name a few) to think about software by getting to grips with it.

Ultimately, I think it’s how engaging and convenient writing J is that has kept me using the language.
When each program is satisfying, it’s easy to come back for more.
By comparison, many of the languages I’ve picked up in the last four years—such as F#, Rust, Factor, Lisp, Haskell, and more—I don’t use nearly as frequently, in large part because the amount of work to arrive at answers for small things just isn’t worth it.
The combination of a low threshold and high reward makes it an obvious choice for me.

There are still areas of J which I haven’t explored—particularly its database jd, and its object-oriented features.
I’m looking forward to exploring these in more detail to deepen my understanding of the languages.

If you’re curious about J, I highly recommend giving it a try!
[NuVoc][nuvoc] has a one-page reference of most of the language, and you can [try it online][playground].
If you’ve got your own thoughts about J, I’d love to hear them—contact details are on the homepage.

Thanks for reading!

[jsoftware]: https://www.jsoftware.com/#/
[nuvoc]: https://code.jsoftware.com/wiki/NuVoc
[playground]: https://jsoftware.github.io/j-playground/bin/html2/
[special-combinations]: https://code.jsoftware.com/wiki/Vocabulary/SpecialCombinations
[trains]: https://code.jsoftware.com/wiki/Vocabulary/fork
