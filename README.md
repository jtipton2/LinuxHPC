# Sierra Installation on a HPC using Spack
>*Joseph B. Tipton, Jr.*
>*December 2024*














### Sierra Intel Compile
* For this they recommend Sierra v5.22 as the previous releases would need Intel classic instead of Intel oneAPI.
* They suggest trying to use the Spack packages `intel-oneapi-compilers` and `intel-oneapi-mpi` and `intel-oneapi-mkl`
* Will then need to set the following ENV variables and paths:
    - I_MPI_CC=
    - I_MPI_CXX=
    - I_MPI_F77=
    - I_MPI_F90=
    - MKLROOT= path where mkl is installed and shoudl be a lib directory underneath that path
* https://spack.readthedocs.io/en/latest/build_systems/inteloneapipackage.html
* https://www.intel.com/content/www/us/en/docs/oneapi/installation-guide-linux/2023-0/spack.html
* https://www.intel.com/content/www/us/en/developer/articles/technical/distribution-of-optimized-hpc-binaries-using-spack.html
* Here is what Sierra team uses on their modules:

![Sandia Sierra Module Settings - A](sandia_sierra_intel_paths_a.png)
![Sandia Sierra Module Settings - B](sandia_sierra_intel_paths_b.png)












