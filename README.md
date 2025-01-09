# Sierra Installation on a HPC using Spack
>*Joseph B. Tipton, Jr.*
>*December 2024*



## Slurm
- Was already installed by ITSD support
- A good potential resource for personal understanding:  https://github.com/SergioMEV/slurm-for-dummies






## Spack

### Install local instance of spack
```
git clone https://github.com/spack/spack.git
source /mnt/home/tvj/software/spack/share/spack/setup-env.sh 
echo $SPACK_ROOT
```

### Populate ~/.spack/packages.yaml with external installed packages
- added "buildable: false" to entries for slurm, openssl, openssh
- see: https://spack-tutorial.readthedocs.io/en/latest/tutorial_configuratoin.html#external-packages
- concretize will now show these packages as [e] meaning "externally installed already"
```
spack external find
spack external find slurm
```

### Install and add GCC compiler and garbage collect
- setting the target of the compiler is important
- otherwise, GCC will be unuseable on the older compute nodes
- adds gcc to /mnt/home/tvj/.spack/linux/compilers.yaml
```
spack compilers
spack compiler find
spack install -j 16 gcc@10.2.0 target="x86_64"
spack compiler add "$(spack location -i gcc@10.2.0)"
spack compilers
spack find
spack gc
spack find
```

### Create Spack environment to test Python 3.9
- This was successful and allowed me to custom install pytetgen via local download and running `python setup.py install`
- I still don't know why pip could not install directly.  See: https://stackoverflow.com/q/79252073
- There are also good instructions on making your own spack installer
- see:  https://spack.readthedocs.io/en/latest/build_systems/pythonpackage.html
```
cd ~/Documents/
spack env create py39test py39test.yaml
spack env activate -p py39test
spack concretize
spack install -j 16
spack env deactivate
spack env activate -p py39test  #needed to set env variables correctly

cd /mnt/home/tvj/software/pytetgen
python setup.py install
```

**py39test.yaml**
```
# py39test.yaml
# Spack Environment file
# Tests an install of Python 3.9
spack:

  specs: 
    - python@3.9
    - py-numpy
    - py-cython
    - py-setuptools
    - py-pip

  view: true
 
  concretizer:
    unify: true
    reuse: false

  packages:
    all:
      require:
      - "%gcc@10.2.0"

  compilers::
    - compiler:
        spec: gcc@=10.2.0
        paths:
          cc: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gcc
          cxx: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/g++
          f77: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran
          fc: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran
        flags: {}
        operating_system: rhel8
        target: x86_64
        modules: []
        environment: {}
        extra_rpaths: []
```

### Create Spack environment to test Open MPI 4.1
- Note that Sierra recommends openmpi@4.0.5
- I could not get this to run without OPAL errors and might be a bug
- ompi 4.1 worked without issues
- Tests
    * `mpirun -np 1 hostname`
    * `mpicc -o mpi_hello_world ./mpi_hello_world`
    * `mpirun -np 2 ./mpi_hello_world`
    * `sbatch test.slurm`
    * `sbatch test_mpi.slurm`

```
cd ~/Documents/
spack env create ompi41v1 ompi41v1.yaml
spack env activate -p ompi41v1
spack concretize
spack install -j 24
spack env deactivate
spack env activate -p ompi41v1  #needed to set env variables correctly
```

**ompi41v1.yaml**
```
# ompi41v1.yaml
# Spack Environment file
# Tests an install of openmpi 4.1
spack:

  specs: 
    - openmpi@4.1+legacylaunchers

  view: true
 
  concretizer:
    unify: true
    reuse: false

  packages:
    all:
      require:
      - "%gcc@10.2.0"
      target:
      - "x86_64"

  compilers::
    - compiler:
        spec: gcc@=10.2.0
        paths:
          cc: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gcc
          cxx: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/g++
          f77: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran
          fc: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran
        flags: {}
        operating_system: rhel8
        target: x86_64
        modules: []
        environment: {}
        extra_rpaths: []
```

**mpi_hello_world.c**
```
//
// mpi_hello_world.c
// Author: Wes Kendall
// Copyright 2011 www.mpitutorial.com
// This code is provided freely with the tutorials on mpitutorial.com. Feel
// free to modify it for your own use. Any distribution of the code must
// either provide a link to www.mpitutorial.com or keep this header intact.
//
// An intro MPI hello world program that uses MPI_Init, MPI_Comm_size,
// MPI_Comm_rank, MPI_Finalize, and MPI_Get_processor_name.
//
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
  // Initialize the MPI environment. The two arguments to MPI Init are not
  // currently used by MPI implementations, but are there in case future
  // implementations might need the arguments.
  MPI_Init(NULL, NULL);

  // Get the number of processes
  int world_size;
  MPI_Comm_size(MPI_COMM_WORLD, &world_size);

  // Get the rank of the process
  int world_rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

  // Get the name of the processor
  char processor_name[MPI_MAX_PROCESSOR_NAME];
  int name_len;
  MPI_Get_processor_name(processor_name, &name_len);

  // Print off a hello world message
  printf("Hello world from processor %s, rank %d out of %d processors\n",
         processor_name, world_rank, world_size);

  // Finalize the MPI environment. No more MPI calls can be made after this
  MPI_Finalize();
}
```

