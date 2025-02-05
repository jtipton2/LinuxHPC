# ORNL Research Cloud

## Login
```
* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
                         Welcome to your ORNL Research Cloud (ORC) Instance

Important Notes:
- Installing Software : Use 'sudo' when elevated privileges are needed. (E.g. To resolve 'permission denied' errors during software installs.)
- Support requests are preferred via tickets (URL below).

Support:
Support Documentation: https://confluence.ornl.gov/display/ORCKB/ORNL+Research+Cloud+KB
Email Contact: orc-ops@ornl.gov
Open a Ticket: https://orc-tickets.ornl.gov
Chat/community: https://ornl.slack.com/ via the channel orc-ornl_research_cloud

Patching:
By default, ORNL Research Cloud systems will automatically patch on the second Tuesday of the month at approximately 10:15 PM.  You will be responsible for periodically rebooting your VM.
Full Information URL : linux-patching.ornl.gov

sudo:
The 'cloud' user on this VM has full sudo.
You may become your ucams user via:
sudo su - ucamsID

Citation Information:
The following acknowledgment should be included in publications and presentations that contain work performed using ORC resources:
This research used resources from the ORNL Research Cloud Infrastructure at the Oak Ridge National Laboratory, which is supported by the Office of Science of the U.S. Department of Energy under Contract No. DE-AC05-00OR22725

* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
```

## Hardware
For testing purposes, I've been given an instance with 32 processors and 256 GB of RAM:
```
[cloud@tvj-orc-1 ~]$ cat /proc/cpuinfo

...

processor       : 31
vendor_id       : AuthenticAMD
cpu family      : 23
model           : 49
model name      : AMD EPYC 7702 64-Core Processor
stepping        : 0
microcode       : 0x830107c
cpu MHz         : 1996.248
cache size      : 512 KB
physical id     : 31
siblings        : 1
core id         : 0
cpu cores       : 1
apicid          : 31
initial apicid  : 31
fpu             : yes
fpu_exception   : yes
cpuid level     : 16
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 clzero xsaveerptr wbnoinvd arat npt nrip_save umip rdpid arch_capabilities
bugs            : sysret_ss_attrs null_seg spectre_v1 spectre_v2 spec_store_bypass retbleed smt_rsb srso
bogomips        : 3992.49
TLB size        : 1024 4K pages
clflush size    : 64
cache_alignment : 64
address sizes   : 48 bits physical, 48 bits virtual
power management:
```

## Software

```
[cloud@tvj-orc-1 ~]$ gcc -dumpmachine
x86_64-redhat-linux

[cloud@tvj-orc-1 ~]$ gcc --version
gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-2)
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[cloud@tvj-orc-1 ~]$ python3 --version
Python 3.9.21

[cloud@tvj-orc-1 ~]$ sudo su

[root@tvj-orc-1 cloud]# dnf install autoconf automake bison bzip2 diffutils flex libtool m4 python3-devel.x86_64 libX11-devel.x86_64 libcurl-devel.x86_64
Last metadata expiration check: 0:05:44 ago on Tue Feb  4 15:40:56 2025.
Package autoconf-2.69-39.el9.noarch is already installed.
Package automake-1.16.2-8.el9.noarch is already installed.
Package bison-3.7.4-5.el9.x86_64 is already installed.
Package bzip2-1.0.8-8.el9.x86_64 is already installed.
Package diffutils-3.7-12.el9.x86_64 is already installed.
Package flex-2.6.4-9.el9.x86_64 is already installed.
Package libtool-2.4.6-46.el9.x86_64 is already installed.
Package m4-1.4.19-1.el9.x86_64 is already installed.
Package python3-devel-3.9.21-1.el9_5.x86_64 is already installed.
Package libX11-devel-1.7.0-9.el9.x86_64 is already installed.
Package libcurl-devel-7.76.1-31.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!

[root@tvj-orc-1 cloud]# exit
exit
```

>[!Important]
>Later during the Intel oneAPI install, CMake install gave an error regarding Curl.  
>To resolve, I also had to `dnf install libcurl-devel.x86_64`




