# Sierra Install using Intel oneAPI
* Sierra DevOps recommend Sierra v5.22 as the previous releases would need Intel classic instead of Intel oneAPI.
* They suggest trying to use the Spack packages `intel-oneapi-compilers` and `intel-oneapi-mpi` and `intel-oneapi-mkl`
* Will then need to set the following ENV variables and paths:
    - I_MPI_CC=
    - I_MPI_CXX=
    - I_MPI_F77=
    - I_MPI_F90=
    - MKLROOT= path where mkl is installed and shoudl be a lib directory underneath that path
* According to Sierra DevOps, v5.22.1 uses the following environment:
  - gcc@10.3.0
  - intel-oneapi-compilers@2024.1.0
  - intel-oneapi-mpi@2021.12.0
  - intel-oneapi-mkl@2024.1.0
* Resources:
  - https://spack.readthedocs.io/en/latest/build_systems/inteloneapipackage.html
  - https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/spack.html
  - https://www.intel.com/content/www/us/en/developer/articles/technical/distribution-of-optimized-hpc-binaries-using-spack.html

## Spack Environment Installation

### sierra522mod.yaml
```
# sierra522mod.yaml
# Spack Environment file
# Tests an install of packages needed to compile and run Sierra v5.22

spack:

  specs: 
    - intel-oneapi-compilers@2024.1.0
    - intel-oneapi-mpi@2021.12.1
    - intel-oneapi-mkl@2024.2.0+cluster

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


### Environment install
```
source /mnt/home/tvj/software/spack/share/spack/setup-env.sh
cd ~/Documents/
spack env create sierra522mod sierra522mod.yaml
spack env activate -p sierra522mod
spack concretize
spack install -j 16
spack env deactivate
spack env activate -p sierra522mod  #needed to set env variables correctly
```

### Environment variables
* explored `env` output to discover what the environment was setting
* added settings for the GCC@10.2.0 settings
* later, I plan to create a custom module... for now, I just set the environment manually via:

```
#
# Environment settings for Sierra 522 MOD
#
export PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/oclfpga/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/bin:/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin:$PATH
export LD_LIBRARY_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/oclfpga/host/linux64/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/compiler/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/opt/mpi/libfabric/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/lib:/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/lib64:$LD_LIBRARY_PATH
export CC=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icx
export CXX=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icpx
export FC=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export F77=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_ROOT=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12
export MKLROOT=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2
export OCL_ICD_FILENAMES=libintelocl_emu.so:libalteracl.so:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/lib/libintelocl.so
export FI_PROVIDER_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/opt/mpi/libfabric/lib/prov:/usr/lib64/libfabric
export I_MPI_CC=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icx
export I_MPI_CXX=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icpx
export I_MPI_F77=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_F90=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_OFI_PROVIDER=tcp
#
# possible other settings to try later
#
export CLASSPATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/share/java/mpi.jar
export CMPLR_ROOT=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1
export CMAKE_PREFIX_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/lib/cmake:.:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view
export CPATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/oclfpga/include:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/include:.:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/include
export NLSPATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/lib/compiler/locale/%l_%t/%N:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/share/locale/%l_%t/%N:.
export LIBRARY_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/lib:.:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/lib
export DIAGUTIL_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/etc/compiler/sys_check/sys_check.sh
```



## Testing Intel oneAPI
* installed `sierra522mod` environment as shown above
* Here are the MPI programs that were created.  Most of these are wrappers and can be read with a text editor.  It turns out that the classic `icc` compiler is no longer included.  It was therefore very important to set the I_MPI_CC variable to point to `icx`.  Otherwise, `mpiicc` and the like would fail.

```
[tvj@bernie bin]$ pwd
/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/bin

[tvj@bernie bin]$ ls
cpuinfo             IMB-MT        mpicc          mpifc    mpiicx
hydra_bstrap_proxy  IMB-NBC       mpicxx         mpigcc   mpiifort
hydra_nameserver    IMB-P2P       mpiexec        mpigxx   mpiifx
hydra_pmi_proxy     IMB-RMA       mpiexec.hydra  mpiicc   mpirun
IMB-MPI1            impi_cpuinfo  mpif77         mpiicpc  mpitune_fast
IMB-MPI1-GPU        impi_info     mpif90         mpiicpx
[tvj@bernie bin]$ 
```
* followed testing proceedures here: https://www.intel.com/content/www/us/en/docs/mpi-library/get-started-guide-linux/2021-14/overview.html

```
[tvj@bernie ~]$ which icx
~/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icx
```

```
[tvj@bernie ~]$ cat hostfile 
cnode001.bernie.cluster
cnode003.bernie.cluster
```

```
mpiicc -o myprog /mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/opt/mpi/test/test.c
```

```
[tvj@bernie ~]$ mpirun -n 2 -ppn 8 -f ./hostfile uptime
 12:16:21 up 75 days,  1:16,  0 users,  load average: 0.00, 0.00, 0.00
 12:16:21 up 75 days,  1:16,  0 users,  load average: 0.00, 0.00, 0.00