**test.slurm**
```
#!/bin/bash

#SBATCH --job-name=test
#SBATCH --nodes=2
#SBATCH --ntasks=16
#SBATCH --time=00:01:00
#SBATCH --partition=slow
#SBATCH --mail-user=tvj@ornl.gov

#
#
#cd $SLURM_SUBMIT_DIR

#
# Load environment modules  
#
#module load mpi/openmpi-x86_64
#module load openmpi/5.0.5-gcc-8.5.0-uepfvox
source /mnt/home/tvj/software/spack/share/spack/setup-env.sh
echo $SPACK_ROOT
spack env activate ompi41t1


#
# Helpful diagnostics for performance troubleshooting
#
echo "======== HOSTNAMECTL ========="
hostnamectl

echo "======== HOST LSCPU ========="
lscpu

echo "======== SLURM JOB DIAGNOSTICS ========"
echo "SLURM_JOB_NUM_NODES is $SLURM_JOB_NUM_NODES"
echo "SLURM_JOB_NODELIST is $SLURM_JOB_NODELIST"
echo "SLURM_NTASKS is $SLURM_NTASKS"
echo "SLURM_TASKS_PER_NODE is $SLURM_TASKS_PER_NODE"
echo "SLURM_CPUS_ON_NODE is $SLURM_CPUS_ON_NODE"
echo "SLURM_JOB_CPUS_PER_NODE is $SLURM_JOB_CPUS_PER_NODE"
echo "SLURM_MEM_PER_NODE is $SLURM_MEM_PER_NODE"
echo "SLURM_MEM_PER_CPU is $SLURM_MEM_PER_CPU"

echo "======= PINGING EACH PROC TO REPORT HOSTNAME ========"
srun hostname

echo "======= PINGING EACH PROC TO REPORT UPTIME ========"
srun uptime
```

**test_mpi.slurm**
```
#!/bin/bash

#SBATCH --job-name=test
#SBATCH --nodes=2
#SBATCH --ntasks=16
#SBATCH --time=00:01:00
#SBATCH --partition=slow
#SBATCH --mail-user=tvj@ornl.gov
#
#
cd $SLURM_SUBMIT_DIR
echo "SLURM_SUBMIT_DIR is $SLURM_SUBMIT_DIR"

#
# Load environment modules on CADES (using GNU compilers)
#
#module load mpi/openmpi-x86_64
#module load openmpi/5.0.5-gcc-8.5.0-uepfvox
source /mnt/home/tvj/software/spack/share/spack/setup-env.sh
echo $SPACK_ROOT
spack env activate ompi41v1

#
# Compile the code
#make
mpicc -o mpi_hello_world ./mpi_hello_world.c

#
# Run the compiled code
# (note, I found all of these to be equivalent on CADES)

#mpirun ./mpi_hello_world
mpiexec ./mpi_hello_world
#srun --mpi=pmi2 ./mpi_hello_world
#srun ./mpi_hello_world
```


### Create Spack environment to test Sierra 5.20 binary
```
cd ~/Documents/
spack env create sierra520 sierra520.yaml
spack env activate -p sierra520
spack concretize
spack install -j 32
spack env deactivate
spack env activate -p sierra520  #needed to set env variables correctly

cd /mnt/home/tvj/software/pytetgen
python setup.py install
```

**sierra520.yaml**
```
# sierra520.yaml
# Spack Environment file
# Tests an install of packages needed to run Sierra workflow
spack:
  # add package specs to the `specs` list

  specs: 
    - openmpi@4.1+legacylaunchers
    - python@3.9
    - py-numpy
    - py-pandas
    - py-matplotlib
    - py-scikit-learn
    - py-cython
    - py-setuptools
    - py-pip
    - seacas

  view: true
 
  concretizer:
    unify: true
    reuse: false

  packages:
    all:
      require:
      - "%gcc@10.2.0"
      target:
      - "x86_64"

  compilers::
    - compiler:
        spec: gcc@=10.2.0
        paths:
          cc: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gcc
          cxx: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/g++
          f77: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran
          fc: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran
        flags: {}
        operating_system: rhel8
        target: x86_64
        modules: []
        environment: {}
        extra_rpaths: []
```






## Open MPI
These were really helpful resources:
- https://www.open-mpi.org/video/?category=general
- https://www.open-mpi.org/video/general/what-is-[open]-mpi-2up.pdf
- https://www.open-mpi.org/video/general/easybuild_tech_talks_01_OpenMPI_part1_20200623.pdf
- https://www.open-mpi.org/video/general/easybuild_tech_talks_01_OpenMPI_part2_20200708.pdf
- https://www.open-mpi.org/video/general/easybuild_tech_talks_01_OpenMPI_part3_20200805.pdf

