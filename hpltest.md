download gcc, g++, gfortran, libopenblas

~~~shell
apt install gcc g++ gfortran libopenblas-*
~~~

download hpl-2.3 to nfs on vm

~~~shell
wget http://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz
~~~

enter container and move hpl-2.3.tar.gz from nfs to another directory and extract

~~~shell
docker attach master
mv /home/nfs/hpl-2.3.tar.gz /home/file
tar -xf hpl-2.3.tar.gz
~~~


go to hpl-2.3 and move Make.Linux_PII_CBLAS file and name as Make.test
~~~shell
mv ./setup/Make.Linux_PII_CBLAS ./Make.test
~~~

change Make.test file
~~~shell
ARCH = test
TOPdir = /home/file/hpl-2.3
MPdir = /openmpi-1.8.1
MPlib = /usr/lib/libmpi.so
LAdir = /usr/lib/openblas-base
LAlib = $(LAdir)/libblas.a
CC = /usr/bin/mpicc
CCFLAGS = $(HPL_DEFS) -fomit-frame-pointer -O3 -funroll-loops -lpthread
LINKER = /usr/bin/mpif77
~~~

run make
~~~shell
make arch=test
~~~

go to hpl-2.3/bin/test and run commend
~~~shell
mpirun -np 4 ./xhpl > 1.txt
~~~