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

### Workflow
<img width="367" alt="image" src="https://user-images.githubusercontent.com/46880289/235099670-60ed9f63-ad10-427f-9a11-85a0f073fc3c.png">



### Build
Here is our detailed guide to help you build environment.

establish an environment in EC2 instance based on Docker see [dockerbuilding.md](./dockerbuilding.md).

establish an environment in EC2 instance based on Podman see [podmanbuilding.md](./podmanbuilding.md).

establish an environment in EC2 instance based on singularity see [singularitybuilding.md](./singularitybuilding.md).


## Dependencies
We use EC2 instance in AWS
Instance type: t2.micro
operation system: ubuntu22.04

## Environments Established You Can Install

These environments have already been set up and are available for you to download.

Podman:
EC2 AMI ID: ami-002acc3ab6e575668
Region: us-east-1c

Singularity:
EC2 AMI ID: ami-0c9b8b367dbd5d7ca
Region: us-east-1a

Docker:
EC2 AMI ID: ami-03734a29ec3d48252
Region: us-east-1

after you launch a instance by AMI, you should follow the [restart.md](./restart.md) to restart the service in virtual meachine to build the operational environment.

## Evaluation and Expected Result

[fiotest.md](./fiotest.md) is the guide file to test and evalute the performance of file I/O.

[hpltest.md](./hpltest.md) is the guide file to test and evaluate the performance cluster calulation.