```
[tvj@bernie ~]$ spack env activate -p sierra520
[sierra520] [tvj@bernie ~]$ ompi_info
                 Package: Open MPI tvj@bernie.sns.gov Distribution
                Open MPI: 4.1.7
  Open MPI repo revision: v4.1.7
   Open MPI release date: Oct 31, 2024
                Open RTE: 4.1.7
  Open RTE repo revision: v4.1.7
   Open RTE release date: Oct 31, 2024
                    OPAL: 4.1.7
      OPAL repo revision: v4.1.7
       OPAL release date: Oct 31, 2024
                 MPI API: 3.1.0
            Ident string: 4.1.7
                  Prefix: /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/openmpi-4.1.7-bj76src2q3j7rmw4x7m3eililaygmczd
 Configured architecture: x86_64-pc-linux-gnu
          Configure host: bernie.sns.gov
           Configured by: tvj
           Configured on: Fri Dec  6 13:45:07 UTC 2024
          Configure host: bernie.sns.gov
  Configure command line: '--prefix=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/openmpi-4.1.7-bj76src2q3j7rmw4x7m3eililaygmczd'
                          '--enable-shared' '--disable-silent-rules'
                          '--disable-sphinx' '--enable-builtin-atomics'
                          '--without-pmi' '--disable-static'
                          '--enable-mpi1-compatibility' '--without-ofi'
                          '--without-fca' '--without-ucc' '--without-mxm'
                          '--without-xpmem' '--without-cma' '--without-psm'
                          '--without-ucx' '--without-psm2' '--without-hcoll'
                          '--without-verbs' '--without-knem'
                          '--without-cray-xpmem' '--without-lsf'
                          '--without-sge' '--without-tm' '--with-slurm'
                          '--without-loadleveler' '--without-alps'
                          '--disable-memchecker'
                          '--with-libevent=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/libevent-2.1.12-ang7uo5zm2h7tqzd7xhqtb66qjkw6zvt'
                          '--with-pmix=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/pmix-5.0.3-azy3tx2erhymkbz4pjcxl44ba4h57ijf'
                          '--with-zlib=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/zlib-ng-2.2.1-hpb2hlkpbr7skqbidn6rtdb7p6zh3rgn'
                          '--with-hwloc=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/hwloc-2.11.1-udjpidphi4dsorjf5a6i76lpug4xecz5'
                          '--disable-java' '--disable-mpi-java'
                          '--with-gpfs=no' '--without-cuda'
                          '--enable-wrapper-rpath'
                          '--disable-wrapper-runpath' '--disable-mpi-cxx'
                          '--disable-cxx-exceptions'
                          '--with-wrapper-ldflags=-Wl,-rpath,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/lib/gcc/x86_64-pc-linux-gnu/10.2.0
                          -Wl,-rpath,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/lib64'
                          '--disable-debug'
                Built by: tvj
                Built on: Fri Dec  6 13:48:41 UTC 2024
              Built host: bernie.sns.gov
              C bindings: yes
            C++ bindings: no
             Fort mpif.h: yes (all)
            Fort use mpi: yes (full: ignore TKR)
       Fort use mpi size: deprecated-ompi-info-value
        Fort use mpi_f08: yes
 Fort mpi_f08 compliance: The mpi_f08 module is available, but due to
                          limitations in the
                          /mnt/home/tvj/software/spack/lib/spack/env/gcc/gfortran
                          compiler and/or Open MPI, does not support the
                          following: array subsections, direct passthru
                          (where possible) to underlying Open MPI's C
                          functionality
  Fort mpi_f08 subarrays: no
           Java bindings: no
  Wrapper compiler rpath: rpath
              C compiler: /mnt/home/tvj/software/spack/lib/spack/env/gcc/gcc
     C compiler absolute: 
  C compiler family name: GNU
      C compiler version: 10.2.0
            C++ compiler: /mnt/home/tvj/software/spack/lib/spack/env/gcc/g++
   C++ compiler absolute: none
           Fort compiler: /mnt/home/tvj/software/spack/lib/spack/env/gcc/gfortran
       Fort compiler abs: 
         Fort ignore TKR: yes (!GCC$ ATTRIBUTES NO_ARG_CHECK ::)
   Fort 08 assumed shape: yes
      Fort optional args: yes
          Fort INTERFACE: yes
    Fort ISO_FORTRAN_ENV: yes
       Fort STORAGE_SIZE: yes
      Fort BIND(C) (all): yes
      Fort ISO_C_BINDING: yes
 Fort SUBROUTINE BIND(C): yes
       Fort TYPE,BIND(C): yes
 Fort T,BIND(C,name="a"): yes
            Fort PRIVATE: yes
          Fort PROTECTED: yes
           Fort ABSTRACT: yes
       Fort ASYNCHRONOUS: yes
          Fort PROCEDURE: yes
         Fort USE...ONLY: yes
           Fort C_FUNLOC: yes
 Fort f08 using wrappers: yes
         Fort MPI_SIZEOF: yes
             C profiling: yes
           C++ profiling: no
   Fort mpif.h profiling: yes
  Fort use mpi profiling: yes
   Fort use mpi_f08 prof: yes
          C++ exceptions: no
          Thread support: posix (MPI_THREAD_MULTIPLE: yes, OPAL support: yes,
                          OMPI progress: no, ORTE progress: yes, Event lib:
                          yes)
           Sparse Groups: no
  Internal debug support: no
  MPI interface warnings: yes
     MPI parameter check: runtime
Memory profiling support: no
Memory debugging support: no
              dl support: yes
   Heterogeneous support: no
 mpirun default --prefix: no
       MPI_WTIME support: native
     Symbol vis. support: yes
   Host topology support: yes
            IPv6 support: no
      MPI1 compatibility: yes
          MPI extensions: affinity, cuda, pcollreq
   FT Checkpoint support: no (checkpoint thread: no)
   C/R Enabled Debugging: no
  MPI_MAX_PROCESSOR_NAME: 256
    MPI_MAX_ERROR_STRING: 256
     MPI_MAX_OBJECT_NAME: 64
        MPI_MAX_INFO_KEY: 36
        MPI_MAX_INFO_VAL: 256
       MPI_MAX_PORT_NAME: 1024
  MPI_MAX_DATAREP_STRING: 128
           MCA allocator: bucket (MCA v2.1.0, API v2.0.0, Component v4.1.7)
           MCA allocator: basic (MCA v2.1.0, API v2.0.0, Component v4.1.7)
           MCA backtrace: execinfo (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA btl: vader (MCA v2.1.0, API v3.1.0, Component v4.1.7)
                 MCA btl: tcp (MCA v2.1.0, API v3.1.0, Component v4.1.7)
                 MCA btl: self (MCA v2.1.0, API v3.1.0, Component v4.1.7)
            MCA compress: gzip (MCA v2.1.0, API v2.0.0, Component v4.1.7)
            MCA compress: bzip (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA crs: none (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                  MCA dl: dlopen (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA event: external (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA hwloc: external (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                  MCA if: linux_ipv6 (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
                  MCA if: posix_ipv4 (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
         MCA installdirs: env (MCA v2.1.0, API v2.0.0, Component v4.1.7)
         MCA installdirs: config (MCA v2.1.0, API v2.0.0, Component v4.1.7)
              MCA memory: patcher (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA mpool: hugepage (MCA v2.1.0, API v3.0.0, Component v4.1.7)
             MCA patcher: overwrite (MCA v2.1.0, API v1.0.0, Component
                          v4.1.7)
                MCA pmix: flux (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA pmix: ext3x (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA pmix: isolated (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA pstat: linux (MCA v2.1.0, API v2.0.0, Component v4.1.7)
              MCA rcache: grdma (MCA v2.1.0, API v3.3.0, Component v4.1.7)
           MCA reachable: weighted (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA shmem: sysv (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA shmem: posix (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA shmem: mmap (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA timer: linux (MCA v2.1.0, API v2.0.0, Component v4.1.7)
              MCA errmgr: default_hnp (MCA v2.1.0, API v3.0.0, Component
                          v4.1.7)
              MCA errmgr: default_tool (MCA v2.1.0, API v3.0.0, Component
                          v4.1.7)
              MCA errmgr: default_orted (MCA v2.1.0, API v3.0.0, Component
                          v4.1.7)
              MCA errmgr: default_app (MCA v2.1.0, API v3.0.0, Component
                          v4.1.7)
                 MCA ess: tool (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA ess: singleton (MCA v2.1.0, API v3.0.0, Component
                          v4.1.7)
                 MCA ess: slurm (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA ess: pmi (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA ess: env (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA ess: hnp (MCA v2.1.0, API v3.0.0, Component v4.1.7)
               MCA filem: raw (MCA v2.1.0, API v2.0.0, Component v4.1.7)
             MCA grpcomm: direct (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA iof: orted (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA iof: hnp (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA iof: tool (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA odls: pspawn (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA odls: default (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA oob: tcp (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA plm: rsh (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA plm: isolated (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA plm: slurm (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA ras: simulator (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
                 MCA ras: slurm (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA regx: reverse (MCA v2.1.0, API v1.0.0, Component v4.1.7)
                MCA regx: naive (MCA v2.1.0, API v1.0.0, Component v4.1.7)
                MCA regx: fwd (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA rmaps: resilient (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
               MCA rmaps: seq (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA rmaps: rank_file (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
               MCA rmaps: round_robin (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
               MCA rmaps: ppr (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA rmaps: mindist (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA rml: oob (MCA v2.1.0, API v3.0.0, Component v4.1.7)
              MCA routed: binomial (MCA v2.1.0, API v3.0.0, Component v4.1.7)
              MCA routed: direct (MCA v2.1.0, API v3.0.0, Component v4.1.7)
              MCA routed: radix (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA rtc: hwloc (MCA v2.1.0, API v1.0.0, Component v4.1.7)
              MCA schizo: slurm (MCA v2.1.0, API v1.0.0, Component v4.1.7)
              MCA schizo: flux (MCA v2.1.0, API v1.0.0, Component v4.1.7)
              MCA schizo: ompi (MCA v2.1.0, API v1.0.0, Component v4.1.7)
              MCA schizo: orte (MCA v2.1.0, API v1.0.0, Component v4.1.7)
              MCA schizo: jsm (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA state: app (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA state: tool (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA state: hnp (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA state: novm (MCA v2.1.0, API v1.0.0, Component v4.1.7)
               MCA state: orted (MCA v2.1.0, API v1.0.0, Component v4.1.7)
                 MCA bml: r2 (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: inter (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: libnbc (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: adapt (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: sync (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: basic (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: han (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: tuned (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: self (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: sm (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                MCA coll: monitoring (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
                MCA fbtl: posix (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA fcoll: individual (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
               MCA fcoll: dynamic (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA fcoll: two_phase (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
               MCA fcoll: vulcan (MCA v2.1.0, API v2.0.0, Component v4.1.7)
               MCA fcoll: dynamic_gen2 (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
                  MCA fs: ufs (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                  MCA io: ompio (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                  MCA io: romio321 (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                  MCA op: avx (MCA v2.1.0, API v1.0.0, Component v4.1.7)
                 MCA osc: rdma (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA osc: pt2pt (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA osc: sm (MCA v2.1.0, API v3.0.0, Component v4.1.7)
                 MCA osc: monitoring (MCA v2.1.0, API v3.0.0, Component
                          v4.1.7)
                 MCA pml: v (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA pml: cm (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA pml: monitoring (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
                 MCA pml: ob1 (MCA v2.1.0, API v2.0.0, Component v4.1.7)
                 MCA rte: orte (MCA v2.1.0, API v2.0.0, Component v4.1.7)
            MCA sharedfp: lockedfile (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
            MCA sharedfp: sm (MCA v2.1.0, API v2.0.0, Component v4.1.7)
            MCA sharedfp: individual (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
                MCA topo: basic (MCA v2.1.0, API v2.2.0, Component v4.1.7)
                MCA topo: treematch (MCA v2.1.0, API v2.2.0, Component
                          v4.1.7)
           MCA vprotocol: pessimist (MCA v2.1.0, API v2.0.0, Component
                          v4.1.7)
[sierra520] [tvj@bernie ~]$ 

```






