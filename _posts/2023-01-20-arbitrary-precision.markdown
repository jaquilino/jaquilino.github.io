---
layout: post
title:  "Arbitrary precision arithmetic"
date:   2023-01-20 8:00:00 -0600
categories: Tech
---

Computers have limits on the size of numbers on which they can
perform arithmetic.
All laptop systems made since about the year 2007
use number **registers** that are 64 bits wide in calculations.
This means that any math calculation performed by the device
must be within a range of 18.4 _quintillion_ values.
If the number can be negative&#8212;a _signed_ value&#8212;
it can range from a negative 9.2 quintillion to a positive 9.2 quintillion.
Limit yourself to positive numbers only,
and it can range from 0 up to the full 18.4 quintillion.

If you are using an older technology that "only" has 32-bit wide counters,
the resulting numbers are limited to a range of 4.2 billion.
Even that is a big number, but one that is reasonably over-filled
in the normal course of human affairs.

These limits on math operations imply that computer-based
operations are inherently **modulo** in nature.
If you have a number that is at the maximum end of the range,
and you need to add one to it, what happens to the number
in the computer (or more precisely, in the CPU)?
It rolls over to 0, resetting the range.

This is similar to the index number representing the day of the week.
The index can range in possible values
from 0 (the beginning day of the week, often Sunday)
to 6 (the last day of the week).
If you are already on the last day of the week,
and you need to refer to _tomorrow_, what is the index?
It rolls over to 0, because it cannot be 7.
The index of days of the week is said to be **modulo 7**,
where modulo refers to the range of possible values.
Computer register values work the same way, just with a much larger
value after modulo.

We also use the term **base** to refer to the usable number of digits
in a numeric computation.
We humans are familiar with numbers that are in a base of 10.
But in human terms, a **place** in a number can range from 0 to 9.
When we hit 9, and add one to it, we put a 1 in front of it
(to indicate how many times we've rolled over from 9),
and reset to 0.
Humans use base 10 numbering.
The truly geeky among you reading this article will deftly point
out that everything always uses base 10 numbering,
just that the number of values to which the _right-hand side_ refers
can vary.
Point taken, let's move on.

The question is:
How does one do math operations on numbers using computers
where the numbers exceed the limits of their registers?

In encryption, for example, we will refer to **keys** that are
1024 or 2048 **bits** in size.
This refers to numbers that are made up of 1024 or 2048 bits,
where a bit is binary value, either 0 or 1.
A 2048-bit number has a range of values
that go from 0 to a 2 raised to the power of 2048.
A truly large number,
but one it is not inconceivable for a computer to calculate.
It might take a little while,
that's the point of it using a really big number to do encryption.
If the computer has a limit of 64 bits in how big of a numeric value
it can make a single calculation on,
how does it perform a calculation on a 2048 bit number?

The answer is that the computer will need to break up
the really big number into a list of smaller numbers,
just as we humans do with our pitifully small, base 10 numbering.
We break our numbers into digits by powers of 10,
where each **place** in the resulting number
represents the number of times that specific power of 10 is applied.
We when say and/or write 2048, we are saying
```
1000 has occurred 2 times
100 has occurred 0 times
10 has occurred 4 times
8 has occured once
```
Add 2000, 40 and 8 and you get 2048.

![Number 2048 in base 10](/assets/base-10-2048.png "Number 2048 in base 10")
<div class="center-caption">
<figcaption>The number 2048 in a list of base 10 numbers</figcaption>
</div>

Computers can be made to do the same thing in the same way,
just that instead of limiting themselves to
a range of 0 to 9 (ie, base 10),
they can be made to make use of a base of 2 to the power of the number of bits
in the width of their registers.
As I write this, that is typically 64 bits,
so 2 raised to the power of 64, or 18.4 quintillion.
Calculations on a 2048-bit key will break down
into a list of 2048 divided by 64, or 32 separate numeric values.

Humans can program a computer to operate on a smaller chunk
than 64, but said human is choosing to "leave money on the table"
by not taking advantage of full computational capability
and have the computer work more slowly.
The human just won't be able to have the computer work in chunks
larger than 64, if that is the size of its internal registers with which
it performs calculations.
(By the way, the size of its registers is referred to as the **word size**
of the CPU.)

An exceptional large number is referred to as an
**arbitrary precision** number.
In the computer, the programmer would organize an arbitrary precision
number is a list of values,
what would be referred to as an **array** in most computer languages.
An array has a length.
Each element in the array corresponds to a _place_
in a human-style numbering system,
where each place represents a power of the base number.

So if you're going to represent a really big number
by a list of values, you further have a choice
of sequencing that list such that
the values of the lowest exponents appear first,
or that the values of the highest exponents appear first.
The former ordering is referred to as **little endian**,
because the little end of the numbers appears first.
The latter ordering is referred to as **big endian**.
Little endian ordering can be an intuitive challenge for humans.
With little endian, the number 2048 would be represented
in the computer as 8402.
CPU hardware that derives from chips that originated with the company Intel,
however, organize numbers in a little endian fashion,
so it is a relatively common way of organizing really big numbers.
