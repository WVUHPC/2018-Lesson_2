---
title: "Interpreted Languages (Python and R): A comparative review"
teaching: 45
exercises: 15
questions:
- "If I know one language, how I can do a similar thing on the other?"
objectives:
- "Expose a comparison between both languages in terms of syntax and and expression"
keypoints:
- "Interpreted languages give you the flexibility to for scientific exploration"
- "R is the standard for statistical analysis"
- "Python is a general purpose language which is fast to code and easy to use"
- "Both languages have a rich ecosystem of external libraries and packages."
---

Python and R are the two most common interpreted languages widely used in scientific computing. The purpose of this episode is reviewing the basic elements of both languages. If you know one of them this double exposure will help you to learn the other language by doing parallels with the known one.

For this exploration we will be using the text based interface for R and IPython for python. That levels up the features for entering commands.

Lets start the journey from the most basic elements, but first lets balance both environments to have an equivalent set of toolboxes. In python we will use a few extra modules. The modules `os` and `math` are part of the Standard Library but we need to import them explicitly. `numpy` and `pandas` are not part of the Standard Library but they are two prominent libraries for numerical computing and data analysis, importing them we have a more fair comparison in terms of capabilities. We will use Python 3.6.0 and R 3.4.1 the two latest versions available on Spruce by the time this lesson was created.

~~~
$ ipython
Python 3.6.6 (default, Jun 28 2018, 05:43:53)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.4.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import os

In [2]: import math

In [3]: import numpy as np

In [4]: import pandas as pd
~~~
{: .source}


### Creating Variables

On an interpreted language, you do not need to declare a variable in advance as it is usually the case with compiled languages. The first time you want to use a variable you declare it and enter a value for it.

#### In R

~~~
> pi
[1] 3.141593
> sqrt(2)
[1] 1.414214
>
> x<-3
> y<-4
> z<-sqrt(x^2 + y^2)
> print(z)
[1] 5
~~~
{: .source}

### Python

~~~
In [5]: math.pi
Out[5]: 3.141592653589793

In [6]: math.sqrt(2)
Out[6]: 1.4142135623730951

In [7]: x=3

In [8]: y=4

In [9]: z=math.sqrt(x**2 + y**2)

In [10]: print(z)
5.0
~~~
{: .source}

In R you can do a couple of things that are very unusual in other programming languages, assigning variables to the right and declaring global variables, also many programming languages keep a special meaning for `.` in particular in object oriented languages. That is not the case for R. See below some examples:

~~~
> 5 -> foo
> foo
[1] 5
> x.glob <<- 1.2345
> x.glob
[1] 1.2345
~~~
{: .source}

### Listing Variables

In R you can use the `ls` function to see the name variables currently defined and `ls.srt` function to see the name, the type and contents of the variables. Python works differently there you have `dir`, `locals` and `globals` and if you are using IPython you will get some extra variables create by the interactive environment that IPython adds on top of the traditional interpreter.

#### R

~~~
> ls()
[1] "foo"    "x"      "x.glob" "y"      "z"     
> ls.str()
foo :  num 5
x :  num 3
x.glob :  num 1.23
y :  num 4
z :  num 5
~~~
{: .source}

#### Python

| Function | Description |
|:---------|:------------|
|dir() | list of in scope variables |
|globals() | dictionary of global variables |
|locals() | dictionary of local variables |

In addition to those above, if you are using IPython you can also get some magics

~~~
In [21]: %who
math	 np	 os	 pd	 x	 y	 z	 

In [22]: %whos
Variable   Type      Data/Info
------------------------------
math       module    <module 'math' from '/opt<...>h.cpython-36m-darwin.so'>
np         module    <module 'numpy' from '/op<...>kages/numpy/__init__.py'>
os         module    <module 'os' from '/opt/l<...>3.6/lib/python3.6/os.py'>
pd         module    <module 'pandas' from '/o<...>ages/pandas/__init__.py'>
x          int       3
y          int       4
z          float     5.0
~~~
{: .source}

### Deleting Variables

In R you use the function `rm` in Python the equivalent is `del`

#### R

~~~
> x <- 3
> x
[1] 3
> rm(x)
> x
Error: object 'x' not found
~~~
{: .source}

#### Python

~~~
In [1]: a=1

In [2]: a
Out[2]: 1

In [3]: del a

In [4]: a
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-4-3f786850e387> in <module>()
----> 1 a

NameError: name 'a' is not defined
~~~
{: .source}

### Creating a Vector

