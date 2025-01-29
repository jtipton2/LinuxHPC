# Sierra Installation on a HPC using Spack
>*Joseph B. Tipton, Jr.*
>*December 2024 - January 2025*

These are my personal notes as I try to learn how to manage a software environment and compile the Sierra simulation suite on a linux HPC.  

## Hardware

The current HPC setup for testing includes the following components:
* headnode
  - 2 x Intel Xeon Scalable Gold 6426Y, 2.5GHz (16-Core) "Sapphire Rapids"
  - 16 x 16GB PC5-38400 4800MHz DDR5 ECC RDIMM
  - NVIDIA 100-Gigabit HDR100 InfiniBand Adapter (ConnectX-6)
* network storage
  - 180TB RAID z2 vdev
  - NVIDIA 100-Gigabit HDR100 InfiniBand Adapter (ConnectX-6)
* switch
  - NVIDIA Mellanox QM8700 - 40-Port Managed HDR InfiniBand 200Gb/s Switch
* compute nodes (x2)
  - NVIDIA Mellanox Technologies MT27500 Family (ConnectX-3)
  - Intel(R) Xeon(R) CPU E5-2637 v2 @ 3.50GHz (8-Core) "Ivy Bridge"

I'm still trying to understand the correct software drivers that should be used for this hardware configuration _specifically as it relates to the Infiniband communication_.  Some notes are found in the  [Intel oneAPI Hardware Driver Troubleshooting](Sierra_Intel_oneAPI.md#hardware-driver-troubleshooting) portion of this project.  Other internet resources are:
* https://developer.nvidia.com/networking/infiniband-software
* https://lists.openfabrics.org/pipermail/libfabric-users/2024-May/001037.html
* OFED
  - https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/
  - https://stackoverflow.com/questions/58622347/what-is-the-difference-between-ofed-mlnx-ofed-and-the-inbox-driver
  - https://www.openfabrics.org/ofed-for-linux/
  - https://serverfault.com/questions/1048740/infiniband-drivers-ofed-or-distro-included
* https://easybuild.io/eum22/011_eum22_mpi_easybuild.pdf


## Benchmark Simulation Summary

The new HPC is called "BERNIE" and uses a new headnode, a new file server, and a new Mellanox Infiniband switch.  For now, ITSD have taken 2 compute nodes from the old cluster (node001, node003) and moved them to the new cluster (cnode001, cnode003).  These are older nodes, each with 8 CPUs with Intel _Ivy Bridge_ architecture.  In this way, I can run a benchmark Sierra explicit simulation (i.e. adagio) to compare performance on the same hardware using different verions of Sierra and different compiler environments.

I use a sample explicit problem that is the dynamic propagation of elastic stress waves in a nested sphere in response to a rapid thermal expansion (due to proton pulse heating). It captures the physics and simulation workflow that I need for spallation target development.

The table below shows comparisons with the simulation run for 20 &mu;s:

Headnode | Compute Nodes | Sierra Version | Environment | OFI | Architecture | Time
--- | --- | --- | --- | --- | --- | ---
OZ3 | 1 & 3 | v4.56.4 | intel-19.0.3 | | fat_x86 | 900 s
BERNIE | 1 & 3 | v5.14 | gcc-8.5.0 | tcp (IPoIB) | fat_x86 | 1,497 s
BERNIE | 1 & 3 | v5.14 | gcc-10.2.0 | tcp (IPoIB) | fat_x86 | 1,558 s
BERNIE | 1 & 3 | v5.22.1 | oneapi-2024.1.0 | tcp (IPoIB) | fat_x86 | 1,566 s
BERNIE | 1 & 3 | v5.22.1 | oneapi-2024.1.0 | mlx | fat_x86 | 1,415 s
BERNIE | 1 & 3 | v5.22.1 | oneapi-2024.1.0 | mlx | sandybridge | 1,372 s

To try and get a better idea of the impact of communication between nodes, I ran the simulation for 10 &mu;s with the following results:

Headnode | Compute Nodes | Sierra Version | Environment | OFI | Architecture | Time
--- | --- | --- | --- | --- | --- | ---
OZ3 | 4 (8 cpus) | v4.56.4 | intel-19.0.3 | | fat_x86 | 873 s
OZ3 | 4 (4 cpus) & 5 (4 cpus) | v4.56.4 | intel-19.0.3 | | fat_x86 | 869 s
BERNIE | 1 (8 cpus) | v5.22.1 | oneapi-2024.1.0 | mlx | fat_x86 | 1,298 s
BERNIE | 1 (4 cpus) & 3 (4 cpus) | v5.22.1 | oneapi-2024.1.0 | tcp | fat_x86 | 1,348 s

As you can see, performance on the new cluster _using the same compute hardware_ is about 50% slower.  I'm still troubleshooting to understand the source of this slowdown.

## Software

My notes on learning the software installation process in a Linux environment are structured as follows:

### [Spack](Spack.md)
_Notes about using Spack to setup linux environments with specific software combinations, especially compilers._

### [Sierra_GCC_OpenMPI](Sierra_GCC_OpenMPI.md) 
_Notes about testing a GCC/OpenMPI environment, compiling Sierra, and running on Slurm._

### [Sierra_Intel_oneAPI](Sierra_Intel_oneAPI.md) 
_Notes about testing an Intel oneAPI environment, compiling Sierra, and running on Slurm._




















