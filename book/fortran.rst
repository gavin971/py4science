=======================================
 F2Py: using Fortran codes from Python
=======================================

While Python is a very flexible tool for numerical work and with Numpy arrays
under the hood is often fast enough for serious usage, there are many valid
reasons to ask for Fortran support.  You may need to use an existing library
written in Fortran, collaborate with a colleague who doesn't program in Python
or want to write a low-level set of routines in Fortran so they can benefit
other Fortran-using colleagues as well as being available to Python.  While
Numpy itself only uses C and Python, the Scipy package is built on a large
foundation of Fortran code, wrapping many well-tested, highly valuable
libraries such as LAPACK, FITPACK, ODRPACK and more.  It would be madness to
abandon the knowledge embedded in these tools, which have stood the test of
time and are still very useful for many purposes.  With Python, we can continue
to benefit from them while using them as part of a modern worfklow, thanks to
Numpy's f2py tool.

As we said, Numpy itself doesn't need Fortran at all, but it does provide the
f2py wrapper generator, so that by installing Numpy you automatically have the
tools needed to create Python wrappers for Fortran libraries (thus making it
possible to compile Scipy).  F2py was originally developed by Pearu Peterson;
here we will only present a brief tutorial introduction of its basic usage and
we refer our readers to the official documentation for complete details.

Below is a description intended for readers comfortable with the Python distutils
machinery, who simply want a quick guide to f2py.  A more tutorial description,
with examples, follows now.


A first f2py example: Fibonacci numbers
---------------------------------------

The Fibonacci numbers, defined by the recursion

.. math::

    f_i = f_{i-1} + f_i

can be computed with the following simple Fortran code:

.. sourcecode:: fortran

	 SUBROUTINE FIB(A,N)
   C
   C     CALCULATE FIRST N FIBONACCI NUMBERS
   C
	 INTEGER N
	 REAL*8 A(N)
	 DO I=1,N
	    IF (I.EQ.1) THEN
	       A(I) = 0.0D0
	    ELSEIF (I.EQ.2) THEN
	       A(I) = 1.0D0
	    ELSE
	       A(I) = A(I-1) + A(I-2)
	    ENDIF
	 ENDDO
	 END

If this code is in the ``fib.f`` file, then we can generate automatically the
sources for a Python module named ``example`` with::

    f2py fib.f -h fib.pyf -m example

The arguments to this call have the following meaning: ``-h fib.pyf`` gives the
name of the generated ``.pyf`` interface file, where ...

Usage for the impatient
-----------------------


Start by building a scratch signature file automatically from your Fortran
sources (in this case all, you can choose only those .f files you need)::

    f2py -m MODULENAME -h MODULENAME.pyf *.f

This writes the file ``MODULENAME.pyf``, making the best guesses it can from
the Fortran sources.  It builds an interface for the module to be accessed as
``import MODULENAME`` from python.

You can then edit the ``.pyf`` file to fine-tune the python interface exhibited
by the resulting extension.  This means for example making unnecessary scratch
areas or array dimensions hidden, or making certain parameters be optional and
take a default value.

Then, write your setup.py file using distutils, and list the .pyf file along
with the Fortran sources it is meant to wrap.  f2py will build the module for
you automatically, respecting all the interface specifications you made in the
.pyf file.

This approach is ultimately easier than trying to get all the declarations
(especially dependencies) right through Cf2py directives in the Fortran
sources.  While C2fpy directives may seem appealing at first, experience seems
to show that they are more time-consuming and prone to subtle errors.  Using
this approach, the first f2py pass can do the bulk of the interface writing and
only fine-tuning needs to be done manually.  I would only recommend embedded
Cf2py directives for very simple problems (where it works very well).

The only drawback of this approach is that the interface and the original
Fortran source lie in different files, which need to be kept in sync.  This
increases a bit the chances of forgetting to update the .pyf file if the
Fortran interface changes (adding a parameter, for example).  However, the
benefit of having explicit, clear control over f2py's behavior far outweighs
this concern.


Using Cf2py directives
----------------------

For simpler cases you may choose to go the route of Cf2py directives. Below
are some tips and examples for this approach.

Here's the signature of a simple Fortran routine:

.. sourcecode:: fortran

	subroutine phipol(j,mm,nodes,wei,nn,x,phi,wrk)

	implicit real *8 (a-h, o-z)
	real *8 nodes(*),wei(*),x(*),wrk(*),phi(*)
	real *8 sum, one, two, half

