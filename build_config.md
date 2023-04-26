# config

**Cloud Computing**

reference：

[https://blog.csdn.net/u014333158/article/details/45172621](https://blog.csdn.net/u014333158/article/details/45172621)

[https://blog.csdn.net/yu_223/article/details/116138062](https://blog.csdn.net/yu_223/article/details/116138062)

[https://blog.csdn.net/weixin_50674454/article/details/124770487](https://blog.csdn.net/weixin_50674454/article/details/124770487)

## Environment Information

- EC2 in AWS
- Instance type: t2.micro
- operation system: ubuntu22.04

Here is the config and preparation need to do to build our clustering environment.

We build a system use different container to build cluster and test performance by openmpi

### 1. Configuring root login in VM

```bash
sudo passwd root
su root
```

### 2. Install NFS service on the main host

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
sudo apt-get install nfs-common
```

- Then, configure NFS shared directory. For example : If we need to share all documents under the /home/ubuntu/monkey/nfs folder, we need to configure the /etc/exports file and add the following line at the end:

```bash
"/home/ubuntu/monkey/nfs" * (rw,sync,no_subtree_check)
```

- Then open the authority of nfs file

```bash
chmod 755 /home/ubuntu/monkey/nfs
```

- after configuring, restart the servce of nfs

```bash
sudo /etc/init.d/nfs-kernel-server restart
```

### 3. Install docker

```bash
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```

- pull image of docker  name：`hmonkey/openmpi14.04:v3`

```bash
docker pull hmonkey/openmpi14.04:v3
```

- some operation about docker
    - Exit the container (must exit in this way, otherwise the container will exit with bash), make sure the container is running in the background
    `ctrl + P + Q`
    - show the running docker
        
        `docker ps -s`
        
    - show all the docker
        
        `docker ps -a`
        
    - reoping the bash of container
        
        `docker attach (name of container: master / slave1 ...)`
        

### 4. Build multi-point mpi through Docker

the image of docker has contained openmpi.

- Establish a master-slave distributed node

```bash
sudo docker run -ti --name master --privileged=true hmonkey/openmpi14.04:v3 /bin/bash
sudo service ssh start
ctrl + P + Q
sudo docker run -ti --name slave1 --privileged=true hmonkey/openmpi14.04:v3 /bin/bash
sudo service ssh start
ctrl + P + Q
sudo docker run -ti --name slave2 --privileged=true hmonkey/openmpi14.04:v3 /bin/bash
sudo service ssh start
ctrl + P + Q
```

### 5. put the addresses of slave1 and slave2 to the hosts file in masters

- in master container `vim /etc/hosts`
- example `hosts` in master
    
    <img width="334" alt="image" src="https://user-images.githubusercontent.com/46880289/234584094-331e0ce0-5c50-4b4c-93b7-593a591a9d7e.png">
    
- use `cat /etc/hosts` in slave1 and slave2 container to find the relative ip address

### 6. Configure passwordless SSH

- this docker image has ssh without password so escape

### 7. mount the nfs file system

- in each contrainer

```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
sudo apt-get install nfs-common
cd /home
sudo mkdir nfs
sudo mount (ip172.31.81.61):/home/ubuntu/monkey/nfs /home/nfs
```

the ip should change to your device ip

<img width="888" alt="image" src="https://user-images.githubusercontent.com/46880289/234584457-9c82643f-68e0-4ec7-86c5-9438b413b95d.png">


### 8. build and compile the mpi program to test the performance

- To configure NFS for distributed program execution, exit the container and go into `monkey/nfs` directory.
- write hello.c
    
    ```python
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
    ```
    
- Enter the master container, check if /home/nfs is synchronized, and compile hello.c
    
    ```bash
    cd /home/nfs
    mpicc -o hello hello.c
    ```
    
- Configure the hostfile and write it using vi (slots can be understood as the number of processes):
    
    ```bash
    # hostfile example
    slave1 slots=4
    slave2 slots=4
    ```
    
- run (the number of np should equals to number of slots)
    
    ```bash
    mpirun -hostfile hostfile -np 8 ./hello
    ```