## Sierra

### Sierra 5.20.1-base-binary-linux-gcc-10.2.0-openmpi-4.0.5
- Followed supplied `README` file
- Unpacked compressed file and placed in `~/software/sierra_5.20/binary`
- `SIERRA_ROOT=/mnt/home/tvj/software/sierra_5.20`
- `/mnt/home/tvj/software/sierra_5.20/binary/sierra_setup.py`

```
[sierra520] [tvj@bernie sierra_5.20]$ SIERRA_ROOT=/mnt/home/tvj/software/sierra_5.20
[sierra520] [tvj@bernie sierra_5.20]$ /mnt/home/tvj/software/sierra_5.20/binary/sierra_setup.py
REMARK: Extracting '/mnt/home/tvj/software/sierra_5.20/binary/sierra-base.tar.gz'
REMARK: Detected package type 'binary'
REMARK: Found no runtime libraries (.so) in lib directory hierarchy '/mnt/home/tvj/software/sierra_5.20/install/runtime/gcc/Compiler/gcc/lib'

************************************************************************
********************* Successfully installed SIERRA ********************
************************************************************************

To use it run the following command:
source /mnt/home/tvj/software/sierra_5.20/install/sierra_init.sh
```

- Note that the init script will try to set `LD_LIBRARY_PATH` and `PATH` to gcc/openmpi within the install directory but these do not exist.  We instead want to use the spack-controlled versions of gcc and openmpi.
- `_SIERRA_INSTALL_DIR=/mnt/home/tvj/software/sierra_5.20/install`
- `export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH`
- `export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH`
- `adagio --version`
```
[sierra520] [tvj@bernie sierra_5.20]$ which adagio
~/software/sierra_5.20/install/apps/bin/adagio
[sierra520] [tvj@bernie sierra_5.20]$ adagio --version
Application: Adagio
  Executable: /mnt/home/tvj/software/sierra_5.20/install/apps/bin/adagio version 5.20.1-0-g19779952 built on Jun 26 2024 12:02:53
  Build Options: linux gcc-10.2.0 release
    Product: ACME                     (2.9.0)
    Product: Adagio                   (5.20.1-0-g19779952)
    Product: FEI                      (2.25.00)
    Product: FETI-DP                  (5.20.1-0-g19779952)
    Product: GDSW                     (5.20.1-0-g19779952)
    Product: Linux                    (4.18.0-553.30.1.el8_10.x86_64)
    Product: SIERRA Framework         (5.20.1-0-g19779952)
    Product: Sandia Toolkit (STK)     (5.20.1-0-g19779952)
    Product: UtilityLib               (5.20.1-0-g19779952)
---------------------------------------------------
There were no errors encountered during execution of this procedure
There were no warnings encountered during execution of this procedure 
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
Execution complete


Timing summary of 1 processor
                 Timer                   Count       CPU Time                Wall Time
---------------------------------------- ----- --------------------- -------------------------
Sierra                                       1 00:00:00.116 (100.0%) 481531:07:20.421 (100.0%)

Took 2.69413e-05 seconds to generate the table above.


Memory summary of 1 processor
           Metric               Largest          Processor          Smallest         Processor
---------------------------- -------------- -------------------- -------------- --------------------
Dynamically Allocated (Heap)        19.2 MB on 0 at 00:00:00.000        19.2 MB on 0 at 00:00:00.000
Largest Free Fragment (Heap) 4.54483e-01 MB on 0 at 00:00:00.000 4.54483e-01 MB on 0 at 00:00:00.000
         Total Memory In Use       451.8 MB on 0                       451.8 MB on 0
           Major Page Faults       727 flts on 0                       727 flts on 0


Performance metric summary
---------------------------------------------------
Min High-water memory usage 74.0 MB
Avg High-water memory usage 74.0 MB
Max High-water memory usage 74.0 MB

Min Available memory per processor 4024.8 MB
Avg Available memory per processor 4024.8 MB
Max Available memory per processor 4024.8 MB

Min No-output time 0.1160 sec
Avg No-output time 0.1160 sec
Max No-output time 0.1160 sec
---------------------------------------------------
There were no errors encountered during parse
There were no warnings encountered during parse
There were no errors encountered during execution
There were no warnings encountered during execution
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
SIERRA execution successful after 00:00:00 (HH:MM:SS)
[sierra520] [tvj@bernie sierra_5.20]$ 
```


