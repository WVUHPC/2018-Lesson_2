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

Being a very young language it is not as well known as the other languages that have being around. The next table shows the age of several languages used in  Scientific Computing.

> ## The Age of Programming Languages (by 2018)
>
> | Language | First appeared | Stable Release |
> |:---------|:---------------|:--------------|
> | Fortran | 1957; 61 years ago | Fortran 2008 (ISO/IEC 1539-1:2010) / 2010; 8 years ago |
> | C | 1972; 46 years ago | C11 / December 2011; 6 years ago |
> | C++ | 1985; 33 years ago | ISO/IEC 14882:2017 / 1 December 2017; 7 months ago |
> | Python | 1990; 27 years ago | 3.7.0 / 27 June 2018; 2.7.15 / 1 May 2018 |
> | R | 1993; 24 years ago | 3.5.1 ("Feather Spray")[3] / July 2, 2018 |
> | Julia | 2012; 6 years ago | 0.6.4 / 9 July 2018 |
>
>{: .source}
{: .callout}

> ## More about programming languages
>
> <img src="http://blog.daveastels.com.s3-website-us-west-2.amazonaws.com/images/languages/PLchart.png" title="Language Chart" alt="Influences of Programming Languages" style="display: block; margin: auto;" />
>
>
> The figure above was found on this Blog:  [Languages you should know](http://blog.daveastels.com.s3-website-us-west-2.amazonaws.com/languages.html)
>
>{: .source}
{: .callout}

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




{% include links.md %}
