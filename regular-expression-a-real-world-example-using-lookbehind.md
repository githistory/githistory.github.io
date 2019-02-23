## Regular Expression — A Real-World Example of Using Lookbehind

When it comes to regular expressions, it almost always brings more clarity to look at real-world examples than textbooks. And here’s one I’ve encountered.

I was working on a project recently where I needed to grab the title text of a video from page source. By inspecting page source, we can be sure that if you look for

```
"title":"..."
```

we’re only going to find 2 occurrences of such pattern, both of which would serve our purpose, so we’ll just have to find the first occurrence and extract the value part.

You might think, how difficult is that? Well, me too on my first try and here’s what I came up with:

```
/title":"([^"]+)"/
```

The [^"]+ basically finds any number of non-double-quote characters up till the first double quote, which is the closing quote character for the value, and store that in a slot after the full regex match.

You can also do this by the way with a non-greedy match with . operator, such as

```
/title":"(.+?)"/
```

This has the same effect, as it’ll stop and return the match on the first encounter of the quote character.

All good, right? Nope. Turns out a youtube video’s title can contain quotes as well. And in the page source, they’re naturally escaped with a preceding backslash. So a title can look like

```
"title":"A \"Fancy\" Title"
```

This would basically make both of above regex patterns match till the double quote character before the word “fancy”, and effectively miss the rest of the video title.

So in order to fix this, what we wanna do here essentially is to tell the program to match till the first quote that’s not preceded by a backslash. Here’s the solution:

```
/"title":"(.*?)(?<!\\)"/
```

We first do a non-greedy any-character match. The `(?<!\\)"` is where the magic happens. The `(?<!a)b` matches a “b” which is not preceded by “a”. Here we matched a double quote character that’s not preceded by backslash. In regular expression, this is called a “lookbehind”, because once the regex engine reaches the double quote character it’ll look behind to make assertions on the previous character.

As you might have guessed there’s also “lookahead” assertion operation which looks at matches beyond the current standing point. So a(?=b) matches an “a” which is followed by a “b” and a(?!b) matches an “a” that’s not followed by a “b”.

To sum it up

```
// find "b" that's preceded by "a"
(?<=a)b
// find "b" that's not preceded by "a"
(?<!a)b
// find "a" that's followed by "b"
a(?=b)
// find "a" that's not followed by "b"
a(?!b)
```

In regular expression, lookahead and lookbehind operations are collectively called “lookaround”. One gotcha however is that lookbehind is only supported in version 49 and beyond of the V8 JS engine. So make sure you have the right browser / nodejs version before you try it out.