## Spack
```
[cloud@tvj-orc-1 ~]$ mkdir software

[cloud@tvj-orc-1 ~]$ cd software/

[cloud@tvj-orc-1 software]$ git clone https://github.com/spack/spack.git
Cloning into 'spack'...
remote: Enumerating objects: 625118, done.
remote: Counting objects: 100% (716/716), done.
remote: Compressing objects: 100% (351/351), done.
remote: Total 625118 (delta 590), reused 365 (delta 365), pack-reused 624402 (from 5)
Receiving objects: 100% (625118/625118), 212.58 MiB | 2.90 MiB/s, done.
Resolving deltas: 100% (296865/296865), done.
Updating files: 100% (12196/12196), done.

[cloud@tvj-orc-1 software]$ source spack/share/spack/setup-env.sh

[cloud@tvj-orc-1 software]$ echo $SPACK_ROOT
/home/cloud/software/spack

[cloud@tvj-orc-1 software]$ spack debug report
* **Spack:** 1.0.0.dev0 (22c3b4099ff5ea93de258117821482f439e6ef0d)
* **Python:** 3.9.21
* **Platform:** linux-rocky9-zen2

[cloud@tvj-orc-1 software]$ spack external find
==> The following specs have been detected on this system and added to /home/cloud/.spack/packages.yaml
autoconf@2.69    bison@3.7.4     diffutils@3.7    gawk@5.1.0    gmake@4.3      m4@1.4.19      perl@5.32.1    tar@1.34
automake@1.16.2  coreutils@8.32  findutils@4.8.0  gettext@0.21  groff@1.22.4   openssh@8.7p1  pkgconf@1.7.3  zlib@1.2.11
binutils@2.35.2  curl@7.76.1     flex@2.6.4       git@2.43.5    libtool@2.4.6  openssl@3.2.2  sed@4.8

[cloud@tvj-orc-1 software]$ spack compilers
==> No compilers available. Run `spack compiler find` to autodetect compilers

[cloud@tvj-orc-1 software]$ spack compiler find
==> Added 1 new compiler to /home/cloud/.spack/linux/compilers.yaml
    gcc@11.5.0
==> Compilers are defined in the following files:
    /home/cloud/.spack/linux/compilers.yaml

[cloud@tvj-orc-1 software]$ spack compilers
==> Available compilers
-- gcc rocky9-x86_64 --------------------------------------------
gcc@11.5.0
```

Sierra is tested on gcc@10.2.0 so I need to install that compiler.  A recent update in Spack now makes that version unsupported, so I'll use the next closest:

```
[cloud@tvj-orc-1 software]$ spack install -j 32 gcc@10.5.0

[cloud@tvj-orc-1 software]$ spack compiler add "$(spack location -i gcc@10.5.0)"

[cloud@tvj-orc-1 software]$ spack compilers
==> Available compilers
-- gcc rocky9-x86_64 --------------------------------------------
gcc@11.5.0  gcc@10.5.0

[cloud@tvj-orc-1 software]$ spack gc

[cloud@tvj-orc-1 software]$ spack find
-- linux-rocky9-zen2 / gcc@11.5.0 -------------------------------
gcc@10.5.0  gcc-runtime@11.5.0  glibc@2.34  gmp@6.3.0  mpc@1.3.1  mpfr@4.2.1  zlib-ng@2.2.3  zstd@1.5.6
==> 8 installed packages
```




## Intel oneAPI Environment

### sierra522.yaml
```
# sierra522.yaml
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
      - "%gcc@10.5.0"
```


