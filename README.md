## Goal:

Build a MariaDB Galera Cluster on GCP


## Steps:

- Create GCP VM x 3
- Install MariaDB-Galera-Cluster
- Create Internal load balancing
- Backup
- Recover


## Create GCP VM x 3

- Why use VM?

http://patrobinson.github.io/2016/11/07/thou-shalt-not-run-a-database-inside-a-container/


- We create 3 GCP VM instances

Use Ubuntu16.04 LTS(20GB SSD Boot disk), n1-standard-2(2CPU, 7.5GB memory)

- Node1 - 192.168.0.10
- Node2 - 192.168.0.11
- Node3 - 192.168.0.12


**Change the ip reflect your needs.**


## Set up  MariaDB-Galera-Cluster

```bash
# Adds the necessary key
sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8

# Adds the repository
sudo add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://ftp.utexas.edu/mariadb/repo/10.1/ubuntu xenial main'

# update apt
sudo apt update -y

# Install MariaDB and rsync(needs to sync the nodes)
sudo apt-get install mariadb-server rsync

# Secure MariaDB
sudo mysql_secure_installation
```

**Complete the above for all three nodes.**



## Configuring

- Copy `my.cnf` to `/etc/mysql/my.cnf`  

**This is just a default conf, you can overriding it reflect your needs**

- Add `/etc/mysql/conf.d/galera.cnf`

```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0
# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
# Galera Cluster Configuration
wsrep_cluster_name="galera_cluster"
wsrep_cluster_address="gcomm://192.168.1.10,192.168.1.11,192.168.1.12"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="192.168.1.10"
wsrep_node_name="Node1"
```


**Do same to Node2 & Node3, Use their respective ip address and Names in this section.**

- Node2

```
# Galera Node Configuration
wsrep_node_address="192.168.1.11"
wsrep_node_name="Node2"
```

- Node3

```
# Galera Node Configuration
wsrep_node_address="192.168.1.12"
wsrep_node_name="Node3"
```

### Starting the Galera Cluster

In each node, stop mysql.

```
sudo systemctl stop mysql
```

- Start Galera Cluster on Node1, the node will now be registered with the cluster

```
sudo galera_new_cluster

mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

Show wsrep_cluster_size = 1


- On Node2 & Node3, 

```
sudo systemctl start mysql
```


- Then on each Node

```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"
```

Show wsrep_cluster_size = 3


**Now MariaDB galera Cluster is up, You can test it by Create DB, insert data...**



# Create Internal load balancing


- Instance group with 3 backends -> Node1, Node2, Node3

- Frontend tcp:3306

- set `static-ip`
 
















