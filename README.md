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

HDP=/usr/hdp/current
export HADOOP_HOME=$HDP/hadoop-client/
export HADOOP_COMMON_HOME=$HDP/hadoop-client/
export HADOOP_HDFS_HOME=$HDP/hadoop-hdfs-client/ 
export HADOOP_MAPRED_HOME=$HDP/hadoop-mapreduce-client/
export HADOOP_SPARK_HOME=$HDP/spark-client/
export HADOOP_HIVE_HOME=$HDP/hive-client/
export HADOOP_YARN_HOME=$HDP/hadoop-yarn-client/

```sh 
# In order to use scp from master node, on master node, as root, edit /etc/hosts and add client node ip address.
# Edit file on master node
sudo nano /etc/hosts

# Add edge host into master node /etc/hosts file
10.132.0.2 edge.c.equipe.internal edge

# Copy hadoop jars file into client node
scp /home/hduser/hadoop-2.6.0.tar.gz hdclient@ztg-client:/home/hdclient



```

> Step 10: Copy hadoop code on client

```sh
# Log to dataiku
sudo su dataiku

# download hadoop distribution
wget http://apache.trisect.eu/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz

tar zxvf hadoop-2.6.0.tar.gz

mv hadoop-2.6.0 hadoop

```

> Step 11: Update environment variables for hdclient in its .bashrc

```sh
# Edit file
nano .bashrc

# Add the following code
#HADOOP VARIABLES START
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
export HADOOP_INSTALL=/home/dataiku/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
#Dont need the exports below.
#export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
#export HADOOP_COMMON_HOME=$HADOOP_INSTALL
#export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib/native"
#HADOOP VARIABLES END

# Run 
source ~/.bashrc

```

> Step 12: Update Configuration files on Client Node

```sh
# Edit /home/dataiku/hadoop/etc/hadoop/core-site.xml
nano /home/dataiku/hadoop/etc/hadoop/core-site.xml

# Chage the configuration to match to the following
<configuration>
<property>
  <name>fs.defaultFS</name>
  <value>hdfs://hadoop-m.c.equipe-1314.internal:8020</value>
  <description>Use HDFS as file storage engine</description>
</property>
</configuration>

# Edit /home/dataiku/hadoop/etc/hadoop/mapred-site.xml
cp /home/dataiku/hadoop/etc/hadoop/mapred-site.xml.template /home/dataiku/hadoop/etc/hadoop/mapred-site.xml
nano /home/dataiku/hadoop/etc/hadoop/mapred-site.xml

# Chage the configuration to match to the following

<configuration>
<property>
 <name>mapreduce.jobtracker.address</name>
 <value>hadoop-m:54311</value>
 <description>The host and port that the MapReduce job tracker runs
  at. If “local”, then jobs are run in-process as a single map
  and reduce task.
</description>
</property>
</configuration>

# On master node 
sudo  adduser  --ingroup  hadoop dataiku

# On master node 
sudo su dataiku
mkdir /home/dataiku/tmp
chmod 777 /home/dataiku/tmp



# Run mapreduce
cd hadoop
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4

# Create a directory structure in HDFS for the new use
hadoop fs -mkdir /user/dataiku/
hadoop fs -chown -R dataiku:dataiku /user/dataiku



```

> Step 12: Spark configuration
```sh
Download spark
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.6.tgz
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.4.tgz

tar zxvf spark-1.6.1-bin-hadoop2.6.tgz
mv spark-1.6.1-bin-hadoop2.6 spark

cp /home/dataiku/spark/conf/spark-env.sh.template /home/dataiku/spark/conf/spark-env.sh
nano /home/dataiku/spark/conf/spark-env.sh

export HADOOP_CONF_DIR=/home/dataiku/hadoop/etc/hadoop/
export HADOOP_CONF_DIR="${HADOOP_CONF_DIR:-/home/dataiku/hadoop/etc/hadoop}"


# From master node copy core-site.xml and yarn-site.xml to client node
chmod 644 *.xml

# change in hadoop in client node
mv hadoop/etc/hadoop/core-site.xml hadoop/etc/hadoop/core-site.xml.bkp
mv hadoop/etc/hadoop/yarn-site.xml hadoop/etc/hadoop/yarn-site.xml.bkp

# copy yarn-site.xml and core-site.xml to hadoop home
cp /home/dataiku/core-site.xml /home/dataiku/hadoop/etc/hadoop/
cp /home/dataiku/yarn-site.xml /home/dataiku/hadoop/etc/hadoop/



```

> Step 12: Update Configuration files on Client Node

```sh
```

> Step 12: Update Configuration files on Client Node

```sh
```