The idea of a variable containing a set of values is implemented very differently on both languages. R uses the a vector that can contain either numbers, strings or logical values but not a mixture of them. Python, on the other side has a variety of options, like sets, tuples and lists depending if you want the object to be mutable or not, if the order matters and if you can repeat values. In the case of a list, you can actually mix types. In addition to those, numpy offers the numpy array that allow you to store a set of numbers all of them of the same kind. Lets see all those options:

#### R

~~~
> a<-c(1,2,3,4,5,3,2,1)
> a
[1] 1 2 3 4 5 3 2 1
> a<-c(1,2,3,4,pi,3,2,1)
> a
[1] 1.000000 2.000000 3.000000 4.000000 3.141593 3.000000 2.000000 1.000000
> a<-c(1,2,3,4,'foo',3,2,1)
> a
[1] "1"   "2"   "3"   "4"   "foo" "3"   "2"   "1"  
> a<-c(1,2,3,4,TRUE,3,2,1)
> a
[1] 1 2 3 4 1 3 2 1
> mode(TRUE)
[1] "logical"
> mode(pi)
[1] "numeric"
> mode(1)
[1] "numeric"
> mode("foo")
[1] "character"
> v1<-c(1,2,3)
> v2<-c(4,5,6)
> c(v1,v2)
[1] 1 2 3 4 5 6
~~~
{: .source}

#### Python

~~~
In [12]: a=(1,2,3,4,5,3,2,1)

In [13]: a
Out[13]: (1, 2, 3, 4, 5, 3, 2, 1)

In [14]: a=[1,2,3,4,5]

In [15]: a.append(3)

In [16]: a
Out[16]: [1, 2, 3, 4, 5, 3]

In [17]: a+[2,1]
Out[17]: [1, 2, 3, 4, 5, 3, 2, 1]

In [18]: np.array(a)
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-18-103c848ebb38> in <module>()
----> 1 np.array(a)

NameError: name 'np' is not defined

In [19]: import numpy as np

In [20]: np.array(a)
Out[20]: array([1, 2, 3, 4, 5, 3])

In [21]: b=np.array(a)

In [22]: c=np.concatenate((a,a))

In [23]: c
Out[23]: array([1, 2, 3, 4, 5, 3, 1, 2, 3, 4, 5, 3])

In [24]: type({1,2,3})
Out[24]: set

In [25]: type((1,2,3))
Out[25]: tuple

In [26]: type([1,2,3])
Out[26]: list

In [27]: type(np.array([1,2,3]))
Out[27]: numpy.ndarray
~~~
{: .source}


### Basic Statistics

Statistics is the stronghold of R, so it is natural to have the most basic operations available right aways without having to load any extra package. Python, from the other side is a general purpose language and relies on numpy for such operations.

#### R

~~~
> x<-c(2,3,5,7,9,11,13,17,19)
> x
[1]  2  3  5  7  9 11 13 17 19
> mean(x)
[1] 9.555556
> median(x)
[1] 9
> sd(x)
[1] 5.981453
> var(x)
[1] 35.77778
> y<- log(x)
> cor(x,y)
[1] 0.9536303
> cov(x,y)
[1] 4.404271
~~~
{: .source}

#### Python

~~~
In [28]: x=np.array((2,3,5,7,9,11,13,17,19))

In [29]: x
Out[29]: array([ 2,  3,  5,  7,  9, 11, 13, 17, 19])

In [30]: np.mean(x)
Out[30]: 9.555555555555555

In [31]: np.median(x)
Out[31]: 9.0

In [32]: np.std(x)
Out[32]: 5.6393677957553425

In [33]: np.var(x)
Out[33]: 31.80246913580247

In [35]: np.var(x, ddof=1)
Out[35]: 35.77777777777778

In [36]: y=np.log(x)

In [37]: y
Out[37]:
array([0.69314718, 1.09861229, 1.60943791, 1.94591015, 2.19722458,
       2.39789527, 2.56494936, 2.83321334, 2.94443898])

In [38]: np.corrcoef(x,y)
Out[38]:
array([[1.        , 0.95363033],
       [0.95363033, 1.        ]])

In [39]: np.cov(x,y)
Out[39]:
array([[35.77777778,  4.40427128],
       [ 4.40427128,  0.59617622]])
~~~
{: .source}

Notice that `np.var()` gives a different value from the R version. To get the same behavior you have to declare ddof=1. The argument ddof stands for
“Delta Degrees of Freedom” the divisor used in the calculation is N - ddof, where N represents the number of elements. By default ddof is zero.

