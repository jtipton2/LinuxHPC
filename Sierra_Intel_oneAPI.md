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

```
[tvj@bernie ~]$ spack env activate -p sierra522mod

[sierra522mod] [tvj@bernie ~]$ spack find
==> In environment sierra522mod
==> 3 root specs
[+] intel-oneapi-compilers@2024.1.0  [+] intel-oneapi-mkl@2024.2.0 +cluster  [+] intel-oneapi-mpi@2021.12.1

-- linux-rhel8-x86_64 / gcc@10.2.0 ------------------------------
cmake@3.26.5        gmake@4.2.1                      intel-oneapi-mkl@2024.2.0   libiconv@1.17      ncurses@6.5      util-macros@1.20.1
gcc-runtime@10.2.0  hwloc@2.11.1                     intel-oneapi-mpi@2021.12.1  libpciaccess@0.17  patchelf@0.17.2  xz@5.4.6
glibc@2.28          intel-oneapi-compilers@2024.1.0  intel-tbb@2021.12.0         libxml2@2.13.4     pkgconf@1.4.2    zlib-ng@2.2.1
==> 18 installed packages
==> 0 concretized packages to be installed (show with `spack find -c`)

[sierra522mod] [tvj@bernie ~]$ spack spec
[+]  intel-oneapi-compilers@2024.1.0%gcc@10.2.0~amd+envmods~nvidia build_system=generic arch=linux-rhel8-x86_64
[+]      ^gcc-runtime@10.2.0%gcc@10.2.0 build_system=generic arch=linux-rhel8-x86_64
[e]      ^glibc@2.28%gcc@10.2.0 build_system=autotools arch=linux-rhel8-x86_64
[+]      ^patchelf@0.17.2%gcc@10.2.0 build_system=autotools arch=linux-rhel8-x86_64
[e]          ^gmake@4.2.1%gcc@10.2.0~guile build_system=generic patches=ca60bd9,fe5b60d arch=linux-rhel8-x86_64
[+]  intel-oneapi-mkl@2024.2.0%gcc@10.2.0+cluster+envmods~gfortran~ilp64+shared build_system=generic mpi_family=mpich threads=none arch=linux-rhel8-x86_64
[+]      ^intel-tbb@2021.12.0%gcc@10.2.0~ipo+shared+tm build_system=cmake build_type=Release cxxstd=default generator=make arch=linux-rhel8-x86_64
[e]          ^cmake@3.26.5%gcc@10.2.0~doc+ncurses+ownlibs~qtgui build_system=generic build_type=Release patches=dbc3892 arch=linux-rhel8-x86_64
[+]          ^hwloc@2.11.1%gcc@10.2.0~cairo~cuda~gl~libudev+libxml2~nvml~oneapi-level-zero~opencl+pci~rocm build_system=autotools libs=shared,static arch=linux-rhel8-x86_64
[+]              ^libpciaccess@0.17%gcc@10.2.0 build_system=autotools arch=linux-rhel8-x86_64
[+]                  ^util-macros@1.20.1%gcc@10.2.0 build_system=autotools arch=linux-rhel8-x86_64
[+]              ^libxml2@2.13.4%gcc@10.2.0+pic~python+shared build_system=autotools arch=linux-rhel8-x86_64
[+]                  ^libiconv@1.17%gcc@10.2.0 build_system=autotools libs=shared,static arch=linux-rhel8-x86_64
[+]                  ^xz@5.4.6%gcc@10.2.0~pic build_system=autotools libs=shared,static arch=linux-rhel8-x86_64
[+]                  ^zlib-ng@2.2.1%gcc@10.2.0+compat+new_strategies+opt+pic+shared build_system=autotools arch=linux-rhel8-x86_64
[+]              ^ncurses@6.5%gcc@10.2.0~symlinks+termlib abi=none build_system=autotools patches=7a351bc arch=linux-rhel8-x86_64
[e]              ^pkgconf@1.4.2%gcc@10.2.0 build_system=autotools arch=linux-rhel8-x86_64
[+]  intel-oneapi-mpi@2021.12.1%gcc@10.2.0~classic-names+envmods~external-libfabric~generic-names~ilp64 build_system=generic arch=linux-rhel8-x86_64
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
#
# not sure how/if these settings are important for my use case...
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

* simple testing proceedures are located here: https://www.intel.com/content/www/us/en/docs/mpi-library/get-started-guide-linux/2021-14/overview.html

> [!TIP]
> Reading through this guide has been extremely helpful:
> Intel® MPI Library Developer Guide for Linux* OS
> https://www.intel.com/content/www/us/en/docs/mpi-library/developer-guide-linux/2021-12/overview.html

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

```
[tvj@bernie ~]$ export I_MPI_DEBUG=1

