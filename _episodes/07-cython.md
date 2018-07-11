---
title: "Cython: Accelerate Python Execution"
teaching: 45
exercises: 15
questions:
- "What is Cython and its relation with Python?"
objectives:
- "Optimize a numerical intensive operation with Cython"
keypoints:
- "Cython is a superset of the Python programming language, designed to give C-like performance with code that is written mostly in Python."
---
# Cython

Cython is a language the extends Python. With minimal changes to a python code,
you can get a compiled code that runs faster. From another perspective, Cython is like a bridge that allows you to interface C and C++ routines inside python.

We will follow a very practical route to give you basic elements that will quickly improve the performance of your code with minimal effort. Scientific computing, performance is central, but also you want to optimize the time spent developing the code. The right balance between both is the goal. Using Python is an efficient way of getting a code that works, but still work must to be done to make it efficient.

Let us take the code for the Sieve of Eratosthenes, the python code was very poor in performance. We are using an interpreted language, but also we are using lists and loops. Those elements make the code run very slowly, that was done on purpose because allow us to explore how to improve from there.

We start splitting the original code in two files. The file that contains the function `SieveOfEratosthenes` will be separated on a file called `sieve.py`:

~~~
def SieveOfEratosthenes(n):

    if n%2 == 0:
        imax=int(n/2)
    else:
        imax=int((n+1)/2)

    neff= 2*(imax-1)+1
    prime = imax*[True]

    print(" Dimension of array: %d\n" % imax)
    print(" Array with indices between 0 to %d\n" % (imax-1))
    print(" Stores primality of odd numbers in range [1, %d]\n\n" % neff)

    # The loop runs over all odd numbers starting at 3
    p=3
    while (p<n/2+1):
        # If prime[p] is true, then it is a prime
        if(prime[int((p-1)/2)] is True):
            # Update all multiples of p
            i = 2*p
            while (i <= neff):
                if (i%2!=0):
                    prime[int((i-1)/2)] = False
                i += p
        p+=2

    counter = 1
    nprimes = 0
    # Print all prime numbers if the number is small otherwise just count the number of primes found
    if (imax < 10000):
        print("%10d " % 2, end='')
        # Starting with index 1 instead of zero because (2*0)+1 = 1 is not prime
        for i in range(1,imax):
            if(prime[i] == 1):
                print("%10d " % (2*i+1), end='')
                counter+=1

            if (counter == 15):
                print("")
                counter=0

        print("\n")

    else:
        for i in range(1,imax):
            if(prime[i] is True):
                nprimes+=1
        print(" Total number of primes found: %d" % nprimes)

    return nprimes
~~~
{: .language-python}

The other file will just contain the `main` execution:

~~~
import sys
import sieve

if __name__ == "__main__":

    if (len(sys.argv) > 1):
        n = int(float(sys.argv[1]))
    else:
        n = 100

    print("")
    print(" Prime numbers up to %d" % n)
    nprimes = sieve.SieveOfEratosthenes(n)
    print(" Function return: %d" % nprimes)
~~~
{: .language-python}

Let us compute how much time will take to find all prime numbers among the first 50 million integers. We do not want to do that on the head node. As several others will be doing the same, we will use a qsub on interactive mode, here I am using `debug`, but use any queue that allows you to enter quickly.

~~~
$ qsub -I -q debug
~~~
{: .bash}

Once you get allocated a compute node use:

~~~
$ module load compilers/python/3.6.0
$ cd 2018-Data-HandsOn/2.Programming/5.Cython
$ time python3 main.py 5E7
~~~
{: .bash}
~~~

 Prime numbers up to 50000000
 Dimension of array: 25000000

 Array with indices between 0 to 24999999

 Stores primality of odd numbers in range [1, 49999999]


 Total number of primes found: 3001133
 Function return: 3001133

real    0m35.606s
user    0m35.560s
sys    0m0.052s
~~~
{: .output}

From above, you see that it takes 35 seconds to compute the prime numbers up to 50 million, there are several reasons for that extreme long execution time, we are using an interpreted language, we are using python's lists instead of a library
like numpy and we are using python's loops instead of vectorized operations again using numpy.

Let us explore what we can do easily with Cython to improve that figure.
We start copying `sieve.py` as `sieve_ct.pyx` and adding this to the first line of the new file:

~~~
#cython: language_level=3, boundscheck=False
~~~
{: .source}

That line is a comment for python but will instruct cython that the code below is python 3. Otherwise, cython interprets print as python 2 and will fail.

Cython works by taking a code like `sieve_ct.pyx` that right now is exactly identical to the original and create a C code `sieve_ct.c` that can be compiled into a shared library `sieve_ct.so` that we can import as any other python module.

There are several ways of compiling Cython extensions. Three ways will demonstrate here: the manual way, using `distutils` and using `pyximport` for transparent compilation.

## Compiling manually

To compile manually you need first to identify where the python headers are located.
On Spruce using the Python 3.6.0 the following command will give you the answer

~~~
$ python3 -c "from distutils import sysconfig; print(sysconfig.get_python_inc())"
~~~
{: .bash}
~~~
/shared/software/languages/python/3.6.0/include/python3.6m
~~~
{: .output}

