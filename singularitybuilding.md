

##  building environment

**Cloud Computing**

## Environment Information

- EC2 in AWS
- Instance type: t2.micro
- operation system: ubuntu20.04

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
singularity build --sandbox master/ docker://hmonkey/openmpi14.04:v3
```

master/ is the built container


## 8. Create nfs directory

```
mkdir nfs
vi hello.c
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
singularity shell --writable master/
singularity run master/
```



## 11. Mount

```
singularity shell -B /home/ubuntu/nfs master/
```

We can see that the nfs folder is already in the container


<img width="521" alt="555" src="https://user-images.githubusercontent.com/33509788/235032949-d823c543-80ad-48da-864c-903f3557ff84.png">



## 12. Compile hello.c

```
mpicc -o hello hello.c
```

<img width="516" alt="4444" src="https://user-images.githubusercontent.com/33509788/235032878-c52573cb-b395-42b4-8783-578ff67758c8.png">




## 13. Run

```
mpirun -np 8 hello
```


<img width="318" alt="3333" src="https://user-images.githubusercontent.com/33509788/235032685-5020a002-34a5-49be-acbb-001336f0f328.png">

 

 

 

 



 

 

 

 

 

 

 

 

 

 