### Important Note
* To avoid installing as `root`, I changed the folder ownership
* When I'm done I need to do `chown -R root:bernie-fem-users  blahdblah`


### Sierra v5.14 Third Party Libraries

```
[tvj@bernie ~]$ cd /data/software/Sierra/5.14_gcc10.2.0/build/distro/code/TPLs_src/spack/spack_tpls
[tvj@bernie spack_tpls]$ ll
total 10
drwxr-x---. 2 tvj users 3 Mar 27  2023 boost
drwxr-x---. 2 tvj users 3 Mar 27  2023 bzip2
drwxr-x---. 2 tvj users 3 May 21  2023 cdt
drwxr-x---. 2 tvj users 3 May 21  2023 cmake
drwxr-x---. 2 tvj users 3 May 28  2023 hdf5
drwxr-x---. 2 tvj users 3 Mar 27  2023 matio
drwxr-x---. 2 tvj users 3 Mar 27  2023 metis
drwxr-x---. 2 tvj users 3 Mar 27  2023 netcdf-c
drwxr-x---. 2 tvj users 3 Mar 27  2023 netlib-lapack
drwxr-x---. 2 tvj users 3 May 21  2023 ninja
drwxr-x---. 2 tvj users 3 Mar 27  2023 parallel-netcdf
drwxr-x---. 2 tvj users 3 Mar 27  2023 parmetis
drwxr-x---. 2 tvj users 3 Mar 27  2023 patchelf
drwxr-x---. 2 tvj users 3 Mar 27  2023 scotch
drwxr-x---. 2 tvj users 3 Mar 27  2023 superlu
drwxr-x---. 2 tvj users 3 May 28  2023 trilinos
drwxr-x---. 2 tvj users 3 Mar 27  2023 umfpack
drwxr-x---. 2 tvj users 3 Mar 27  2023 y12m
drwxr-x---. 2 tvj users 3 Mar 27  2023 yaml-cpp
drwxr-x---. 2 tvj users 3 Mar 27  2023 zlib


-rw-r-----. 1 tvj users 130620992 Mar 27  2023 boost-1.77.0.tar.gz
-rw-r-----. 1 tvj users 815486 Mar 27  2023 bzip2-1.0.8.tar.gz
-rw-r-----. 1 tvj users 1126185 May 21  2023 cdt-oldmaster.tar.gz
-rw-r-----. 1 tvj users 11004923 May 21  2023 cmake-3.25.2.tar.gz  <<<---***
-rw-r-----. 1 tvj users 13058029 May 28  2023 hdf5-1.10.7.tar.gz  <<<---***
-rw-r-----. 1 tvj users 8535070 Mar 27  2023 matio-1.5.9.tar.gz  <<<---***
-rw-r-----. 1 tvj users 293187 Mar 27  2023 metis-5.1.0.tar.gz
-rw-r-----. 1 tvj users 5999520 Mar 27  2023 netcdf-c-4.7.4.tar.gz  <<<---***
-rw-r-----. 1 tvj users 7534567 Mar 27  2023 netlib-lapack-3.9.0.tar.gz
-rw-r-----. 1 tvj users 229479 May 21  2023 ninja-1.11.1.tar.gz  <<<---***
-rw-r-----. 1 tvj users 2549195 Mar 27  2023 parallel-netcdf-1.12.1.tar.gz  <<<---***
-rw-r-----. 1 tvj users 5293797 Mar 27  2023 parmetis-4.0.3.tar.gz
-rw-r-----. 1 tvj users 188702 Mar 27  2023 patchelf-0.10.tar.gz
-rw-r-----. 1 tvj users 4804966 Mar 27  2023 scotch-6.0.4.tar.gz
-rw-r-----. 1 tvj users 2561295 Mar 27  2023 superlu-5.2.1.tar.gz
-rw-r-----. 1 tvj users 181954955 May 28  2023 trilinos-2023.04.20.tar.gz
-rw-r-----. 1 tvj users 1409440 Mar 27  2023 umfpack-5.1.0.tar.gz
-rw-r-----. 1 tvj users 178055 Mar 27  2023 y12m-1.0.0.tar.gz
-rw-r-----. 1 tvj users 3308674 Mar 27  2023 yaml-cpp-0.6.3.tar.gz
-rw-r-----. 1 tvj users 821813 Mar 27  2023 zlib-2.1.0.devel.tar.gz
```