### Environment Install
```
[cloud@tvj-orc-1 software]$ spack env create sierra522 sierra522.yaml
==> Created environment sierra522 in: /home/cloud/software/spack/var/spack/environments/sierra522
==> Activate with: spack env activate sierra522

[cloud@tvj-orc-1 software]$ spack env activate -p sierra522

[sierra522] [cloud@tvj-orc-1 software]$ spack concretize
==> Concretized 3 specs:
 -   tfmkwy3  intel-oneapi-compilers@2024.1.0%gcc@10.5.0~amd+envmods~nvidia build_system=generic arch=linux-rocky9-zen2
 -   j2jvb2x      ^gcc-runtime@10.5.0%gcc@10.5.0 build_system=generic arch=linux-rocky9-zen2
[e]  uu2ssga      ^glibc@2.34%gcc@10.5.0 build_system=autotools arch=linux-rocky9-zen2
 -   g2w3tvd      ^patchelf@0.17.2%gcc@10.5.0 build_system=autotools arch=linux-rocky9-zen2
[e]  ugbzoaq          ^gmake@4.3%gcc@10.5.0~guile build_system=generic patches=599f134 arch=linux-rocky9-zen2
 -   mk34k5p  intel-oneapi-mkl@2024.2.0%gcc@10.5.0+cluster+envmods~gfortran~ilp64+shared build_system=generic mpi_family=mpich threads=none arch=linux-rocky9-zen2
 -   uahtckp      ^intel-tbb@2022.0.0%gcc@10.5.0~ipo+shared+tm build_system=cmake build_type=Release cxxstd=default generator=make arch=linux-rocky9-zen2
 -   7b6hjke          ^cmake@3.31.5%gcc@10.5.0~doc+ncurses+ownlibs~qtgui build_system=generic build_type=Release arch=linux-rocky9-zen2
[e]  phjasxa              ^curl@7.76.1%gcc@10.5.0+gssapi+ldap~libidn2~librtmp~libssh~libssh2+nghttp2 build_system=autotools libs=shared,static tls=openssl arch=linux-rocky9-zen2
 -   snnsvia              ^ncurses@6.5%gcc@10.5.0~symlinks+termlib abi=none build_system=autotools patches=7a351bc arch=linux-rocky9-zen2
 -   ihpi7tu              ^zlib-ng@2.2.3%gcc@10.5.0+compat+new_strategies+opt+pic+shared build_system=autotools arch=linux-rocky9-zen2
 -   3xtwcjn          ^hwloc@2.11.1%gcc@10.5.0~cairo~cuda~gl~level_zero~libudev+libxml2~nvml~opencl+pci~rocm build_system=autotools libs=shared,static arch=linux-rocky9-zen2
 -   3ralz5j              ^libpciaccess@0.17%gcc@10.5.0 build_system=autotools arch=linux-rocky9-zen2
 -   epl2i3z                  ^util-macros@1.20.1%gcc@10.5.0 build_system=autotools arch=linux-rocky9-zen2
 -   wdurisw              ^libxml2@2.13.5%gcc@10.5.0~http+pic~python+shared build_system=autotools arch=linux-rocky9-zen2
 -   7yytcfy                  ^libiconv@1.17%gcc@10.5.0 build_system=autotools libs=shared,static arch=linux-rocky9-zen2
 -   i6am5v3                  ^xz@5.4.6%gcc@10.5.0~pic build_system=autotools libs=shared,static arch=linux-rocky9-zen2
[e]  mzcft5n              ^pkgconf@1.7.3%gcc@10.5.0 build_system=autotools arch=linux-rocky9-zen2
 -   k7gwoc3  intel-oneapi-mpi@2021.12.1%gcc@10.5.0~classic-names+envmods~external-libfabric~generic-names~ilp64 build_system=generic arch=linux-rocky9-zen2

[sierra522] [cloud@tvj-orc-1 software]$ spack install -j 32

...

[sierra522] [cloud@tvj-orc-1 software]$ spack env deactivate

[cloud@tvj-orc-1 software]$ spack env activate -p sierra522

[sierra522] [cloud@tvj-orc-1 software]$ env > env.out
```

>[!NOTE]
>Use the `env` output to determine how Spack is setting the environment variables.