[tvj@bernie ~]$ mpirun -n 1 IMB-MPI1
[0] MPI startup(): Intel(R) MPI Library, Version 2021.12  Build 20240213 (id: 4f55822)
[0] MPI startup(): Copyright (C) 2003-2024 Intel Corporation.  All rights reserved.
[0] MPI startup(): library kind: release
[0] MPI startup(): libfabric version: 1.18.1-impi
[0] MPI startup(): libfabric provider: mlx
[1736959004.001121] [bernie:29714:0]        ib_iface.c:1060 UCX  ERROR ibv_create_cq(cqe=4096) failed: Cannot allocate memory : Please set max locked memory (ulimit -l) to 'unlimited' (current: 64 kbytes)
Abort(1614735) on node 0 (rank 0 in comm 0): Fatal error in PMPI_Init_thread: Unknown error class, error stack:
MPIR_Init_thread(192)........: 
MPID_Init(1645)..............: 
MPIDI_OFI_mpi_init_hook(1653): 
create_vni_context(2253).....: OFI endpoint open failed (ofi_init.c:2253:create_vni_context:Input/output error)

[tvj@bernie ~]$ dmesg -T
[Wed Jan 15 11:37:57 2025] infiniband mlx5_0: create_qp:3047:(pid 29714): Create QP type 2 failed
[Wed Jan 15 11:37:57 2025] infiniband mlx5_0: create_qp:3047:(pid 29714): Create QP type 9 failed
[Wed Jan 15 11:37:57 2025] infiniband mlx5_0: create_qp:3047:(pid 29714): Create QP type 4 failed
```

**Libfrabric Support**
* This section of the developer guide is very useful
* Intel MPI Library uses the Open Fabrics Interfaces (OFI) framework which is implemented through libfabric libraries
* `FI_PROVIDER_PATH` identifies the libfabric provider libraries
* OFI includes support for mlx, tcp, psm2, psm3, sockets, verbs, and more
* These can be manually set via `FI_PROVIDER`
* It appears that IMPI correctly identifies that we have MLX support, however, something is wrong with UCX
* Let's try some diagnostics:

```
[tvj@bernie ~]$ fi_info
provider: mlx
    fabric: mlx
    domain: mlx
    version: 1.4
    type: FI_EP_UNSPEC
    protocol: FI_PROTO_MLX
provider: psm3
    fabric: IB/OPA-0xfe80000000000000
    domain: mlx5_0
    version: 705.1000
    type: FI_EP_RDM
    protocol: FI_PROTO_PSMX3
provider: psm3
    fabric: IB/OPA-0xfe80000000000000
    domain: mlx5_0
    version: 705.1000
    type: FI_EP_RDM
    protocol: FI_PROTO_PSMX3
provider: psm3
    fabric: IB/OPA-0xfe80000000000000
    domain: mlx5_0
    version: 705.1000
    type: FI_EP_RDM
    protocol: FI_PROTO_PSMX3
provider: mlx;ofi_rxm
    fabric: mlx
    domain: mlx
    version: 118.10
    type: FI_EP_RDM
    protocol: FI_PROTO_RXM
provider: tcp
    fabric: 192.168.201.0/24
    domain: ib0
    version: 118.10
    type: FI_EP_RDM
    protocol: FI_PROTO_XNET

