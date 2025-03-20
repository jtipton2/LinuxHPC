# AWS GovCloud

Amazon created a physically different AWS option for the US Government that follows regulations (including ITAR, EAR) and allows access only by US Citizens (https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-differences.html).

## AWS ParallelCluster 
* Overview
  * Open source tool created by AWS to create HPC environments
  * https://aws.amazon.com/hpc/parallelcluster/
  * https://github.com/aws/aws-parallelcluster
* Background Materials
  * https://github.com/aws-samples/aws-hpc-tutorials
  * Links to: https://www.hpcworkshops.com/
  * Material is already about 2 years old
    * https://www.hpcworkshops.com/03-parallel-cluster-cli/11-1stjob.html
    * https://www.hpcworkshops.com/05-fsx-for-lustre/09-performance-test.html
    * https://www.hpcworkshops.com/07-efa/04-complie-run-osu.html
* Best Tutorial Materials
  * https://catalog.workshops.aws/nwp-on-aws/en-US
  * https://aws.amazon.com/blogs/hpc/install-optimized-software-with-spack-configs-for-aws-parallelcluster/
* New(er) Material to Explore
  *  https://aws.amazon.com/blogs/hpc/you-told-us-we-needed-to-re-think-hpc-in-the-cloud-so-we-did/
  *  https://aws.amazon.com/hpc/resources/


## Access Proceedure

### Part I - Install PCLuster
* cloudtamer.ornl.gov
* Cloud Management --> Cloud Access Roles --> My Cloud Access Roles --> STS-target-Engineering-Admin --> Cloud Access --> Select an Account --> Web Access
* From there I can launch a console and follow the pcluster CLI instructions to create a HPC instance
* https://docs.aws.amazon.com/parallelcluster/latest/ug/install-v3-virtual-environment.html

```
[cloudshell-user@ip-10-0-158-59 ~]$ source ~/apc-ve/bin/activate
(apc-ve) [cloudshell-user@ip-10-0-158-59 ~]$ export AWS_ACCESS_KEY_ID=XXXX
(apc-ve) [cloudshell-user@ip-10-0-158-59 ~]$ export AWS_SECRET_ACCESS_KEY=XXXX
(apc-ve) [cloudshell-user@ip-10-0-158-59 ~]$ export AWS_SESSION_TOKEN=XXXX
(apc-ve) [cloudshell-user@ip-10-0-158-59 ~]$ pcluster configure --config config-hpc.yaml
INFO: Configuration file config-hpc.yaml will be written.
Press CTRL-C to interrupt the procedure.


Allowed values for AWS Region ID:
1. us-gov-east-1
2. us-gov-west-1
AWS Region ID [us-gov-west-1]:  
Allowed values for EC2 Key Pair Name:
1. tipton-aws-1
2. tipton-aws-2
EC2 Key Pair Name [tipton-aws-1]: 1
Allowed values for Scheduler:
1. slurm
2. awsbatch
Scheduler [slurm]: 
Allowed values for Operating System:
1. alinux2
2. alinux2023
3. ubuntu2004
4. ubuntu2204
5. rhel8
6. rocky8
7. rhel9
8. rocky9
Operating System [alinux2]: 
Head node instance type [t3.micro]: c5n.18xlarge
Number of queues [1]: 
Name of queue 1 [queue1]: 
Number of compute resources for queue1 [1]: 
Compute instance type for compute resource 1 in queue1 [t3.micro]: c5n.18xlarge
The EC2 instance selected supports enhanced networking capabilities using Elastic Fabric Adapter (EFA). EFA enables you to run applications requiring high levels of inter-node communications at scale on AWS at no additional charge (https://docs.aws.amazon.com/parallelcluster/latest/ug/efa-v3.html).
Enable EFA on c5n.18xlarge (y/n) [y]: y
Maximum instance count [10]: 
Enabling EFA requires compute instances to be placed within a Placement Group. Please specify an existing Placement Group name or leave it blank for ParallelCluster to create one.
Placement Group name []: 
Automate VPC creation? (y/n) [n]: y
Allowed values for Availability Zone:
1. us-gov-west-1a
2. us-gov-west-1b
3. us-gov-west-1c
Availability Zone [us-gov-west-1a]: 
Allowed values for Network Configuration:
1. Head node in a public subnet and compute fleet in a private subnet
2. Head node and compute fleet in the same public subnet
Network Configuration [Head node in a public subnet and compute fleet in a private subnet]: 
Beginning VPC creation. Please do not leave the terminal until the creation is finalized
Creating CloudFormation stack...
Do not leave the terminal until the process has finished.
Stack Name: parallelclusternetworking-pubpriv-20250307185832 (id: XXXX)
Status: parallelclusternetworking-pubpriv-20250307185832 - CREATE_COMPLETE      
The stack has been created.
Configuration file written to config-hpc.yaml
You can edit your configuration file or simply run 'pcluster create-cluster --cluster-configuration config-hpc.yaml --cluster-name cluster-name --region us-gov-west-1' to create your cluster.
(apc-ve) [cloudshell-user@ip-10-0-158-59 ~]$
```

