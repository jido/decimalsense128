# decimalsense128
A decimal number format that makes sense

Have you ever looked at the IEEE-754 decimal number standard and wondered what in the world were they thinking?
Decimal numbers do not need be that complex.

This is an attempt at making a decimal number format similar to decimal128, but with _less complications_. It is an extension of the [decimalsense number format](https://github.com/jido/decimalsense).

Numbers
=======

Normal number range, 34 significant digits:

**1.0e-8191** to **9.99...99e+8191**

(hexadecimal _314D C6448D93 38C15B0A 00000000_ to _7FFDED09 BEAD87C0 378D8E63 FFFFFFFF_)

Subnormal number range (non-zero, variable precision):

**1.0e-8211** to **9.999,999,999,999,999,999e-8192**

(hexadecimal _1_ to _8AC72304 89E7FFFF_)

Zero:

> hexadecimal _0_

One:

> hexadecimal _3FFE314D C6448D93 38C15B0A 00000000_

Ten:

> hexadecimal _4000314D C6448D93 38C15B0A 00000000_

Infinity:

> hexadecimal _7FFF0000 00000000 00000000 00000000_

Negative infinity:

> hexadecimal _FFFF0000 00000000 00000000 00000000_

Decimalsense numbers are _monotonic_, the unsigned part is ordered when cast as integer, and _unique_: 
there is only one way to encode a particular nonzero number.
Not a Number (NaN) can have several representations.

According to IEEE, positive and negative zero are equal which is the only exception to unique number representation.

Format
======

~~~
seeeeeee eeeeeeem mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm
mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm
~~~

   `s` = sign bit
   
   `e` = 14-bit exponent
   
   `m` = 113-bit mantissa

 * If `e` and the first 49 bits of `m` are all 0 then it is a subnormal number
 * If `e` is all 1 and the first bit of `m` is 1 then it is an _Infinity_ or _NaN_

For platforms that don't natively support 128 bit quantities, an alternative format can be used - [see below](#alternative-format).

Normal numbers
--------------

 * Exponent is offset by 8191
 * The mantissa is normalised, it goes from _1e33_ to _9.99...99e33_

Take _8191_ away from the 14-bit exponent value to read the actual exponent

Subnormal numbers
-----------------

Equivalent to 128-bit integers (ignoring the sign bit) from _0_ to _9,999,999,999,999,999_;

Their exponent is _-8177_; the mantissa encodes numbers between 0.0 and 1.0 with a lower precision (between 1 and 19 significant digits.)

Alternative Format
==================

This is intended for easy conversion between decimalsense numbers and decimal representation on platforms without 128 bit number support.
This alternative format is _not_ monotonic but the uniqueness of number representation is preserved.

~~~
shhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh hhhhhhhh
llllllll llllllll llllllll llllllll llllllll llllllll lleeeeee eeeeeeee
~~~

**Word 1:**

   `s` = sign bit
   
   `h` = 63 bits for the first 19 decimal digits of mantissa

**Word 2:**

   `l` = 50 bits for the last 15 decimal digits of mantissa
   
   `e` = 14-bit exponent

* If the first 15 bits of `h` are all 1 then it is an _Infinity_ or _NaN_
* Otherwise, if **word 2** value is 0 or hexadecimal _80000000 00000000_ then it is a subnormal number

Normal numbers
--------------

 * Exponent is offset by _8192_
 * The mantissa is normalised, it goes from _1e33_ to _9.99...99e33_

Take _8192_ away from the 14-bit exponent value to read the actual exponent;

Multiply `h` by _1e15_ then add _1e33_ to get the first 19 digits of the mantissa;

Add `l` to fill in the last 15 digits of the mantissa

Subnormal numbers
-----------------

If the sign bit of **word 2** is set, then the format is equivalent to normal numbers but with reduced precision (value of `l` is 0)

If the sign bit of **word 2** is not set:

 * Exponent becomes _-8192_
 * Mantissa goes from _1,000,000,000,000,000_ to _9.999,999,999,999,999,99e32_ (non-zero mantissa)
 
Multiply `h` by _1e15_ to read the actual value of the mantissa