[tvj@bernie ~]$ ucx_info -v
# Library version: 1.15.0
# Library path: /lib64/libucs.so.0
# API headers version: 1.15.0
# Git branch '', revision 348d14f
# Configured with: --build=x86_64-redhat-linux-gnu --host=x86_64-redhat-linux-gnu --program-prefix= --disable-dependency-tracking --prefix=/usr --exec-prefix=/usr --bindir=/usr/bin --sbindir=/usr/sbin --sysconfdir=/etc --datadir=/usr/share --includedir=/usr/include --libdir=/usr/lib64 --libexecdir=/usr/libexec --localstatedir=/var --sharedstatedir=/var/lib --mandir=/usr/share/man --infodir=/usr/share/info --disable-optimizations --disable-logging --disable-debug --disable-assertions --disable-params-check --without-java --enable-cma --without-cuda --without-gdrcopy --with-verbs --without-cm --without-knem --with-rdmacm --without-rocm --without-xpmem --without-fuse3 --without-ugni

[tvj@bernie ~]$ lspci | grep Mellanox
99:00.0 Infiniband controller: Mellanox Technologies MT28908 Family [ConnectX-6]

[tvj@bernie ~]$ ibv_devinfo
hca_id:	mlx5_0
	transport:			InfiniBand (0)
	fw_ver:				20.41.1000
	node_guid:			a088:c203:0096:05fc
	sys_image_guid:			a088:c203:0096:05fc
	vendor_id:			0x02c9
	vendor_part_id:			4123
	hw_ver:				0x0
	board_id:			MT_0000000222
	phys_port_cnt:			1
		port:	1
			state:			PORT_ACTIVE (4)
			max_mtu:		4096 (5)
			active_mtu:		4096 (5)
			sm_lid:			35
			port_lid:		35
			port_lmc:		0x00
			link_layer:		InfiniBand

[tvj@bernie ~]$ ucx_info -d | grep Trans
#      Transport: self
#      Transport: tcp
#      Transport: tcp
#      Transport: tcp
#      Transport: sysv
#      Transport: posix
```

> [!IMPORTANT]
> https://www.intel.com/content/www/us/en/developer/articles/technical/improve-performance-and-stability-with-intel-mpi-library-on-infiniband.html
> This resource helps to identify the error with lack of dc, rc, and ud transports.
> This was fixed as follows:

```
dnf install -y ucx*

ucx_info -d | grep Transp
#      Transport: self
#      Transport: tcp
#      Transport: tcp
#      Transport: tcp
#      Transport: sysv
#      Transport: posix
#      Transport: rc_verbs
#      Transport: ud_verbs
#      Transport: cma
```

```
[tvj@bernie ~]$ export FI_PROVIDER=mlx
[tvj@bernie ~]$ export I_MPI_DEBUG=1
[tvj@bernie ~]$ mpirun -n 16 -ppn 8 -f ./hostfile ./myprog
[0] MPI startup(): Intel(R) MPI Library, Version 2021.12  Build 20240213 (id: 4f55822)
[0] MPI startup(): Copyright (C) 2003-2024 Intel Corporation.  All rights reserved.
[0] MPI startup(): library kind: release
[0] MPI startup(): libfabric version: 1.18.1-impi
[0] MPI startup(): libfabric provider: mlx
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
```

```
[tvj@bernie ~]$ mpirun -n 16 -ppn 8 IMB-MPI1
[0] MPI startup(): Intel(R) MPI Library, Version 2021.12  Build 20240213 (id: 4f55822)
[0] MPI startup(): Copyright (C) 2003-2024 Intel Corporation.  All rights reserved.
[0] MPI startup(): library kind: release
[0] MPI startup(): libfabric version: 1.18.1-impi
[0] MPI startup(): libfabric provider: mlx
[1736961102.822146] [bernie:39388:0]        ib_iface.c:1060 UCX  ERROR ibv_create_cq(cqe=4096) failed: Cannot allocate memory : Please set max locked memory (ulimit -l) to 'unlimited' (current: 64 kbytes)
Abort(1614735) on node 0 (rank 0 in comm 0): Fatal error in PMPI_Init_thread: Unknown error class, error stack:
MPIR_Init_thread(192)........: 
MPID_Init(1645)..............: 
MPIDI_OFI_mpi_init_hook(1653): 
create_vni_context(2253).....: OFI endpoint open failed (ofi_init.c:2253:create_vni_context:Input/output error)
```

> [!WARNING]
> I don't know why this error is occurring.  If I set `FI_PROVIDER=tcp`, then the pingpong test code runs without error.



## Testing Slurm with Intel oneAPI
* `test_mpi.slurm` script
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
# Load environment
# V3:  Sierra v5.22 compiled with Intel oneAPI
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
_SIERRA_INSTALL_DIR=/data/software/Sierra/5.22/install
export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH
export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH
export I_MPI_OFI_PROVIDER=tcp

#
# Compile the code
#make
mpicc -o mpi_hello_world ./mpi_hello_world.c

#
# Run the compiled code
mpirun ./mpi_hello_world
#mpiexec ./mpi_hello_world
#srun --mpi=pmi2 ./mpi_hello_world
#srun ./mpi_hello_world
```

