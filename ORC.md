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
[cloud@sierra-benchmark ~]$ cat /proc/cpuinfo

...

processor       : 31
vendor_id       : AuthenticAMD
cpu family      : 25
model           : 160
model name      : AMD EPYC 9754 128-Core Processor
stepping        : 2
microcode       : 0xaa00213
cpu MHz         : 2250.000
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
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves avx512_bf16 clzero xsaveerptr wbnoinvd arat npt lbrv nrip_save tsc_scale vmcb_clean pausefilter pfthreshold v_vmsave_vmload avx512vbmi umip pku ospke avx512_vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg avx512_vpopcntdq la57 rdpid fsrm arch_capabilities
bugs            : sysret_ss_attrs null_seg spectre_v1 spectre_v2 spec_store_bypass srso
bogomips        : 4500.00
TLB size        : 1024 4K pages
clflush size    : 64
cache_alignment : 64
address sizes   : 48 bits physical, 57 bits virtual
power management:
```

## Software

```
[cloud@sierra-benchmark ~]$ gcc -dumpmachine
x86_64-redhat-linux

[cloud@sierra-benchmark ~]$ gcc --version
gcc (GCC) 11.5.0 20240719 (Red Hat 11.5.0-2)
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

[cloud@sierra-benchmark ~]$ python3 --version
Python 3.9.21

[cloud@sierra-benchmark ~]$ sudo su

[root@sierra-benchmark cloud]# dnf install autoconf automake bison bzip2 diffutils flex libtool m4 python3-devel.x86_64 libX11-devel.x86_64
Last metadata expiration check: 3:07:55 ago on Mon Feb  3 06:21:22 2025.
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
Dependencies resolved.
Nothing to do.
Complete!

[root@sierra-benchmark cloud]# exit
exit
```

## Spack
```
[cloud@sierra-benchmark ~]$ mkdir software

[cloud@sierra-benchmark software]$ git clone https://github.com/spack/spack.git
Cloning into 'spack'...
remote: Enumerating objects: 624261, done.
remote: Counting objects: 100% (108/108), done.
remote: Compressing objects: 100% (57/57), done.
remote: Total 624261 (delta 79), reused 51 (delta 51), pack-reused 624153 (from 3)
Receiving objects: 100% (624261/624261), 210.95 MiB | 4.49 MiB/s, done.
Resolving deltas: 100% (296544/296544), done.
Updating files: 100% (12185/12185), done.

[cloud@sierra-benchmark software]$ source spack/share/spack/setup-env.sh

[cloud@sierra-benchmark software]$ echo $SPACK_ROOT
/home/cloud/software/spack

[cloud@sierra-benchmark software]$ spack external find
==> The following specs have been detected on this system and added to /home/cloud/.spack/packages.yaml
binutils@2.35.2  curl@7.76.1    findutils@4.8.0  gettext@0.21  gmake@4.3     m4@1.4.19      openssl@3.2.2  pkgconf@1.7.3  tar@1.34
coreutils@8.32   diffutils@3.7  gawk@5.1.0       git@2.43.5    groff@1.22.4  openssh@8.7p1  perl@5.32.1    sed@4.8        zlib@1.2.11
autoconf@2.69  automake@1.16.2  bison@3.7.4  flex@2.6.4  libtool@2.4.6

[cloud@sierra-benchmark ~]$ spack compilers
==> No compilers available. Run `spack compiler find` to autodetect compilers

[cloud@sierra-benchmark ~]$ spack compiler find
==> Added 1 new compiler to /home/cloud/.spack/linux/compilers.yaml
    gcc@11.5.0
==> Compilers are defined in the following files:
    /home/cloud/.spack/linux/compilers.yaml

[cloud@sierra-benchmark ~]$ spack compilers
==> Available compilers
-- gcc rocky9-x86_64 --------------------------------------------
gcc@11.5.0
```

>[!Important]
>Later during the Intel oneAPI install, CMake install gave an error regarding Curl
>To resolve, I also had to `dnf install libcurl-devel.x86_64`


## Intel oneAPI Environment

**sierra522.yaml**
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
```


