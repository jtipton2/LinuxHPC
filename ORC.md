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


>[!Important]
>Also later, the Sierra compile failed with an error that there was no python in `/usr/bin`.  
>To resolve, I also had to `sudo ln -sf /usr/bin/python3 /usr/bin/python`




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



## Sierra Install
* run installation script
```
[cloud@tvj-orc-1 ~]$ SIERRA_ROOT=/home/cloud/software/sierra_5.22

[cloud@tvj-orc-1 ~]$ cd ${SIERRA_ROOT}

[cloud@tvj-orc-1 sierra_5.22]$ /home/cloud/software/source/sierra_setup.py --procs=32 --keep-build-dir
REMARK: Extracting '/home/cloud/software/source/sandia.tar.gz'
REMARK: Detected package type 'source'
REMARK: Building Sierra and logging output to '/home/cloud/software/sierra_5.22/build/build.log'

************************************************************************
********************* Successfully installed SIERRA ********************
************************************************************************

To use it run the following command:
source /home/cloud/software/sierra_5.22/install/sierra_init.sh
```

* the environment variables script contains
```
_SIERRA_INSTALL_DIR=/home/cloud/software/sierra_5.22/install
export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH
export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH
```

* test the executable
```
[cloud@tvj-orc-1 sierra_5.22]$ source /home/cloud/software/sierra_5.22/install/sierra_init.sh

[cloud@tvj-orc-1 sierra_5.22]$ which adagio
~/software/sierra_5.22/install/apps/bin/adagio

[cloud@tvj-orc-1 sierra_5.22]$ adagio --version
MPI startup(): PMI server not found. Please set I_MPI_PMI_LIBRARY variable if it is not a singleton case.
[0] MPI startup(): Intel(R) MPI Library, Version 2021.12  Build 20240213 (id: 4f55822)
[0] MPI startup(): Copyright (C) 2003-2024 Intel Corporation.  All rights reserved.
[0] MPI startup(): library kind: release
[0] MPI startup(): Load tuning file: "/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12/opt/mpi/etc/tuning_generic_shm.dat"
[0] MPI startup(): ===== CPU pinning =====
[0] MPI startup(): Rank    Pid      Node name  Pin cpu
[0] MPI startup(): 0       1197920  tvj-orc-1  0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31
[0] MPI startup(): I_MPI_CC=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icx
[0] MPI startup(): I_MPI_CXX=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/icpx
[0] MPI startup(): I_MPI_F90=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
[0] MPI startup(): I_MPI_F77=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/compiler/2024.1/bin/ifx
[0] MPI startup(): I_MPI_ROOT=/home/cloud/software/spack/var/spack/environments/sierra522/.spack-env/view/mpi/2021.12
[0] MPI startup(): I_MPI_FABRICS=shm
[0] MPI startup(): I_MPI_DEBUG=5
Application: Adagio
  Executable: /home/cloud/software/sierra_5.22/install/apps/bin/adagio version 5.22.1-0-g7f404f06 built on Feb  5 2025 11:23:25
  Build Options: linux oneapi-2024.1.0 release
    Product: ACME                     (2.9.0)
    Product: Adagio                   (5.22.1-0-g7f404f06)
    Product: FEI                      (5.21.00)
    Product: FETI-DP                  (5.22.1-0-g7f404f06)
    Product: GDSW                     (5.22.1-0-g7f404f06)
    Product: Linux                    (5.14.0-503.21.1.el9_5.x86_64)
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
Sierra                                       1 00:00:00.071 (100.0%) 482992:40:22.092 (100.0%)

Took 0.00120401 seconds to generate the table above.


Memory summary of 1 processor
           Metric               Largest          Processor          Smallest         Processor
---------------------------- -------------- -------------------- -------------- --------------------
Dynamically Allocated (Heap)      2148.4 MB on 0 at 00:00:00.000      2148.4 MB on 0 at 00:00:00.000
Largest Free Fragment (Heap) 2.35764e-01 MB on 0 at 00:00:00.000 2.35764e-01 MB on 0 at 00:00:00.000
         Total Memory In Use      2519.6 MB on 0                      2519.6 MB on 0
           Major Page Faults         0 flts on 0                         0 flts on 0


Performance metric summary
---------------------------------------------------
Min High-water memory usage 143.6 MB
Avg High-water memory usage 143.6 MB
Max High-water memory usage 143.6 MB

Min Available memory per processor 8054.3 MB
Avg Available memory per processor 8054.3 MB
Max Available memory per processor 8054.3 MB

Min No-output time 0.0707 sec
Avg No-output time 0.0707 sec
Max No-output time 0.0707 sec
---------------------------------------------------
There were no errors encountered during parse
There were no warnings encountered during parse
There were no errors encountered during execution
There were no warnings encountered during execution
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
SIERRA execution successful after 00:00:00 (HH:MM:SS)
[cloud@tvj-orc-1 sierra_5.22]$
```



