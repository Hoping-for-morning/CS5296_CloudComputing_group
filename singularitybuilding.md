

##  building environment

**Cloud Computing**

## Environment Information

- EC2 in AWS
- Instance type: t2.micro
- operation system: ubuntu22.04

Here is the config and preparation need to do to build our clustering environment.

We build a system use different container to build cluster and test performance by openmpi.

The following is the deployment of singularity.



## 1. Install dependencies

```
sudo apt-get update && sudo apt-get install -y \
  build-essential \
  uuid-dev \
  libgpgme-dev \
  squashfs-tools \
  libseccomp-dev \
  wget \
  pkg-config \
  git \
  cryptsetup-bin
```



## 2. Install the go environment

Singularity is written in Go, and Go needs to be installed

Download the appropriate version of Go from [https://golang.org/dl/](https://links.jianshu.com/go?to=https://golang.org/dl/) to /usr/local.

```
cd /usr/local
sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
export PATH=$PATH:/usr/local/go/bin
source $HOME/.profile
```



## 3. Download singularity

```
git clone https://github.com/sylabs/singularity.git
cd singularity
```



## 4. Download missing dependencies

```
git submodule update --init
sudo apt-get install libglib2.0-dev
```

 

## 5. Compile

```
./mconfig && \
  make -C ./builddir && \
  sudo make -C ./builddir install
./mconfig --prefix=/opt/singularity
```



## 6. Verify

```
singularity --help
```

<img width="416" alt="image" src="https://user-images.githubusercontent.com/33509788/234894472-55b6f63b-27e6-4f3a-bd8c-5ebea311b5ef.png">

Confirm that singularity has been installed



## 7. Pull the openmpi image and build a singularity sandbox container

```
singularity build --sandbox mpi/ docker://hmonkey/openmpi14.04:v3
```

mpi/ is the built container(In our AMI, the container is named master/)



## 8. Create nfs1 directory

```
mkdir nfs1
vi ma.c
```



## 9. Write test file

```
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



 

## 10. Go into the container

```
singularity shell --writable mpi/
singularity run mpi/
```



## 11. Mount

```
singularity shell -B /home/ubuntu/nfs1 mpi/
```

We can see that the nfs1 folder is already in the container

<img width="416" alt="image" src="https://user-images.githubusercontent.com/33509788/234894678-1ea0f0ad-2ae6-4514-a8cf-64d2b08be8ab.png">



## 12. Compile ma.c

```
mpicc -o ma ma.c
```

<img width="222" alt="image" src="https://user-images.githubusercontent.com/33509788/234894721-578a56d7-fdfb-40bb-b665-16c28977587b.png">



## 13. Run

```
mpirun -np 8 ma
```

<img width="409" alt="image" src="https://user-images.githubusercontent.com/33509788/234894787-7c9a55ef-3dae-4324-a516-5eca7196d414.png">

 

 

 

 



 

 

 

 

 

 

 

 

 

 
