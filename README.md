# decimalsense128
A decimal number format that makes sense

Have you ever looked at the IEEE-754 decimal number standard and wondered what in the world were they thinking?
Decimal numbers do not need be that complex.

This is an attempt at making a decimal number format similar to decimal128, but with _less complications_. It is an extension of the [decimalsense number format](https://github.com/jido/decimalsense).

Numbers
=======

Normal number range, 35 significant digits:

**1.0e-512** to **9.99...99e+511**

(hexadecimal _1ED09 BEAD87C0 378D8E64 00000000_ to _7FF34261 72C74D82 2B878FE7 FFFFFFFF_)

Subnormal number range (non-zero, between 1 and 19 significant digits):

**1.0e-531** to **9.999,999,999,999,999,999e-513**

(hexadecimal _1_ to _8AC72304 89E7FFFF_)

Zero:

> hexadecimal _0_

One:

> hexadecimal _4001ED09 BEAD87C0 378D8E64 00000000_

Ten:

> hexadecimal _4021ED09 BEAD87C0 378D8E64 00000000_

Infinity:

> hexadecimal _7FFF0000 00000000 00000000 00000000_

Negative infinity:

> hexadecimal _FFFF0000 00000000 00000000 00000000_

Decimalsense numbers are _monotonic_, the unsigned part is ordered when cast as integer, and _unique_: 
there is only one way to encode a particular nonzero number.
Not a Number (NaN) can have several representations.

According to IEEE, positive and negative zero are equal which is the only exception to unique number representation.

----

Format
======

~~~
seeeeeee eeemmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm
mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm
~~~

   `s` = sign bit
   
   `e` = 10-bit exponent
   
   `m` = 117-bit mantissa

 * If `e` and the first 53 bits of `m` are all 0 then it is a subnormal number
 * If `e` and the first 5 bits of `m` are all 1 then it is an _Infinity_ or _NaN_

For platforms that don't natively support 128 bit quantities, an alternative format can be used - [see below](#alternative-format).

Normal numbers
--------------

 * Exponent is offset by 512
 * The mantissa is normalised, it goes from _1e34_ to _9.99...99e34_

Take _512_ away from the 10-bit exponent value to read the actual exponent

Subnormal numbers
-----------------

 * Exponent is _-512_
 * The mantissa encodes numbers between 0.0 and 1.0 with a lower precision

Equivalent to 128-bit integers (ignoring the sign bit) from _0_ to _9,999,999,999,999,999,999_;

Multiply `m` by _1e15_ to read the actual mantissa value

----

Alternative Format
==================

This is intended for easy conversion between decimalsense numbers and decimal representation on platforms without 128 bit number support.
This alternative format is _not_ monotonic but the uniqueness of number representation is preserved.

~~~
shhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh
eeeeeeee eellllll llllllll llllllll llllllll llllllll llllllll llllllll
~~~

**Word 1:**

   `s` = sign bit
   
   `h` = 63 bits for the first 19 decimal digits of mantissa

**Word 2:**
   
   `e` = 10-bit exponent

   `l` = 54 bits for the last 16 decimal digits of mantissa

* If the first 15 bits of `h` are all 1 then it is an _Infinity_ or _NaN_
* If `e` is all 0 and the first two bits of `l` are 1 then it is a subnormal number
(**word 2** value is hexadecimal _300000 00000000_ or _310000 00000000_)

Normal numbers
--------------

 * Exponent is offset by _512_
 * The mantissa is normalised, it goes from _1e34_ to _9.99...99e34_

Take _512_ away from the 10-bit exponent value to read the actual exponent;

Multiply `h` by _1e16_ then add _1e34_ to get the first 19 digits of the mantissa;

Add `l` to fill in the last 16 digits of the mantissa

Examples:

> One = hexadecimal
> ~~~
> 00000000 00000000
> 80000000 00000000
> ~~~
>
> Two = hexadecimal
> ~~~
> 0DE0B6B3 A7640000
> 80000000 00000000
> ~~~

Subnormal numbers
-----------------

 * Exponent is _-512_
 * Mantissa goes from _1,000,000,000,000,000_ to _9.999,999,999,999,999,999e33_ (non-zero mantissa)

Copy bit 48 of **word 2** into bit 63 of **word 1** then multiply **word 1** by _1e15_ to read the actual value of the mantissa