## Sierra Test

* set environment
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
#
# sierra environment load
#
_SIERRA_INSTALL_DIR=/home/cloud/software/sierra_5.22/install
export PATH=${_SIERRA_INSTALL_DIR}/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/engine:${_SIERRA_INSTALL_DIR}/tools/contrib/bin:${_SIERRA_INSTALL_DIR}/tools/sntools/job_scripts:${_SIERRA_INSTALL_DIR}/apps/bin:$PATH
export PYTHONPATH=${_SIERRA_INSTALL_DIR}/tools/tpls/utilities:$PYTHONPATH
```

* run test program
* I use a sample explicit problem that is the dynamic propagation of elastic stress waves in a nested sphere in response to a rapid thermal expansion (due to proton pulse heating). It captures the physics and simulation workflow that I need for spallation target development.

```
[cloud@tvj-orc-1 sierratest]$ ll
total 119728
-rw-------. 1 cloud cloud     5216 Feb  5 11:50 LasagnaOpt_Dynamic.i
-rw-------. 1 cloud cloud     5741 Feb  5 11:50 LasagnaOpt_Dynamic.slurm
-rw-------. 1 cloud cloud 71010020 Feb  5 11:50 LasagnaOpt_Temps.g
-rw-------. 1 cloud cloud 51540580 Feb  5 11:50 LasagnaOpt_noShell.g
-rw-------. 1 cloud cloud    22693 Feb  5 11:50 MatlProps_Structural.i
-rw-------. 1 cloud cloud      692 Feb  5 11:50 README.txt

[cloud@tvj-orc-1 sierratest]$ decomp --processors 32 LasagnaOpt_noShell.g >> decomp.log 2>&1

[cloud@tvj-orc-1 sierratest]$ decomp --processors 32 LasagnaOpt_Temps.g >> decomp.log 2>&1

[cloud@tvj-orc-1 sierratest]$ cat hostfile
tvj-orc-1

[cloud@tvj-orc-1 sierratest]$ mpiexec -n 32 -ppn 32 -f ./hostfile adagio -i LasagnaOpt_Dynamic.i

[cloud@tvj-orc-1 sierratest]$ epu -add_processor -processor_count 32 -extension e LasagnaOpt_Dynamic_Block_Pulses >> epu.log 2>&1

[cloud@tvj-orc-1 sierratest]$ epu -add_processor -processor_count 32 -extension e LasagnaOpt_Dynamic_Shroud_Pulses >> epu.log 2>&1

[cloud@tvj-orc-1 sierratest]$ rm *.e.* *.g.*
```

* The head of the `LasagnaOpt_Dynamic.log` file shows `Simd vector width 2`.  I might be able to improve that if I can figure out how to compile for Zen2 processors.
```
+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----

        Directory /home/cloud/sierratest/
       Executable /home/cloud/software/sierra_5.22/install/apps/bin/adagio
            Built Feb  5 2025 11:23:25
    Build Options linux oneapi-2024.1.0 release
      Run Started Feb  5 2025 12:01:27
             User cloud
     Architecture tvj-orc-1
             Host tvj-orc-1
         Hardware x86_64
          Running Linux
       Processors 32
Simd vector width 2

      Product                  Version            Qualifier
-------------------- ---------------------------- ---------
                ACME 2.9.0
              Adagio 5.22.1-0-g7f404f06
                 FEI 5.21.00
             FETI-DP 5.22.1-0-g7f404f06
                GDSW 5.22.1-0-g7f404f06
               Linux 5.14.0-503.21.1.el9_5.x86_64
    SIERRA Framework 5.22.1-0-g7f404f06
