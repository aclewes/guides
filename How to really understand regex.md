# What is regex
So what are regular expressions (regex/regexes)? [Wikipedia](https://en.wikipedia.org/wiki/Regular_expression) helpfully tells us regex is:

> _in theoretical computer science and formal language theory, a sequence of characters that define a search pattern. Usually this pattern is then used by string searching algorithms for "find" or "find and replace" operations on strings._

Lovely. The last sentence is definitely the more _useful_ part of that description. Regular expressions are kind of a scripting language that acts as a turbocharged version of the old asterisk/question mark wildcards. However, just like someone who isn't computer literate won't understand that `wh*` can be used to match `who`, `what`, `where`, `when` and `why`; someone who doesn't know regular expressions definitely won't understand that [`bu|[rn]t|[coy]e|[mtg]a|j|iso|n[hl]|[ae]d|lev|sh|[lnd]i|[po]o|ls` matches the last names of elected US presidents but not their opponents](https://www.xkcd.com/1313/) (excluding Trump/Clinton).

In the hands of someone who knows how to properly wield it, regex is an incredibly powerful tool for not only finding but also tearing apart and replacing text. Assuming you had an appropriate file move program, you could easily create a regex to convert `IT Crowd - s01e01.mkv` to `IT Crowd/Season 1/Episode 1.mkv` and any other combination of series name, season number and episode number in the same format (in case you're wondering, the expression and substitution are `(.+) - s0*(\d+)e0*(\d+)\.(.+)` and `$1/Season $2/Episode $3.$4` respectively).

![Everybody stand back! I know regular expressions!](https://i.imgur.com/ZIA1Hw5.png)

# How to write regex
This is the basic usage of regex. You need to do some kind of string searching or manipulation and because you're a lazy developer, you want an expression that'll do it for you. Assuming you can't find one on stackoverflow, you'll probably need to write it yourself.

> Note: It's **extremely** useful to write regex inside of a regex debugger with an example of the text you're working against. I almost always do this because it's easier and quicker than having to debug and expression in-vivo.
There are many regex debuggers out there, but the one I prefer to use is [regex101](https://regex101.com/) as it not only shows the expressions results, a breakdown of your expression in tree form and the match results all in real-time, but also includes a library of valid tokens in case you forget one (I always forget the format of lookahead/lookbehind) and a step-by-step debugger, which is **absolutely indispensible** when optimising an expression.

Basic regular expressions are built up of three main elements: **strings**, **quantifiers** and **groups**.

For the following we'll use the example regex `Al+([aey]|ai)n` which matches Alan, Alen, Allan, Allen, and Allyn.

### Strings
Strings are the bits of text in the expression that say _what_ should be matched. In the example the strings are `A`, `l`, `ai`, `n`, and `[aey]`.

The first four strings are simple **Literals**, which simply mean 'match these specific characters in this specific order'. `A` will match any instance of 'A', while `ai` will match any instance of 'ai', but not 'a', 'i', or 'ia'.

`[aey]` is _slightly_ different, being **Character Class** rather than a Literal. `[aey]` will match any instance of 'a', 'e', or 'y'. An easy way to think of it is that `ai` is `a AND i`, while `[aey]` is `a OR e OR y`. Character Classes can also be **inverted** to match any character _except_ the ones listed; `[^abc]` will match any character that _isn't_ 'a', 'b', or 'c'.

There are also some more advanced strings that aren't in our example:

* `.` will match any character (except newline by default, but this is configurable).
* `[x-y]` will match any character between 'x' and 'y' (e.g. `[a-g]` is the equivalent of `[abcdefg]`).
* `\w` will match any 'word' character (a-z, A-Z, and underscore), `\W` will match the inverse.
* `\s` will match any 'whitespace' character (space, tab, carriage return, newline, and page break), `\S` will match the inverse.
* `\d` will match any digit, `\D` will match the inverse.
* `\b` will match 'word boundaries', the point at the start or the end of a sequence of 'word' characters (but won't match any of the characters). For example, running `\b\w` on `One Two Three` will match 'O', 'T', and 'T'.

### Quantifiers
Quantifiers are the biggest thing that transforms regex from a simple find to something much more flexible by allowing to define how many times a String can appear in a row. You apply a quantifier by appending it to a String, for example `a+`. There are five main types of quantifiers:

* `?` means 'zero or one instances of the string'.
* `{x}` means 'x instances of the string'.
* `{x,y}` means 'between x and y instances of the string'.
* `+` means 'one or more instances of the string'.
* `*` means 'zero or more instances of the string`.

In our example, we use the `+`  Quantifier on the `l` so we can match both 'Alan' and 'Allan', however this will also match 'Allllllllllllllan'. If we want to exclude Alans with more than two l's, we can swap the `+` out for `{1,2}`.

> Note: **A Quantifier will only apply to the single character or Character Class directly preceding it.**
For example, in the previous paragraph `Al+` matches 'Allll', not 'AlAlAlAl'.

### Groups
If Quantifiers let us define how many times a String can appear in a row, then Groups let us define how many times entire _groups_ of Strings can appear in a row. They also let us create the equivalent of Character Classes for Strings, and 'capture' pieces of the text to use later.

If we wanted to make a regex to match strings of 'ha's of any length (e.g. 'ha', 'hahaha', or 'hahahahahahahahahahaha'), then we can't really use Character Classes or String Quantifiers. Instead, we want to use a Group to match one or more instances of the multi-character Literal `ha`.

In our example, we use the `([aey]|ai)` Group to match _either_ `[aey]` or `ai`. You can use this method to combine as many possibilities as you want (e.g. `[aey]` can be rewritten as `(a|e|y)` if you really want to).

Finally, the most important use of Groups is to 'capture' text. Say we wanted to find all text in parentheses, assuming no nested ones, we _could_ use `\(.+?\)` but if we run it against 'This is some text (with some other text in parens)', we run into a problem:

> (with some other text in parens)

We get the parentheses being included in the result! We don't want that, we just want the text _inside_ the parentheses.

# How to read regex
# How to optimise regex
[Amazingly useful blog post by the author of regex101.com](https://firasdib.com/blog/a-brief-tour-of-regular-expressions/)