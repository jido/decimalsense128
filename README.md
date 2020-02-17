# decimalsense128
A decimal number format that makes sense

Have you ever looked at the IEEE-754 decimal number standard and wondered what in the world were they thinking?
Decimal numbers do not need be that complex.

This is an attempt at making a decimal number format similar to decimal128, but with _less complications_. It is an extension of the [decimalsense number format](https://github.com/jido/decimalsense).

Numbers
=======

Normal number range, 35 significant digits:

**1.0e-511** to **9.99...99e+510**

(hexadecimal _11ED09 BEAD87C0 378D8E64 00000000_ to _7FD34261 72C74D82 2B878FE7 FFFFFFFF_)

Subnormal number range (non-zero, between 1 and 19 significant digits):

**1.0e-530** to **9.999,999,999,999,999,999e-512**

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

 * If `e` is all 0 then it is a subnormal number (the first 53 bits of `m` are also 0)
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

Multiply `m` by _1e16_ to read the actual mantissa value

----

Alternative Format
==================

This format provides more precision than regular decimalsense numbers on platforms without 128 bit number support.
It is essentially identical to Decimalsense with 64 bit worth of extra precision (except subnormal numbers).

This alternative format is monotonic and the uniqueness of number representation is preserved.

~~~
seeeeeee eeehhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh
llllllll llllllll llllllll llllllll llllllll llllllll llllllll llllllll
~~~

**Word 1:**

   `s` = sign bit
   
   `e` = 10-bit exponent

   `h` = 53 bits for the first 16 decimal digits of mantissa

**Word 2:**
   
   `l` = 64 bits for the last 19 decimal digits of mantissa

* If `e` is all 0 then it is a subnormal number (**word 1** value ignoring the sign bit is _0_)
* If `e` is all 1 and the first bit of `h` is 1 then it is an _Infinity_ or _NaN_

Normal numbers
--------------

 * Exponent is offset by _512_
 * The mantissa is normalised, it goes from _1e34_ to _9.99...99e34_

Take _512_ away from the 10-bit exponent value to read the actual exponent;

Multiply `h` by _1e19_ then add _1e34_ to get the first 16 digits of the mantissa;

Add `l` to fill in the last 19 digits of the mantissa

Examples:

> One = hexadecimal
> ~~~
> 40000000 00000000
> 00000000 00000000
> ~~~
>
> Two = hexadecimal
> ~~~
> 402386F2 6FC10000
> 00000000 00000000
> ~~~

Subnormal numbers
-----------------

 * Exponent is _-512_
 * Mantissa goes from _10,000,000,000,000,000_ to _9.999,999,999,999,999,999e34_ (non-zero mantissa)

 Multiply `l` by _1e16_ to read the actual value of the mantissa
