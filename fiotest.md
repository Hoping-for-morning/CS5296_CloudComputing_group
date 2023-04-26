# FIO_test

### here is the file I/O testing pipeline for our cluster

### 1. write sequential write test file to nfs file system

- vim 1m-seq-PREwrite.job
    
    ```bash
    [global]
    ioengine=libaio
    runtime=300
    direct=1
    rw=write
    bs=1M
    iodepth=32
    group_reporting
    [test ]
    time_based
    stonewall
    directory=/home/nfs  #containers’directory
    size=10G
    numjobs=1
    ```
    

### 2. write random write test file to nfs system

- vim 4k-rand-write.job
    
    ```bash
    [global]
    ioengine=libaio
    runtime=300
    direct=1
    norandommap=1
    rw=randwrite
    bs=4k
    iodepth=32
    group_reporting
    ramp_time=6
    [test ]
    time_based
    stonewall
    directory=/home/nfs  #containers’directory
    size=10G
    numjobs=1
    ```
    

### 3. write random read test file to nfs system

- vim 4k-rand-read.job
    
    ```bash
    [global]
    ioengine=libaio
    runtime=300
    direct=1
    norandommap=1
    rw=randread
    bs=4k
    iodepth=32
    group_reporting
    ramp_time=6
    [test ]
    time_based
    stonewall
    directory=/home/nfs  #containers’directory
    size=10G
    numjobs=1
    ```
    

### 4. write test shell script to test

- vim fio_test.sh
    
    ```bash
    #!/bin/bash
    job="1m-seq-PREwrite.job 4k-rand-read.job 4k-rand-wirte.job"
    for x in $job
    do
    echo *****$x*****
    fio --client slave1 ${x} --client slave2 ${x}
    sleep 180
    done
    ```
    

### 5. install fio tool and test

- install
    
    ```bash
    apt-get install fio
    ```
    
- start server on two slaves
    
    ```bash
    docker attack slave
    fio --server
    ```
    
- test read and write on master
    
    ```bash
    sh fio_test.sh
    ```