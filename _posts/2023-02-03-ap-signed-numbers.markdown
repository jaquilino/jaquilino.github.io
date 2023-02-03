---
layout: post
title:  "Arbitrary precision -- Signed numbers in C"
date:   2023-02-03 8:00:00 -0600
categories: Tech
---

This is a follow-on discussion of an earlier [post](/tech/2023/01/20/arbitrary-precision.html){:target="_blank"} on
arbitrary precision arithmetic.
In the [post](/tech/2023/01/26/ap-addu.markdown){:target="_blank"},
we looked at C code to add two numbers that were positive,
that is, ranging from 0 up to a large value.
In this post, we examine the use of **signed** integers.
We continue the use of the C programming language.

First, we deal with the question of representing a number
so that it may range in value from a low of the maximum negative value
up to a high of the maximum positive value.
One technique would be to use a seperate variable to indicate
whether a value is signed or not.
This is generally not done.
For every number you want to use in your program,
you now need two variables.
Then, how would you default it?
If the variable was not supplied,
is the program correct in assuming the numbers are NOT signed?
The C language does not rely on a seperate indicator of signed or unsigned.

Another approach is to **reserve** one bit of the numeric value
to **mark** whether a number is negative or positive.
Which bit to use is somewhat arbitrary,
but standard practice has become to leverage the highest bit
as the sign indicator.
Using any one bit means that the range of values of the number
is constrained by one bit.
A 16-bit number may actually only apply 15 of those bits to a number,
with the other representing whether the value is negative or positive.
So a 16-bit C compiler can only represent signed values up to maximums
of 2 to the power of 15 (which is 32,767).
This means 16-bit signed numbers can range from -32767 to 32767.
The CPU for which a C compiler is producing code MAY be engineered
where its numeric registers always make provision for a sign.
A CPU may have 17-bit registers, instead of 16, where one of the bits
is the sign indicator.
But the C language does not require this machine architecture,
and if there were such a machine architecuture,
what would do with the extra bit if the number on which it
intended to operate did not need a sign indicator?
Use all 17 bits to represent a larger positive value,
or just pretend the machine hardware does not exist?
The hardware architect has not added any value,
the problem still exists.

The C language assumes numbers are signed unless the developer indicates otherwise.
If you declare a variable to be of the type ```int```, it will be signed.
The declare that a variable will always be positive (0 or higher),
it must be explicitly declared as ```unsigned```.
(Just declaring it ```unsigned``` with no other type tells the C compiler it is an ```unsigned int```.
It can also be an ```unsigned``` ```char```, ```short```, or ```long```).

The C language does not require which bit of a register
would be the sign indicator,
but the convention is that the highest bit is used.
This is the bit furthest to the left.

A value of negative 1 is represented by turning that high bit ON.
All following bits are also turned ON.
In a 16-bit machine, -1 would have the bit pattern ```FFFF FFFF```.
If you iteratively subtract 1 from a negative value,
after having done so for the maximum range, 
the result is the negative sign indicator followed by all ZERO bits.
This may seem counter-intuitive,
but it ends up delivering a convenient numeric relationship.

Add 1 to -1 and you get zero (0).
Add 16-bit ```0000 0001``` to ```FFFF FFFF```, and you also get 0 ```0000 0000```.
All bits ON is the natural inverse of all bits OFF.

To change a positive value into its negative counterpart (to **negate** it),
you subtract 1 from the positive value and then
perform an operation called a **two's complement***.
Two's complement just turns all bits in a register to their opposite values.
To tell the C compiler to take the two's complement of any number,
you put a tilde in front of the number (or variable).
```
      int
      negate_pos2neg(int x)
      {
          return (~(x - 1));
      }
```
So the algorithm to negate a positive number is subtract 1 from it and then takes its two's complement.
To negate a negative number, reverse the algorithm,
that is take its two's complement and then add 1 to the result.
```
      int
      negate_neg2pos(int x)
      {
          return ((~x) + 1));
      }
```

These mirror algorithms enable one to perform subtraction
by negating the number you are subtracting by,
and adding it to the number you are subtracting from.
To subtract 374 from 482, add -374 to 482.
If you are subtracing one number from another,
and are assuming the result will still be an unsigned value,
the number you are subtracting by MUST be less than or equal to
the number you are subtracting from.
If the result may be signed, you don't care.
Addition is **commutative**, meaning it doesn't matter what
order you apply the numbers in.
Algebraically, this is noted as ```A + B = B + A```.
You can even add a negative number to another negative number.
The result will be an even more negative number.

