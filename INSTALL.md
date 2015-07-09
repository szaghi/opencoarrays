INSTALLATION

See also the implementation status in the file STATUS

Introduction
------------

This file targets system administrators and developers who require access to
the entire OpenCoarrays distribution, including experimental libraries that
are intended only for advanced users.  Most end users will find it simpler 
to install the subset of the OpenCoarrays distribution that is included in  
the tar ball on http://www.opencoarrays.org.  Users can either access that
tar ball directly and build using CMake or Make or install the contents of
the tar ball using package management software such as Macports for OS X,
apt-get for Linux, or Cygwin for Windows.

The following documentation provides instructions on how to set up the coarray
support on GNU Fortran, using the OpenCoarrays library, and how to compile and
run a simple coarray program.

As prerequisite for using OpenCoarrays, gfortran from GCC 5 is required; see
below for how to obtain it.  The OpenCoarrays libraries can also build with
other compilers than GCC 5.


OpenCoarrays Installation
-------------------------

OpenCoarrays contains four libraries; if in doubt, use the MPI version:

* single
  Provides a stub library with a single image. To be used for testing or
  to turn a parallel program into a single-image program to avoid dependencies
  on other libraries.
  Requires only a C compiler.

* mpi - recommended version
  Communication library base on the Message Passing Interface (MPI); requires
  an MPI 3 implementation. (MPI 2 works with some restrictions.)
  MPICH, Open MPI, MVAPICH are common free implementations; vendor MPI
  implementations are typically based on either of them.  As MPI 3 is new and
  several MPI implementations have recently improved the one-sided
  communication (RMA, remote-memory access), consider using the newest release.
  We recommend to use MPICH or MVAPICH.

