# Restart Steps
## Podman
### 1. Configuring root login in VM

~~~shell
su root
~~~
password: root

### 2. Start the container

~~~shell
podman start master
podman start slave1
podman start slave2
~~~

### 3. mount the nfs file system to every container  and start ssr service
The ip address should be replaced with the Private IPv4 addresses of your ec2 instance.
~~~shell
podman attach master
sudo mount 172.31.81.135:/home/ubuntu/monkey/nfs /home/nfs
sudo service ssh start
ctrl+P+Q

podman attach slave1
sudo mount 172.31.81.135:/home/ubuntu/monkey/nfs /home/nfs
sudo service ssh start
ctrl+P+Q

podman attach slave1
sudo mount 172.31.81.135:/home/ubuntu/monkey/nfs /home/nfs
sudo service ssh start
ctrl+P+Q

~~~

### 4. Put the addresses of slave1 and slave2 to the hosts file in masters

Check the addresses of slave1 and slave2. 

~~~shell
podman attach slave1
cat /etc/hosts

ctrl+P+Q
podman attach slave2
cat /etc/hosts
~~~

Then edit masters /etc/hosts file (using `vi /etc/hosts` command) to add addresses of slave1 and slave2.

~~~shell
podman attach master
vi /etc/hosts
~~~

 Adding the two lines are like: 

~~~
ip_address	slave1
ip_address	slave2
~~~
### 5. FIO test
Start server on two slaves
~~~shell
fio --server
~~~
Test read and write on master
~~~shell
sh fio_test.sh
~~~
### 6. HPL test
Test on master. Change directory to hpl-2.3/bin/test and run command.
~~~shell
podman attach master
cd home/file/hpl-2.3/bin/test
mpirun -np 4 ./xhpl > 1.txt
~~~
## Docker
When verifying tests about docker containers in an instance created with the EC2 AMI (docker) we provide, the operations to restart the containers are similar to those of Podman.
## Singularity
The container can be used directly without additional operations.