* run script using `srun ./mpi_hello_world`
```
SLURM_SUBMIT_DIR is /mnt/home/tvj/Documents
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
Hello world from processor cnode003.bernie.cluster, rank 0 out of 1 processors
```

* run script using
  - `srun --mpi=pmi2 ./mpi_hello_world`
  - `mpirun ./mpi_hello_world`
  - `mpiexec ./mpi_hello_world`
```
SLURM_SUBMIT_DIR is /mnt/home/tvj/Documents
Hello world from processor cnode003.bernie.cluster, rank 8 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 9 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 15 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 0 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 1 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 2 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 4 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 5 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 7 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 3 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 10 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 11 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 12 out of 16 processors
Hello world from processor cnode001.bernie.cluster, rank 6 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 13 out of 16 processors
Hello world from processor cnode003.bernie.cluster, rank 14 out of 16 processors
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




## Testing Sierra v5.22
* set environment
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
_SIERRA_INSTALL_DIR=/data/software/Sierra/5.22/install
export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH
export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH
export I_MPI_OFI_PROVIDER=tcp
```
  
* check Sierra executable
```
[tvj@bernie 1_SIERRA_Troubleshooting]$ which adagio
/data/software/Sierra/5.22/install/apps/bin/adagio

[tvj@bernie 1_SIERRA_Troubleshooting]$ adagio --version
Application: Adagio
  Executable: /data/software/Sierra/5.22/install/apps/bin/adagio version 5.22.1-0-g7f404f06 built on Jan  8 2025 15:15:16
  Build Options: linux oneapi-2024.1.0 release
    Product: ACME                     (2.9.0)
    Product: Adagio                   (5.22.1-0-g7f404f06)
    Product: FEI                      (5.21.00)
    Product: FETI-DP                  (5.22.1-0-g7f404f06)
    Product: GDSW                     (5.22.1-0-g7f404f06)
    Product: Linux                    (4.18.0-553.30.1.el8_10.x86_64)
    Product: SIERRA Framework         (5.22.1-0-g7f404f06)
    Product: Sandia Toolkit (STK)     (5.22.1-0-g7f404f06)
    Product: UtilityLib               (5.22.1-0-g7f404f06)
---------------------------------------------------
There were no errors encountered during execution of this procedure
There were no warnings encountered during execution of this procedure 
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
Execution complete


Timing summary of 1 processor
                 Timer                   Count       CPU Time                Wall Time
---------------------------------------- ----- --------------------- -------------------------
Sierra                                       1 00:00:00.337 (100.0%) 482442:51:53.822 (100.0%)

Took 0.000233889 seconds to generate the table above.


Memory summary of 1 processor
           Metric               Largest          Processor          Smallest         Processor
---------------------------- -------------- -------------------- -------------- --------------------
Dynamically Allocated (Heap)        30.9 MB on 0 at 00:00:00.000        30.9 MB on 0 at 00:00:00.000
Largest Free Fragment (Heap) 1.81366e-01 MB on 0 at 00:00:00.000 1.81366e-01 MB on 0 at 00:00:00.000
         Total Memory In Use       593.4 MB on 0                       593.4 MB on 0
           Major Page Faults         0 flts on 0                         0 flts on 0


Performance metric summary
---------------------------------------------------
Min High-water memory usage 174.5 MB
Avg High-water memory usage 174.5 MB
Max High-water memory usage 174.5 MB

Min Available memory per processor 4024.8 MB
Avg Available memory per processor 4024.8 MB
Max Available memory per processor 4024.8 MB

Min No-output time 0.3368 sec
Avg No-output time 0.3368 sec
Max No-output time 0.3368 sec
---------------------------------------------------
There were no errors encountered during parse
There were no warnings encountered during parse
There were no errors encountered during execution
There were no warnings encountered during execution
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
SIERRA execution successful after 00:00:00 (HH:MM:SS)
```
  
