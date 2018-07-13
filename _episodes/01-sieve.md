---
title: "The Sieve of Eratosthenes"
teaching: 45
exercises: 15
questions:
- "How one algorithm looks in 7 different languages?"
objectives:
- "Learn basic syntax from several languages used in scientific computing"
keypoints:
- "A literal translation of an algorithm ends up in a poorly efficient code."
- "Each language has its own way of doing things. Knowing how to adapt the algorithm to the language is the way to create efficient code."
---

# Programming Languages for Scientific Computing

There are literally hundreds of programming languages. However, not all of them are suitable or commonly used for Scientific Computing. On this lesson we will present a selection of the most prominent languages for Scientific Computing today.

Without entering in subtle details, we can classify programming languages in several ways, with some shadow zones in between. Programming languages can be:

1. Interpreted or Compiled

2. Static Typed or Dynamic Typed.

In general compiled languages are also static typed and interpreted languages are dynamically typed. Examples of compiled languages are C/C++ and Fortran, in the category of interpreted we can mention Python, R and Julia. Java is an special case where code is compiled in a intermediate representation called "bytecode" that is interpreted by the Java Virtual Machine during execution.

The difference is even more diffuse by technologies like Just-in-Time compilation that uses compilation during runtime for an usually interpreted language.

There are only 3 languages that we can consider as truly High-Performance Programming Languages (PL), this trinity is made by Fortran, C and C++. Other languages, like Python and R even if used very often in Scientific Computing are not High-Performance at its core. Julia is a newer language filling the gap in this classification.

Fortran, C and C++ are all of them static typed, compiled and the most prominent paradigms for Parallel Computing in CPUs OpenMP and MPI has official support for those 3 languages. Among the list of languages that we will consider today they are also the older ones.

The next table shows the age of several languages used in  Scientific Computing.

> ## The Age of Programming Languages (by 2018)
>
> | Language | First appeared | Stable Release |
> |:---------|:---------------|:--------------|
> | Fortran | 1957; 61 years ago | Fortran 2008 (ISO/IEC 1539-1:2010) / 2010; 8 years ago |
> | C | 1972; 46 years ago | C11 / December 2011; 6 years ago |
> | C++ | 1985; 33 years ago | ISO/IEC 14882:2017 / 1 December 2017; 7 months ago |
> | Python | 1990; 27 years ago | 3.7.0 / 27 June 2018; 2.7.15 / 1 May 2018 |
> | R | 1993; 24 years ago | 3.5.1 ("Feather Spray")/ July 2, 2018 |
> | Java | 1995; 23 years ago |  Java 10, released on March 20, 2018 |
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

## The Sieve of Eratosthenes

In order to illustrate all those languages in a short glimpse we will be showing how to implement the same algorithm in all those 7 languages. The algorithm is called the Sieve of Eratosthenes and it is a classical algorithm for finding all prime numbers up to any given limit. These are the rules for the implementation.

1. All implementations will be as close as possible to each other. In fact, that is a bad idea from the performance point of view but, for our purpose will facilitate the comparison in syntax rather how to achieve top performance with each PL.

2. No external libraries will be used, with the sole exception of Input/Output routines. Interpreted languages take advantage of external libraries to improve performance, those external languages are usually written in a compiled language, that is the case of several R and Python libraries.

3. The fairly accelerate the algorithm we will skip all even numbers, there is just one prime even number and we know what it is.

<img src="https://upload.wikimedia.org/wikipedia/commons/9/94/Animation_Sieve_of_Eratosth.gif" />

For this exercises, please enter in interactive mode as the head node cannot take all users executing at the same time. The Sieve was chosen for being very CPU intensive but with little memory requirements.

### R

Lets start with interpreted languages. R is one of the most used languages for statistical analysis. The sieve here is implemented as an Rscript and uses a full dimension for the array. It expects one integer argument as the upper limit for the sieve, otherwise defaults to 100.

~~~
#!/usr/bin/env Rscript

SieveOfEratosthenes <- function(n) {
  if (n < 2) return(NULL)
  primes <- rep(T, n)
  sprintf(" Dimension of array %d", length(primes))

  # 1 is not prime
  primes[1] <- F
  for(i in seq(n)) {
    if (primes[i]) {
      j <- i * i
      if (j > n){
      	 if (sum(primes, na.rm=TRUE)<10000) {
	    # Select just the indices of True values
	    print(which(primes))
	 }  
	 return(sum(primes, na.rm=TRUE))
	 }	 
      primes[seq(j, n, by=i)] <- F
    }
  }
  return()
}

# Collecting arguments from the command
args = commandArgs(trailingOnly=TRUE)

# Default number is n=100
if (length(args)==0) {
  n = 100
} else {
  n = strtoi(args[1])
}

sprintf(" Prime numbers up to %d", n)
nprimes<-SieveOfEratosthenes(n)
sprintf(" Total number of primes found: %d", nprimes)
~~~
{: .language-r}

Now lets get some timing on how much it takes for the first 100 million
integers. This timing was taking on Spruce but your numbers could be different based on which compute node are you running.

~~~
$ time ./SieveOfEratosthenes.R 100000000
~~~
{: .language-bash}
~~~
[1] " Prime numbers up to 100000000"
[1] " Total number of primes found: 5761455"

real    0m24.492s
user    0m23.672s
sys     0m0.823s
~~~
{: .output}

