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



## Misc. Install Notes:

### GCC environment error in Spack
- I submitted a bug report:  https://github.com/spack/spack/issues/47924
- The spack environment fails to set GCC env paths.  So these must be set manually:
   - append to PATH:  `/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin`
   - append to LD_LIBRATH_PATH:  `/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/lib64`
   - add `CC=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gcc`
   - add `CXX=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/g++`
   - add `F77=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran`
   - add `F90=/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin/gfortran`


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

