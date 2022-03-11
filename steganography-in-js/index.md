---
title: Steganography with Zero Width Characters (Z-Chars)
date: 2022-03-09
---

{{< rawhtml img >}}
<img src="https://user-images.githubusercontent.com/5805251/156551854-29ef4800-e455-4e86-bb98-deef7dd0d6b7.svg" style="width: 100%;height: 320px" />
{{< /rawhtml >}}

__Steganography the practice of hiding a message in something that is not secret. Certain unicode characters are zero-width, meaning that they won't be printed and can't be seen. Using these character we can encode characters and hide messages.__

In this post I'll provide a quick explanation of some of the key aspects of [Z-Chars][Z-Chars Demo]. The post might interest Javascript devs who are curious about unicode and the bidirectional algorithm.

**POLITE NOTICE** There's enough trash on the web without hidden characters gumming up our messages, please don't use this in the wild.

## Hiding characters in text

Wherever text is represented digitally, it's likely to be encoded using Unicode. Some Unicode characters are intentionally invisible, a carriage return for example (`↵`) can't be seen although we can see it's effect. Such characters are known as [control characters][control-characters].

Because not all languages read from left to right, some require special mark-up to ensure they run from right to left. Take the word "Egypt" as an example.

{{< rawhtml half >}}
<figure>
    <div>Egypt</div>
    <code>⟶</code>
    <figcaption>Latin characters read left to right</figcaption>
</figure>
<figure>
    <div>مصر</div>
    <code>⟵</code>
    <figcaption>Arabic characters read right to left</figcaption>
</figure>
{{< /rawhtml >}}

Computers need to know which direction these characters run in and one way to indicate this is with control characters.

There are nine such characters as specified in the [Unicode Bidirectional Algorithm][Unicode Bidirectional Algorithm]. The nine characters are:

```
U+2066   LEFT-TO-RIGHT ISOLATE
U+2067   RIGHT-TO-LEFT ISOLATE
U+2068   FIRST-STRONG ISOLATE
U+202A   LEFT-TO-RIGHT EMBEDDING
U+202B   RIGHT-TO-LEFT EMBEDDING
U+202D   LEFT-TO-RIGHT OVERRIDE
U+202E   RIGHT-TO-LEFT OVERRIDE

U+202C   POP DIRECTIONAL FORMATTING
U+2069   POP DIRECTIONAL ISOLATE
```

I'm not going to be digging much into the details but hopefully the names alone provide some indication of their use. Let's explore how they'll print in the browser. We'll be using the UTF-16 reference listed above. In JS we can access these characters with `'\uCODE'`. The letter "A" for example is `'\u0041'`.

```js
'\u0041'
// "A"

'\u0041\u0042\u0043'
// "ABC"

'\u2066\u2067\u2068\u202A\u202B\u202D\u202E\u202C\u2069'
// "" [Nothing is visible]
```
As you can see, our nine characters print as an empty string "". Now let's try to insert these characters between other letters.

```js
'A' +
'\u2066\u202A\u202D' +
'B' +
'\u2066\u202A\u202D' +
'C'
// "A⁦‪‭B⁦‪‭C"
```

Again the control characters between the letters can't be seen.

Because the assumed use for this is hiding characters in the latin alphabet, we know the text will always run left to right, to guarantee this it's safest to limit ourselves to the three left-to-right control characters:

```
U+2066   LEFT-TO-RIGHT ISOLATE
U+202A   LEFT-TO-RIGHT EMBEDDING
U+202D   LEFT-TO-RIGHT OVERRIDE
```

We now have the means to hide characters, all we need now is a way to encode them to other characters.

## Encoding and Decoding

So we have our three characters with which we can encode our hidden letters and this is great because if you have an understanding of binary, you'll know that we only need two `0` and `1`. Let's see how we can use these two characters to represent the letter "X".

```js
"X".charCodeAt(0)
// 88
"X".charCodeAt(0).toString(2);
// "1011000"
```

`charCodeAt` gives us the unicode character reference for our letter, `toString(2)` then converts this to a binary string or a number with a base of 2. To get back to our "X" we can use `fromCodePoint` to reverse the process.

```js
parseInt("1011000",2)
// 88
String.fromCodePoint(88)
// "X"
String.fromCodePoint(parseInt("1011000",2))
// "X"
```

Because we know we can represent "X" using 0 and 1, it's a short step to swapping these characters out to the control characters that can't be seen.

```
1 => U+2066
0 => U+202A
1 => U+2066
1 => U+2066
0 => U+202A
0 => U+202A
0 => U+202A
```

So an X hidden between A and B might look like this:

```js
'A'+'\u2066\u202A\u2066\u2066\u202A\u202A\u202A'+'B'
// A⁦‪⁦⁦‪⁦‪B
```

Because we have three characters however, it makes sense to use them all. This changes our base (or radix) to three. Encoding and decoding three characters look like this:

```js
// ENCODING

"X".charCodeAt(0)
// 88
"X".charCodeAt(0).toString(3);
// "10021" [Comprises three characters 0,1 and 2]

// DECODING

parseInt("10021",3)
// 88
String.fromCodePoint(88)
// "X"
String.fromCodePoint(parseInt("10021",3))
// "X"
```

Let's take another example by hiding "HIDDEN" within the word "VISIBLE". Because our z-chars fit between our visible letters, the number of characters we can encode is the number of visible letters minus one. So our visible letters demark our hidden ones.

Here's a visualisation of the encoding:

{{< rawhtml >}}
<figure>
    <span>V‭‭⁦⁦I‭‭⁦‪S‭‪‪‭I‭‪‪‭B‭‪‭⁦L‭‭‭⁦E⁩</span>
    <figcaption>Characters represented by zero-width characters</figcaption>
</figure>

<figure>
    <span>V<code>2200</code>I<code>2201</code>S<code>2112</code>I<code>2112</code>B<code>2120</code>L<code>2220</code>E</span>
    <figcaption>Characters represented by 0,1,2</figcaption>
</figure>

<figure>
    <span>V<code>H</code>I<code>I</code>S<code>D</code>I<code>D</code>B<code>E</code>L<code>N</code>E</span>
    <figcaption>Characters decoded</figcaption>
</figure>
{{< /rawhtml >}}

## Summary

Other than a few functions to split strings and interpolate the hidden characters, this all but covers the core concepts behind Z-Chars. Pray you never use them to this effect.

Please take a look through the [source code] if you're curiouser still.

## Links

- [What every JavaScript developer should know about Unicode]
- [An introduction to bidi markup]
- [Unicode Bidirectional Algorithm]
- [Z Cars theme music]

[source code]: https://github.com/robstarbuck/z-chars
[Z-Chars Demo]: https://robstarbuck.github.io/z-chars-demo/
[bidi basics]: https://www.w3.org/International/articles/inline-bidi-markup/uba-basics
[Unicode Bidirectional Algorithm]: http://www.unicode.org/reports/tr9/
[An introduction to bidi markup]: https://www.w3.org/International/articles/inline-bidi-markup/
[control-characters]: https://en.wikipedia.org/wiki/Control_character
[What every JavaScript developer should know about Unicode]: https://dmitripavlutin.com/what-every-javascript-developer-should-know-about-unicode/
[charCodeAt]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt#examples
[Z Cars theme music]: https://www.youtube.com/watch?v=wL1HnDGTAK8