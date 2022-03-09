---
date: 2022-03-01
title: Count to 1023 on Two Hands
---

__If you can accept that ten binary digits `1111111111` represents 1023 in decimal, we can of course use our ten fingers to represent any number counting up to it.__

{{< iframe "https://robstarbuck.github.io/finbin/" "Finger Binary" >}}

This is a brief whistle-stop tour of a JS project that converts a number to it's handy binary representation. In building it I learnt a little about JS [Bitwise Operators][BitwiseOperators] and their application. This post covers the utility of those operators and might interest any JS devs who are curious about their usage.

Let's start with a single closed fist as our 0, you can click the fingers or adjust the input to change the value.

{{< iframe "https://robstarbuck.github.io/finbin/?initialBinaryValue=0&maxBinaryValue=31" "Starting from zero" >}}

I'm sure you're getting the gist. Wherever a finger is extended we count the digit in our calculation. Let's represent that most rock-and-roll of numbers, 18.

{{< iframe "https://robstarbuck.github.io/finbin/?lockValue=true&maxBinaryValue=18" "The number 18" >}}

Translating the decimal value "18" to it's binary value "10010" looks like this:

```
Binary         1     0     0     1     0
Bit Value      16    8     4     2     1
Bit Position   5     4     3     2     1
```

These bit positions of course correspond to fingers. The number 18 has **1s** in bit positions 2 and 5 represented by our pinky and index fingers.

We can prove our binary value equals 18 using `parseInt` and including a radix (or base) of 2 as our second argument. 

```js
parseInt("10010", 2);
// 18
```

The radix tells parseInt we're dealing with a binary number (2 numbers, 1 and 0) and not a decimal (10 numbers, 0-9, parseInt's default value).

> Anyone who ever mistakenly tried `["1","2","3"].map(parseInt)` learnt the hard way about this second argument.

It is of course possible to derive our "18" value from an array of booleans:

```
[true, false, false, true, false] => 10010 => 18
```

But that's well covered ground and not what I'm exploring here. Instead we'll be deriving our boolean values (is a finger extended) from our number.

### Calculating the max value for a given number of fingers

The maximum value that can be represented by 10 fingers is 2 to the power of 10 minus one. Or `(2 ** 10) -1` in JS.

The following chart shows the calculation for 2 to the power n.

```
n        0    1    2    3    4    5    6    7    8    9    10
2 ** n   1    2    4    8    16   32   64   128  256  512  1024
Finger   1    2    3    4    5    6    7    8    9    10   11
```

So our tenth finger will represent the number 512 but this isn't our maximum as we've yet to count all the fingers that came before it and doing so gives us our maximum value 1023.

```js
1 + 2 + 4 + 8 + 16 + 32 + 64 + 128 + 256 + 512
// 1023
```

To avoid totalling these numbers it's easier to take our [11th value 1024](https://robstarbuck.github.io/finbin/?initialBinaryValue=1024&maxBinaryValue=2047) and minus one. 

We can the parity of these values can be proven in JS.

```js
(1023).toString(2);
// "1111111111"
(1023).toString(2).length;
// 10
(2 ** 10) -1 === 1023;
// true
parseInt("1111111111",2) === 1023
// true
```

### Each Fingers Value

Another thing we need to know is the value of each finger from right to left. Just as `2 ** 10` gave us the value of our 11th finger. We can use `2 ** n` to calculate each finger's binary value.

```js
Array(10)
    .fill(null)
    .map((_, i) => 2 ** i);
    // [ 1, 2, 4, 8, 16, 32, 64, 128, 256, 512 ]
```

### Should a finger be extended
As we loop through the value of each finger we can use the **AND** [(&)][BitwiseAnd] bitwise operator to determine if a value is included in our total and therefore the applicable finger extended. This is completely different to the logical AND represented by "&&". We'll explore this problem for our most knarliest of numbers `17`.


{{< iframe "https://robstarbuck.github.io/finbin/?lockValue=true&maxBinaryValue=17" "The Knarly Number" >}}

To get our `Result` in the following examples we use our operator "&" against each finger's corresponding `Value`. In decimal this would look like:

- `1 & 17`
- `2 & 17`
- `4 & 17`
- `8 & 17`
- `16 & 17`

Here are the same calculations visualised in binary.

```
Input                  00001   00010   00100   01000   10000
Value (17)             10001   10001   10001   10001   10001
Input & Value (Result) 00001   00000   00000   00000   10000
Result === Input       TRUE    FALSE   FALSE   FALSE   TRUE
```
I hope you can see what's happening here, wherever a 1 appears in the same bit position for both the `Input` and the `Value` the result `Input & Value` gets a 1 in the same position. If the result matches the input we know that a finger should be extended. Let's run that calculation in JS to prove we're right.

```js
[1,2,4,8,16].map(v => (v & 17) === v)
// [true, false, false, false, true]
```

### Closing and extending fingers

Our last task is closing and extending fingers or taking away and adding values. Let's say that we have a single hand with all of it's fingers extended, giving us the number 31 or `11111` in binary. Let's close that most British of digits, the pinky finger, represented by the number 16 or `10000`.

Our operator in this case will be **XOR** [(^)][Bitwise^], this returns a `1` for every bit position where the values differ.

```
Input (16)            10000
Value (31)            11111
Input ^ Value (15)    01111  
```

Because XOR is only interested in the difference between the two binary values, we can use the same operator to extend our pinky finger, adding it back to the hand.


```
Value (15)            01111
Input (16)            10000
Value XOR Input (31)  11111  
```

## Summary

We've proven that an array of boolean values can equally be represented by a single number. Whilst Bitwise operators provide a powerful toolset for manipulating that number it is not without its limitations. The biggest limitation is the number of elements (or fingers) we can represent. Because of JS's implmentation we are restricted to a maximum safe value of 9007199254740991, retrievable with the MAX_SAFE_INTEGER constant:

```js
Number.MAX_SAFE_INTEGER
// 9007199254740991
// 11111111111111111111111111111111111111111111111111111
```

These 52 `1`s represent 53 elements in an array, meaning we can safely operate on no larger number.

Finally I'd be doubtful that anyone would thank you for the inclusion of bitwise operators in source code, their usage is certainly a little obscure but they've been fun to play around with.

[BitwiseAnd]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_AND
[BitwiseOperators]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators#binary_bitwise_operators
[Bitwise^]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_XOR