```

```
[tvj@bernie ~]$ mpirun -n 2 -ppn 8 -f ./hostfile ./myprog 
[1736788625.693157] [cnode001:521140:0]          select.c:630  UCX  ERROR   no active messages transport to <no debug data>: self/memory - Destination is unreachable
Abort(1614735) on node 0 (rank 0 in comm 0): Fatal error in PMPI_Init: Unknown error class, error stack:
MPIR_Init_thread(192)............: 
MPID_Init(1645)..................: 
MPIDI_OFI_mpi_init_hook(1703)....: 
insert_addr_table_roots_only(473): OFI get address vector map failed
```

> [!CAUTION]
> After much trial and error, I discovered that I needed to set `export I_MPI_OFI_PROVIDER=tcp` to get this to work.
> I still don't understand what is happening.  This seems to tell Intel to use ethernet instead of the Mellanox Infiniband.  More troubleshooting is needed.

```
[tvj@bernie ~]$ export I_MPI_OFI_PROVIDER=tcp
[tvj@bernie ~]$ mpirun -n 2 -ppn 8 -f ./hostfile ./myprog 
Hello world: rank 0 of 2 running on cnode001.bernie.cluster
Hello world: rank 1 of 2 running on cnode001.bernie.cluster
[tvj@bernie ~]$ mpirun -n 16 -ppn 8 -f ./hostfile ./myprog 
Hello world: rank 0 of 16 running on cnode001.bernie.cluster
Hello world: rank 1 of 16 running on cnode001.bernie.cluster
Hello world: rank 2 of 16 running on cnode001.bernie.cluster
Hello world: rank 3 of 16 running on cnode001.bernie.cluster
Hello world: rank 4 of 16 running on cnode001.bernie.cluster
Hello world: rank 5 of 16 running on cnode001.bernie.cluster
Hello world: rank 6 of 16 running on cnode001.bernie.cluster
Hello world: rank 7 of 16 running on cnode001.bernie.cluster
Hello world: rank 8 of 16 running on cnode003.bernie.cluster
Hello world: rank 9 of 16 running on cnode003.bernie.cluster
Hello world: rank 10 of 16 running on cnode003.bernie.cluster
Hello world: rank 11 of 16 running on cnode003.bernie.cluster
Hello world: rank 12 of 16 running on cnode003.bernie.cluster
Hello world: rank 13 of 16 running on cnode003.bernie.cluster
Hello world: rank 14 of 16 running on cnode003.bernie.cluster
Hello world: rank 15 of 16 running on cnode003.bernie.cluster
[tvj@bernie ~]$ 
```




## Compiling Sierra v5.22

> [!IMPORTANT]
> Initially, the trilinos third party library install failed with python header warnings.
> The solution was to first install the following package on the headnode:
> `sudo su`
> `yum install python36-devel.x86_64`

* manually set environment (to avoid Sierra's own spack install of TPLs):
```
export PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/oclfpga/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/bin:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/bin:/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/bin:$PATH
export LD_LIBRARY_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/oclfpga/host/linux64/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/opt/compiler/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/opt/mpi/libfabric/lib:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/lib:/mnt/home/tvj/software/spack/opt/spack/linux-rhel8-x86_64/gcc-8.5.0/gcc-10.2.0-tmw25cg7ifqdxosrybfowaalx25ol5ow/lib64:$LD_LIBRARY_PATH
export CC=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icx
export CXX=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icpx
export FC=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export F77=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_ROOT=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12
export MKLROOT=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mkl/2024.2
export OCL_ICD_FILENAMES=libintelocl_emu.so:libalteracl.so:/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/lib/libintelocl.so
export FI_PROVIDER_PATH=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/mpi/2021.12/opt/mpi/libfabric/lib/prov:/usr/lib64/libfabric
export I_MPI_CC=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icx
export I_MPI_CXX=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/icpx
export I_MPI_F77=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_F90=/mnt/home/tvj/software/spack/var/spack/environments/sierra522mod/.spack-env/view/compiler/2024.1/bin/ifx
```

* run installation script:
```
SIERRA_ROOT=/data/software/Sierra/5.22
cd ${SIERRA_ROOT}
/data/software/Sierra/5.22/5.22.1-base-source/source/sierra_setup.py --procs=40 --sierra-build-options=--verbose
```

* the installation produces an environment script that contains:
```
_SIERRA_INSTALL_DIR=/data/software/Sierra/5.22/install
export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH
export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH
```




## Troubleshooting Sierra v5.22







## Additional resources to explore
- [ ] Learn how to run the test and benchmark packages that come with the oneAPI installation
- [ ] https://www.intel.com/content/www/us/en/docs/mpi-library/developer-guide-linux/2021-11/job-schedulers-support.html
- [ ] https://www.intel.com/content/www/us/en/docs/mpi-library/developer-reference-windows/2021-8/mpiexec-hydra.html
- [ ] https://community.intel.com/t5/Intel-MPI-Library/Configuration-for-Intel-impi/m-p/1553070#M11253
- [ ] https://www.intel.com/content/www/us/en/developer/articles/technical/mpi-library-2019-over-libfabric.html
- [ ] https://www.intel.com/content/www/us/en/docs/mpi-library/developer-reference-linux/2021-8/ofi-capable-network-fabrics-control.html
- [ ] https://www.intel.com/content/www/us/en/developer/articles/technical/mpi-library-2019-over-libfabric.html
- [ ] https://www.intel.com/content/www/us/en/developer/articles/technical/improve-performance-and-stability-with-intel-mpi-library-on-infiniband.html
