# ps0 #

## Ubuntu 18.04 server ##

### static IP address ###

Ubuntu 18.04 uses netplan as default tool to configure network. See [configure network static IP address in Ubuntu](https://www.tecmint.com/configure-network-static-ip-address-in-ubuntu/)

* add the following to [/etc/netplan/50-cloud-init.yaml](50-cloud-init.yaml)
  ```sh
  sudo vim /etc/netplan/50-cloud-init.yaml
  ```

  ```sh
  network:
      ethernets:
          enp2s0:
              dhcp4: no
              dhcp6: no
              addresses: [192.168.1.90/24]
              gateway4: 192.168.1.1
              nameservers:
                      addresses: [8.8.8.8,8.8.4.4]
      version: 2
  ```
* apply the changes
  ```sh
  sudo netplan apply
  ```
* verify IP address
  ```sh
  ifconfig -a
  ```
  
### Install Docker ###

### Install and Configurate SnapRAID ###

There is no dpkg that one can use to install directly. Following instruction from [here](https://zackreed.me/setting-up-snapraid-on-ubuntu/). To avoid adding un-necessary dependency after compilation, I compile the SnapRAID inside a docker container (of Ubuntu 18.04). 

* Run docker image Ubuntu 18.04
  ```sh
  docker run -i -t ubuntu:18.04 /bin/bash
  ```
* Inside the container.
  ```sh
  apt-get update && apt-get upgrade -y
  apt-get install gcc git make wget -y
  cd /home
  wget https://github.com/amadvance/snapraid/releases/download/v11.2/snapraid-11.2.tar.gz
  tar xzvf snapraid-11.2.tar.gz
  cd snapraid-11.2/
  ./configure
  make && make check
  cd ..
  tar cvfz snapraid-11.2-ubuntu.tar.gz snapraid-11.2.tar.gz
  make install
  cd ..
  cp ~/snapraid-11.2/snapraid.conf.example /etc/snapraid.conf
  cd ..
  ```
* On host machine
  ```sh
  docker cp <container_name>:/home/snapraid-11.2-ubuntu.tar.gz .
  tar xvfz snapraid-11.2-ubuntu.tar.gz
  cd snapraid-11.2
  sudo make install
  cd ..
  sudo cp snapraid.conf.example /etc/snapraid.conf
  ```
* Configure [/etc/fstab](fstab)
  * create mount points
    ```
    sudo mkdir -p /depot/{data1,data2,data3}
    sudo mkdir -p /depot/parity
    ```
  * find the UUID of partitions to use
    ```sh
    blkid
    ```
  * edit /etc/fstab to mount the drives
  * mount drives
    ```sh
    sudo mount -a
    ```
* Configure [/etc/snapraid.conf](snapraid.conf)
  After editing snapraid.conf, and then run 
  ```sh
  sudo vim /etc/snapraid.conf
  snapraid sync
  ```
  We are done!
  
## Dockers containers ##