### Part II - Launch HPC Instance
* First, I tweak the `config-hpc.yaml` file based on the template from a tutorial
* https://catalog.workshops.aws/nwp-on-aws/en-US/1-create-cluster/a-create-cluster
```
Region: us-gov-west-1
Image:
  Os: alinux2
HeadNode:
  InstanceType: c5n.18xlarge
  Networking:
    SubnetId: subnet-XXXX
  LocalStorage:
    RootVolume:
      Size: 50
  Dcv:
    Enabled: true
  Ssh:
    KeyName: tipton-aws-1
Scheduling:
  Scheduler: slurm
  SlurmQueues:
  - Name: queue1
    ComputeResources:
    - Name: c5n18xlarge
      Instances:
      - InstanceType: c5n.18xlarge
      MinCount: 0
      MaxCount: 10
      DisableSimultaneousMultithreading: true
      Efa:
        Enabled: true
    Networking:
      PlacementGroup:
        Enabled: true
      SubnetIds:
      - subnet-XXXX
SharedStorage:
  - Name: FsxLustre0
    StorageType: FsxLustre
    MountDir: /shared
    FsxLustreSettings:
      StorageCapacity: 1200
      DeploymentType: SCRATCH_2
      DataCompressionType: LZ4
```

* Next, launch the cluster
```
(apc-ve) [cloudshell-user@ip-10-0-158-59 ~]$ pcluster create-cluster --cluster-configuration config-hpc.yaml --cluster-name OZ4AWS --region us-gov-west-1
{
  "cluster": {
    "clusterName": "OZ4AWS",
    "cloudformationStackStatus": "CREATE_IN_PROGRESS",
    "cloudformationStackArn": "arn:aws-us-gov:cloudformation:us-gov-west-1:XXXX",
    "region": "us-gov-west-1",
    "version": "3.12.0",
    "clusterStatus": "CREATE_IN_PROGRESS",
    "scheduler": {
      "type": "slurm"
    }
  },
  "validationMessages": [
    {
      "level": "INFO",
      "type": "DeletionPolicyValidator",
      "message": "The DeletionPolicy is set to Delete. The storage 'FsxLustre0' will be deleted when you remove it from the configuration when performing a cluster update or deleting the cluster."
    },
    {
      "level": "WARNING",
      "type": "DcvValidator",
      "message": "With this configuration you are opening DCV port XXXX to the world (0.0.0.0/0). It is recommended to restrict access."
    }
  ]
}
```
* Once the instance is launched, I can go back to the web interface and launch the EC2 widget.  That gives me the address of the instance.
* Then I can follow these instructions to access the EC2 HPC instance via Putty:  https://ornl.servicenowservices.com/kb_view.do?sysparm_article=KB0011577


### Part III - Install Spack & Needed Packages
* Get needed packages for Sierra install
```
sudo yum install autoconf automake bison bzip2 diffutils flex libtool m4 python3-devel.x86_64 libX11-devel.x86_64 libcurl-devel.x86_64
which python3
which python
sudo ln -sf /usr/bin/python3 /usr/bin/python
```

