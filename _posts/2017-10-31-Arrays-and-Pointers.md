---
layout: post
title: C arrays and pointers
---

There are some pretty widespread misconceptions about arrays and pointers in C.
Arrays and pointers are different but related things. I think the idea’s pretty
easy to get, but you have to approach it the right way.

So, first, go read the post about [Types and Objects](2017-10-31-Types-and-Objects).

Array types in C are complete object types, for the most part like any
other object type.  Recall that a complete object type describes a set of
values and how they are represented in memory.  An array type has an
*element type* ``E`` and a *bound* ``N``.  Its values are sequences of
``N`` values of type ``E``, and it stores them sequentially, one right
after another.

So, for example, if you have a pointer to an array type (written in C’s somewhat
baroque syntax as ``float (*)[10]``), then what you have is a pointer to an
array object consisting of 10 ``float``s laid out sequentially.

The element type of an array can be any complete object type, meaning that
it can itself be an array type.  For example, the type ``int (*)[4][3]``
connotes a pointer to an array of 4 arrays of 3 ``int``s each, which (by a
simple application of that layout rule) has the same underlying layout as
an array of 12 ``int``s.

(Note that this produces a row-major (really an "outer-major") layout for
nested arrays, where a small change in an earlier index causes a large jump
in addresses.  This arises completely unavoidably from the rules of the
language.  However, it's unfortunate for scientific programming because
many algorithms are written assuming the column-major ordering used by
languages such as Fortran which provide native multi-dimensional arrays;
naively porting such algorithms to C can lead to catastrophically bad
performance.)

By implication, it is impossible to directly express a "jagged"
multi-dimensional array, where different elements have different lengths,
because this would require the elements to have different types from each
other.

Anyway, in terms of basic layout, array types are just normal complete
object types.  But there are some special things about arrays:

First, the contexts that implicitly turn l-values into r-values turn array
l-values into pointer r-values by taking the address of the first element.
This is often called "array-to-pointer decay". So an l-value of type
``int[4]`` will turn into an r-value of type ``int*``, and similarly, an
l-value of type ``float[4][10]`` will turn into an r-value of type
``float(*)[10]``.  These contexts include pointer-arithmetic operators:
if you have a declared array like ``short x[8];``, then ``x + 5``
causes the array l-value on the left side to decay to a ``short*``, which
the operator then increases by ``5 * sizeof(short)``.  That’s also how
subscripting works in C, since ``x[5]`` is defined to be the same as
``*(x + 5)``.

Second, since call arguments generally cause l-values to turn into r-values,
and therefore turn arrays into pointers, it makes some sense to apply the
same treatment to parameters. So if you have a function parameter declared
as ``int p[10]``, this is treated exactly the same as ``int *`` in
both the type of the function and the type of expressions that refer to ``p``.

Finally, you don’t always have to give an explicit element count: you can
have an *incomplete array type* like ``int[]``.  The meaning of this depends
on the context.  If a variable is declared with such a type but is given
an explicit initializer, the number of elements will simply be inferred from
the initializer.  If a parameter is declared with such a type, the parameter
type immediately decays to be a pointer like normal.  Otherwise, incomplete
array types are not complete object types and are subject to various
restrictions on their use.
