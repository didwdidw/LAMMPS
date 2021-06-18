Download latest stable version from https://lammps.sandia.gov/download.html.

$ tar xf lammps-stable.tar.gz
$ cd lammps-29Oct20/src
$ module load intel/2020u4
$ module load intelmpi/2020u4
$ cp MAKE/OPTIONS/Makefile.intel_cpu_intelmpi MAKE/MINE/Makefile.intel_cpu_intelmpi_O3
$ vim MAKE/MINE/Makefile.intel_cpu_intelmpi_O3

Modify the flags to enhance the performance and follow the competition requirements:
    OPTFLAGS =  -xHost -O2 -fp-model fast=2 -no-prec-div -qoverride-limits -qopt-zmm-usage=high
    -> OPTFLAGS =  -xCORE-AVX512 -O3 -fno-alias -fp-model fast=2 -no-prec-div -qoverride-limits -qopt-zmm-usage=high

    FFT_INC = -DFFT_MKL -DFFT_SINGLE
    -> FFT_INC = -DFFT_MKL

$ make clean-all
$ make no-all
$ make no-lib
$ make yes-manybody yes-molecule yes-replica yes-kspace yes-asphere yes-rigid yes-snap yes-user-omp yes-user-reaxc 
$ make yes-user-intel
$ make -j 40 intel_cpu_intelmpi_O3

After doing these, we now have a executable file "lmp_intel_cpu_intelmpi_O3".

To run the Lennard-Jones benchmark:
    Modify the input file first, add the following:
        newton off
        atom_modify sort 1250 0.0
        
    $ module load intel/2020u4
    $ module load intelmpi/2020u4
    $ srun -np 320 ../lammps-29Oct20/src/lmp_intel_cpu_intelmpi_O3 -sf intel -in ../inputs/in.lj -pk intel 0 omp 1 mode double


To run the Rhodopsin benchmark:
    $ cp inputs/data.rhodo $WORKDIR
    Modify the input file first, add the following:
        atom_modify sort 500 0.0
        
    $ module load intel/2020u4
    $ module load intelmpi/2020u4
    $ srun -np 320 ../lammps-29Oct20/src/lmp_intel_cpu_intelmpi_O3 -sf intel -in ../inputs/in.rhodo -pk intel 0 omp 1 mode double