The mean is normally calculated as `x.sum() / N`, where `N = len(x)`. If, however, `ddof` is specified, the divisor `N - ddof` is used instead.

In standard statistical practice, `ddof=1` provides an unbiased estimator of the variance of a hypothetical infinite population. `ddof=0` provides a maximum likelihood estimate of the variance for normally distributed variables.

### Sequences of Numbers

Both R and Python offers ready-to-use functions for creating numerical sequences.
In R they will produce vectors but in Python you can decide if creating the sequence. Also in python terminology sequence is the word used to describe the various ways of storing sets of variables. There are six sequence types in Python: strings, Unicode strings, lists, tuples, buffers, and xrange objects.
In python 2 the function `range()` used to create lists, python 3 creates iterators that work differently in their internals, you need to convert them into a list if you want to compute all the values explicitly.

#### R

~~~
> a<-1:5
> a
[1] 1 2 3 4 5
> b < seq(from=1, to=5, by=2)
Error: object 'b' not found
> b <- seq(from=1, to=5, by=2)
> b
[1] 1 3 5
> b <- seq(from=1, to=15, by=2)
> b
[1]  1  3  5  7  9 11 13 15
> rep(1, times=10)
 [1] 1 1 1 1 1 1 1 1 1 1
> c<-9:1
> c
[1] 9 8 7 6 5 4 3 2 1
> d<-seq(from=1,to=3,length.out=20)
> d
 [1] 1.000000 1.105263 1.210526 1.315789 1.421053 1.526316 1.631579 1.736842
 [9] 1.842105 1.947368 2.052632 2.157895 2.263158 2.368421 2.473684 2.578947
[17] 2.684211 2.789474 2.894737 3.000000
~~~
{: .source}

#### Python

~~~
In [40]: a=range(1,6)

In [41]: a
Out[41]: range(1, 6)

In [42]: a=list(range(1,6))

In [43]: a
Out[43]: [1, 2, 3, 4, 5]

In [44]: list(range(1,6,2))
Out[44]: [1, 3, 5]

In [45]: list(range(1,16,2))
Out[45]: [1, 3, 5, 7, 9, 11, 13, 15]

In [46]: 10*[1]
Out[46]: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]

In [47]: list(range(9,0,-1))
Out[47]: [9, 8, 7, 6, 5, 4, 3, 2, 1]

In [50]: np.linspace(1,3,20)
Out[50]:
array([1.        , 1.10526316, 1.21052632, 1.31578947, 1.42105263,
       1.52631579, 1.63157895, 1.73684211, 1.84210526, 1.94736842,
       2.05263158, 2.15789474, 2.26315789, 2.36842105, 2.47368421,
       2.57894737, 2.68421053, 2.78947368, 2.89473684, 3.        ])
~~~
{: .source}

Remember that lists in python are very flexible but also very ineficient for numerical calculations, use as much as possible `numpy`. By using numpy you replace `range()` that in python 3 returns an iterator, by `np.arange()` that returns a numpy array. Also you replace the `10*[1]` in numpy with `np.ones()`

~~~
In [55]: np.arange(1,6)
Out[55]: array([1, 2, 3, 4, 5])

In [56]: np.ones(10)
Out[56]: array([1., 1., 1., 1., 1., 1., 1., 1., 1., 1.])

In [57]: np.arange(1,6)
Out[57]: array([1, 2, 3, 4, 5])

In [58]: np.arange(1,16,2)
Out[58]: array([ 1,  3,  5,  7,  9, 11, 13, 15])

In [59]: np.arange(9,0,-1)
Out[59]: array([9, 8, 7, 6, 5, 4, 3, 2, 1])
~~~
{: .source}

### Selecting vectors elements

In this particular item R and Python (using numpy) works very differently. In R indexes start in 1 but Python follows the C convention of start indexing by 0

#### R

~~~
> fib=c(0,1,1,2,3,5,8,13,21,34)
> fib
 [1]  0  1  1  2  3  5  8 13 21 34
