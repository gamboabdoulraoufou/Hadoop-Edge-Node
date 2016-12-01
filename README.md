# Hadoop-Edge-Node
The procedure below explains setting up an edge node for clients to access the Hadoop Cluster for submitting jobs.

> Step 1: Install Java JDK

```sh
# Update package source
sudo apt-get update

# Install Java SDK
sudo apt-get install default-jdk

# Check installation. You should see somthing like this: java version "x.x.x"
java -version

```

> Step 2: Add user dataiku

```sh
# Add user on client server
sudo adduser dataiku

# Add user on HDFS

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
```

> Step 1: Install Java JDK

```sh
```

> Step 1: Install Java JDK

```sh
```

> Step 1: Install Java JDK

```sh
```

> Step 1: Install Java JDK

```sh
```
