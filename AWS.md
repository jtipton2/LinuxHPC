# AWS GovCloud

Amazon created a physically different AWS option for the US Government that follows regulations (including ITAR, EAR) and allows access only by US Citizens (https://docs.aws.amazon.com/govcloud-us/latest/UserGuide/govcloud-differences.html).

## AWS ParallelCluster 
* Open source tool created by AWS to create HPC environments
* https://aws.amazon.com/hpc/parallelcluster/
* https://github.com/aws/aws-parallelcluster

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
* First, I tweak the `config-hpc.yaml` file based on tutorials:
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

### Part IV - Install Sierra