**Environment Install**
```
[cloud@sierra-benchmark software]$ spack env create sierra522 sierra522.yaml
==> Created environment sierra522 in: /home/cloud/software/spack/var/spack/environments/sierra522
==> Activate with: spack env activate sierra522

[cloud@sierra-benchmark software]$ spack env activate -p sierra522

[sierra522] [cloud@sierra-benchmark software]$ spack concretize
==> Fetching https://ghcr.io/v2/spack/bootstrap-buildcache-v1/blobs/sha256:82ec278bef26c42303a2c2c888612c0d37babef615bc9a0003530e0b8b4d3d2c
==> Fetching https://ghcr.io/v2/spack/bootstrap-buildcache-v1/blobs/sha256:0c5831932608e7b4084fc6ce60e2b67b77dab76e5515303a049d4d30cd772321
==> Installing "clingo-bootstrap@=spack%gcc@=10.2.1~docs+ipo+optimized+python+static_libstdcpp build_system=cmake build_type=Release generator=make patches=bebb819,ec99431 arch=linux-centos7-x86_64" from a buildcache
==> Warning: The default behavior of tarfile extraction has been changed to disallow common exploits (including CVE-2007-4559). By default, absolute/parent paths are disallowed and some mode bits are cleared. See https://access.redhat.com/articles/7004769 for more details.
==> Concretized 3 specs:
 -   uwad3jv  intel-oneapi-compilers@2024.1.0%gcc@11.5.0~amd+envmods~nvidia build_system=generic arch=linux-rocky9-x86_64_v4
 -   2ar6fxk      ^gcc-runtime@11.5.0%gcc@11.5.0 build_system=generic arch=linux-rocky9-x86_64_v4
[e]  wg5xyks      ^glibc@2.34%gcc@11.5.0 build_system=autotools arch=linux-rocky9-x86_64_v4
 -   ahwnesp      ^patchelf@0.17.2%gcc@11.5.0 build_system=autotools arch=linux-rocky9-x86_64_v4
[e]  n6b3fv4          ^gmake@4.3%gcc@11.5.0~guile build_system=generic patches=599f134 arch=linux-rocky9-x86_64_v4
 -   evm7ow4  intel-oneapi-mkl@2024.2.0%gcc@11.5.0+cluster+envmods~gfortran~ilp64+shared build_system=generic mpi_family=mpich threads=none arch=linux-rocky9-x86_64_v4
 -   iw6kavq      ^intel-tbb@2022.0.0%gcc@11.5.0~ipo+shared+tm build_system=cmake build_type=Release cxxstd=default generator=make arch=linux-rocky9-x86_64_v4
 -   64ovbhr          ^cmake@3.31.5%gcc@11.5.0~doc+ncurses+ownlibs~qtgui build_system=generic build_type=Release arch=linux-rocky9-x86_64_v4
[e]  bxd4dez              ^curl@7.76.1%gcc@11.5.0+gssapi+ldap~libidn2~librtmp~libssh~libssh2+nghttp2 build_system=autotools libs=shared,static tls=openssl arch=linux-rocky9-x86_64_v4
 -   kez2ryj              ^ncurses@6.5%gcc@11.5.0~symlinks+termlib abi=none build_system=autotools patches=7a351bc arch=linux-rocky9-x86_64_v4
 -   l5gd3ws              ^zlib-ng@2.2.3%gcc@11.5.0+compat+new_strategies+opt+pic+shared build_system=autotools arch=linux-rocky9-x86_64_v4
 -   44zqaq7          ^hwloc@2.11.1%gcc@11.5.0~cairo~cuda~gl~level_zero~libudev+libxml2~nvml~opencl+pci~rocm build_system=autotools libs=shared,static arch=linux-rocky9-x86_64_v4
 -   r4lgjol              ^libpciaccess@0.17%gcc@11.5.0 build_system=autotools arch=linux-rocky9-x86_64_v4
 -   hcazmcf                  ^util-macros@1.20.1%gcc@11.5.0 build_system=autotools arch=linux-rocky9-x86_64_v4
 -   d7doi3t              ^libxml2@2.13.5%gcc@11.5.0~http+pic~python+shared build_system=autotools arch=linux-rocky9-x86_64_v4
 -   vxdyael                  ^libiconv@1.17%gcc@11.5.0 build_system=autotools libs=shared,static arch=linux-rocky9-x86_64_v4
 -   ghvwc6i                  ^xz@5.4.6%gcc@11.5.0~pic build_system=autotools libs=shared,static arch=linux-rocky9-x86_64_v4
[e]  pxe5cvs              ^pkgconf@1.7.3%gcc@11.5.0 build_system=autotools arch=linux-rocky9-x86_64_v4
 -   kyh7l2f  intel-oneapi-mpi@2021.12.1%gcc@11.5.0~classic-names+envmods~external-libfabric~generic-names~ilp64 build_system=generic arch=linux-rocky9-x86_64_v4


```
