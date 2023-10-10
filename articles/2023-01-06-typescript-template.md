// title: TypeScript as an HTML Templating Format
// date: 2023-01-06
// author: Ibzan

# TypeScript as an HTML Templating Format

I made a [website to serve as a character reference][ref] recently.
It&rsquo;s [written in TypeScript using Next.js][ref source], and it got me thinking.

Is TypeScript (or JavaScript) a really good format for producing HTML from templates?

## Why Next.js

In short: I chose to deploy on CloudFlare.
CloudFlare already hosts this blog and the domain it lives on, so I figured that I may as well re-use the same infrastructure for it.
CloudFlare pages are also extremely easy to set up _if you are using a readily-supported framework_.
This blog does not use such a framework ([since I wrote it myself][blog blog]), so I wanted something easier this time.

My requirements were pretty simple:

- Static HTML export
- Can populate templates in a reasonable manner
- Won&rsquo;t be prohibitive down the line if I want to add functionality
- Supported by CloudFlare

That last condition really narrowed it down a lot.
Browsing the [CloudFlare Pages framework guides][cf pages fws], I decided that Next.js looked suitable for my puposes, even if it meant tangling with JavaScript and/or TypeScript.
Neither of these languages are ones I use professionally or for fun.
Being involved in data, Python is by far the most dominant language, and is probably my most experienced tool for typical jobs like this.
Python also has good templating solutions like [Jinja][jinja] and [Mako][mako], so I wouldn&rsquo;t need to implement my own (like I did for this blog).

However, ease of deployment was a major factor for me.
CloudFlare workers do support [a collection of languages][cf pages langs], but Node looked to me to be the most common option and well supported.
A lot of the frameworks that come with templates for CloudFlare Pages are also written using Node, so in I went!

## Writing Some TypeScript

JavaScript is an ok language in my opinion.
It gets some things horribly wrong, like the type system, certain function behaviour, and some poor old design decisions that have hung around.
On the other hand, it allows for some relatively expressive code out of the box.
I think JavaScript is probably one of the more competitive options when trying to do literate programming.

TypeScript in theory fixes a lot of my issues with raw JavaScript.
I think it&rsquo;s a testament to the design of TypeScript&rsquo;s type system (try saying that ten times fast!) that it manages to encapsulate so much icky JavaScript behaviour in a language that can be incrementally adopted.
In addition, union types in the way that TypeScript has to implement them are a very interesting concept I think about exploring more whenever I think about a type system for a while.

However, for this project (at least at an early stage), using the full extent of TypeScript&rsquo;s power was going to be far from necessary.
All I really wanted to do was find some way to generate some static HTML.
The [Next.js template][project template] I started with includes an option to do just that, while also providing live reload for development.

Exploring the template, I found fairly quickly how it was generating HTML: it&rsquo;s just in the TypeScript code.
As I understand it, this is a feature of [JSX][JSX] which allows the programmer to directly embed HTML into a suitable TypeScript (or, since TypeScript is a superset of JavaScript, JavaScript) file.
This eases a lot of the potential pain when handling templates between the file they are written in and the script that populates them.
Everything can instead be passed around in-code with minimal fuss.
It&rsquo;s no surprise that JSX became popular as a result of React and other front-end libraries since it makes this so easy.
At the same time, it means the programmer doesn&rsquo;t need to learn a separate template language — everything they already know in TypeScript is valid.

I will admit it feels a bit funny writing things in basically HTML again, instead of using some format like Markdown (which this blog is written in) to generate it.
However, for a project of this scale, it&rsquo;s really convenient.
You can just get straight down to implementing stuff with minimal fuss.

I spent far longer on this project tweaking CSS to get the appearance acceptable that thinking about how to template the pages and generate them.
That in itself was a huge boon, saving me a lot of time.
So, we reach the crux of this blog post:

## Is TypeScript the best templating language?

Maybe.
I don&rsquo;t really have enough recent experience with the competitors to say.
In particular, they&rsquo;re more flexible in that they can generate output beyond just HTML.

For generating a webpage, however, I think TypeScript with JSX may be the simplest option.
Everything can be written in one place in one language.
Tooling and support is generally good.
Interacting with the Node ecosystem is bearable.

In conclusion, I think it works well for a nice quick job like this, while remaining robust and extensible for future updates.
Naturally, after saying this, I&rsquo;m sure I&rsquo;ll come to regret my words.
Until then, however, everything is great!

[blog blog]: /2022-07-25-blog-generator
[cf pages fws]: https://developers.cloudflare.com/pages/framework-guides/
[cf pages langs]: https://developers.cloudflare.com/pages/platform/build-configuration/#language-support-and-tools
[jinja]: https://jinja.palletsprojects.com/en/3.1.x/
[JSX]: https://www.typescriptlang.org/docs/handbook/jsx.html
[mako]: https://www.makotemplates.org/
[project template]: https://github.com/vercel/next.js/tree/canary/examples/with-static-export
[ref]: https://ref.ibzan.co.uk
[ref source]: https://github.com/IbzanHyena/ref.ibzan.co.uk

