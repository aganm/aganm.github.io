---
layout: post
title: Door Oriented Programming
subtitle: "or: What Can Doors Teach Us About Software"
gh-repo: aganm/aganm.github.io
gh-badge: [star, fork, follow]
tags: [test]
comments: true
author: Michael Aganier
---

https://aganm.github.io/2020-02-28-sample-markdown/Let me ask you this: is there such a thing as objectively right or wrong design?
I remember watching a video about the design of doors. I love doors. The video
was about different kinds of designs of doors and how they affect people's
expectations. When you see a handle, you expect that you're expected to pull
on that handle.  When you see no handle, you expect that you're expected to
push on it. And the world keeps turning. But not every door is designed like
that.  What happens when it's the opposite? Because it does happen in the real
world.  Sometimes they just wanted to try something new and exiting, other
times it was an installation mistake. But what happens when you design a door
with a handle that is meant to be pushed? Whether it's volountary or not, you
always get the same thing: you get crap. People will try to use the door wrong
all the time. People see a handle, they expect it's there to be pulled, then
they pull on it, realise they used it wrong, and push.  And therefore to the
question I asked, yes, there is such a thing as objectively right and wrong
design. When a design encourages people using a thing wrong, the design, is
wrong.

[![It's not you. Bad doors are everywhere.](https://99percentinvisible.org/app/uploads/2016/02/pulldoors.jpg)](https://youtu.be/yY96hTb8WgI "It's not you. Bad doors are everywhere.")

# Type Names

When a door is designed right, you don't even think about it, you just use it
because the design was intuitive and it didn't need you to stop to think twice
over opening this door. Now when it comes to type names in programming, I see
this a lot: one type is named in a certain way, and then another type in a
different kind of way. And when you use the first type, you expect something
from the future types, but then you get surprised that it is not the case.
And you stop to think twice over it.

# The Problem

What I propose is a design for basic type names that is intuitive and doesn't
make you stop twice or even thrice about it. The usual mathy type names might
look something along the lines of:

~~~
float       // A float
Vector3     // A vector of 3 floats
Rect4       // A rectangle of 4 floats
~~~

For commonly used types, it may seem not so bad, but it gets worse as we introduce
other types. For doubles, what do we do? Maybe add a d after the types:

~~~
double
Vector3d
Rect4d
~~~

What if I want integers, but 64-bits long? Maybe I'll use an explicit size primitive,
and that will reflect on other names as well:

~~~
i64
Vector3i64
Rect4i64
~~~

There is no way to intuitively reason about how these names should be combined
from the top of your head because they are arbitrarily named. Some are of
implicit size that we assume (`float`), some are explicit size (`i64`), some
omit the primitive type  (`Vector3`), some specify the primitive type and the
size (`Vector3i64`).

Such a naming style is all over the place and impossible to think intuitively
about. The following solution will fix these incongruities once and for all.

# The Solution

Here is the design I use:

First step, do I want a floating point number, a signed integer, or an unsigned integer?

~~~
f     // Floating point
i     // Signed integer
u     // Unsigned integer
~~~

One might like `s` for signed, personally I like `i` for integer. This
is preferencial, you can choose whichever letters you like, as long as
the overall design is consistent within itself.

Second step, what size do I want this primitive to be? Depending if the language
supports it, this value could be almost anything starting from 8 in powers of 2.

~~~
f32   // 32-bit floating point
i64   // signed 64-bit integer
u128  // unsigned 128-bit integer
~~~

Fun fact: Zig will let you choose specific bit sizes for your types, like `u29`.

From here, you can choose to end it, and you have a primitive.
Or, if you continue, you can make complex math types.

Third step: do I want a vector type, a rect type, a matrix type?
And if I want a complex type, what dimension is this complex type of?

~~~
i64rect    // rect for rect, a rect (4 members, x,y,w,h) of 64-bit signed integer
f32v3      // v for vector, so vector3 (3 members, x,y,z) of 32-bit floating point
f64m4x4    // m for matrix, so a matrix of dimension 4 by 3 of 64-bit floating point
~~~

Originally, I made this type design for C, so because it didn't have tuples, I also
added handy common math types that are handy to have, for example:

~~~
f32sincos    // the result of a sincos function in 32-bit floating point (a struct of two f32, sin, and cos)
~~~

Here is what a math type catalogue could look like in my design, and is in fact
what I use:

~~~
f32                               // single value
f32sincos                         // result of a sincos
f32v2, f32v3, f32v4               // vectors
f32m2x2, f32m2x3, f32m2x4         // matrixes that start with a dimension of 2
f32m3x2, f32m3x3, f32m3x4         // matrixes that start with a dimension of 3
f32m4x2, f32m4x3, f32m4x4         // matrixes that start with a dimension of 4
f32rect                           // rect
f32quat                           // quaternion
f32aabb2, f32aabb3, f32aabb4      // aabb bounding boxes
f32ray2, f32ray3, f32ray4         // ray types (origin vector + direction vector)
~~~

Now combine the types above with all of these possible primitives:

~~~
f32     // 32-bit floating point
f64     // 64-bit floating point
f128    // 128-bit floating point
i8      // signed 8-bit integer
i16     // signed 16-bit integer
i32     // signed 32-bit integer
i64     // signed 64-bit integer
i128    // signed 128-bit integer
u8      // unsigned 8-bit integer
u16     // unsigned 16-bit integer
u32     // unsigned 32-bit integer
u64     // unsigned 64-bit integer
u128    // unsigned 128-bit integer
~~~

Maybe I need some ultra high precision 4x3 matrix math for whatever reason: `f128m4x3`.
A 4 element unsigned 8-bit per channel RGBA value: `u8v4` (swizzled with rgba members).
Maybe I want a 2d position based on integers because it's a tilemap: `i32v2`.

For **22** different kinds of types, and for **13** different kinds of primitives, that's **286** possible combinations.
286 and I never have to stop thinking what I'm gonna type next for any type I
want. I never have to stop to think when opening that door.

I also like to have more types for common units of measurement. These are
simple structs that store a single primitive. Having these in type checked structs go
a long way stopping simple mistakes such as passing the wrong type of unit in the
wrong place by accident:

~~~
f32seconds
f32meters
f32kilograms
~~~

With this design, one can come up with any math type right off the top of their
head. All you have to do is follow this consistent rule:

    < primitive >< size >[< complex type >[< dimension of complex type >]]
 
Which, in my design, translates to:

1. Choose a primitive letter: `f`, `u`, `i`
2. Choose a size in bits: `8`, `16`, `32`, `64`, `128`
3. Optional: if you want a complex type, choose a complex type: `v` for vector, `m` for matrix, `sincos`, etc.
4. Optional: if the complex type requires a dimension, then choose the dimension of the type.
             a `v` for vector requires a dimension, a `m` for matrix requires a dimension,
             but not a `sincos`, the dimension of the `sincos` is embedded into it, it's always 2 values.

This design is: consistent, predictable, intuitive, easy to use. If this was a
door, it is the kind of door design that I can use right without having to stop
to think about it.

# SIMD

SIMD can be complex, and there's more than one way to go about it.
The way you go about it will change what makes sense as far as naming goes,
but the point is SIMD type names can be just as intuitive with this design.
The way I went about my SIMD type names is, I reused the exact same rules
explained above, with one extra rule: for a SIMD type, put a 'm' in front of
it, 'm' as in Many or Multiple (Single Instruction Multiple Data).

    [< m for SIMD >]< primitive >< size >[< complex type >[< dimension of complex type >]]

If I want multiple f32s, `mf32`.
Multiple 64-bit signed integers? `mi64`.
Multiple sincos values in double precision? `mf64sincos`.
Multiple vector 4s of f32s? `mf32v4`.
Multiple 30x39 matrixes of 96-bit floats? `mf96m30x39`.

Okay, I don't have a `mf96m30x39` type, but you get the point.
I don't ever have to think twice how a type is named, it's always the same simple
rule. The door always pulls when there's a handle, and it always pushes when
there's no handle.


---

>    
> 
> Rennnnnt!
> 
> 
> ..
> 
> 
> It's time to pay rent.
> 
> 
> ..
> 
> 
> You'll get your rent when you fix this damn door!
> 
>