### Environment Variables
* explored `env` output to discover what the environment was setting
* later, I plan to create a custom module... for now, I just set the environment manually via:
```
#
# Environment settings for Sierra 522
#
export PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/opt/oclfpga/bin:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/bin:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/bin:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/bin:/home/cloud/software/spack/opt/spack/linux-rocky9-zen2/gcc-11.5.0/gcc-10.5.0-fcs4ppfe3bzb4bu3bxfzvgg5r5t5cpw6/bin:$PATH
export LD_LIBRARY_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/opt/oclfpga/host/linux64/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/opt/compiler/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/opt/mpi/libfabric/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/lib:/home/cloud/software/spack/opt/spack/linux-rocky9-zen2/gcc-11.5.0/gcc-10.5.0-fcs4ppfe3bzb4bu3bxfzvgg5r5t5cpw6/lib64:$LD_LIBRARY_PATH
export CC=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx
export CXX=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icpx
export FC=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
export F77=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_ROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12
export FI_PROVIDER_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/opt/mpi/libfabric/lib/prov:/usr/lib64/libfabric:.
export I_MPI_CC=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx
export I_MPI_CXX=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icpx
export I_MPI_F77=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
export I_MPI_F90=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
export MKLROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2
#
# not sure how/if these settings are important for my use case...
#
export PKG_CONFIG_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/lib/pkgconfig:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/lib/pkgconfig:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/lib/pkgconfig:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/lib64/pkgconfig:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/lib/pkgconfig:/usr/share/pkgconfig:/usr/lib64/pkgconfig:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/share/pkgconfig:.
export FPGA_VARS_DIR=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/opt/oclfpga
export DIAGUTIL_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/etc/compiler/sys_check/sys_check.sh:.
export MANPATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/share/man:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/share/man:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/share/man:/usr/share/man:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/man:.:
export CMAKE_PREFIX_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/lib/cmake:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view:.
export CMPLR_ROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1
export ACLOCAL_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/share/aclocal:/usr/share/aclocal:.
export LIBRARY_PATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/lib:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/lib:.
export OCL_ICD_FILENAMES=libintelocl_emu.so:libalteracl.so:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/lib/libintelocl.so:.
export CLASSPATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/share/java/mpi.jar:.
export INTELFPGAOCLSDKROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/opt/oclfpga
export NLSPATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/lib/compiler/locale/%l_%t/%N:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/share/locale/%l_%t/%N:.
export CPATH=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/opt/oclfpga/include:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mkl/2024.2/include:/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/include:.
#
# later settings found via troubleshooting
#
export FI_PROVIDER=shm
export I_MPI_FABRICS=shm
```




### Testing Intel oneAPI installation functionality

