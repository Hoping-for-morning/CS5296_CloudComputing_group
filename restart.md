# Restart Steps

## 1. Configuring root login in VM

~~~shell
sudo passwd root
su root
~~~

password: root

## 2. Start the container

~~~shell
podman start master
podman start slave1
podman start slave2
~~~

## 3. mount the nfs file system to every container  and start ssr service

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

## 4. Put the addresses of slave1 and slave2 to the hosts file in masters

check the addresses of slave1 and slave2. 

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