### Sierra v5.14 Compile
* The problem appears to be that Sierra uses its own local Spack to install third party libraries and report back to the program compiling Sierra.  So I can't start it from with my own Spack environment.  They are working on using Spack for everything in the future.
* For now, we tried deactivating spack and manually setting environment paths and variables:
* Had an issue with error saying compiler was not C++11 standard compliant.  That is not true.  The problem turned out to be because the clocks on the headnode and storage server were not in sync.
* The next issue was with HDF5.  A newer version was already installed on the system for SEACAS.  The newer version used different number of arguments and caused the errors.
* I then tried to set specific versions of the TPLs in my Spack YAML file.  But there were lots of concretization conflicts with SEACAS.  The problem is that SEACAS is newer.
* So I ended up creating a `sierra514` environment that just has GCC and OpenMPI.  I then will have a `pysierra` environment that has all the python modules and pytetgen and SEACAS.

```
export PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/bin:/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin:$PATH
export OMPI_CC=gcc OMPI_CXX=g++ OMPI_FC=gfortran
export LD_LIBRARY_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/lib
export CC=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gcc
export CXX=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/g++
export MPIF90=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/bin/mpif90
export MPICXX=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/bin/mpic++
export MPICC=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/bin/mpicc
export MPIFC=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/bin/mpifort
export MPIF77=/mnt/home/tvj/software/spack/var/spack/environments/sierra514/.spack-env/view/bin/mpif77

SIERRA_ROOT=/data/software/Sierra/5.14_gcc10.2.0

cd ${SIERRA_ROOT}

./sierra_setup.py --procs 40
```