```
[cloud@tvj-orc-1 ~]$ which gcc
~/software/spack/opt/spack/linux-rocky9-zen2/gcc-11.5.0/gcc-10.5.0-fcs4ppfe3bzb4bu3bxfzvgg5r5t5cpw6/bin/gcc

[cloud@tvj-orc-1 ~]$ gcc --version
gcc (Spack GCC) 10.5.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[cloud@tvj-orc-1 ~]$ which icx
~/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx

[cloud@tvj-orc-1 ~]$ which icx
~/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx

[cloud@tvj-orc-1 ~]$ icx --version
Intel(R) oneAPI DPC++/C++ Compiler 2024.1.0 (2024.1.0.20240308)
Target: x86_64-unknown-linux-gnu
Thread model: posix
InstalledDir: /home/cloud/software/spack/opt/spack/linux-rocky9-zen2/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-tfmkwy3xabqoecs6uqyctxv7e43pqpk6/compiler/2024.1/bin/compiler
Configuration file: /home/cloud/software/spack/opt/spack/linux-rocky9-zen2/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-tfmkwy3xabqoecs6uqyctxv7e43pqpk6/compiler/2024.1/bin/compiler/../icx.cfg

[cloud@tvj-orc-1 ~]$ cat hostfile
tvj-orc-1

[cloud@tvj-orc-1 ~]$ mpirun -n 4 -ppn 32 -f ./hostfile uptime
 16:01:33 up 43 min,  1 user,  load average: 0.02, 0.06, 0.48
 16:01:33 up 43 min,  1 user,  load average: 0.02, 0.06, 0.48
 16:01:33 up 43 min,  1 user,  load average: 0.02, 0.06, 0.48
 16:01:33 up 43 min,  1 user,  load average: 0.02, 0.06, 0.48

[cloud@tvj-orc-1 ~]$ mpiicc -o mpihello /home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/opt/mpi/test/test.c

[cloud@tvj-orc-1 ~]$ export I_MPI_DEBUG=5

[cloud@tvj-orc-1 ~]$ mpirun -n 16 -ppn 32 -f ./hostfile ./mpihello
[0] MPI startup(): Intel(R) MPI Library, Version 2021.12  Build 20240213 (id: 4f55822)
[0] MPI startup(): Copyright (C) 2003-2024 Intel Corporation.  All rights reserved.
[0] MPI startup(): library kind: release
[0] MPI startup(): libfabric version: 1.18.1-impi
[0] MPI startup(): libfabric provider: tcp
[0] MPI startup(): Load tuning file: "/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/opt/mpi/etc/tuning_generic_shm-ofi.dat"
[0] MPI startup(): Number of NICs:  1
[0] MPI startup(): ===== NIC pinning on tvj-orc-1 =====
[0] MPI startup(): Rank    Pin nic
[0] MPI startup(): 0       eth0
[0] MPI startup(): 1       eth0
[0] MPI startup(): 2       eth0
[0] MPI startup(): 3       eth0
[0] MPI startup(): 4       eth0
[0] MPI startup(): 5       eth0
[0] MPI startup(): 6       eth0
[0] MPI startup(): 7       eth0
[0] MPI startup(): 8       eth0
[0] MPI startup(): 9       eth0
[0] MPI startup(): 10      eth0
[0] MPI startup(): 11      eth0
[0] MPI startup(): 12      eth0
[0] MPI startup(): 13      eth0
[0] MPI startup(): 14      eth0
[0] MPI startup(): 15      eth0
[0] MPI startup(): ===== CPU pinning =====
[0] MPI startup(): Rank    Pid      Node name  Pin cpu
[0] MPI startup(): 0       90944    tvj-orc-1  {0,1}
[0] MPI startup(): 1       90945    tvj-orc-1  {2,3}
[0] MPI startup(): 2       90946    tvj-orc-1  {4,5}
[0] MPI startup(): 3       90947    tvj-orc-1  {6,7}
[0] MPI startup(): 4       90948    tvj-orc-1  {8,9}
[0] MPI startup(): 5       90949    tvj-orc-1  {10,11}
[0] MPI startup(): 6       90950    tvj-orc-1  {12,13}
[0] MPI startup(): 7       90951    tvj-orc-1  {14,15}
[0] MPI startup(): 8       90952    tvj-orc-1  {16,17}
[0] MPI startup(): 9       90953    tvj-orc-1  {18,19}
[0] MPI startup(): 10      90954    tvj-orc-1  {20,21}
[0] MPI startup(): 11      90955    tvj-orc-1  {22,23}
[0] MPI startup(): 12      90956    tvj-orc-1  {24,25}
[0] MPI startup(): 13      90957    tvj-orc-1  {26,27}
[0] MPI startup(): 14      90958    tvj-orc-1  {28,29}
[0] MPI startup(): 15      90959    tvj-orc-1  {30,31}
[0] MPI startup(): I_MPI_CC=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx
[0] MPI startup(): I_MPI_CXX=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icpx
[0] MPI startup(): I_MPI_F90=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
[0] MPI startup(): I_MPI_F77=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
[0] MPI startup(): I_MPI_ROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12
[0] MPI startup(): I_MPI_MPIRUN=mpirun
[0] MPI startup(): I_MPI_BIND_WIN_ALLOCATE=localalloc
[0] MPI startup(): I_MPI_HYDRA_TOPOLIB=hwloc
[0] MPI startup(): I_MPI_RETURN_WIN_MEM_NUMA=-1
[0] MPI startup(): I_MPI_INTERNAL_MEM_POLICY=default
[0] MPI startup(): I_MPI_DEBUG=5
Hello world: rank 0 of 16 running on tvj-orc-1
Hello world: rank 1 of 16 running on tvj-orc-1
Hello world: rank 2 of 16 running on tvj-orc-1
Hello world: rank 3 of 16 running on tvj-orc-1
Hello world: rank 4 of 16 running on tvj-orc-1
Hello world: rank 5 of 16 running on tvj-orc-1
Hello world: rank 6 of 16 running on tvj-orc-1
Hello world: rank 7 of 16 running on tvj-orc-1
Hello world: rank 8 of 16 running on tvj-orc-1
Hello world: rank 9 of 16 running on tvj-orc-1
Hello world: rank 10 of 16 running on tvj-orc-1
Hello world: rank 11 of 16 running on tvj-orc-1
Hello world: rank 12 of 16 running on tvj-orc-1
Hello world: rank 13 of 16 running on tvj-orc-1
Hello world: rank 14 of 16 running on tvj-orc-1
Hello world: rank 15 of 16 running on tvj-orc-1
```