* Install Spack
  * https://aws.amazon.com/blogs/hpc/install-optimized-software-with-spack-configs-for-aws-parallelcluster/
  * > Third, you will need to decide on an instance type for your cluster’s head node. The Spack installer script uses the head node CPU architecture to determine what compilers and optimizations to install. Thus, we recommend matching the CPU generation and architecture of your head node to the instances you will run your HPC jobs on. For example, if you’re building a cluster with hpc6id.32xlarge compute nodes, a c6i.xlarge would be an appropriate head node instance type. Be aware that Spack does not support t2, t3, or t4 family instances, which some people use for lightweight head nodes. Choose from c, m, or r instance types instead to avoid this limitation.
  * > Finally, Spack needs a shared directory where you can install it. This helps ensure Spack and the software it can manage are accessible from every node in your cluster. If your cluster will run in a single Availability Zone, we recommend you use Amazon FSx for Lustre for shared storage. 
```
curl -sL -O https://raw.githubusercontent.com/spack/spack-configs/main/AWS/parallelcluster/postinstall.sh
sudo bash postinstall.sh
tail -f /var/log/spack-postinstall.log
```

* This appears to install `intel-oneapi-compilers` using `gcc@12`.  I already know this version of `gcc` will not work with Sierra, so remove the `intel-oneapi-compilers` package.
* Might need to also remove this from `/shared/spack/etc/spack/compilers.yaml`
* I ended up with duplicate entries and got confused.  I ended up uninstalling using the hash.
```
# had to remove the other version of oneapi built with gcc@12
# also needed to then remove this from the compilers
spack find --format "{name}-{version}-{hash}"
spack uninstall /46k6kokhbateyvghfxsm5uykwlu4odcm
```

* install GCC compiler and add to spack compilers yaml
```
spack install -j 32 gcc@10.5.0
spack compiler add "$(spack location -i gcc@10.5.0)"
# this made a file entry in /home/ec2-user/.spack/linux/compilers.yaml
# had to add a prepend path to the compiler for binutils following what was done in /shared/spack/etc/spack/compilers.yaml
spack compilers
```

* install Intel OneAPI compilers and add to spack compilers yaml
```
spack install -j 64 intel-oneapi-compilers@2024.1.0%gcc@10.5.0
spack load intel-oneapi-compilers
spack compiler find
```

* install `libfabric`, `intel-oneapi-mpi`, and `intel-oneapi-mkl`
* https://catalog.workshops.aws/nwp-on-aws/en-US/1-create-cluster/e-install-intel
```
spack install -j 64 libfabric fabrics=efa,tcp,udp,sockets,verbs,shm,mrail,rxd,rxm %oneapi
spack install -j 64 intel-oneapi-mpi+external-libfabric%oneapi
spack install -j 64 intel-oneapi-mkl@2024.2.0+cluster^intel-oneapi-mpi%oneapi
```


### Part IV - Install Sierra
* The Sierra install spins up its own spack instance to install third party libraries (TPLs).  It will crash if Spack is already running on the cluster.  Nicely, the AWS Spack script automatically integrates with the module environment.  I can thus copy the module scripts, make my own Sierra module, and disable spack from loading on the cluster.
* Use `module avail` to see listings.  Then use `module show XXXX` to get the settings:

