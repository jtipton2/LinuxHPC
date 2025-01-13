# Spack

## Install local instance of spack
```
git clone https://github.com/spack/spack.git
source /mnt/home/tvj/software/spack/share/spack/setup-env.sh 
echo $SPACK_ROOT
```

## Populate ~/.spack/packages.yaml with external installed packages
- Slurm was already installed by ITSD support.  A good potential resource for personal understanding:  https://github.com/SergioMEV/slurm-for-dummies
- added "buildable: false" to entries for slurm, openssl, openssh
- see: https://spack-tutorial.readthedocs.io/en/latest/tutorial_configuratoin.html#external-packages
- concretize will now show these packages as [e] meaning "externally installed already"
```
spack external find
spack external find slurm
```

## Install and add GCC compiler and garbage collect
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

## Create Spack environment to test Python 3.9
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

## Create Spack environment to test Open MPI 4.1
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


## Create Spack environment to test Sierra 5.20 binary
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