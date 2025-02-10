# Sierra Installation on a HPC using Spack
>*Joseph B. Tipton, Jr.*
>*December 2024 - January 2025*

These are my personal notes as I try to learn how to manage a software environment and compile the Sierra simulation suite on a linux HPC.  

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

### [ORNL Research Cloud](ORC.md) 
_Notes about testing Sierra on a virtual machine using AMD processosrs on the ORNL Research Cloud._




## Hardware

The current HPC setup on BERNIE for testing includes the following components:
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


## Drivers

I'm still trying to understand the correct software drivers that should be used for this hardware configuration _specifically as it relates to the Infiniband communication_.  Some notes are found in the  [Intel oneAPI Hardware Driver Troubleshooting](Sierra_Intel_oneAPI.md#hardware-driver-troubleshooting) portion of this project.  Other internet resources are:
* https://developer.nvidia.com/networking/infiniband-software
* https://lists.openfabrics.org/pipermail/libfabric-users/2024-May/001037.html
* OFED
  - https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/
  - https://stackoverflow.com/questions/58622347/what-is-the-difference-between-ofed-mlnx-ofed-and-the-inbox-driver
  - https://www.openfabrics.org/ofed-for-linux/
  - https://serverfault.com/questions/1048740/infiniband-drivers-ofed-or-distro-included
* https://easybuild.io/eum22/011_eum22_mpi_easybuild.pdf

A summary of my understanding is:
* Firmware and drivers is important to performance.
* Current Mellanox drivers are DOCA.
* Previous Mellanox drivers are OFED.
* ConnectX-3 and older cards cannot use OFED or DOCA.  You insead have to use the stock RedHat supplied drivers.


## Performance Testing

- https://www.intel.com/content/www/us/en/developer/articles/technical/intel-mpi-benchmarks.html
  * https://orfeo-doc.areasciencepark.it/examples/MPI-communication/
  * https://openbenchmarking.org/test/pts/intel-mpi&eval=87eaf65bb9165c1f55818e7cb6d2dac014567573#metrics
  * https://www.ibm.com/support/pages/how-do-i-check-open-mpi-112-infiniband-support-using-intel-mpi-benchmark
- https://mvapich.cse.ohio-state.edu/benchmarks/
  * This is available via Spack!
  * https://packages.spack.io/package.html?name=osu-micro-benchmarks
- https://hpctoolkit.org/
- https://www.top500.org/project/linpack/

* use `ifconfig` to get connection names
* use `ethtool` to get maximum speeds
* `ibstat` is another command to use for Infiniband info
```
[tvj@bernie ~]$ ethtool eno1
Settings for eno1:
	Supported ports: [ TP ]
	Supported link modes:   10baseT/Half 10baseT/Full
	                        100baseT/Half 100baseT/Full
	                        1000baseT/Full
	Supported pause frame use: Symmetric
	Supports auto-negotiation: Yes
	Supported FEC modes: Not reported
	Advertised link modes:  10baseT/Half 10baseT/Full
	                        100baseT/Half 100baseT/Full
	                        1000baseT/Full
	Advertised pause frame use: Symmetric
	Advertised auto-negotiation: Yes
	Advertised FEC modes: Not reported
	Speed: 1000Mb/s
	Duplex: Full
	Auto-negotiation: on
	Port: Twisted Pair
	PHYAD: 1
	Transceiver: internal
	MDI-X: on (auto)
netlink error: Operation not permitted
        Current message level: 0x00000007 (7)
                               drv probe link
	Link detected: yes

[tvj@bernie ~]$ ethtool ib0
Settings for ib0:
	Supported ports: [  ]
	Supported link modes:   Not reported
	Supported pause frame use: No
	Supports auto-negotiation: No
	Supported FEC modes: Not reported
	Advertised link modes:  Not reported
	Advertised pause frame use: No
	Advertised auto-negotiation: No
	Advertised FEC modes: Not reported
	Speed: 10000Mb/s
	Duplex: Full
	Auto-negotiation: off
	Port: Other
	PHYAD: 0
	Transceiver: internal
netlink error: Operation not permitted
	Link detected: yes
[tvj@bernie ~]$ 
```


### How to setup and administer a HPC
- [ ] https://en.wikipedia.org/wiki/OpenHPC
- [ ] https://csrd-conundrums.medium.com/a-basic-hpc-c590cf249dde
- [ ] https://www.admin-magazine.com/HPC/Articles/Building-an-HPC-Cluster
- [ ] https://www.admin-magazine.com/HPC/Articles/real_world_hpc_setting_up_an_hpc_cluster