* This creates an init script `source /data/software/Sierra/5.14_gcc10.2.0/install/sierra_init.sh` that generates the following:

```
_SIERRA_INSTALL_DIR=/data/software/Sierra/5.14_gcc10.2.0/install
export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH
export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH
```


### Sierra Optimization
* try running `spack arch` to deduce the architecture or `spack debug report`
    - headnode is `linux-rhel8-sapphirerapids`
    - compute node is `linux-rhel8-ivybridge`
* EX:  `spack config add concretizer:targets:host_compatible:false`
* EX:  `spack install mpileaks@1.1.2 %gcc@4.7.3 cppflags="-O3 -floop-block"`
* https://stackoverflow.com/a/10647460/28628026
* https://spack.readthedocs.io/en/latest/getting_started.html#manual-compiler-configuration
* https://groups.google.com/g/spack/c/2cExxjIvuOI
* https://www.chpc.utah.edu/documentation/software/single-executable.php
* https://groups.google.com/g/spack/c/aFsGIhsDKCg/m/OtzR__unCgAJ
* Maybe try to pass flags via: `./sierra_setup.py --procs 20 --sierra-build-options=--verbose`



### Sierra Intel Compile
* For this they recommend Sierra v5.22 as the previous releases would need Intel classic instead of Intel oneAPI.
* They suggest trying to use the Spack packages `intel-oneapi-compilers` and `intel-oneapi-mpi` and `intel-oneapi-mkl`
* Will then need to set the following ENV variables and paths:
    - I_MPI_CC=
    - I_MPI_CXX=
    - I_MPI_F77=
    - I_MPI_F90=
    - MKLROOT= path where mkl is installed and shoudl be a lib directory underneath that path
* https://spack.readthedocs.io/en/latest/build_systems/inteloneapipackage.html
* https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/spack.html
* https://www.intel.com/content/www/us/en/developer/articles/technical/distribution-of-optimized-hpc-binaries-using-spack.html
* Here is what Sierra team uses on their modules:

![Sandia Sierra Module Settings - A](sandia_sierra_intel_paths_a.png)
![Sandia Sierra Module Settings - B](sandia_sierra_intel_paths_b.png)












## Misc. Install Notes:

### gcc@10.2.0 on the Compute Nodes
- It turns out that the gcc@10.2.0 that was installed didn't work on the architecture of the compute nodes
- I think I need to instead use:  spack install gcc@10.2.0 target="x86_64"
- I also learned that `mpicc` is just a wrapper for `gcc` and can be viewed in detail via `mpicc --showme mpi_hello_world.c`
- see:  https://docs.open-mpi.org/en/main/man-openmpi/man1/ompi-wrapper-compiler.1.html
```
[tvj@cnode001 Documents]$ ~/software/spack/var/spack/environments/ompi41t1/.spack-env/view/bin/mpicc
[tvj@cnode001 Documents]$ ~/software/spack/var/spack/environments/ompi41t1/.spack-env/view/bin/mpicc --version
[tvj@cnode001 Documents]$ mpicc --showme mpi_hello_world.c 
/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-icelake/gcc-8.5.0/gcc-10.2.0-xdrxrw6qdcaqljdci3lj23su6ayg5g5u/bin/gcc mpi_hello_world.c -I/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/openmpi-4.1.7-bj76src2q3j7rmw4x7m3eililaygmczd/include -pthread -L/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/openmpi-4.1.7-bj76src2q3j7rmw4x7m3eililaygmczd/lib -L/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/hwloc-2.11.1-udjpidphi4dsorjf5a6i76lpug4xecz5/lib -L/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/libevent-2.1.12-ang7uo5zm2h7tqzd7xhqtb66qjkw6zvt/lib -Wl,-rpath,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-icelake/gcc-8.5.0/gcc-10.2.0-xdrxrw6qdcaqljdci3lj23su6ayg5g5u/lib/gcc/x86_64-pc-linux-gnu/10.2.0 -Wl,-rpath,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-icelake/gcc-8.5.0/gcc-10.2.0-xdrxrw6qdcaqljdci3lj23su6ayg5g5u/lib64 -Wl,-rpath -Wl,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/openmpi-4.1.7-bj76src2q3j7rmw4x7m3eililaygmczd/lib -Wl,-rpath -Wl,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/hwloc-2.11.1-udjpidphi4dsorjf5a6i76lpug4xecz5/lib -Wl,-rpath -Wl,/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-10.2.0/libevent-2.1.12-ang7uo5zm2h7tqzd7xhqtb66qjkw6zvt/lib -lmpi
[tvj@cnode001 Documents]$ gcc --version
gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-22)
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[tvj@cnode001 Documents]$ which gcc
/usr/bin/gcc
[tvj@cnode001 Documents]$ /mnt/home/tvj/software/spack/opt/spack/linux-rhel8-icelake/gcc-8.5.0/gcc-10.2.0-xdrxrw6qdcaqljdci3lj23su6ayg5g5u/bin/gcc --version
Illegal instruction (core dumped)
[tvj@cnode001 Documents]$ 
```