With that knowledge we compile the code with this two steps:

~~~
$ cython -3 sieve_ct.pyx
$ gcc -shared -fPIC -O2 -Wall -lm -I/shared/software/languages/python/3.6.0/include/python3.6m sieve_ct.c -o sieve_ct.so
~~~
{: .bash}

A copy of `main.py` with a couple of small modifications can help us to test the new module. We are renaming the module to avoid loading the original `sieve.py` instead.
As we rename the code as `sieve_ct.pyx` and the final module will be `sieve_ct` we need to change the main like this:

~~~
import sys
import sieve_ct

if __name__ == "__main__":

    if (len(sys.argv) > 1):
        n = int(sys.argv[1])
    else:
        n = 100

    print("")
    print(" Prime numbers up to %d" % n)
    nprimes = sieve_ct.SieveOfEratosthenes(n)
    print(" Function return: %d" % nprimes)
~~~
{: .language-python}

Let us get some timing using the new cython modules (remember to use `qsub -I`)

~~~
$ module load compilers/python/3.6.0
$ cd 2018-Data-HandsOn/2.Programming/5.Cython
$ time python3 main_ct.py 5E7
~~~
{: .bash}
~~~
 Prime numbers up to 50000000
 Dimension of array: 25000000

 Array with indices between 0 to 24999999

 Stores primality of odd numbers in range [1, 49999999]


 Total number of primes found: 3001133
 Function return: 3001133

real    0m17.602s
user    0m17.549s
sys    0m0.056s
~~~
{: .output}

We are basically cutting in a half the time needed. Not bad for a code that has not been changed at all.

## Compiling with distutils

Another way of compiling the code is using `distutils`. Create a file `setup.py`
and enter this lines:

~~~
from distutils.core import setup
from Cython.Build import cythonize

setup(
    name='sieve',
    ext_modules=cythonize('sieve_ct.pyx')
    )
~~~
{: .language-python}

We are using the `cythonize` module from Cython that allow us to compile the code without actually running `cython` and the `gcc` command.

~~~
$ python3 setup.py build_ext --inplace
~~~
{: .bash}
~~~
running build_ext
building 'sieve_ct' extension
creating build
creating build/temp.linux-x86_64-3.6
gcc -pthread -Wno-unused-result -Wsign-compare -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I/shared/software/languages/python/3.6.0/include/python3.6m -c sieve_ct.c -o build/temp.linux-x86_64-3.6/sieve_ct.o
creating build/lib.linux-x86_64-3.6
gcc -pthread -shared -flto -ffat-lto-objects -flto-partition=none build/temp.linux-x86_64-3.6/sieve_ct.o -o build/lib.linux-x86_64-3.6/sieve_ct.cpython-36m-x86_64-linux-gnu.so
~~~
{: .output}

As you can see that save you from knowing the compilation line. The `--inplace` is just to get the resulting `.so` file in the same place as the `main_ct.py`. Otherwise, you will have to look inside the build folder for the `.so` file generated.

The new file is called `sieve_ct.cpython-36m-x86_64-linux-gnu.so`, you can get timings again and noticing that the new code is one or two seconds faster, just because the compilation line add a bit higher optimization level.

## Using cython transparently with pyximport

The third option is to use `pyximport`, use interactive mode and not running on the head node.

~~~
$ module load compilers/python/3.6.0
$ cd 2018-Data-HandsOn/2.Programming/5.Cython
$ rm *.so
~~~
{: .bash}

And run IPython:

~~~
$ ipython3
Python 3.6.0 (default, Jan 24 2017, 12:13:37)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.3.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import pyximport

In [2]: pyximport.install()
Out[2]: (None, <pyximport.pyximport.PyxImporter at 0x7fe26a7766a0>)

In [3]: import sieve_ct

In [4]:%timeit sieve_ct.SieveOfEratosthenes(1E7)
...
...
15.3 s ± 269 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
~~~
{: .source}

The module `pyximport` will take care of compile the code transparently for you. On this example we are using the magic `%timeit` from IPython that help us to benchmark the function, revealing that it takes now 15 seconds to compute the primes.

## Adding static types

Until now, we have not change the function `SieveOfEratosthenes(n)` from the original used on our first episode. We can get another boost in performance by using "typed variables".

In python, you can use variables to store all sort of types and changing them during execution, for example, you can declare `a=1` and later change it into `a='one'`.
That kind of flexibility is very welcome on an interpreted language but it comes with high-performance overhead. Without knowing the type of variables in advance, the interpreter cannot produce on-the-fly the kind of optimized code that a programming language like C can produce.

In Cython, you can declare the type of variables with a minimal change on the code.
Make a copy of `sieve_ct.pyx` as `sieve_st.pyx` and add this line inside the function definition:

~~~
#cython: language_level=3, boundscheck=False

import sys

def SieveOfEratosthenes(n):

    # Type declared variables for cython
    cdef int imax, neff, p, i, counter, nprimes

    if n%2 == 0:
        imax=int(n/2)
    else:
        imax=int((n+1)/2)

    neff= 2*(imax-1)+1
    prime = imax*[True]
