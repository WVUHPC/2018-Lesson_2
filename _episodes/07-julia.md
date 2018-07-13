---
title: "Julia: The best of two worlds"
teaching: 45
exercises: 15
questions:
- "What is Julia, and why should I learn another language?"
objectives:
- "Learn the basics of Julia and its use in scientific computing"
keypoints:
- "Julia offers the advantages of interpreted languages like R or Python and
good performance, approaching that of statically-compiled languages like C"
---

The Julia programming language fills the gap between the world of compiled languages like C/C++ and Fortran and interpreted languages like Python and R.

Julia is a flexible dynamic language, appropriate for scientific and numerical computing but with performance comparable to traditional statically-typed languages.

Being a very young language it is not as well known as the other languages that have being around for a long time.

# Introduction to Julia Programming

As we have done for the other languages, we offer here a very brief introduction to the 5 basic elements to know about in every programming languages. Variables, Control Flow, Input/Output, Packages and Debugging/Profiling/Benchmarking.

## Variables

Variables are declared as in most interpreted languages, just use a good name and set the values

~~~
x = 3.14
n = 10
~~~
{: .source}

Julia is case sensitive for variables and accept both variables and strings to be UTF8

~~~
salut = 안녕하세요"
\alpha<tab>=1.0
~~~
{: .source}

Julia offers primitive numeric types for both integers and floating-point numbers (truncated real numbers). The following table describes them

### Integer variables

|Type	|Signed	|Number of bits	|Smallest value	|Largest value|
|:----|:------|:--------------|:--------------|:------------|
|Int8	  | yes |	  8	|-2^7	|2^7 - 1|
|UInt8  | no  |	  8	|0	|2^8 - 1|
|Int16  | yes |	 16	|-2^15	|2^15 - 1|
|UInt16	| no  |  16	|0	|2^16 - 1|
|Int32  | yes |	 32	|-2^31	|2^31 - 1|
|UInt32 | no  |	 32	|0	|2^32 - 1|
|Int64  | yes |	 64	|-2^63	|2^63 - 1|
|UInt64	| no  |  64	|0	|2^64 - 1|
|Int128	| yes |	128	|-2^127	|2^127 - 1|
|UInt128| no  |	128	|0	|2^128 - 1|
|Bool   |     |	  8	| false (0)	| true (1)|

### Floating-point variables

|Type	|Precision	|Number of bits|
|:----|:----------|:-------------|
|Float16	|half	  |16|
|Float32	|single	|32|
|Float64	|double	|64|

Julia offers a middle position with variables, you do not need to be too concern about them but if you offer information about the variables, Julia can take that information and give you performance advantages out of it.

You can get access to the type of a variable at a given time. Contrary to what you can think from the mathematical point of view the integer 1 is not the same as the floating point 1.0. Most machines nowadays are 64 bits but still you can get information about the Word size of the machine where you are running with `Sys.WORD_SIZE`

~~~
julia> a=1
1

julia> typeof(a)
Int64

julia> a=1.0
1.0

julia> typeof(a)
Float64

julia> Sys.WORD_SIZE
64
~~~
{: .source}

One of the big differences between python 2 and 3 is dividing integers. For an expression such as `2/3` python 2 returns 0, python 3 returns `0.6666666666`. Julia works like python 3, promoting both variables into floating point numbers and returning another floating.

For floating point numbers, you can create Float32 or Float64 by adding a `f` or `e` as we can see below:

~~~
julia> a=3.14f0
3.14f0

julia> typeof(a)
Float32

julia> b=3.14e0
3.14

julia> typeof(b)
Float64
~~~
{: .source}

A machine can only represent truncated real numbers. The distance between 1 and the closest number that the machine can represent that is larger than 1 is called the machine epsilon. This example shows how get the epsilon and why knowing about it can be important

~~~
julia> eps()
2.220446049250313e-16

julia> eps()/2
1.1102230246251565e-16

julia> 1+(eps()/2)==1
true
~~~
{: .source}

The truncated representation of real numbers results in gaps from one number to the next real number representable as different, the functions `prevfloat` and `nextfloat` can give you exactly what is the next real number.

~~~
julia> prevfloat(1.0)
0.9999999999999999

julia> nextfloat(1.0)
1.0000000000000002  
~~~
{: .source}

If you are working with numbers bigger than those representable with the primitives of the language, the GNU Multiple Precision Arithmetic Library (GMP) and the GNU MPFR Library were your alternatives to work with them (some applications in Statistical Mechanics rely on those ultra high precision numerics). However, working with GMP and MPFR have been always a challenge as even the simplest operations require many lines of C code. Julia wrap that complexity inside de BigInt and BigFloat datatypes and allow you to go to huge numbers for the price of much lower performance.

~~~
julia> typemax(Int64)
9223372036854775807

julia> typemax(Int64)+1
-9223372036854775808

julia> BigInt(typemax(Int64))+1
9223372036854775808

julia> BigInt(typemax(Int64))+BigInt(typemax(Int128))
170141183460469231740910675752738881534

julia> BigInt(typemax(Int64))*BigInt(typemax(Int128))
1569275433846670190788806172341447372284678185363
~~~
{: .source}

Julia allow to express equations in a way that is very close to the way math is written in paper. Something quite unusual in any other programming language.
See for example the polynomials

~~~
julia> x=2
2

julia> 6x
12

julia> 6x^2
24

julia> (1+6x)^2
169

julia> (6x)^2+2(6x)+1
169

julia> (6x)*(6x+2)+1
169
~~~
{: .source}

Mathematical operations are very similar to other interpreted languages the next block explore a few of them:

~~~
julia> x=2
2

julia> y=3
3

julia> x+y
5

julia> x%y
2

julia> y%2
1

julia> y%x
1

julia> x^y
8

julia> x+=y
5

julia> x
5

julia> y
3
~~~
{: .source}

### Complex and Rational Numbers

Complex numbers are created by adding the suffix im to the imaginary partition

~~~
julia> sqrt(2)
1.4142135623730951

julia> sqrt(2im)
1.0 + 1.0im

julia> sqrt(complex(-1))
0.0 + 1.0im

julia> sqrt(-1)
ERROR: DomainError:
sqrt will only return a complex result if called with a complex argument. Try sqrt(complex(x)).
Stacktrace:
 [1] sqrt(::Int64) at ./math.jl:434
~~~
{: .source}

Rational numbers are created using the `//` operator

~~~
julia> 1//3
1//3

julia> (1//3)^3
1//27

julia> 3(1//3)
1//1

julia> 3(1/3)
1.0

julia> 1/3
0.3333333333333333

julia> 2(1//3)
2//3

julia> 2(1/3)
0.6666666666666666
~~~
{: .source}




{% include links.md %}