```
[ec2-user@ip-10-0-0-39 ~]$ module show intel-oneapi-compilers/2024.1.0-gcc-10.5.0-ic4d364
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-x86_64_v3/intel-oneapi-compilers/2024.1.0-gcc-10.5.0-ic4d364:

module-whatis    Intel oneAPI Compilers. Includes: icx, icpx, ifx, and ifort. Releases before 2024.0 include icc/icpc LICENSE INFORMATION: By downloading and using this software, you agree to the terms and conditions of the software license agreements at https://intel.ly/393CijO.
conflict         intel-oneapi-compilers
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/.
prepend-path     ACL_BOARD_VENDOR_PATH /opt/Intel/OpenCLFPGA/oneAPI/Boards
setenv           CMPLR_ROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga/include
prepend-path     DIAGUTIL_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/etc/compiler/sys_check/sys_check.sh
setenv           FPGA_VARS_ARGS
setenv           FPGA_VARS_DIR /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga
setenv           INTELFPGAOCLSDKROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga/host/linux64/lib:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/compiler/lib:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib
prepend-path     NLSPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib/compiler/locale/%l_%t/%N
prepend-path     OCL_ICD_FILENAMES libintelocl_emu.so:libalteracl.so:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib/libintelocl.so
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib/pkgconfig
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1
prepend-path     MANPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/share/man
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga/bin
setenv           CC /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/icx
setenv           CXX /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/icpx
setenv           F77 /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/ifx
setenv           FC /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/ifx
setenv           INTEL_ONEAPI_COMPILERS_ROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl
append-path      MANPATH
-------------------------------------------------------------------


[ec2-user@ip-10-0-0-39 ~]$ module show intel-oneapi-mpi/2021.14.0-oneapi-2024.1.0-qeh6djk
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-skylake_avx512/intel-oneapi-mpi/2021.14.0-oneapi-2024.1.0-qeh6djk:

module-whatis    Intel MPI Library is a multifabric message-passing library that implements the open-source MPICH specification. Use the library to create, maintain, and test advanced, complex applications that perform better on high-performance computing (HPC) clusters based on Intel processors.
module           load intel-oneapi-runtime/2024.1.0-oneapi-2024.1.0-qpf3qxi
module           load libfabric/1.22.0-oneapi-2024.1.0-wvnsyjs
conflict         intel-oneapi-mpi
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/.
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib64
prepend-path     CLASSPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/share/java/mpi.jar
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/include
setenv           I_MPI_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/lib
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/lib
prepend-path     MANPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/share/man:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/share/man:/usr/share/man::/opt/slurm/share/man:
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/bin
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/lib/pkgconfig
setenv           INTEL_ONEAPI_MPI_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r
append-path      MANPATH
-------------------------------------------------------------------


[ec2-user@ip-10-0-0-39 ~]$ module show intel-oneapi-mkl/2024.2.0-oneapi-2024.1.0-2kfcrkx
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-skylake_avx512/intel-oneapi-mkl/2024.2.0-oneapi-2024.1.0-2kfcrkx:

module-whatis    Intel oneAPI Math Kernel Library (Intel oneMKL; formerly Intel Math Kernel Library or Intel MKL), is a library of optimized math routines for science, engineering, and financial applications. Core math functions include BLAS, LAPACK, ScaLAPACK, sparse solvers, fast Fourier transforms, and vector math.
module           load intel-oneapi-mpi/2021.14.0-oneapi-2024.1.0-qeh6djk
module           load intel-oneapi-runtime/2024.1.0-oneapi-2024.1.0-qpf3qxi
module           load intel-oneapi-tbb/2022.0.0-gcc-12.4.0-f5ibkfg
conflict         intel-oneapi-mkl
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/.
setenv           MKLROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib/cmake
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/include
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib/
prepend-path     NLSPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/share/locale/%l_%t/%N
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/bin/
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib/pkgconfig
setenv           INTEL_ONEAPI_MKL_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt
-------------------------------------------------------------------


[ec2-user@ip-10-0-0-39 ~]$ module show libfabric/1.22.0-oneapi-2024.1.0-wvnsyjs
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-skylake_avx512/libfabric/1.22.0-oneapi-2024.1.0-wvnsyjs:

module-whatis    The Open Fabrics Interfaces (OFI) is a framework focused on exporting fabric communication services to applications.
conflict         libfabric
prepend-path     PATH /opt/amazon/efa/bin
prepend-path     MANPATH /opt/amazon/efa/share/man
prepend-path     PKG_CONFIG_PATH /opt/amazon/efa/lib64/pkgconfig
prepend-path     CMAKE_PREFIX_PATH /opt/amazon/efa/.
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib64
setenv           LIBFABRIC_ROOT /opt/amazon/efa/
append-path      MANPATH
-------------------------------------------------------------------


[ec2-user@ip-10-0-0-39 ~]$ module show intel-oneapi-runtime/2024.1.0-oneapi-2024.1.0-qpf3qxi
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-skylake_avx512/intel-oneapi-runtime/2024.1.0-oneapi-2024.1.0-qpf3qxi:

module-whatis    Package for OneAPI compiler runtime libraries redistributables. LICENSE INFORMATION: By downloading and using this software, you agree to the terms and conditions of the software license agreements at https://intel.ly/393CijO.
conflict         intel-oneapi-runtime
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-runtime-2024.1.0-qpf3qxifb6wkcie2uxbcftwtzen3t3qb/.
setenv           INTEL_ONEAPI_RUNTIME_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-runtime-2024.1.0-qpf3qxifb6wkcie2uxbcftwtzen3t3qb
-------------------------------------------------------------------


[ec2-user@ip-10-0-0-39 ~]$ module show intel-oneapi-tbb/2022.0.0-gcc-12.4.0-f5ibkfg
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-x86_64_v4/intel-oneapi-tbb/2022.0.0-gcc-12.4.0-f5ibkfg:

module-whatis    Intel oneAPI Threading Building Blocks (oneTBB) is a flexible performance library that simplifies the work of adding parallelism to complex applications across accelerated architectures, even if you are not a threading expert.
conflict         intel-oneapi-tbb
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/.
setenv           TBBROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/..
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/..
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../include
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../lib/intel64/gcc4.8
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../lib/intel64/gcc4.8
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../lib/pkgconfig
setenv           INTEL_ONEAPI_TBB_ROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc
-------------------------------------------------------------------


[ec2-user@ip-10-0-0-39 ~]$ module show gcc/10.5.0-gcc-12.4.0-ypgh42q
-------------------------------------------------------------------
/shared/spack/share/spack/modules/linux-amzn2-skylake_avx512/gcc/10.5.0-gcc-12.4.0-ypgh42q:

module-whatis    The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, and Go, as well as libraries for these languages.
module           load gmp/6.3.0-gcc-12.4.0-nvqmuj3
module           load mpc/1.3.1-gcc-12.4.0-6qm7spa
module           load mpfr/4.2.1-gcc-12.4.0-infmbmz
module           load zlib-ng/2.2.1-gcc-12.4.0-ywvdco5
module           load zstd/1.5.6-gcc-12.4.0-r4wkqmj
conflict         gcc
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/bin
prepend-path     MANPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/share/man
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/.
setenv           CC /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/bin/gcc
setenv           FC /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/bin/gfortran
setenv           F77 /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/bin/gfortran
setenv           GCC_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg
append-path      MANPATH
-------------------------------------------------------------------
```