### Python

Our second interpreted language is Python, the implementation below is not using numpy, the numerical library. This is big handicap for this implementation having to rely only on very flexible but efficient python lists. Lists are part of the core language. Numpy, even not being part of the core or even the standard library is the defacto standard for manipulate arrays as we will see later today.

~~~
#!/usr/bin/env python3

import sys

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
        if(prime[int((p-1)/2)] == 1):
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
        print("%8d " % 2, end='')
        # Starting with index 1 instead of zero because (2*0)+1 = 1 is not prime
        for i in range(1,imax):
            if(prime[i] == 1):
                print("%8d " % (2*i+1), end='')
                counter+=1

            if (counter == 15):
                print("")
                counter=0

        print("\n")

    for i in range(1,imax):
        if(prime[i] is True):
            nprimes+=1

    return nprimes + 1

if __name__ == "__main__":

    if (len(sys.argv) > 1):
        n = int(sys.argv[1])
    else:
        n = 100

    print("")
    print(" Prime numbers up to %d" % n)
    nprimes = SieveOfEratosthenes(n)
    print(" Total number of primes found: %d" % nprimes)
~~~
{: .language-python}

And the timing (Obs: it is very slow!), so be patient

~~~
$ time ./SieveOfEratosthenes.py 100000000
~~~
{: .language-bash}
~~~
 Prime numbers up to 100000000
 Dimension of array: 50000000

 Array with indices between 0 to 49999999

 Stores primality of odd numbers in range [1, 99999999]


 Total number of primes found: 5761455

real    1m21.595s
user    1m21.505s
sys     0m0.111s
~~~
{: .output}

### Java

The big advantage of Java is being able to run with the same bytecode on machines running different Operating Systems like Linux and Windows. There are a number of scientific packages written in Java and packages for Data Analysis

~~~
class SieveOfEratosthenes
{
    void sieveOfEratosthenes(int n)
    {
        // Create a boolean array, we will avoid all even numbers
	// so the element i on the array stores true or false if
	// the number 2*i+1 is actually prime
	int imax, neff, counter=1, nprimes=0;
	boolean prime[];
	if (n%2==0)
	    imax=n/2;
	else
	    imax=(n+1)/2;
	neff= 2*(imax-1)+1;
	prime = new boolean[imax];

	// Lets assume that all odd numbers are primes
        for(int i=0;i<imax;i++)
            prime[i] = true;

	System.out.format(" Dimension of array: %d\n", imax);
	System.out.format(" Array with indices between 0 to %d\n", imax-1);
	System.out.format(" Stores primality of odd numbers in range [1, %d]\n\n", neff);

	// The loop runs over all odd numbers starting at 3
        for(int p = 3; p <n/2+1; p+=2)
	    {
		// If prime[p] is true, then it is a prime
		if(prime[(p-1)/2] == true)
		    {
			// Update all multiples of p
			for(int i = p*2; i <= neff; i += p)
			    {
			    if (i%2!=0) prime[(i-1)/2] = false;
			    }
		    }
	    }

        // Print all prime numbers if the number is small otherwise just count the number of primes found
	if (imax < 10000) {
	    System.out.format("%8d ", 2);
	    // Starting with index 1 instead of zero because (2*0)+1 = 1 is not prime
	    for(int i = 1; i < imax; i++){
		if(prime[i] == true){
		    System.out.format("%8d ", 2*i+1);
		    counter++;
		}
		if (counter == 15){
		    System.out.format("\n");
		    counter=0;
		}
	    }
	    System.out.format("\n");
	}
	else {
	    for(int i = 1; i < imax; i++){
		if(prime[i] == true)
		    nprimes ++;
	    }
	    System.out.format(" Total number of primes found: %d\n", nprimes+1);
	}
    }

    // Main program, reads command line to get upper limit for Sieve
    public static void main(String args[])
    {
        int n=1;
	if (args.length > 0) {
	    try {
		n = Integer.parseInt(args[0]);
	    } catch (NumberFormatException e) {
		System.err.println("Argument" + args[0] + " must be an integer.");
		System.exit(1);
	    }
	}
	else
	    n = 100;

	System.out.format("\n");
	System.out.format(" Prime numbers up to %d\n", n);
        SieveOfEratosthenes g = new SieveOfEratosthenes();
        g.sieveOfEratosthenes(n);
    }
}
~~~
{: .language-java}

We need to compile the Java code in what is called "bytecode". That is different from "machine code". The resulting file has the ".class" extension and is interpreted by the Java Virtual Machine. The same ".class" file can run exactly as it is on another machine.

Load the corresponding module for Java JDK and compile the sources as follows:

~~~
$ module load compilers/java/jdk1.8.0
$ javac SieveOfEratosthenes.java
~~~
{: .language-bash}

To execute, notice that the extension ".class" is not added on the command line. The java interpreter (JVM) assumes that extension

~~~
$ time java SieveOfEratosthenes 100000000
~~~
{: .language-bash}
~~~
 Prime numbers up to 100000000
 Dimension of array: 50000000
 Array with indices between 0 to 49999999
 Stores primality of odd numbers in range [1, 99999999]

 Total number of primes found: 5761455

real    0m1.137s
user    0m1.132s
sys     0m0.049s
~~~
{: .source}

### C 


{% include links.md %}
