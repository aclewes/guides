# What is regex
So what are regular expressions (regex/regexes)? [Wikipedia](https://en.wikipedia.org/wiki/Regular_expression) helpfully tells us regex is:

> _in theoretical computer science and formal language theory, a sequence of characters that define a search pattern. Usually this pattern is then used by string searching algorithms for "find" or "find and replace" operations on strings._

Lovely. The last sentence is definitely the more _useful_ part of that description. Regular expressions are kind of a scripting language that acts as a turbocharged version of the old asterisk/question mark wildcards. However, just like someone who isn't computer literate won't understand that `wh*` can be used to match `who`, `what`, `where`, `when` and `why`; someone who doesn't know regular expressions definitely won't understand that [`bu|[rn]t|[coy]e|[mtg]a|j|iso|n[hl]|[ae]d|lev|sh|[lnd]i|[po]o|ls` matches the last names of elected US presidents but not their opponents](https://www.xkcd.com/1313/) (excluding Trump/Clinton).

In the hands of someone who knows how to properly wield it, regex is an incredibly powerful tool for not only finding but also tearing apart and replacing text. Assuming you had an appropriate file move program, you could easily create a regex to convert `IT Crowd - s01e01.mkv` to `IT Crowd/Season 1/Episode 1.mkv` and any other combination of series name, season number and episode number in the same format.
# How to write regex
This is the basic usage of regex. You need to do some kind of string manipulation and you want an expression that'll do it for you. Assuming you can't find one on stackoverflow, you'll probably need to write it yourself.

> Note: It's **extremely** useful to write regex inside of a regex debugger with an example of the text you're working against. I almost always do this because it's easier and quicker than having to debug and expression in-vivo.
There are many regex debuggers out there, but the one I prefer to use is [regex101](https://regex101.com/) as it not only shows the expressions results, a breakdown of your expression in tree form and the match results all in real-time, but also includes a library of valid tokens in case you forget one (I always forget the format of lookahead/lookbehind) and a step-by-step debugger, which is **absolutely indispensible** when optimising an expression.

For the following we'll use the example regex `Ste(v|f|ph)[ea]n?` which matches Steven, Steve, Stephen _and_ Stefan.

Regular expressions are built up of three main elements: **strings**, **quantifiers** and **groups**.

Strings are the bits of text in the expression that say _what_ should be matched. In the example the strings are `Ste`, `v`, `f`, `ph`, `e`, `a` and `n`.

# How to read regex
# How to optimise regex
[Amazingly useful blog post by the author of regex101.com](https://firasdib.com/blog/a-brief-tour-of-regular-expressions/)