It is a little odd that it is defalting to `tcp` for the libfabric provider.  Let's try to manually select `shm`, indicating that all processors are on shared memory:

```
[cloud@tvj-orc-1 ~]$ export FI_PROVIDER=shm
[cloud@tvj-orc-1 ~]$ export I_MPI_FABRICS=shm
[cloud@tvj-orc-1 ~]$ mpirun -n 16 -ppn 32 -f ./hostfile ./mpihello
[0] MPI startup(): Intel(R) MPI Library, Version 2021.12  Build 20240213 (id: 4f55822)
[0] MPI startup(): Copyright (C) 2003-2024 Intel Corporation.  All rights reserved.
[0] MPI startup(): library kind: release
[0] MPI startup(): Load tuning file: "/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/opt/mpi/etc/tuning_generic_shm.dat"
[0] MPI startup(): ===== CPU pinning =====
[0] MPI startup(): Rank    Pid      Node name  Pin cpu
[0] MPI startup(): 0       90994    tvj-orc-1  {0,1}
[0] MPI startup(): 1       90995    tvj-orc-1  {2,3}
[0] MPI startup(): 2       90996    tvj-orc-1  {4,5}
[0] MPI startup(): 3       90997    tvj-orc-1  {6,7}
[0] MPI startup(): 4       90998    tvj-orc-1  {8,9}
[0] MPI startup(): 5       90999    tvj-orc-1  {10,11}
[0] MPI startup(): 6       91000    tvj-orc-1  {12,13}
[0] MPI startup(): 7       91001    tvj-orc-1  {14,15}
[0] MPI startup(): 8       91002    tvj-orc-1  {16,17}
[0] MPI startup(): 9       91003    tvj-orc-1  {18,19}
[0] MPI startup(): 10      91004    tvj-orc-1  {20,21}
[0] MPI startup(): 11      91005    tvj-orc-1  {22,23}
[0] MPI startup(): 12      91006    tvj-orc-1  {24,25}
[0] MPI startup(): 13      91007    tvj-orc-1  {26,27}
[0] MPI startup(): 14      91008    tvj-orc-1  {28,29}
[0] MPI startup(): 15      91009    tvj-orc-1  {30,31}
[0] MPI startup(): I_MPI_CC=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx
[0] MPI startup(): I_MPI_CXX=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icpx
[0] MPI startup(): I_MPI_F90=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
[0] MPI startup(): I_MPI_F77=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
[0] MPI startup(): I_MPI_ROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12
[0] MPI startup(): I_MPI_MPIRUN=mpirun
[0] MPI startup(): I_MPI_BIND_WIN_ALLOCATE=localalloc
[0] MPI startup(): I_MPI_HYDRA_TOPOLIB=hwloc
[0] MPI startup(): I_MPI_RETURN_WIN_MEM_NUMA=-1
[0] MPI startup(): I_MPI_INTERNAL_MEM_POLICY=default
[0] MPI startup(): I_MPI_FABRICS=shm
[0] MPI startup(): I_MPI_DEBUG=5
Hello world: rank 0 of 16 running on tvj-orc-1
Hello world: rank 1 of 16 running on tvj-orc-1
Hello world: rank 2 of 16 running on tvj-orc-1
Hello world: rank 3 of 16 running on tvj-orc-1
Hello world: rank 4 of 16 running on tvj-orc-1
Hello world: rank 5 of 16 running on tvj-orc-1
Hello world: rank 6 of 16 running on tvj-orc-1
Hello world: rank 7 of 16 running on tvj-orc-1
Hello world: rank 8 of 16 running on tvj-orc-1
Hello world: rank 9 of 16 running on tvj-orc-1
Hello world: rank 10 of 16 running on tvj-orc-1
Hello world: rank 11 of 16 running on tvj-orc-1
Hello world: rank 12 of 16 running on tvj-orc-1
Hello world: rank 13 of 16 running on tvj-orc-1
Hello world: rank 14 of 16 running on tvj-orc-1
Hello world: rank 15 of 16 running on tvj-orc-1
[cloud@tvj-orc-1 ~]$
```