...
...
~~~
{: python}

The line `cdef int imax, neff, p, i, counter, nprimes` instruct cython that all those variables will only contain integers for the entire execution. Save the file and get new timings for the new code.

Here we are using the `pyximport` inside IPython and timing with the magic `%timeit`. Remember to use `qsub -I`

~~~
$ ipython3
Python 3.6.0 (default, Jan 24 2017, 12:13:37)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.3.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import pyximport

In [2]: pyximport.install()
Out[2]: (None, <pyximport.pyximport.PyxImporter at 0x7f573945def0>)

In [3]: import sieve_st

In [4]: %timeit sieve_st.SieveOfEratosthenes(5E7)
...
...
8.34 s ± 818 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
~~~
{: .source}

## Working with C arrays and pointers

One of the reasons why the Python code that we proposed initially runs so slow is that we are storing the primality condition for all odd numbers as a big list.
Python list are very flexible, they are mutable, meaning that you can add and remove elements to the list and change the values on the fly. Those features are nice but also comes with a performance tag. This time we need to introduce more changes.

This is the final code and we will explain the changes:

~~~
#cython: language_level=3, boundscheck=False
from libc.stdio cimport printf
from libc.stdlib cimport malloc, free

import sys

def SieveOfEratosthenes(n):

    cdef int imax, neff, p, i, counter, nprimes

    if n%2 == 0:
        imax=int(n/2)
    else:
        imax=int((n+1)/2)

    neff= 2*(imax-1)+1

    cdef int *prime = <int *> malloc(imax * sizeof(int))
    for i in range(imax):
        prime[i]=1    

    print(" Dimension of array: %d\n" % imax)
    print(" Array with indices between 0 to %d\n" % (imax-1))
    print(" Stores primality of odd numbers in range [1, %d]\n\n" % neff)

    # The loop runs over all odd numbers starting at 3
    p=3
    while (p<n/2+1):
        # If prime[p] is true, then it is a prime
        if(prime[int((p-1)/2)] == 1):
            # Update all multiples of p
            i = 2*p
            while (i <= neff):
                if (i%2!=0):
                    prime[int((i-1)/2)] = 0
                i += p
        p+=2

    counter = 1
    nprimes = 0
    # Print all prime numbers if the number is small otherwise just count the number of primes found
    if (imax < 10000):
        print("%10d " % 2, end='')
        # Starting with index 1 instead of zero because (2*0)+1 = 1 is not prime
        for i in range(1,imax):
            if(prime[i] == 1):
                print("%10d " % (2*i+1), end='')
                counter+=1

            if (counter == 15):
                print("")
                counter=0

        print("\n")

    else:
        for i in range(1,imax):
            if(prime[i] == 1):
                nprimes+=1
        print(" Total number of primes found: %d" % nprimes)

    free(prime)
    return nprimes
~~~
{: .language-python}

First, we add some imports from `libc`, those imports are very similar to `#include <stdlib>` and `#include <stdio>`

~~~
#cython: language_level=3, boundscheck=False
from libc.stdio cimport printf
from libc.stdlib cimport malloc, free
~~~
{: .language-python}

The next step is to replace the Phyton `prime` list into a Cython C-like array (actually a pointer).

~~~
cdef int *prime = <int *> malloc(imax * sizeof(int))
for i in range(imax):
    prime[i]=1    
~~~
{: .language-python}

The other changes in the code consist from changing storing and comparing Booleans (`True` and `False`) for integers (1 or 0)

The final change is freeing the memory for `prime` array. Python offers a convenient garbage collector, but in C, you are responsible from freeing the memory that you are not using, for this particular case is not a big deal, as the program will finish after evaluating the primes, but in more complex situations freeing memory is an important issue for keeping the code efficient by avoiding memory leaks.

Now, it is time for benchmarking the new code.

~~~
$ ipython
Python 3.6.0 (default, Jan 24 2017, 12:13:37)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.3.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import pyximport

In [2]: pyximport.install()
Out[2]: (None, <pyximport.pyximport.PyxImporter at 0x7f783904c668>)

In [3]: import sieve_ar

In [4]: %timeit sieve_ar.SieveOfEratosthenes(5E7)
...
...
1.59 s ± 35 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)
~~~
{: .source}

We have successfully lowered the time to compute the prime numbers up to 50 million from 39 seconds to 1.5 seconds.

Let us summarize our success in the next table:

|Code        |Description |Timing (Using %timeit)|
|:--------|:-----------|:---------------------|
|`sieve.py` |Original code| 39 s ± 1.01 s per loop (mean ± std. dev. of 7 runs, 1 loop each)|
|`sieve_ct.pyx`| Original code with Cython compilation|16.4 s ± 1.19 s per loop (mean ± std. dev. of 7 runs, 1 loop each)|
|`sieve_st.pyx`| Using typed variables|7.18 s ± 112 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)|
|`sieve_ar.pyx`| Using C arrays and pointers|1.62 s ± 24.5 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)|

{% include links.md %}
