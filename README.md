## Goal:

Build a MariaDB Galera Cluster on GCP


## Steps:

- Create GCP VM x 3
- Install MariaDB-Galera-Cluster
- Create Internal load balancing
- Backup
- Recover


### Create GCP VM x 3

- Why use VM?

http://patrobinson.github.io/2016/11/07/thou-shalt-not-run-a-database-inside-a-container/


- We create 3 GCP VM instances

Use Ubuntu16.04 LTS(20GB SSD Boot disk), n1-standard-2(2CPU, 7.5GB memory)

- Node1 - 192.168.0.10
- Node2 - 192.168.0.11
- Node3 - 192.168.0.12


### Set up  MariaDB-Galera-Cluster

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



