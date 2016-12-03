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
hadoop-m.c.equipe.internal hadoop-m

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

> Step 10: Create Hadoop user _Master node_

```sh
# Create a directory structure in HDFS for the new use
hadoop fs -mkdir /user/dataiku/
hadoop fs -chown -R dataiku:dataiku /user/dataiku

# On master node 
sudo su dataiku
mkdir /home/dataiku/tmp
chmod 777 /home/dataiku/tmp

```

> Step 10: Download Hadoop ditribution _Client node_

```sh
# Log to dataiku
sudo su dataiku

# download hadoop distribution
wget http://apache.trisect.eu/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz

# Unpack
tar zxvf hadoop-2.6.0.tar.gz

# Rename Hadoop folder
mv hadoop-2.6.0 hadoop

```

> Step 11: Update environment variables for dataiku in its .bashrc _Client node_

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

> Step 12: Configure HDFS _Client Node_

```sh
# Copy core-site.xml and yarn-site.xml file from master node to client node (comment topology_script proprety in core-site.xml)
scp /etc/hadoop/2.4.2.0-258/0/core-site.xml dataiku@edge:/home/dataiku/hadoop/etc/hadoop/
scp /etc/hadoop/2.4.2.0-258/0/yarn-site.xml dataiku@edge:/home/dataiku/hadoop/etc/hadoop/

# Edit yarn-env.sh
nano /home/dataiku/hadoop/etc/hadoop/yarn-env.sh

# Add JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64

# HDFS 
hadoop hs -ls /user/

```

> Step 13: Configure Mapreduce _Client node_

```sh
# Copy mapred-site.xml from master node to client node
scp /etc/hadoop/2.4.2.0-258/0/mapred-site.xml dataiku@edge:/home/dataiku/hadoop/etc/hadoop/

OR

# Create mapred-site.xml file
cp /home/dataiku/hadoop/etc/hadoop/mapred-site.xml.template /home/dataiku/hadoop/etc/hadoop/mapred-site.xml

# Edit
nano /home/dataiku/hadoop/etc/hadoop/mapred-site.xml

# Edit hadoop-env.sh
nano /home/dataiku/hadoop/etc/hadoop/hadoop-env.sh

# Add the following content
The /etc/hadoop/hadoop-env.sh sets the maximum java heap memory for Hadoop, by Default it is:

   export HADOOP_CLIENT_OPTS="-Xmx128m $HADOOP_CLIENT_OPTS"
This Xmx setting is too low, simply change it to this and rerun

   export HADOOP_CLIENT_OPTS="-Xmx2048m $HADOOP_CLIENT_OPTS"
 

# Chage the configuration to match to the following
<configuration>
<property>
 <name>mapreduce.jobtracker.address</name>
 <value>hadoop-m</value>
 <description>The host and port that the MapReduce job tracker runs
  at. If “local”, then jobs are run in-process as a single map
  and reduce task.
</description>
</property>
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>

# Run mapreduce
cd hadoop
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4

```

> Step 14: Configure Spark _Client node_

```sh
# Download spark (http://spark.apache.org/downloads.html)
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.6.tgz

# Unpack spark
tar zxvf spark-1.6.1-bin-hadoop2.6.tgz

# Remane Spark folder
mv spark-1.6.1-bin-hadoop2.6 spark

# Create spark configuration file
cp /home/dataiku/spark/conf/spark-env.sh.template /home/dataiku/spark/conf/spark-env.sh

# Edit Spark configuration file
nano /home/dataiku/spark/conf/spark-env.sh

# Add the following lines
export HADOOP_CONF_DIR=/home/dataiku/hadoop/etc/hadoop/

# Edit 
nano .bashrc

# Add the foolwing line
export SPARK_HOME=/home/dataiku/spark
export PATH=$PATH:$SPARK_HOME/bin

# Run
source ~/.bashrc

# Test Spark
# Go to spark folder
cd /home/dataiku/spark

# Run spark shell on yarn
./bin/spark-shell --master yarn-client

# Run scala command
sc.parallelize(Seq(1, 2, 3)).sum()

# Exit spark shell
:quit
```

> Step 15: Configure hive

```sh
# Download hive
wget http://mirrors.ukfast.co.uk/sites/ftp.apache.org/hive/hive-1.2.1/apache-hive-1.2.1-bin.tar.gz

# Unpack Hive
tar zxvf apache-hive-1.2.1-bin.tar.gz

# Rename hive folder
mv apache-hive-1.2.1-bin hive

# Edit 
nano .bashrc

# Add the foolwing line
export HIVE_HOME=/home/dataiku/hive
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:/home/dataiku/hadoop/lib/*:.
export CLASSPATH=$CLASSPATH:home/dataiku/hadoop/lib/*:.
export HADOOP_USER_CLASSPATH_FIRST=true

# Run
source ~/.bashrc

# Copy hive-site.xml form master node to client
scp /etc/hive/conf/hive-site.xml dataiku@edge:/home/dataiku/hive/conf
scp /etc/hive/conf/hive-env.sh dataiku@edge:/home/dataiku/hive/conf

# Edit hive-env.sh
nano /home/dataiku/hive/conf/hive-env.sh

# Add the following lines
hadoop fs -mkdir /tmp 
hadoop fs -mkdir /user/hive/warehouse
hadoop fs -chmod g+w /tmp 
hadoop fs -chmod g+w /user/hive/warehouse


dataiku@edge:~$ sed -i -e 's/system:java.io.tmpdir/tmp/g' /home/dataiku/hive/conf/hive-site.xml
sed -i -e 's/system:user.name/user.name/g' /home/dataiku/hive/conf/hive-site.xml
sed -i -e 's/${tmp}/tmp/g' /home/dataiku/hive/conf/hive-site.xml

export HADOOP_USER_CLASSPATH_FIRST=true


cat /home/dataiku/hive/conf/hive-site.xml | grep system:user.name


# Test hive
hive

show database;

```

> Step 16: Configure R

```sh

```

