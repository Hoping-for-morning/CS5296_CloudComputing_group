# Podman

## Reference

https://docs.podman.io/en/latest/markdown/podman-run.1.html

https://linux.how2shout.com/how-to-install-podman-on-ubuntu-20-04-lts-focal-fossa/

https://www.linode.com/docs/guides/using-podman/#searching-for-images

## Environment Information

- EC2 in AWS 
- Instance type: t2.micro 
- operation system: ubuntu22.04
- EBS: 30GB
- Podman: 3.4.4

Here is the config and preparation need to do to build our clustering environment.

We build a system use different container to build cluster and test performance by openmpi.

The following is the deployment of Podman.

## 1. Configuring root login in VM

~~~shell
sudo passwd root
su root
~~~

##  2. Install NFS service on the main host (same as Docker part)

~~~shell
sudo apt-get update
sudo apt-get install nfs-kernel-server
sudo apt-get install nfs-common
~~~

## 3. Install Podman

~~~shell
sudo apt update
sudo apt install podman
~~~

## 4. Pull OpenMPI image

1. Download and pull images

~~~
podman pull docker://hmonkey/openmpi14.04:v3
~~~

![image-20230427103154366](https://user-images.githubusercontent.com/62580908/234894505-ee61be89-e6db-4e30-962a-30d197b9ca06.png)


2. List all Images

~~~shell
podman images
~~~

![image-20230427103347053](https://user-images.githubusercontent.com/62580908/234894951-a27de8e2-5f9a-4ff0-9dd6-6b4c5c63c626.png)

## 5. Create a model container and commit image

1. Create a model container 

~~~
sudo podman run -dit -ti --privileged=true --name model hmonkey/openmpi14.04:v3 /bin/bash
~~~

![image-20230427103846200](https://user-images.githubusercontent.com/62580908/234895057-9addeb5c-4778-4395-98a6-a6d91b53ee03.png)

2. check container status

~~~shell
podman ps -a
~~~

![image-20230427104014978](https://user-images.githubusercontent.com/62580908/234895166-0bae28cc-9124-4439-8b66-2a1bfca80803.png)

3. Get running container command-line access:

~~~shell
podman attach model
~~~

4. Install NFS service

~~~shell
sudo apt-get update
sudo apt-get install nfs-kernel-server
sudo apt-get install nfs-common
~~~

5. mount the nfs file system

~~~shell
mount the nfs file system
cd /home
sudo mkdir nfs
sudo mount (ip172.31.81.61):/home/ubuntu/monkey/nfs /home/nfs
~~~

6. commit a podman image

~~~
Ctrl P+Q
podman commit COINTAINER_ID
~~~

![image-20230427110726603](https://user-images.githubusercontent.com/62580908/234895245-283fc919-d7c2-4114-880d-c132844c82ab.png)

~~~shell
podman images
~~~

![image-20230427110911875](https://user-images.githubusercontent.com/62580908/234895301-fb9a3d87-ed5d-4c01-8ba4-1aa2a5ffb5ef.png)

A new image `9cb83ab47a04`

## 6. Create master and slave container

1. Create master and slave container

~~~shell
sudo podman run -dit -ti --privileged=true --name master 9cb83ab47a04 /bin/bash

sudo podman run -dit -ti --privileged=true --name slave1 9cb83ab47a04 /bin/bash

sudo podman run -dit -ti --privileged=true --name slave2 9cb83ab47a04 /bin/bash

~~~

2. list the container

~~~shell
podman ps -s
~~~

![podmancontainers](https://user-images.githubusercontent.com/62580908/234895576-1d77a8bb-56d5-4319-b03d-12940a930509.png)

3. mount the nfs file system to every container

~~~
sudo mount (ip172.31.81.61):/home/ubuntu/monkey/nfs /home/nfs
~~~
4. start ssh service for every container

~~~shell
sudo service ssh start
~~~

##  7. Put the addresses of slave1 and slave2 to the hosts file in masters(same as Docker part)

## 8. Build and compile the mpi program to test the performance

1. To configure NFS for distributed program execution, exit the container and go into `monkey/nfs` directory.

1. write hello.c

~~~c
#include <stdio.h>
#include "mpi.h"
int main(int argc, char *argv[])
{
    int myrank, size;
    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int namelen;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &myrank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);
    MPI_Get_processor_name(processor_name, &namelen);
    printf("Processor %d of %d on %s: Hello World!\n", myrank, size, processor_name);
    MPI_Finalize();
    return 0;
}

~~~

2. Enter the master container, check if /home/nfs is synchronized, and compile hello.c

~~~shell
cd /home/nfs
mpicc -o hello hello.c
~~~

2. Configure the hostfile and write it using vi (slots can be understood as the number of processes):

~~~shell
# hostfile example
slave1 slots=4
slave2 slots=4
~~~

3. run (the number of np should equals to number of slots)

~~~shell
mpirun -hostfile hostfile -np 8 ./hello
~~~



![image-20230427115428351](https://user-images.githubusercontent.com/62580908/234895671-d02a0f64-027f-4961-bc2c-34bfa0ed19ae.png)



