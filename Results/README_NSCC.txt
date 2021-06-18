Download latest stable version from https://lammps.sandia.gov/download.html.

$ tar xf lammps-stable.tar.gz
$ cd lammps-29Oct20/src

We have installed our own Open MPI and CUDA package
Load the essential environmental variable for them:

For CUDA: 
$ export PATH=$HOME/scratch/packages/cuda-10.0/bin:$PATH
$ export LD_LIBRARY_PATH=/usr/lib64:$HOME/scratch/packages/cuda-10.0/lib64:$LD_LIBRARY_PATH
$ export INCLDUE=$HOME/scratch/packages/cuda-10.0/include:$INCLUDE

For Open MPI:
$ export PATH=/home/users/industry/isc2020/iscst07/scratch/didwdidw/opt/openmpi-4.1.1/bin:$PATH
$ export LD_LIBRARY_PATH=/home/users/industry/isc2020/iscst07/scratch/didwdidw/opt/openmpi-4.1.1/lib:$LD_LIBRARY_PATH
$ export INCLUDE=/home/users/industry/isc2020/iscst07/scratch/didwdidw/opt/openmpi-4.1.1/include:$INCLUDE

For using kokkos package:
$ export KOKKOS_PATH=/home/users/industry/isc2020/iscst07/scratch/didwdidw/lammps-29Oct20/lib/kokkos

$ cp MAKE/OPTIONS/Makefile.kokkos_cuda_mpi MAKE/MINE/Makefile.cuda100Opt
$ vim MAKE/MINE/Makefile.cuda100Opt

Base on the environment, modify MAKE/OPTIONS/Makefile.kokkos_cuda_mpi
Also, add additional flags to enhance the performance:
    export MPICH_CXX = $(KOKKOS_ABSOLUTE_PATH)/bin/nvcc_wrapper
    -> #export MPICH_CXX = $(KOKKOS_ABSOLUTE_PATH)/bin/nvcc_wrapper

    CCFLAGS =   -g -O3
    -> CCFLAGS =   -g -O3 -std=c++14 --default-stream per-thread

    LINKFLAGS = -g -O3
    -> LINKFLAGS = -g -O3 -std=c++14

    KOKKOS_ARCH = Kepler35
    -> KOKKOS_ARCH = BDW, VOLTA70
       KOKKOS_CUDA_OPTIONS = "enable_lambda"

$ make clean-all
$ make no-all
$ make no-lib
$ make yes-manybody yes-molecule yes-replica yes-kspace yes-asphere yes-rigid yes-snap yes-user-omp yes-user-reaxc 
$ make yes-kokkos
$ make -j 24 cuda100Opt

After doing these, we now have a executable file "lmp_cuda100Opt".

To run the Lennard-Jones benchmark:
    Export essential environmental variables for CUDA and Open MPI
    $ mpirun -np 20 ../lammps-29Oct20/src/lmp_cuda100Opt -k on g 4 -sf kk -pk kokkos -in ../inputs/in.lj


To run the Rhodopsin benchmark:
    $ cp inputs/data.rhodo $WORKDIR
    Modify in.rhodo first: Add suffix "/kk/host" behind bond_style and improper_style
    Export essential environmental variables for CUDA and Open MPI 
    $ mpirun -np 8 ../lammps-29Oct20/src/lmp_cuda100Opt -k on g 4 -sf kk -pk kokkos -in ../inputs/in.rhodo