---
title: OAI EPC Installation

# View.
#   1 = List
#   2 = Compact
#   3 = Card
view: 2

# Optional header image (relative to `static/img/` folder).
header:
  caption: ""
  image: ""

---

# EPC Installation

## All EPC components ( HSS, MME, SPGW ) in one virtual machine (Ubuntu Bionic 18.04) Hosted on host (Ubuntu Xenial 16.04).



### STEP 1 : Create virtual machine on host computer (Ubuntu Xenial 16.04)


**1. Update Host computer**
```
sudo apt-get update
sudo apt-get upgrade
```

**2. Install uvtool and virt-manager to manager**
```
sudo apt -y install uvtool
sudo apt install virt-manager
```

```
sudo uvt-simplestreams-libvirt sync release=bionic arch=amd64
```

```
ssh-keygen
```

```
sudo uvt-kvm create --memory 8192 --disk 50 --cpu 4  openair-cn arch=amd64 release=bionic
```

-----
### STEP 2 : Login into virtual machine and Setup ssh

```
sudo uvt-kvm ip openair-cn  
```

replace ``[IP_ADDRESS]`` with output of previous command
```
ssh -i ~/.ssh/id_rsa ubuntu@[IP_ADDRESS]
```

**Update VM**
```
sudo passwd ubuntu
sudo apt update
sudo apt upgrade
```
**Enable ssh**
```
ssh-keygen -t dsa
```

**Enable password authentication**

```
sudo nano /etc/ssh/sshd_config
```
set ``PasswordAuthentication yes``

-----

### STEP 3 : Login into virtual machine and Setup ssh

```
git clone https://github.com/OPENAIRINTERFACE/openair-cn.git
git clone https://github.com/OPENAIRINTERFACE/openair-cn-cups.git
```

**1. Install and Configure Cassandra for Installation of HSS**


```
cd ~/openair-cn/scripts/
./build_cassandra --check-installed-software --force
```

```
nodetool status
```

```
sudo service cassandra stop
sudo rm -rf /var/lib/cassandra/data/system/*
sudo rm -rf /var/lib/cassandra/commitlog/*
sudo rm -rf /var/lib/cassandra/data/system_traces/*
sudo rm -rf /var/lib/cassandra/saved_caches/*
```

```
sudo nano /etc/cassandra/cassandra.yaml
```


``cluster_name: "HSS Cluster"``
``seeds: "<Cassandra_Server_IP>"``
``listen_address: <Cassandra_Server_IP>``
``rpc_address: <Cassandra_Server_IP>``
``endpoint_snitch: GossipingPropertyFileSnitch``


```
sudo service cassandra start
Check cassandra service status
```

**2. Build and Install HSS**
```
./build_hss_rel14 --check-installed-software --force
./build_hss_rel14 --clean
```

**3. Install MME**
```
./build_mme --check-installed-software --force
./build_mme --clean
```

**4. Install SPGW-U**

```
cd ~/openair-cn-cups/build/scripts
./build_spgwu -I -f
./build_spgwu -c -V -b Debug -j
```


**5. Install SPGW-C**

```
cd ~/openair-cn-cups/build/scripts
./build_spgwc -I -f
./build_spgwc -c -V -b Debug -j
```
