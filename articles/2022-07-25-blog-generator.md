// title: Hello, World: Making a Static Blog Generator from Scratch
// author: Ibzan
// date: 2022-07-25

# Hello, World: Making a Static Blog Generator from Scratch

For the first blog post on this site, I thought an introduction to the blog generator that it is made by is in order.
I wrote it myself over the course of a few days in the [J programming language][J], an array language based on [APL][APL].
All in all, the project spans five (or maybe six) GitHub repos, and was quite fun to work on.

## J

I expect to be writing about J a decent amount on this blog, and the blog generator was written in J, so I will quickly introduce the language.
At a future point in time I may write an entire blog post introducing J as a language and link to it instead, since I expect it will be a useful resource for J-related posts.

### Why J?

Because I've been writing short J programs recently and wanted to try a larger project.
So far, all of my J code has been intended to run from the J terminal.
This would be different - it needs to be run from the command line, ideally on a CI runner somewhere.
I also wanted to see how J tackles larger projects.

### About J

As an array language, J is quite different to the paradigms people are often most used to.
The array style is generally declarative, strict, dynamically typed, interpreted, and very good at dealing with data that fits into a rectangle.

An array in the sense of an array language is mathematically a tensor.
Within programming, these are often called arrays as a result of their implementations.
An array has a _rank_, which is the number of dimensions in it.
A rank zero array is just a single element.
A rank one array is a vector, rank two is a matrix (although they do not behave exactly the same as a matrix does in linear algebra at all times), and so on.

