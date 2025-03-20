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


### Part III - Install Spack & Set Environment
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

```
spack install -j 32 gcc@10.5.0
spack compiler add "$(spack location -i gcc@10.5.0)"
# this made a file entry in /home/ec2-user/.spack/linux/compilers.yaml
# had to add a prepend path to the compiler following what was done in /shared/spack/etc/spack/compilers.yaml
spack compilers
 
spack install -j 64 intel-oneapi-compilers@2024.1.0%gcc@10.5.0

# had to remove the other version of oneapi built with gcc@12
# also needed to then remove this from the compilers
spack find --format "{name}-{version}-{hash}"
spack uninstall /46k6kokhbateyvghfxsm5uykwlu4odcm


spack install -j 64 libfabric fabrics=efa,tcp,udp,sockets,verbs,shm,mrail,rxd,rxm %oneapi

spack load intel-oneapi-compilers
spack compiler find
"""
  - compiler:
      spec: oneapi@=2024.1.0
      paths:
        cc: /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-12.4.0/intel-oneapi-compilers-2024.1.0-46k6kokhbateyvghfxsm5uykwlu4odcm/compiler/latest/bin/icx
        cxx: /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-12.4.0/intel-oneapi-compilers-2024.1.0-46k6kokhbateyvghfxsm5uykwlu4odcm/compiler/latest/bin/icpx
        f77: /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-12.4.0/intel-oneapi-compilers-2024.1.0-46k6kokhbateyvghfxsm5uykwlu4odcm/compiler/latest/bin/ifx
        fc: /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-12.4.0/intel-oneapi-compilers-2024.1.0-46k6kokhbateyvghfxsm5uykwlu4odcm/compiler/latest/bin/ifx
      flags: {}
      operating_system: amzn2
      target: x86_64
      modules: []
      environment:
        prepend_path:
          PATH: /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-7.3.1/binutils-2.37-qvccg7zpskturysmr4bzbsfrx34kvazo/bin
      extra_rpaths:
        - /shared/spack/opt/spack/linux-amzn2-x86_64_v3/gcc-7.3.1/gcc-12.4.0-pttzchh7o54nhmycj4wgzw5mic6rk2nb/lib64
"""

spack install -j 64 intel-oneapi-mpi+external-libfabric%oneapi

spack install -j 64 intel-oneapi-mkl@2024.2.0+cluster^intel-oneapi-mpi%oneapi

```


### Part IV - Install Sierra
```
module load /home/ec2-user/sierraenv.mod
SIERRA_ROOT=/shared/sierra_5.22
cd ${SIERRA_ROOT}

nohup /shared/source/sierra_setup.py --procs=64 --keep-build-dir &
```
