## This file is heavily modified from the Makefile distributed by with
## ccx.

## pkg-config is used to pull all flags from local enviroment.  You
## must set this up.  Typically this would be defining
## PKG_CONFIG_PATH=$PKG_CONFIG_PATH:~/local/lib64/pkgconfig or
## similar.  You will also have to add .pc file for each library you
## use.  I did this because I have several machines with different
## library enviroments.  You could alternatively use compiler flags
## such as -L/path/to/lib -I/path/to/include -llib.  Examples of this
## on my distribution are in the comments below




################################################################################
################################################################################
##                                                                            ##
## Flags related to CPU, optimizations, etc                                   ##
## GPU based optimization later in the make file                              ##
##                                                                            ##
################################################################################
################################################################################

CCX_VERSION = 2.22
CCX_EXEC = ccx_$(CCX_VERSION)

CC=gcc
FC=gfortran

CFLAGS=-march=native -g -O0 ## debugging  -pg
FFLAGS=-march=native -g -O0 ## debugging  -pg
LDFLAGS += -g
# CFLAGS=-march=native -O2 -Wall ## conservative
# FFLAGS=-march=native -O2 -Wall ## conservative
## CFLAGS=-march=native -O3 -Wall ## -flto ## aggressive -fprofile-generate -fprofile-use
## FFLAGS=-march=native -O3 -Wall ## -flto ## aggressive -fprofile-generate -fprofile-use

## Integer8.  Note at least Arpack and Pardiso need to be compiled for
## Integer8 as well.  Spooles does not appear to be int8 according to Guido
## LONGLONG = -DLONGLONG
## CFLAGS  += $(LONGLONG)
## FFLAGS  += -fdefault-integer-8
## INTEXT = 64

## Other CCX Options
CFLAGS  += -DARCH="Linux" -DMATRIXSTORAGE 
LDFLAGS +=

## Multi Threaded and MPI
CFLAGS += -DUSE_MT
#CFLAGS += -DCALCULIX_MPI -fopenmp

## This is now default for calculix and relates to CFD
CFLAGS += -DNETWORKOUT

################################################################################
################################################################################
##                                                                            ##
## Flags related to CPU based solvers                                         ##
##                                                                            ##
################################################################################
################################################################################

## SPOOLES
CFLAGS  += -I/usr/include/spooles -I/usr/include/spooles/MT -DSPOOLES
LDFLAGS += -lspooles -lpthread
## CFLAGS  += `pkg-config --cflags spooles` -DSPOOLES
## LDFLAGS += `pkg-config --libs spooles`

## ARPACK
CFLAGS  += -I/usr/include/arpack -DARPACK
LDFLAGS += -larpack
## CFLAGS  += `pkg-config --cflags arpack$(INTEXT)` -DARPACK
## LDFLAGS += `pkg-config --libs arpack$(INTEXT)`

## TAUCS
## CFLAGS  += -DTAUCS
## LDFLAGS += -ltaucs -lmetis

## LAPACK
CFLAGS  += -I/usr/include/openblas
LDFLAGS += -lopenblas
## LDFLAGS += -lreflapack -lopenblas
## CFLAGS  += `pkg-config --cflags lapack$(INTEXT)`
## LDFLAGS += `pkg-config --libs lapack$(INTEXT)`

## BLAS
CFLAGS  += -I/usr/include/openblas
LDFLAGS += -lopenblas
## CFLAGS  += `pkg-config --cflags blas$(INTEXT)`
## LDFLAGS += `pkg-config --libs blas$(INTEXT)`

## PARDISO
CFLAGS  += -I/opt/intel/oneapi/mkl/2023.1.0/include -DPARDISO 
LDFLAGS += -L/opt/intel/oneapi/mkl/2023.1.0/lib/intel64 -Wl,--no-as-needed -lmkl_gf_lp64 -lmkl_gnu_thread -lmkl_core -lmkl_blacs_intelmpi_lp64 -lgomp -lpthread -lm -ldl
## CFLAGS  += `pkg-config --cflags pardiso` -DPARDISO
## LDFLAGS += `pkg-config --libs pardiso`

## PASTIX
CFLAGS  += -I/home/kalupaika/projects/pastix/include -DPASTIX
LDFLAGS += -L/home/kalupaika/projects/pastix/lib -lpastixf -lpastix -lpastix_kernels -lspmf -lspm -lhwloc -lscotch -lscotcherrexit -lmetis

################################################################################
################################################################################
##                                                                            ##
## Recipes                                                                    ##
##                                                                            ##
################################################################################
################################################################################

## .cu file so not have a default implicit rule.  Define all implicit rules used.
.SUFFIXES: .o .c .cu .f90 .f

.c.o :
	$(CC) $(CFLAGS) -c $<
.f.o :
	$(FC) $(FFLAGS) -c $<
.f90.o :
	$(FC) $(FFLAGS) -c $<
.cu.o:
	$(NVCC) -DCUDACUSP -c $<

include Makefile.inc

SCCXMAIN = CalculiX.c

## Define all the object file rules to identify dependencies
OCCXCU = $(SCCXCU:.cu=.o)
OCCXF = $(SCCXF:.f=.o)
OCCXF9 = $(SCCXF90:.f90=.o)
OCCXC = $(SCCXC:.c=.o)
OCCXMAIN = $(SCCXMAIN:.c=.o)

## Link to math and standard c
CFLAGS += -lm -lc

$(CCX_EXEC): $(OCCXMAIN) $(CCX_EXEC).a
	./date.pl; 
	$(CC) $(CFLAGS) -c $(SCCXMAIN) $(LDFLAGS); 
	$(FC)  -Wall -g -o $@ $(OCCXMAIN) $(CCX_EXEC).a $(LDFLAGS)

$(CCX_EXEC).a: $(OCCXF) $(OCCXC) $(OCCXF9)
	ar vr $@ $?

clean:
	rm -rf *.a *.o $(CCX_EXEC)*