The above is correctly handled by f2py, but it can't know what is meant to be
input/output and what the relations between the various variables are (such as
integers which are array dimensions).  If we add the following f2py
directives, the generated python interface is a lot nicer:

.. sourcecode:: fortran

    subroutine phipol(j,mm,nodes,wei,nn,x,phi,wrk)
    c
    c       Lines with Cf2py in them are directives for f2py to generate a better
    c	python interface.  These must come _before_ the Fortran variable
    c       declarations so we can control the dimension of the arrays in Python.
    c
    c       Inputs:
    Cf2py   integer check(0<=j && j<mm),depend(mm) :: j
    Cf2py   real *8 dimension(mm),intent(in) :: nodes
    Cf2py   real *8 dimension(mm),intent(in) :: wei
    Cf2py   real *8 dimension(nn),intent(in) :: x
    c
    c       Outputs:
    Cf2py   real *8 dimension(nn),intent(out),depend(nn) :: phi
    c
    c       Hidden args:
    c       - scratch areas can be auto-generated by python
    Cf2py   real *8 dimension(2*mm+2),intent(hide,cache),depend(mm) :: wrk
    c       - array sizes can be auto-determined
    Cf2py   integer intent(hide),depend(x):: nn=len(x)
    Cf2py   integer intent(hide),depend(nodes) :: mm = len(nodes)
    c
    implicit real *8 (a-h, o-z)
    real *8 nodes(*),wei(*),x(*),wrk(*),phi(*)
    real *8 sum, one, two, half


The f2py directives should come immediately after the 'subroutine' line and
before the Fortran variable lines. This allows the f2py dimension directives to
override the Fortran var(*) directives.

If the Fortran code uses var(N) instead of var(*), the f2py directives can be
placed after the Fortran declarations.  This mode is preferred, as there is
less redundancy overall.  The result is much simpler:

.. sourcecode:: fortran

    subroutine phipol(j,mm,nodes,wei,nn,x,phi,wrk)
    c
    implicit real *8 (a-h, o-z)
    real *8 nodes(mm),wei(mm),x(nn),wrk(2*mm),phi(nn)
    real *8 sum, one, two, half
    c
    c       The Cf2py lines allow f2py to generate a better Python interface.
    c
    c       Inputs:
    Cf2py   integer check(0<=j && j<mm),depend(mm) :: j
    Cf2py   intent(in) :: nodes
    Cf2py   intent(in) :: wei
    Cf2py   intent(in) :: x
    c
    c       Outputs:
    Cf2py   intent(out) :: phi
    c
    c       Hidden args:
    c       - scratch areas can be auto-generated by python
    Cf2py   intent(hide,cache) :: wrk
    c       - array sizes can be auto-determined
    Cf2py   integer intent(hide),depend(x):: nn=len(x)
    Cf2py   integer intent(hide),depend(nodes) :: mm = len(nodes)


Since python can automatically manage memory, it is possible to hide the need
for manually passed 'work' areas.  The C/python wrapper to the underlying
fortran routine will allocate the memory for the needed work areas on the fly.
This is done by specifying intent(hide,cache).  'hide' tells f2py to remove the
variable from the argument list and 'cache' tells it to auto-generate it.

In cases where the allocation cost becomes a performance problem, one can
remove the 'hide' part and make it an optional argument.  In this case it will
only be generated if not given.  For this, the line above should be changed
to:

.. sourcecode:: fortran

    Cf2py   real *8 dimension(2*mm+2),intent(cache),optional,depend(mm) :: wrk

Note that this should only be done after _proving_ that the scratch areas are
causing a performance problem.  The 'cache' directive causes f2py to keep
cached copies of the scratch areas, so no unnecessary mallocs should be
triggered.

Since f2py relies on Numpy arrays, all dimensions can be determined from
the arrays themselves and it is not necessary to pass them explicitly.


With all this, the resulting f2py-generated docstring becomes::

    phipol - Function signature:
      phi = phipol(j,nodes,wei,x)
    Required arguments:
      j : input int
      nodes : input rank-1 array('d') with bounds (mm)
      wei : input rank-1 array('d') with bounds (mm)
      x : input rank-1 array('d') with bounds (nn)
    Return objects:
      phi : rank-1 array('d') with bounds (nn)


Debugging
---------

For debugging, use the ``--debug-capi`` option to f2py.  This causes the
extension modules to print detailed information while in operation.  In
distutils, this must be passed as an option in the f2py_options to the
Extension constructor.