### pytetgen
- Tried to install pytetgen manually - SUCCESS!! - using python@3.9
- I had to do this manually
```
[py39test] [tvj@bernie Documents]$ cd ../software/pytetgen/
[py39test] [tvj@bernie pytetgen]$ ll
total 60
drwxr-xr-x. 3 tvj users     4 Dec  4 11:44 docs
-rw-r--r--. 1 tvj users 34523 Dec  4 11:44 LICENSE.txt
-rw-r--r--. 1 tvj users   166 Dec  4 11:44 MANIFEST.in
drwxr-xr-x. 3 tvj users     7 Dec  4 11:44 pytetgen
-rw-r--r--. 1 tvj users  2804 Dec  4 11:44 README.rst
-rw-r--r--. 1 tvj users    19 Dec  4 11:44 requirements.txt
-rw-r--r--. 1 tvj users  2215 Dec  4 11:45 setup.py
[py39test] [tvj@bernie pytetgen]$ python setup.py install
/mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/setuptools/_distutils/extension.py:134: UserWarning: Unknown Extension options: 'compiler_directives'
  warnings.warn(msg)
/mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/setuptools/_distutils/cmd.py:66: SetuptoolsDeprecationWarning: setup.py install is deprecated.
!!

        ********************************************************************************
        Please avoid running ``setup.py`` directly.
        Instead, use pypa/build, pypa/installer or other
        standards-based tools.

        See https://blog.ganssle.io/articles/2021/10/setup-py-deprecated.html for details.
        ********************************************************************************

!!
  self.initialize_options()
/mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/setuptools/_distutils/cmd.py:66: EasyInstallDeprecationWarning: easy_install command is deprecated.
!!

        ********************************************************************************
        Please avoid running ``setup.py`` and ``easy_install``.
        Instead, use pypa/build, pypa/installer or other
        standards-based tools.

        See https://github.com/pypa/setuptools/issues/917 for details.
        ********************************************************************************

!!
  self.initialize_options()
warning: no files found matching 'pytetgen/pytetgen.cpp'
Compiling pytetgen/pytetgen.pyx because it changed.
[1/1] Cythonizing pytetgen/pytetgen.pyx
warning: pytetgen/pytetgen.pyx:90:12: Unreachable code
In file included from /mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/numpy/_core/include/numpy/ndarraytypes.h:1909,
                 from /mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/numpy/_core/include/numpy/ndarrayobject.h:12,
                 from /mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/numpy/_core/include/numpy/arrayobject.h:5,
                 from pytetgen/pytetgen.cpp:1271:
/mnt/home/tvj/software/spack/var/spack/environments/py39test/.spack-env/view/lib/python3.9/site-packages/numpy/_core/include/numpy/npy_1_7_deprecated_api.h:17:2: warning: #warning "Using deprecated NumPy API, disable it with " "#define NPY_NO_DEPRECATED_API NPY_1_7_API_VERSION" [-Wcpp]
   17 | #warning "Using deprecated NumPy API, disable it with " \
      |  ^~~~~~~
pytetgen/tetgen.cxx: In member function 'bool tetgenbehavior::parse_commandline(int, char**)':
pytetgen/tetgen.cxx:3144:53: warning: passing argument 2 to 'restrict'-qualified parameter aliases with argument 1 [-Wrestrict]
 3144 |           brio_threshold = (int) strtol(workstring, (char **) &workstring, 0);
      |                                         ~~~~~~~~~~  ^~~~~~~~~~~~~~~~~~~~~
pytetgen/tetgen.cxx:3173:54: warning: passing argument 2 to 'restrict'-qualified parameter aliases with argument 1 [-Wrestrict]
 3173 |             hilbert_limit = (int) strtol(workstring, (char **) &workstring, 0);
      |                                          ~~~~~~~~~~  ^~~~~~~~~~~~~~~~~~~~~
pytetgen/tetgen.cxx: In member function 'int tetgenmesh::add_steinerpt_in_segment(tetgenmesh::face*, int)':
pytetgen/tetgen.cxx:19171:7: warning: variable 'success' set but not used [-Wunused-but-set-variable]
19171 |   int success;
      |       ^~~~~~~
zip_safe flag not set; analyzing archive contents...
__pycache__.pytetgen.cpython-39: module references __file__
```