* run test program on cluster (without slurm)

```
[tvj@bernie 1_SIERRA_Troubleshooting]$ cat hostfile
cnode001.bernie.cluster
cnode003.bernie.cluster

[tvj@bernie 1_SIERRA_Troubleshooting]$ mpiexec -n 16 -ppn 8 -f ./hostfile  adagio -i LasagnaOpt_Dynamic.i

[tvj@bernie 1_SIERRA_Troubleshooting]$ tail LasagnaOpt_Dynamic.log 
Min No-output time 1388.7954 sec
Avg No-output time 1392.8354 sec
Max No-output time 1394.2110 sec
---------------------------------------------------
There were no errors encountered during parse
There was 1 warning encountered during parse
There were no errors encountered during execution
There were no warnings encountered during execution
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
SIERRA execution successful after 00:23:21 (HH:MM:SS)
```

* run test program with `-r ssh` option
```
[tvj@bernie 1_SIERRA_Troubleshooting]$ mpiexec -r ssh -n 16 -ppn 8 -f ./hostfile  adagio -i LasagnaOpt_Dynamic.i

[tvj@bernie 1_SIERRA_Troubleshooting]$ tail LasagnaOpt_Dynamic.log 
Min No-output time 9.5244 sec
Avg No-output time 9.6027 sec
Max No-output time 9.6256 sec
---------------------------------------------------
There were no errors encountered during parse
There were no warnings encountered during parse
There were no errors encountered during execution
There was 1 warning encountered during execution
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
SIERRA execution successful after 00:00:09 (HH:MM:SS)
```



