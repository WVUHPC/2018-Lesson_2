---
title: "Fortran inside R"
teaching: 45
exercises: 15
questions:
- "How to accelerate numerical intensive operations in R"
objectives:
- "Using Fortran inside R for a matrix operation."
keypoints:
- "You can integrate compiled languages like Fortran or C inside R to optimize specific routines. The resulting code becomes restricted to only run on similar machines."
---

## Using compiled code inside R

We will show how you can accelerate executing of numerical intensive operations by moving those routines into a compiled language but still using R for calling the functions:

On the `2018-Data-HandsOn/2.Programming/9.Fortran_R`

~~~
!
! simple subroutine to compute factorial
!
subroutine facto(n, answer)
    integer n, answer, i

    answer = 1
    do i = 2,n
        answer = answer * i
    end do
end subroutine
~~~
{: .language-fortran}

We compile the code with:

~~~
$ gfortran facto.f90 -c
$ ls
~~~
{: .language-bash}
~~~
facto.f90	facto.o
~~~
{: .output}

And we create the shared Library

~~~
$ gfortran -shared -o facto.so facto.o
~~~
{: .language-bash}

Now we can go into R and load the library

~~~
> dyn.load("facto.so")
> .Fortran("facto",n=as.integer(40),answer=as.numeric(1.0))
$n
[1] 40

$answer
[1] 8.159153e+47
~~~
{: .source}

## The Sieve Of Eratosthenes (Reloaded)

Now we can go back to the original algorithm this morning and see if we can take advantage of this for improving our execution in R

We do not need to chage the original Fortran code, the subroutine works fine for insert it in R

~~~
$ gfortran -c SieveOfEratosthenes.f90
$ gfortran -shared -o SieveOfEratosthenes.so SieveOfEratosthenes.o
~~~

Now that we have our new library we can use it inside R

~~~
> dyn.load("SieveOfEratosthenes.so")
> .Fortran("SieveOfEratosthenes",n=as.integer(40),nprimes=as.integer(1))
  Dimension of array:           20
  Array with indices between 0 to           19
  Stores primality of odd numbers in range [1,39]
         2         3         5         7        11        13        17        19        23        29        31        37        41
$n
[1] 40

$nprimes
[1] 13

> .Fortran("SieveOfEratosthenes",n=as.integer(100000000),nprimes=as.integer(1))
  Dimension of array:     50000000
  Array with indices between 0 to     49999999
  Stores primality of odd numbers in range [1,99999999]
$n
[1] 100000000

$nprimes
[1] 5761456
~~~
{: .source}

### Wrapping into a function

~~~
SieveOfEratosthenes <- function(num) {
  dyn.load("SieveOfEratosthenes.so")
  retvals <- .Fortran("SieveOfEratosthenes",n = as.integer(num), nprimes = as.integer(1))
  return(retvals$nprimes)
}
~~~
{: .language-r}

### Benchmarking in R

There are several packages in R for Benchmarking, we will demonstrate two of them: "microbenchmark" and "rbenchmark"

~~~
> microbenchmark(SieveOfEratosthenes(100000000), times=3)
~~~
{: .language-r}
~~~
Dimension of array:     50000000
Array with indices between 0 to     49999999
Stores primality of odd numbers in range [1,99999999]
Dimension of array:     50000000
Array with indices between 0 to     49999999
Stores primality of odd numbers in range [1,99999999]
Dimension of array:     50000000
Array with indices between 0 to     49999999
Stores primality of odd numbers in range [1,99999999]
Unit: seconds
                     expr     min       lq     mean   median       uq
SieveOfEratosthenes(1e+08) 1.92569 1.932376 1.936352 1.939062 1.941683
    max neval
1.944303     3
~~~
{: .output}

~~~
> library("rbenchmark")
> benchmark(SieveOfEratosthenes(100000000), replications=3)
~~~
{: .language-r}
~~~
  Dimension of array:     50000000
  Array with indices between 0 to     49999999
  Stores primality of odd numbers in range [1,99999999]
  Dimension of array:     50000000
  Array with indices between 0 to     49999999
  Stores primality of odd numbers in range [1,99999999]
  Dimension of array:     50000000
  Array with indices between 0 to     49999999
  Stores primality of odd numbers in range [1,99999999]
  Dimension of array:     50000000
  Array with indices between 0 to     49999999
  Stores primality of odd numbers in range [1,99999999]
                        test replications elapsed relative user.self sys.self