* create a custom module `sierraenv.mod` to load these environments
  * I added the `I_MPI_XX` settings, otherwise there's a bug in IMPI that tries to find `icc` which is not installed
  * I added the GCC path to `binutils` which is what is listed in the `compilers.yaml` file
```
#%Module

module-whatis   "Sets up environment to be able to compile Sierra 5.22"

# from module show intel-oneapi-compilers/2024.1.0-gcc-10.5.0-ic4d364
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/.
prepend-path     ACL_BOARD_VENDOR_PATH /opt/Intel/OpenCLFPGA/oneAPI/Boards
setenv           CMPLR_ROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga/include
prepend-path     DIAGUTIL_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/etc/compiler/sys_check/sys_check.sh
#setenv           FPGA_VARS_ARGS
setenv           FPGA_VARS_DIR /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga
setenv           INTELFPGAOCLSDKROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga/host/linux64/lib:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/compiler/lib:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib
prepend-path     NLSPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib/compiler/locale/%l_%t/%N
prepend-path     OCL_ICD_FILENAMES libintelocl_emu.so:libalteracl.so:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib/libintelocl.so
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/lib/pkgconfig
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1
prepend-path     MANPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/share/man
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/opt/oclfpga/bin
setenv           CC /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/icx
setenv           CXX /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/icpx
setenv           F77 /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/ifx
setenv           FC /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/ifx
setenv           INTEL_ONEAPI_COMPILERS_ROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl
#append-path      MANPATH

# from module show intel-oneapi-runtime/2024.1.0-oneapi-2024.1.0-qpf3qxi
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-runtime-2024.1.0-qpf3qxifb6wkcie2uxbcftwtzen3t3qb/.
setenv           INTEL_ONEAPI_RUNTIME_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-runtime-2024.1.0-qpf3qxifb6wkcie2uxbcftwtzen3t3qb

# from module show libfabric/1.22.0-oneapi-2024.1.0-wvnsyjs
prepend-path     PATH /opt/amazon/efa/bin
prepend-path     MANPATH /opt/amazon/efa/share/man
prepend-path     PKG_CONFIG_PATH /opt/amazon/efa/lib64/pkgconfig
prepend-path     CMAKE_PREFIX_PATH /opt/amazon/efa/.
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib64
setenv           LIBFABRIC_ROOT /opt/amazon/efa/
#append-path      MANPATH

# from module show intel-oneapi-mpi/2021.14.0-oneapi-2024.1.0-qeh6djk
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/.
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib
prepend-path     LD_LIBRARY_PATH /opt/amazon/efa/lib64
prepend-path     CLASSPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/share/java/mpi.jar
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/include
setenv           I_MPI_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/lib
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/lib
prepend-path     MANPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/share/man:/shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/share/man:/usr/share/man::/opt/slurm/share/man:
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/bin
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r/mpi/2021.14/lib/pkgconfig
setenv           INTEL_ONEAPI_MPI_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mpi-2021.14.0-qeh6djkva3u5nk7diymkg7bakhuzhe2r
#append-path      MANPATH
setenv           I_MPI_CC /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/icx
setenv           I_MPI_CXX /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/icpx
setenv           I_MPI_F77 /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/ifx
setenv           I_MPI_F90 /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-10.5.0/intel-oneapi-compilers-2024.1.0-ic4d364p36e4wbteuxpiq4pbzeazv4nl/compiler/2024.1/bin/ifx

# from module show intel-oneapi-tbb/2022.0.0-gcc-12.4.0-f5ibkfg
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/.
setenv           TBBROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/..
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/..
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../include
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../lib/intel64/gcc4.8
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../lib/intel64/gcc4.8
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc/tbb/2022.0/env/../lib/pkgconfig
setenv           INTEL_ONEAPI_TBB_ROOT /shared/spack/opt/spack/linux-amzn2-x86_64_v4/gcc-12.4.0/intel-oneapi-tbb-2022.0.0-f5ibkfg75ntm5bdcetf233ibnkr5cggc

# from module show intel-oneapi-mkl/2024.2.0-oneapi-2024.1.0-2kfcrkx
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/.
setenv           MKLROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2
prepend-path     CMAKE_PREFIX_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib/cmake
prepend-path     CPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/include
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib
prepend-path     LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib/
prepend-path     NLSPATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/share/locale/%l_%t/%N
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/bin/
prepend-path     PKG_CONFIG_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt/mkl/2024.2/lib/pkgconfig
setenv           INTEL_ONEAPI_MKL_ROOT /shared/spack/opt/spack/linux-amzn2-skylake_avx512/oneapi-2024.1.0/intel-oneapi-mkl-2024.2.0-2kfcrkxjdvlgascv7zu4b5vj3ry4yqbt

# finally set the location to GCC@10.5.0
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/bin
prepend-path     LD_LIBRARY_PATH /shared/spack/opt/spack/linux-amzn2-skylake_avx512/gcc-12.4.0/gcc-10.5.0-ypgh42qqo2n4aiweu2de6bg6muf7klrg/lib64
prepend-path     PATH /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-7.3.1/binutils-2.37-qvccg7zpskturysmr4bzbsfrx34kvazo/bin
```

* Now you can remove the `/etc/profile.d/spack.sh` entry to keep it from autoloading.  It was just running `source /shared/spack/share/spack/setup-env.sh` which can be done manually.
* SSH into a new Putty terminal, and run the Sierra installation
* `nohup` is important to run the installer in the background so it doesn't terminate when the shell is closed
```
module load /home/ec2-user/sierraenv.mod
SIERRA_ROOT=/shared/sierra_5.22
cd ${SIERRA_ROOT}
nohup /shared/source/sierra_setup.py --procs=32 --keep-build-dir &
```