J, like most array languages, is read right-to-left and has no "operator" precedence (strictly speaking J primitives are not operators, but it's close enough for now).
Instead, the interpreter applies operators in the order it encounters them.
This system has both advantages and disadvantages.
The main advantage is that the lack of operator precedence greatly simplifies writing code and interpreting it, especially in a language with a large number of primitives.
The disadvantage, however, is that it can make quickly reading code misleading, especially for people who are used to left-to-right and operator precedence rules.

```j
   NB. Lack of precedence rules can cause unexpected behaviour for newcomers
   5 * 2 + 1
%%% 5 * 2 + 1 %%%
   (5 * 2) + 1
%%% (5 * 2) + 1 %%%
   1 + 5 * 2
%%% 1 + 5 * 2 %%%
   NB. If we wanted to double then add 1, can use primitives instead
   >:@+: 5
%%% >:@+: 5 %%%
```

One of the main features of array languages is broadcasting of operations.
Where an imperative language would have one write a `for` loop, or a functional language would require `map`, array languages instead implicitly broadcast an operation across their input arrays.
The precise semantics of this differ from language to language, but in J it depends on the ranks of the inputs.
To demonstrate, here are some examples.

```j
   NB. Numbers 0 to 4
   i. 5
%%% i. 5 %%%
   NB. Add one to them
   1 + i. 5
%%% 1 + i. 5 %%%
   NB. Alternatively, add to another vector
   10 11 12 13 14 + i. 5
%%% 10 11 12 13 14 + i. 5 %%%
   NB. A two-dimensional array
   i. 3 4
%%% i. 3 4 %%%
   NB. Can broadcast across it in three ways
   NB. Way one: a single element
   1 + i. 3 4
%%% 1 + i. 3 4 %%%
   NB. Way two: vectors
   1 1 1 1 , 2 2 2 2 ,: 3 3 3 3
%%% 1 1 1 1 , 2 2 2 2 ,: 3 3 3 3 %%%
   (1 1 1 1 , 2 2 2 2 ,: 3 3 3 3) + i. 3 4
%%% (1 1 1 1 , 2 2 2 2 ,: 3 3 3 3) + i. 3 4 %%% 
  NB. Way three: another matrix
  (10 * i. 3 4) + i. 3 4
%%% (10 * i. 3 4) + i. 3 4 %%%
```

In addition to this, J has a wide variety of primitives.
Like, a lot of primitives.
The wiki has a [page dedicated to the vocabulary available][NuVoc], which covers everything available.
Combined with broadcasting, this allows J code to be extremely terse, whether that is a good thing or not.

## md: Markdown Processor

The first element of the blog generator is converting the Markdown source into static HTML for putting on a website.
Enter the first project: [md][md].
md is a very barebones Markdown to HTML processor that supports a small subset of Markdown that I like enough to want to use, plus a small extension.
At the time of writing, md supports the following Markdown features:

- Unordered lists (ha)
- Code blocks
- Inline text formatting (**strong**, _emphasised_, ~~deleted~~, `monospace`)
- Headers
- [Hyperlinks][home]
- Paragraphs
- Line breaks
- As an extension, the ability to run J code in the document

Although small, this is generally all I use in a typical block of writing.
Any further features required can always be added later, since it's my project.
Or I can move to a less janky system.

md's strategy is as follows.
First, it breaks the input text into paragraphs, which are separated by two line feed (LF) characters.
Then, it splits each paragraph into sections separated by line breaks (space space LF) and adds an HTML line break inside each of them.
Then, it classifies the paragraph based on a guess about what kind of paragraph it is.

This leads to md having a lot of edge cases that don't work very well.
Nested lists, lists embedded within paragraphs and, empty lines in code blocks all fail to work as they should according to the Markdown specification.
If I move away from md for any reason, it will be because I want to avoid having to completely restructure it to make it more robust and support more features.

## blog: Static Blog Generator

md generates the HTML.
But we need a way of processing a collection of files into a site.
Enter: [blog][blog].
blog is also written in J, and generates HTML files and an index page from a given directory containing Markdown source and assets.

blog uses template HTML files to insert relevant data from the Markdown into pre-made pages.
It also extracts metadata from the file to generate the index page, and copies assets into the finished site.

The biggest pain point when creating blog was actually that last one - copying assets.
J does not have any in-built recursive directory copying utilities, and the one I wrote somehow resulted in the interpreted segfaulting.
As a backup, I resorted to invoking `cp -r` on the command line and not thinking too hard about it.

Aside from that, blog is mostly uninteresting glue code to perform the aforementioned tasks.

## setup-j: CI Glue

J is not a widely-popular language.
This means that a lot of the common infrastructure you might find for other languages is missing.
I was able to find a [Docker image][docker] that creates a J environment, which had a grand total of 560 Docker pulls at time of writing.
For comparison, Python's official Docker image has over 1 billion pulls.

When deciding how to deploy this site, I settled on using GitHub Actions to build and deploy the site on each push.
This necessitates a step that either installs the J interpreter or runs J within a Docker image.
After some further deliberation I opted to install the J interpreter in the CI runner, and then invoke it from the command line in later steps.

As you may expect, I didn't find any pre-existing Actions that did this, so I had to write my own.
Fortunately, composite actions (i.e. an action that just runs shell scripts) are relatively easy to make and a mere 16 lines of YAML and 17 lines of bash were needed to install J.
The action can be found [here][setup-j] should you wish to use it yourself.

## Blog Repo Glue

There isn't much to write about the repo the blog itself lives in, containing HTML templates, CSS, and the dependencies as submodules for ease of installation.
To run the CI itself was just a matter of sticking a bit more glue to go with my glue.
One CI script later and the blog is being automatically re-deployed on commit.
Glorious.

## Conclusion

Despite all the difficulties involved with various stages of the project, over all it was a lot of fun to throw together.
As a language, J is not particularly well-suited to this genre of problem. Part of this is because of the array model that J uses, called the flat array model.
In the flat array model, all arrays are rectangular pieces of data in some number of dimensions.
A string in J is a vector (rank one array) of characters.
This is one of the problems - to represent a collection of strings of varying length, they must either be padded with fill characters (typically spaces) to form a rectangle, or boxed (stored behind a pointer) so that the array of _boxes_ can be a rectangle.
Since I wanted to avoid adding extra whitespace to the documents I'm working with, boxing was required, but this adds complexity to the code needed to manipulate the text.

There are different array models out there.
Of particular note in this scenario is the based array model, employed by [BQN][BQN].
Alongside BQN's representation of strings, it vastly simplifies storing an array of different-length strings.
I won't go into too much detail about it here (you can read further about it in the [BQN documentation][based]), but it allows for storing a vector of strings as one would in a typical programming language, which eases manipulation.

However, there are certain parts that are still very elegant within J to write.
Lots of small operations are quite pleasant or have nice solutions.
I enjoy iterating on solutions in J to find a smaller or more elegant form.
For example, one operation performed by blog is replacing the file extension of the Markdown source with `html`.
I use `'.' <;._1 @ , ]` to split the file extensions, which is quite a nice approach (in my opinion).
First it prepends the character `.` to the filename. Then, `<;._1` splits the string into chunks based on its first character.
My earlier solutions accomplished the same task but more verbosely.
Reducing snippets like this can be a fun diversion and is one of the draws of the language to me.

I expect to write plenty more J in the future, although it is far from the only language I like writing.
Rust, F#, Python, Haskell, C#, and sometimes C are also languages I like a lot (or at least have a love-hate relationship with), so expect to see an appearance from them at some point on this blog.
There are also a few other languages I have in mind that I might try out at some point.
Outside of programming I may write about any other hobbies that I feel particularly inspired to, but so far I don't have any plans.

Until next time,  
Ibzan

[APL]: https://aplwiki.com/
[based]: https://mlochbaum.github.io/BQN/doc/based.html
[blog]: https://github.com/IbzanHyena/blog/
[docker]: https://www.docker.com/
[BQN]: https://mlochbaum.github.io/BQN/index.html
[home]: index.html
[J]: https://www.jsoftware.com/
[md]: https://github.com/IbzanHyena/md/
[NuVoc]: https://code.jsoftware.com/wiki/NuVoc
[setup-j]: https://github.com/IbzanHyena/setup-j
