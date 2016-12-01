# Hadoop-Edge-Node
The procedure below explains setting up an edge node for clients to access the Hadoop Cluster for submitting jobs.

> Step 1: Install Java JDK

```sh
# Update package source
sudo apt-get update

# Install some prerequisite packages
sudo apt-get -y install openssh scp curl wget python tar unzip telnet telnet-server

# Install Java SDK
sudo apt-get install default-jdk

# Check installation. You should see somthing like this: java version "x.x.x"
java -version

```

> Step 2: Add user dataiku

```sh
# Add user on client server
sudo adduser dataiku

```

> Step 3: Disable IPV6 on Ubuntu

```sh
# Edit sysctl.conf file
sudo nano /etc/sysctl.conf

```
> Add the following lines at the end
#disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1

```sh
# Executesysctl to disable ipv6
sudo sysctl -p

# Check if ipv6 is disabled
cat /proc/sys/net/ipv6/conf/all/disable_ipv6

# The command bellow should return: 1

```

> Step 4: Enable SSH (execute these commands on client node only)

```sh
# Log into dataiku account
sudo su dataiku
cd

# Generate ssh key
ssh-keygen -t rsa -P ""

# Add ssh key to authorized host
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

# log to localhost
ssh dataiku@localhost

# logout
exit

# Change right on authorized_keys
chmod 600 $HOME/.ssh/authorized_keys

```

> Step 5: Modify sshd_config file _master node_
Set PermitRootLogin to yes
Uncomment PasswordAuthentication yes
Comment PasswordAuthentication no

```sh
# create a copy of sshd_config file
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit sshd_config file and change current configuration
sudo nano /etc/ssh/sshd_config

# Initialise configuration
sudo /etc/init.d/sshd –t                             
sudo service sshd restart

```

> Step 6: Add user dataiku on _master node_

```sh
# Add user dataiku on master node
sudo adduser dataiku

```
> Step 7: Add master host into client /etc/hosts file

```sh
# Edit file 
sudo nano /etc/hosts

# Add master host into client /etc/hosts file
10.132.0.12 hadoop-m.c.equipe.internal hadoop-m

``` 

> Step 8: Add ssh key from client node to master node

``` sh
# Copy key
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop-m

# Test connection. You should access without password.
ssh hadoop-m

```

> Step 9: Modify sshd_config file _master node_ to set initial configuration
Set PermitRootLogin to yes
Uncomment PasswordAuthentication yes
Comment PasswordAuthentication no

```sh
# create a copy of sshd_config file
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak2
sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config

```

> Step 10: Install Hadoop on Client Node
We need hadoop to compile the jars on client and submit the job but we will not run any hadoop daemons.

```sh 
# In order to use scp from master node, on master node, as root, edit /etc/hosts and add client node ip address.
# Edit file on master node
sudo nano /etc/hosts

# Add edge host into master node /etc/hosts file
10.132.0.2 edge.c.equipe.internal edge

# Copy hadoop jars file into client node
scp /home/hduser/hadoop-2.6.0.tar.gz hdclient@ztg-client:/home/hdclient


```

> Step 1: Install Java JDK

```sh
```

> Step 1: Install Java JDK

```sh
```