## Testing Sierra v5.22 with Slurm
* see Slurm script above
* I use a sample explicit problem that is the response of a nested sphere to a rapid thermal expansion due to proton pulse heating.  It captures the physics and simulation workflow that I need for spallation target development.
* It runs best when invoked via `mpiexec -np ${SLURM_NTASKS} adagio -i LasagnaOpt_Dynamic.i`
* When invoked via `srun --mpi=pmi2 adagio -i LasagnaOpt_Dynamic.i`, the simulation seems to take much longer to begin.  It also seems to increase the simulation time by ~1% (which could be in the noise... I haven't explored further).
* When invoked via `sierra --np ${SLURM_NTASKS} --run adagio -i $INPUT_FILENAME`
  - Sierra appears to be a python script that itself invokes another python script called `launch`.
  - This then tries to start teh simulation using `mpiexec` with a `-r ssh` option.  This causes a crash as shown.
  - If I manually remove `-r ssh`, then everything runs fine.  And I also don't need the `- envlist option`.
  - Oddly enough, when I invoke _outside_ of slurm, I can run `mpiexec` with and without `-r ssh`.

```
NOTICE: Found adagio in PATH at /data/software/Sierra/5.22/install/apps/bin/adagio
***** Executing sierra batch script *****
Executing: /mnt/home/tvj/Documents/1_SIERRA_Troubleshooting/sierra_batch_script_526691.sh

***** Executing sierra *****
Executing: launch -n 16 adagio -i LasagnaOpt_Dynamic.i
Obtained cached build options from '/data/software/Sierra/5.22/install/apps/bin/.build_metadata'
mpiexec -r ssh -np 16 -envlist PATH,LD_LIBRARY_PATH,PWD adagio -i LasagnaOpt_Dynamic.i
[mpiexec@cnode001.bernie.cluster] Error: Unable to run bstrap_proxy on cnode003 (pid 526696, exit code 65280)
[mpiexec@cnode001.bernie.cluster] poll_for_event (../../../../../src/pm/i_hydra/libhydra/demux/hydra_demux_poll.c:157): check exit codes error
[mpiexec@cnode001.bernie.cluster] HYD_dmx_poll_wait_for_proxy_event (../../../../../src/pm/i_hydra/libhydra/demux/hydra_demux_poll.c:206): poll for event error
[mpiexec@cnode001.bernie.cluster] HYD_bstrap_setup (../../../../../src/pm/i_hydra/libhydra/bstrap/src/intel/i_hydra_bstrap.c:1063): error waiting for event
[mpiexec@cnode001.bernie.cluster] Error setting up the bootstrap proxies
[mpiexec@cnode001.bernie.cluster] Possible reasons:
[mpiexec@cnode001.bernie.cluster] 1. Host is unavailable. Please check that all hosts are available.
[mpiexec@cnode001.bernie.cluster] 2. Cannot launch hydra_bstrap_proxy or it crashed on one of the hosts.
[mpiexec@cnode001.bernie.cluster]    Make sure hydra_bstrap_proxy is available on all hosts and it has right permissions.
[mpiexec@cnode001.bernie.cluster] 3. Firewall refused connection.
[mpiexec@cnode001.bernie.cluster]    Check that enough ports are allowed in the firewall and specify them with the I_MPI_PORT_RANGE variable.
[mpiexec@cnode001.bernie.cluster] 4. Ssh bootstrap cannot launch processes on remote host.
[mpiexec@cnode001.bernie.cluster]    Make sure that passwordless ssh connection is established across compute hosts.
[mpiexec@cnode001.bernie.cluster]    You may try using -bootstrap option to select alternative launcher.
unknown option -- -
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]
           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]
           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
           [-i identity_file] [-J [user@]host[:port]] [-L address]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] destination [command]
Command: 'mpiexec -r ssh -np 16 -envlist PATH,LD_LIBRARY_PATH,PWD adagio -i LasagnaOpt_Dynamic.i' failed with exit status 255
ERROR: Previous command failed (exitcode=255)
```



## Additional questions and resources to explore

### How to optimize compiling of Sierra
* The slow queue nodes are `linux-rhel8-ivybridge`
* Currently, I compile on the headnode using `x86_64` generic targets.
* Sierra DevOps recommended compiling on the compute node using the native `ivybridge` target.
* Spack keeps track of architectures via https://github.com/spack/spack/blob/develop/lib/spack/external/archspec/json/cpu/microarchitectures.json
* I can see if this worked through an adagio log file which will have `Simd vector width 4` if it works and `2` otherwise.

### Understanding MPI libfabric
* Why doesn't `I_MPI_OFI_PROVIDER=mlx` work?
* `fi_info` output shows `mlx` as an option
* Is it best practice to just use `tcp` with `IBoIP`?

### How to benchmark machine
- [ ] Learn how to run the test and benchmark packages that come with the oneAPI installation
- [ ] https://www.intel.com/content/www/us/en/developer/articles/technical/intel-mpi-benchmarks.html
- [ ] https://mvapich.cse.ohio-state.edu/benchmarks/
- [ ] https://hpctoolkit.org/

### Resources to read
- [ ] https://www.intel.com/content/www/us/en/developer/articles/technical/mpi-library-2019-over-libfabric.html
- [ ] https://www.intel.com/content/www/us/en/docs/mpi-library/developer-reference-linux/2021-8/ofi-capable-network-fabrics-control.html
- [ ] https://www.intel.com/content/www/us/en/developer/articles/technical/improve-performance-and-stability-with-intel-mpi-library-on-infiniband.html
