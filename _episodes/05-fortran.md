---
title: "Fortran: Intensive Numerics"
teaching: 15
exercises: 15
questions:
- "Why to use Fortran if it is so old?"
objectives:
- "Learn the basics of Fortran and its long tradition in Numerical Computing"
keypoints:
- "The most important libraries for Matrix computation and Linear Algebra, BLAS and LAPACK were written in Fortran and they are still in used today. They are behind Numpy in Python and R can be compiled to use it too."
---

### Matrix inversion

The purpose here is not to show the algorithm behind matrix inversion but to show how that could be achieve in several programming languages using external libraries, in particular we will show you the problem solved using LAPACK for Fortran, GSL for C and Numpy for python

Lets start with the Fortran version. BLAS and LAPACK are a set of well known libraries to perform Linear Algebra calculations. This is a simple example of inverting a real matrix.

~~~
program inverse_matrix
  implicit none
  double precision, allocatable, dimension(:,:) :: a, ainv
  double precision, allocatable, dimension(:) :: work
  integer :: i,j, lwork

  integer :: info, lda,	m, n
  integer, allocatable, dimension(:) :: ipiv

  integer deallocatestatus
  character(len=15) :: mformat='(100(E14.6,1x))'

  external dgetrf
  external dgetri

  n = 4
  lda = n
  lwork = n*n
  allocate (a(lda,n))
  allocate (ainv(lda,n))
  allocate (work(lwork))
  allocate (ipiv(n))

  call random_seed()
  call random_number(a)

  print '(" ")'
  print*,"LU matrix:"  
  do i = 1, n
     write(*,mformat) (a(i,j), j = 1, n)
  end do

  print '(" ")'
  ! dgetrf computes an lu factorization of a general m-by-n matrix a
  ! using partial pivoting with row interchanges.

  m=n
  lda=n

  ! store a in ainv to prevent it from being overwritten by lapack
  ainv = a

  call dgetrf( m, n, ainv, lda, ipiv, info )

  if(info.eq.0)then
     print '(" LU decomposition successful ")'
  endif
  if(info.lt.0)then
     print '(" LU decomposition:  illegal value ")'
     stop
  endif
  if(info.gt.0)then
     write(*,'(a,i4)') 'LU decomposition return',info
  endif

  print '(" ")'
  print*,"LU matrix:"
  do i = 1, n
     write(*,mformat) (ainv(i,j), j = 1, n)
  end do

  !  dgetri computes the inverse of a matrix using the lu factorization
  !  computed by dgetrf.
  call dgetri(n, ainv, n, ipiv, work, lwork, info)

  print '(" ")'
  if (info.ne.0) then
     stop 'Matrix inversion failed!'
  else
     print '(" Inverse successful ")'
  endif

  print '(" ")'
  print*,"Inverse matrix:"
  do i = 1, n
     write(*,mformat)(ainv(i,j), j = 1, n)
  end do

  print '(" ")'

  deallocate (a, stat = deallocatestatus)
  deallocate (ainv, stat = deallocatestatus)
  deallocate (ipiv, stat = deallocatestatus)
  deallocate (work, stat = deallocatestatus)

  print '(" done ")'
  print '(" ")'

  stop
end program inverse_matrix
~~~
{: .source}

{% include links.md %}