> fib[1]
[1] 0
> fib[-3]
[1]  0  1  1  3  5  8 13 21 34
> fib[1:3]
[1] 0 1 1
> fib > 10
 [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE
~~~
{: .source}

#### Python

~~~
In [60]: fib=np.array((0,1,1,2,3,5,8,13,21,34))

In [61]: fib
Out[61]: array([ 0,  1,  1,  2,  3,  5,  8, 13, 21, 34])

In [62]: fib[0]
Out[62]: 0

In [63]: fib[-1]
Out[63]: 34

In [64]: fib[0:3]
Out[64]: array([0, 1, 1])

In [65]: fib>10
Out[65]:
array([False, False, False, False, False, False, False,  True,  True,
        True])
~~~
{: .source}

As you can see negative numbers for indices have different meanings with R vectors and numpy arrays

~~~
In [71]: fib[np.arange(len(fib))!=3]
Out[71]: array([ 0,  1,  1,  3,  5,  8, 13, 21, 34])
~~~
{: .source}

### Matrices

In R a matrix is a different object than a vector, but can be create from it. In Python using numpy there is no fundamental difference, numpy arrays can be created with several dimensions.

#### R

~~~
> fib
 [1]  0  1  1  2  3  5  8 13 21 34
> a<- matrix(fib,2,5)
> a
     [,1] [,2] [,3] [,4] [,5]
[1,]    0    1    3    8   21
[2,]    1    2    5   13   34
> a[1,1]
[1] 0
> a[,1]
[1] 0 1
> a[1,]
[1]  0  1  3  8 21
~~~
{: .source}

#### Python

~~~
In [75]: fib
Out[75]: array([ 0,  1,  1,  2,  3,  5,  8, 13, 21, 34])

In [76]: a=fib.reshape(2,5)

In [79]: a
Out[79]:
array([[ 0,  1,  1,  2,  3],
       [ 5,  8, 13, 21, 34]])

In [80]: b=fib.reshape(5,2).T

In [81]: b
Out[81]:
array([[ 0,  1,  3,  8, 21],
       [ 1,  2,  5, 13, 34]])

In [82]: a[0,0]
Out[82]: 0

In [83]: a[0]
Out[83]: array([0, 1, 1, 2, 3])

In [84]: b[0,0]
Out[84]: 0

In [85]: b[0]
Out[85]: array([ 0,  1,  3,  8, 21])

In [87]: b[:,0]
Out[87]: array([0, 1])
~~~
{: .source}

### Operations with vectors and Matrices

Both in R and Python R vectors and matrices and numpy arrays can be added, multiplied element by element and executing general element-by-element functions even including higher functions.

#### R

~~~
> fib
 [1]  0  1  1  2  3  5  8 13 21 34
>
> b <- 9 : 0
> length(b)
[1] 10
> length(fib)
[1] 10
> fib+b
 [1]  9  9  8  8  8  9 11 15 22 34
> 2*fib
 [1]  0  2  2  4  6 10 16 26 42 68
> fib/3
 [1]  0.0000000  0.3333333  0.3333333  0.6666667  1.0000000  1.6666667
 [7]  2.6666667  4.3333333  7.0000000 11.3333333
> fib^2
 [1]    0    1    1    4    9   25   64  169  441 1156
> sqrt(fib)
 [1] 0.000000 1.000000 1.000000 1.414214 1.732051 2.236068 2.828427 3.605551
 [9] 4.582576 5.830952
> log(fib)
 [1]      -Inf 0.0000000 0.0000000 0.6931472 1.0986123 1.6094379 2.0794415
 [8] 2.5649494 3.0445224 3.5263605
> sin(fib)
 [1]  0.0000000  0.8414710  0.8414710  0.9092974  0.1411200 -0.9589243
 [7]  0.9893582  0.4201670  0.8366556  0.5290827
~~~
{: .source}

~~~
In [88]: fib
Out[88]: array([ 0,  1,  1,  2,  3,  5,  8, 13, 21, 34])

In [89]: b=np.arange(9,-1,-1)

In [90]: fib.shape
Out[90]: (10,)

In [91]: b.shape
Out[91]: (10,)

In [92]: fib+b
Out[92]: array([ 9,  9,  8,  8,  8,  9, 11, 15, 22, 34])

In [93]: 2*fib
Out[93]: array([ 0,  2,  2,  4,  6, 10, 16, 26, 42, 68])

In [94]: fib/3
Out[94]:
array([ 0.        ,  0.33333333,  0.33333333,  0.66666667,  1.        ,
        1.66666667,  2.66666667,  4.33333333,  7.        , 11.33333333])

In [95]: fib**2
Out[95]: array([   0,    1,    1,    4,    9,   25,   64,  169,  441, 1156])

In [96]: np.sqrt(fib)
Out[96]:
array([0.        , 1.        , 1.        , 1.41421356, 1.73205081,
       2.23606798, 2.82842712, 3.60555128, 4.58257569, 5.83095189])

In [97]: np.log(fib)
/opt/local/bin/ipython:1: RuntimeWarning: divide by zero encountered in log
  #!/opt/local/Library/Frameworks/Python.framework/Versions/3.6/bin/python3.6
Out[97]:
array([      -inf, 0.        , 0.        , 0.69314718, 1.09861229,
       1.60943791, 2.07944154, 2.56494936, 3.04452244, 3.52636052])

In [98]: np.sin(fib)
Out[98]:
array([ 0.        ,  0.84147098,  0.84147098,  0.90929743,  0.14112001,
       -0.95892427,  0.98935825,  0.42016704,  0.83665564,  0.52908269])
~~~
{: .source}

### Basic matrix operations

The fundamental matrix operations can be perform with quite different syntax on both languages.

#### R

~~~
> fib=c(0, 1, 1, 2,  3,  5,  8, 13, 21, 34, 55, 89, 144, 233, 377, 610)
> m<-matrix(fib,4,4)
> m
     [,1] [,2] [,3] [,4]
[1,]    0    3   21  144
[2,]    1    5   34  233
[3,]    1    8   55  377
[4,]    2   13   89  610
> t(m)
     [,1] [,2] [,3] [,4]
[1,]    0    1    1    2
[2,]    3    5    8   13
[3,]   21   34   55   89
[4,]  144  233  377  610
> t(fib)
     [,1] [,2] [,3] [,4] [,5] [,6] [,7] [,8] [,9] [,10] [,11] [,12] [,13] [,14]
[1,]    0    1    1    2    3    5    8   13   21    34    55    89   144   233
     [,15] [,16]
[1,]   377   610
> a<-m
> b<-t(m)
> a+b
     [,1] [,2] [,3] [,4]
[1,]    0    4   22  146
[2,]    4   10   42  246
[3,]   22   42  110  466
[4,]  146  246  466 1220
> a*b
     [,1] [,2]  [,3]   [,4]
[1,]    0    3    21    288
[2,]    3   25   272   3029
[3,]   21  272  3025  33553
[4,]  288 3029 33553 372100
> a%*%b
      [,1]   [,2]   [,3]   [,4]
[1,] 21186  34281  55467  89748
[2,] 34281  55471  89752 145223
[3,] 55467  89752 145219 234971
[4,] 89748 145223 234971 380194
> diag(4)
     [,1] [,2] [,3] [,4]
[1,]    1    0    0    0
[2,]    0    1    0    0
[3,]    0    0    1    0
[4,]    0    0    0    1
~~~
{: .source}

~~~
In [101]: fib=np.array((0, 1, 1, 2,  3,  5,  8, 13, 21, 34, 55, 89, 144, 233, 377, 610))

In [102]: m=fib.reshape(4,4).T

In [103]: m
Out[103]:
array([[  0,   3,  21, 144],
       [  1,   5,  34, 233],
       [  1,   8,  55, 377],
       [  2,  13,  89, 610]])

In [104]: m.T
Out[104]:
array([[  0,   1,   1,   2],
       [  3,   5,   8,  13],
       [ 21,  34,  55,  89],
       [144, 233, 377, 610]])

In [105]: fib.T
Out[105]:
array([  0,   1,   1,   2,   3,   5,   8,  13,  21,  34,  55,  89, 144,
       233, 377, 610])

In [106]: fib.reshape(-1,len(fib))
Out[106]:
array([[  0,   1,   1,   2,   3,   5,   8,  13,  21,  34,  55,  89, 144,
        233, 377, 610]])

In [107]: fib2 = fib[np.newaxis]

In [108]: fib
Out[108]:
array([  0,   1,   1,   2,   3,   5,   8,  13,  21,  34,  55,  89, 144,
       233, 377, 610])

In [109]: fib2
Out[109]:
array([[  0,   1,   1,   2,   3,   5,   8,  13,  21,  34,  55,  89, 144,
        233, 377, 610]])

In [110]: fib2.T
Out[110]:
array([[  0],
       [  1],
       [  1],
       [  2],
       [  3],
       [  5],
       [  8],
       [ 13],
       [ 21],
       [ 34],
       [ 55],
       [ 89],
       [144],
       [233],
       [377],
       [610]])

In [111]: a=m

In [112]: b=m.T

In [113]: a+b
Out[113]:
array([[   0,    4,   22,  146],
       [   4,   10,   42,  246],
       [  22,   42,  110,  466],
       [ 146,  246,  466, 1220]])

In [114]: a*b
Out[114]:
array([[     0,      3,     21,    288],
       [     3,     25,    272,   3029],
       [    21,    272,   3025,  33553],
       [   288,   3029,  33553, 372100]])

In [115]: np.dot(a,b)
Out[115]:
array([[ 21186,  34281,  55467,  89748],
       [ 34281,  55471,  89752, 145223],
       [ 55467,  89752, 145219, 234971],
       [ 89748, 145223, 234971, 380194]])

In [116]: np.eye(4)
Out[116]:
array([[1., 0., 0., 0.],
       [0., 1., 0., 0.],
       [0., 0., 1., 0.],
       [0., 0., 0., 1.]])
~~~
{: .source}

### Defining functions

Functions in R and Python works very similar, with the extra on the python side that being Python capable of Object oriented programming. You can also create classes, that can encapsulate methods and variables. As both are interpreted languages, the arguments of the function do not need to be typed.

#### R

~~~
> cv<-function(x) sd(x)/mean(x)
> cv(1:10)
[1] 0.5504819
>
> gcd<-function(a,b){
+    if(b==0) return(a)
+    else return(gcd(b,a%%b))
+ }
> gcd(144,260)
[1] 4
~~~

#### Python

~~~
In [127]: def cv(x):
     ...:     return np.std(x,ddof=1)/np.mean(x)
     ...:
     ...:

In [128]: x=np.arange(1,11)

In [129]: cv(x)
Out[129]: 0.5504818825631803

In [130]: def gcd(a,b):
     ...:     if(b==0):
     ...:         return a
     ...:     else:
     ...:         return gcd(b,a%b)
     ...:     

In [131]: gcd(144,260)
Out[131]: 4
~~~
{: .source}


### Reading Datasets

In data analysis datasets represent tables that can be identified by columns. In R datasets are intrinsic to the language, in the case of Python you have use an external package called `pandas` to achieve a similar functionality.

#### R

~~~
> ds<-read.csv('plantTraits.csv')
> ds
X    pdias  longindex durflow height begflow mycor vegaer vegsout
1   Aceca   96.840 0.00000000       2      7       5     2      0       0
2   Aceps  110.720 0.00000000       3      8       4     2      0       0
3   Agrca    0.060 0.66666667       3      2       6     2      0       1
4   Agrst    0.080 0.48888889       2      2       7     1      2       0
5   Ajure    1.480 0.47619048       3      2       5     2      2       0
6   Allpe    2.330 0.50000000       3      5       4     0      0       0
7   Anaar    0.380 0.90476190       3      2       6     2      0       0
8   Anene    2.550 0.06666667       3      2       3     2      0       2
9   Angsy    1.480 0.21052632       3      3       7     2      0       0
...
~~~
{: .source}

#### Python

~~~
In [135]: ds=pd.read_csv('plantTraits.csv')

In [136]: ds
Out[136]:
    Unnamed: 0    pdias  longindex  durflow  ...   epizoo  aquat  windgl  unsp
0        Aceca    96.84   0.000000        2  ...      0.0    0.0     1.0   0.0
1        Aceps   110.72   0.000000        3  ...      0.0    0.0     1.0   0.0
2        Agrca     0.06   0.666667        3  ...      0.0    0.0     0.0   1.0
3        Agrst     0.08   0.488889        2  ...      0.0    0.0     0.0   1.0
4        Ajure     1.48   0.476190        3  ...      0.0    0.0     0.0   0.0
5        Allpe     2.33   0.500000        3  ...      0.0    0.0     0.0   1.0
6        Anaar     0.38   0.904762        3  ...      0.0    0.0     0.0   1.0
7        Anene     2.55   0.066667        3  ...      1.0    0.0     0.0   0.0
8        Angsy     1.48   0.210526        3  ...      0.0    1.0     0.0   0.0
9        Antod     0.52   0.369565        3  ...      1.0    0.0     0.0   0.0
~~~
{: .source}

After loading a CSV file the dataset can be manipulated in quite similar way on both sides

#### R

~~~
> ds['pdias']
       pdias
1     96.840
2    110.720
3      0.060
4      0.080
5      1.480
6      2.330
7      0.380
8      2.550
9      1.480
...
~~~
{: .source}

#### Python

~~~
In [137]: ds['pdias']
Out[137]:
0        96.84
1       110.72
2         0.06
3         0.08
4         1.48
5         2.33
6         0.38
7         2.55
8         1.48
9         0.52
...
~~~
{: .sources}



{% include links.md %}