* gasnet
  Communication library based on GASNet (Global Address Space Network,
  http://gasnet.lbl.gov/). More difficult to setup and use but has typically a
  higher performance than MPI. Requires that the same compiler is used for both
  the compilation of GASNet itself and for the library.

* armci - unsupported/not working
  Communication library based on ARMCI (Aggregate Remote Memory Copy Interface,
  http://hpc.pnl.gov/armci/).  The libcaf_armci library is currently not
  supported in OpenCoarrays and uses an older ABI.
  Only included in OpenCoarrays to make it simpler for interested parties to
  implement a working libcaf_armci without starting from scratch.

Note:
* It is sufficient to compile only the library you are interested in.
* Any C compiler can be used. In particular, older GCC or different compilers
  work as well.	 However, for MPI ABI compatibility, using a recent GCC for
  compilation is preferred


Prerequisites
-------------

- Single: (none except for a C compiler)
- MPI: Installed MPI implementation with the "mpicc" wrapper either in the
  path
- GASNet: Installed GASNet; same C compiler as used for compiling GASNet;
  knowledge where GASNet has been installed.


Build the libraries with CMake (preferred)
------------------------------------------
Although CMake (http://www.cmake.org) is the preferred build system, we also 
provide a static Makefile inside the src source directory.  CMake is 
cross-platform and more automated than Make and therefore less work to maintain.
Our CMake scripts require build in a directory outside the source tree.  As an 
example, one might set the requisite environment variables and generate the
Makefiles inside a neighboring directory in the bash shell as follows:

mkdir Build
cd Build
CC=mpicc FC=mpif90 cmake ../opencoarrays/ -DCMAKE_INSTALL_PREFIX=${PWD}

whereupon the following commands respectively build the OpenCoarrays static
libraries libcaf_mpi.a and libcaf_single.a, install them in the present 
working inside a subdirectory named "lib", and run all tests:

make
make install
ctest

We recommend running the tests to ensure a correct installation.  The tests 
directory includes unit tests, integration tests (small applications), and 
performance tests.  Please report any test failures to opencoarrays@googlegroups.com.  

In order to work around compiler-specific bugs or missing language features, the
CMake set-up creates include files named "compiler_capabilities.txt"  into the source 
tree. Typing "make clean" inside the top level of your build directory will remove these.  

Advanced options (these affect only three tests and most users should not use them): 
  -DLEGACY_ARCHITECTURE=OFF enables the use of FFT libraries that employ AVX instructions
  -DHIGH_RESOLUTION_TIMER=ON enables timers that tick once per clock cycle

How to build OpenCoarrays with Make
-----------------------------------
To build with Make, use steps such as the following: 

cd src
make single mpi

which builds libcaf_single.a and libcaf_mpi.a.  Or drop the "single mpi" above
to build the default library (libcaf_mpi.a).

You might need to edit the "make.inc" file for your system settings, e.g.
you migt need to remove the "-Werror" option from the compile flags, set
a different compiler or similar.

For GASNet, set GASNET_MAK to point to your GASNet installation
with the conduit which should be used.

In order to activate efficient strided-array transfer support, uncomment
the -DSTRIDED flag inside the "make.inc" file.


Compiling with Make on CRAY:

* Rename "make.inc.Cray-XE" to "make.inc"
* Run: module swap PrgEnv-cray PrgEnv-gnu
* You may need to update the GASNet path and the used compiler for GASNet
  or change other settings
* Run "make" with the library which should be created

We currently don't have a "make install" option.  If you build with Make,
then install the libraries in the directly of your choice by simply copying
from where Make puts them in their respective source-tree subdirectory 
(src/single/libcaf_single.a, src/mpi/libcaf_mpi.a, etc.) to the location
of your choice.

Many of the tests have static Makefiles in their respective subdirectories
(e.g., src/tests/integration/coarrayHelloWorld/Makefile).  Please run several
tests to ensure verify correct installation and execution of the library.

How to compile and run a coarray program
----------------------------------------

The following explanation assumes the reader will use the OpenCoarrays MPI transport layer and
has installed it to the path /opt/opencarrays/, in which case there will be one subdirectory
/opt/opencoarrays/lib containing libcaf_mpi.a and another subdirectory /opt/opencoarrays/mod
containing a compiler generated module file.  Otherwise, substitute the appropriate path
for paths that appear below.

We originally OpenCoarrays for use with compilers that are capable of generating calls directly
to an OpenCoarrays transport layer (e.g., libcaf_mpi.a) automatically.  In response to a user
request, however, we now offer limited support for use with other compilers.  Both use cases
are described below.

1. Use with an OpenCoarrays-enabled compiler (e.g., GCC 5.0.1 or later):  
A hello.f90 program can compiled in a single step using the following
string:

  mpif90 -fcoarray=lib -L/opt/opencoarrays/ hello.f90 \
         -lcaf_mpi -o hello

For running the program:

  mpirun -np m ./hello

where m is the number of images you want to use.

2. Use with an non-OpenCoarrays-enabled compiler:  
If a user's compiler provides limited or no coarray support, the user can directly reference a
subset of the OpenCoarrays procedures and types through a Fortran module named "opencoararys"
in the "mod" subdirectory of the user's chosen OpenCoarrays installation path.  The module
provides wrappers for some basic Fortran 2008 parallel programming capabilities along with some 
more advanced features that have been proposed for Fortran 2015 in the draft Technical 
Specification TS18508 Additional Parallel Features in Fortran (visit www.opencoarrays.org for a
link to this document, which gets updated periodically).  An example of the use of the opencoarrays
Fortran module is in the tally_images_numbers.F90 file in the test suite.  The following commands
compile that program:

  cd src/tests/integration/extensions/
  mpif90 -L /opt/opencoarrays/lib/ -I /opt/opencoarrays/mod/ -fcoarray=lib tally_image_numbers.f90 -lcaf_mpi -o tally
  mpirun -np 8 ./tally

GNU Fortran Configuration
-------------------------

The coarray support on GFortran is available since version GCC 5.

GCC 5 can either be build as outlined below; however, there are also unofficial
binary builds https://gcc.gnu.org/wiki/GFortranBinaries ; some Linux
distributions already offer GCC 5 as preview.

For building GCC from source, first download GCC 5 - either as snapshot from
the mirror or using the latest version by using the SVN trunk/GIT master.
We provide quick instructions on how to build a minimal GCC version.
See also https://gcc.gnu.org/wiki/GFortranBinaries#FromSource and the
official and complete instructions at https://gcc.gnu.org/install/.

1) From the main directory create a build directory using:

   mkdir build

2) GCC requires various tools and packages like GMP, MPFR, MPC. These and others
   can be automatically downloaded typing the following command inside the main
   directory:

   ./contrib/download_prerequisites

3) Inside the build directory type the following configure string customizing
   the installation path:

   ../configure --prefix=/your/path --enable-languages=c,c++,fortran \
                --disable-multilib

4) Type

   make -jN

   where N is the number of cores on your machine + 1. (This may take a while).

5) Type:

   make install