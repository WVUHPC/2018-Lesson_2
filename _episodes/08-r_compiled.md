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





{% include links.md %}
