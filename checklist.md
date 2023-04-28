## Abstract

Our topic is to explore the impact of container usage on HPC performance. We build simple HPC cluster through different types of containers. Here is the artifact appendix to help prepare the environment run our experiment and evaluate the result.

## Checklist

check the item bellow will help you build the environment and install the dependencies packages correctly.

- VM with Linux opertaion system
- install Docker
- install podman
- install nfs server
- install go environment
- install singularity
- pull the mpi image
- install fio
- install gcc g++
- install gfortran
- install libopenblas
- install hpl-2.3


## Description

Here is our detailed guide 

establish an environment in EC2 instance based on Docker see [dockerbuilding.md](./dockerbuilding.md).

establish an environment in EC2 instance based on Podman see [podmanbuilding.md](./podmanbuilding.md).

establish an environment in EC2 instance based on singularity see [singularitybuilding.md](./singularitybuilding.md).


### dependencies
We use EC2 instance in AWS
Instance type: t2.micro
operation system: ubuntu22.04

## Environments Established you can install

These environments have already been set up and are available for you to download.

Podman:
EC2 AMI ID: ami-002acc3ab6e575668
Region: us-east-1c

Singularity:
EC2 AMI ID: ami-0c9b8b367dbd5d7ca
Region: us-east-1a

after you launch a instance by AMI, you should follow the [restart.md](./restart.md) to restart the service in virtual meachine to build the operational environment.

## Evaluation and expected result

[fiotest.md](./fiotest.md) is the guide file to test and evalute the performance of file I/O.

[hpltest.md](./hpltest.md) is the guide file to test and evaluate the performance cluster calulation.