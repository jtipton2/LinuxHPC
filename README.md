# Sierra Installation on a HPC using Spack
>*Joseph B. Tipton, Jr.*
>*December 2024 - January 2025*

These are my personal notes as I try to learn how to manage a software environment and compile the Sierra simulation suite on a linux HPC.

The new HPC is called "BERNIE" and uses a new headnode, a new file server, and a new Mellanox Infiniband switch.  For now, ITSD have taken 2 compute nodes from the old cluster (node001, node003) and moved them to the new cluster (cnode001, cnode003).

In this way, I can run a benchmark Sierra explicit simulation to compare performance on the same hardware using different verions of Sierra and different compiler environments.  The table below shows current results:

Headnode | Compute Nodes | Sierra Version | Environment | Time
--- | --- | --- | --- | ---
OZ3 | 1 & 3 | v4.56.4 | intel-19.0.3 | 900 s
BERNIE | 1 & 3 | v5.14 | gcc-8.5.0 | 1,497 s
BERNIE | 1 & 3 | v5.14 | gcc-10.2.0 | 1,558 s
BERNIE | 1 & 3 | v5.22.1 | oneapi-2024.1.0 | 1,406 s





