1 SieveOfEratosthenes(1e+08)            3   5.814        1     5.556    0.248
  user.child sys.child
1          0         0
~~~
{: .output}

### Profiling code

Lets import the original code:

~~~
> source("SieveOfEratosthenes.R")
 [1]  2  3  5  7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71 73 79 83 89 97
>
> SieveOfEratosthenes(100000000)
[1] 5761455
~~~
{: .source}

~~~
> source("SieveOfEratosthenes.R")
 [1]  2  3  5  7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71 73 79 83 89 97
>
> SieveOfEratosthenes(100000000)
[1] 5761455
> Rprof("run.out")
> SieveOfEratosthenes(100000000)
[1] 5761455
> Rprof(NULL)
> summaryRprof("run.out")
$by.self
                      self.time self.pct total.time total.pct
"pmin"                     4.20    36.59       4.20     36.59
"SieveOfEratosthenes"      3.84    33.45      11.48    100.00
"seq.default"              3.20    27.87       7.40     64.46
"sum"                      0.24     2.09       0.24      2.09

$by.total
                      total.time total.pct self.time self.pct
"SieveOfEratosthenes"      11.48    100.00      3.84    33.45
"seq.default"               7.40     64.46      3.20    27.87
"seq"                       7.40     64.46      0.00     0.00
"pmin"                      4.20     36.59      4.20    36.59
"sum"                       0.24      2.09      0.24     2.09

$sample.interval
[1] 0.02

$sampling.time
[1] 11.48
~~~
{: .source}

Using memory profiling:

~~~
> Rprof("run_mem2.out", memory.profiling=TRUE)
> SieveOfEratosthenes(100000000)
[1] 5761455
> Rprof(NULL)
> summaryRprof("run_mem.out", memory="both")
$by.self
                      self.time self.pct total.time total.pct mem.total
"pmin"                     4.06    35.43       4.06     35.43    3340.0
"SieveOfEratosthenes"      3.94    34.38      11.46    100.00    5905.4
"seq.default"              3.18    27.75       7.26     63.35    4818.8
"sum"                      0.26     2.27       0.26      2.27       7.9
"max"                      0.02     0.17       0.02      0.17       7.7

$by.total
                      total.time total.pct mem.total self.time self.pct
"SieveOfEratosthenes"      11.46    100.00    5905.4      3.94    34.38
"seq.default"               7.26     63.35    4818.8      3.18    27.75
"seq"                       7.26     63.35    4818.8      0.00     0.00
"pmin"                      4.06     35.43    3340.0      4.06    35.43
"sum"                       0.26      2.27       7.9      0.26     2.27
"max"                       0.02      0.17       7.7      0.02     0.17

$sample.interval
[1] 0.02

$sampling.time
[1] 11.46
~~~
{: .source}

Monitoring the garbage collector

~~~
> gcinfo()
Error in gcinfo() : argument "verbose" is missing, with no default
> gcinfo(TRUE)
[1] FALSE
> SieveOfEratosthenes(100000000)
Garbage collection 37 = 19+5+13 (level 1) ...
19.9 Mbytes of cons cells used (59%)
1786.2 Mbytes of vectors used (86%)
Garbage collection 38 = 19+5+14 (level 2) ...
19.8 Mbytes of cons cells used (58%)
959.6 Mbytes of vectors used (46%)
Garbage collection 39 = 20+5+14 (level 0) ...
19.8 Mbytes of cons cells used (58%)
1468.2 Mbytes of vectors used (71%)
Garbage collection 40 = 20+6+14 (level 1) ...
19.8 Mbytes of cons cells used (58%)
1213.9 Mbytes of vectors used (58%)
Garbage collection 41 = 21+6+14 (level 0) ...
19.8 Mbytes of cons cells used (58%)
1322.9 Mbytes of vectors used (64%)
Garbage collection 42 = 22+6+14 (level 0) ...
19.8 Mbytes of cons cells used (58%)
1372.7 Mbytes of vectors used (66%)
Garbage collection 43 = 23+6+14 (level 0) ...
19.8 Mbytes of cons cells used (58%)
1385.5 Mbytes of vectors used (67%)
Garbage collection 44 = 24+6+14 (level 0) ...
19.8 Mbytes of cons cells used (58%)
1386.3 Mbytes of vectors used (67%)
Garbage collection 45 = 25+6+14 (level 0) ...
19.8 Mbytes of cons cells used (58%)
1387.9 Mbytes of vectors used (67%)
[1] 5761455
~~~
{: .source}




{% include links.md %}