Sandia Toolkit (STK) 5.22.1-0-g7f404f06
          UtilityLib 5.22.1-0-g7f404f06

+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----
Reading LasagnaOpt_Dynamic.i                                Feb  5 2025 12:01:28
```

* The tail of the `LasagnaOpt_Dynamic.log` file shows the speed of execution:
```
### Total Wall Clock Run Time Used ###: 523.53
### Total Number of Steps Taken ###: 2317
```

* And when I run with 16 procs:
```
### Total Wall Clock Run Time Used ###: 995.39
### Total Number of Steps Taken ###: 2317
```

* This compares with ~900 seconds that I was seeing on the OZ3 cluster with slow queue.  I think I can declare success at this point!
* The ORC administrator next granted me access to a full node with an allotment of 250 cpus.  According to `lscpu`, the node used AMD EPYC 9754 128-Core Processors.  When I ran `cpuinfo`, it appeared that all 250 cpus were on the same socket.  That ended up creating problems when I tried to run the test case on anything more than 127 procs.  They were able to fix this, and I was able to run the test case to check scaling:

Arch | Procs | Time (s)
--- | ---: | ---:
AMD EPYC 7702 | 16 | 995
AMD EPYC 7702 | 32 | 524
AMD EPYC 9754 | 64 | 232
AMD EPYC 9754 | 128 | 146
AMD EPYC 9754 | 200 | 99
AMD EPYC 9754 | 250 | 84


## Misc. Notes

### Hostname

I had initially set my hostname to `sierra-benchmark`.  The Sierra compile job quickly crashed:

```
REMARK: EXECUTING COMMAND: 'bake --local --verbose -j 32  --installdir=/home/cloud/software/sierra_5.22/install/apps --customer=generic --package=base --no-build-cache' in directory '/home/cloud/software/sierra_5.22/build/distro/code'
icpx: error: unknown argument: '-showme:version'
Traceback (most recent call last):
  File "/home/cloud/software/sierra_5.22/install/tools/sntools/engine/bake", line 90, in <module>
    sys.exit(mymodule.run())
  File "/home/cloud/software/sierra_5.22/install/tools/sntools/engine/lib/bake.py", line 382, in run
    return BakeEntry().execute()
  File "/home/cloud/software/sierra_5.22/install/tools/sntools/engine/lib/bake.py", line 54, in __init__
    if self.data.mpi.vendor == "intelmpi":
  File "/home/cloud/software/sierra_5.22/install/tools/sntools/runtime_data.py", line 343, in mpi
    subprocess.check_output(["mpicxx", "-showme:version"])
  File "/usr/lib64/python3.9/subprocess.py", line 424, in check_output
    return run(*popenargs, stdout=PIPE, timeout=timeout, check=True,
  File "/usr/lib64/python3.9/subprocess.py", line 528, in run
    raise CalledProcessError(retcode, process.args,
subprocess.CalledProcessError: Command '['mpicxx', '-showme:version']' returned non-zero exit status 1.
REMARK: Command exited with status '1'
```

The `-showme:version` should only be for the Open MPI environment.  I could correctly test by `mpicxx -compile-info`.  When I look at the `runtime_data.py` file, it seems that somehow the script thinks I am a host_name of `ats2`.  It turns out the `hosts.py` utility has a `hosts` variable that is set by stripping anything past `-` from the hostname.  With `sierra` left, it then thought it was on a local system (listed in `hosts.json`) and overrode the settings.  We found this by putting `print(self.hosts)` in the script as a debug tool.  The fix was to delete the VM instance and give the system a better name.

### GCC version

Rocky9 comes with `gcc@11.5.0` so I thought I'd just try to use that natively.  The Sierra compile failed with an error:

```
[cloud@tvj-orc-1 sierra_5.22]$ grep "error:" /home/cloud/software/sierra_5.22/build/build.log
/home/cloud/software/sierra_5.22/build/distro/code/Salinas/util/RandomNumbers.H:24:37: error: unknown type name 'size_t'; did you mean 'std::size_t'?
```

I then tried to go back to using `gcc@10.2.0` but this is now depreciated on the current version of Spack.  I finally found success with `gcc@10.5.0`.
