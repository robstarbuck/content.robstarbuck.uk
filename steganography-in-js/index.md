---
title: Steganography in JS
date: 2022-03-01
---

# Steganography with Zero Width Characters (Z-Chars)

__Steganography the practice of hiding a message in something that is not secret. Certain unicode characters are zero-width, meaning that they won't be printed and can't be seen. Using these character we can encode others to hide messages.__

In this post I'll provide a quick explanation of some of the key source code for [Z-Chars](https://robstarbuck.github.io/z-chars-demo/). The post might interest JS devs who are curious about unicode and the bidirectional algorhtym.

## Hiding text in text

Wherever text is represented digitally, it's likely to be encoded using Unicode. Some Unicode characters are intentionally invisible, a carriage return for example (`↵`) can't be seen although we can see it's effect. These are known as [control characters][control-characters].

The same is true for other characters. Not all languages read from left to right and some languages require special mark-up to ensure they run from right to left. Take the word "Egypt" as an example.

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

Because computers need to know which direction these characters run in and one way to indicate this is with control characters.

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

I'm not going to be digging much into the details but hopefully the names alone provide some indication of their use. Let's use these code references to explore how these will print in the browser:

```js
'\u0041\u0042\u0043'
// "ABC"

'\u2066\u2067\u2068\u202A\u202B\u202D\u202E\u202C\u2069'
// ""

'\u2066\u2067\u2068\u202A\u202B\u202D\u202E\u202C\u2069'.length
// 9
```
As you can see, our nine characters print as an empty string "". Now let's try to these characters between other letters.

```js
'A' +
'\u2066\u202A\u202D' +
'B' +
'\u2066\u202A\u202D' +
'C'
// "A⁦‪‭B⁦‪‭C"
```

As you can see the control characters between the letters don't print and can't be seen.

Because the assumed use for this is hiding characters in the latin alphabet we know the text will always run left to right, to guarantee this it's safest limiting ourselves to the three left-to-right control characters:

```
U+2066   LEFT-TO-RIGHT ISOLATE
U+202A   LEFT-TO-RIGHT EMBEDDING
U+202D   LEFT-TO-RIGHT OVERRIDE
```

We now have the means to hide characters, all we need now is a way to encode them to other characters.

## Encoding

So we have our three characters with which we can encode our hidden letters and this is great because if you have an understanding of binary, you'll know that we only need two `0` and `1`. Let's how we can use these two characters to represent the letter "Z".

```js
"Z".charCodeAt(0)
// 90

"Z".charCodeAt(0).toString(2);
// 1011010
```

`charCodeAt` gives us the unicode character reference for our letter, `toString(2)` then converts this to binary or a number with a base of 2. To get back to our "Z" we can use `fromCodePoint` to reverse the process.

```js
parseInt("1011010",2)
// 90
String.fromCodePoint(90)
// "Z"
String.fromCodePoint(parseInt("1011010",2))
// "Z"
```

Because we know we can represent "Z" using 0 and 1, it's a short step to swapping these characters out to ones that can't be seen.

```
1 => \u2066
0 => \u202A
1 => \u2066
1 => \u2066
0 => \u202A
1 => \u2066
0 => \u202A
```

So a Z hidden between A and B might look like this:

```js
'A\u2066\u202A\u2066\u2066\u202A\u2066\u202AB'
// A⁦‪⁦⁦‪⁦‪B
```

These are the basics but there are a couple of details to mention. 





{{< rawhtml >}}
<figure>
    <span>V<code>    </code>I<code>    </code>S<code>    </code>I<code>    </code>B<code>    </code>L<code>    </code>E</span>
    <figcaption>Characters are invisible</figcaption>
</figure>
<figure>
    <span>V<code>◒◒◓◓</code>I<code>◒◒◓◑</code>S<code>◒◑◑◒</code>I<code>◒◑◑◒</code>B<code>◒◑◒◓</code>L<code>◒◒◒◓</code>E</span>
    <figcaption>Characters represented by ◓◑◒</figcaption>
</figure>
<!--
<figure>
    <span>V<code>2200</code>I<code>2201</code>S<code>2112</code>I<code>2112</code>B<code>2120</code>L<code>2220</code>E</span>
    <figcaption>Characters represented by 0,1,2</figcaption>
</figure>
-->
<figure>
    <span>V<code> H  </code>I<code> I  </code>S<code> D  </code>I<code>  D </code>B<code>  E </code>L<code>  N </code>E</span>
    <figcaption>Characters decoded</figcaption>
</figure>
{{< /rawhtml >}}


# Links

[An introduction to bidi markup]
[What every JavaScript developer should know about Unicode]

[bidi basics]: https://www.w3.org/International/articles/inline-bidi-markup/uba-basics
[Unicode Bidirectional Algorithm]: http://www.unicode.org/reports/tr9/
[An introduction to bidi markup]: https://www.w3.org/International/articles/inline-bidi-markup/
[control-characters]: https://en.wikipedia.org/wiki/Control_character
[What every JavaScript developer should know about Unicode]: https://dmitripavlutin.com/what-every-javascript-developer-should-know-about-unicode/
[charCodeAt]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt#examples