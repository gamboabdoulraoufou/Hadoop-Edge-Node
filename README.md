# Hadoop-Edge-Node
The procedure below explains setting up an edge node for clients to access the Hadoop Cluster for submitting jobs.
http://www.aerospike.com/docs/connectors/ashadoop/handson/edgeNodeSetup.html

> Step 1: Install Java JDK

```sh
# Update package source
sudo apt-get update

# Install some prerequisite packages
sudo apt-get -y install scp curl wget tar unzip
sudo apt-get -y install xinetd telnetd
sudo apt-get install openssh-server
sudo apt-get install openssh-client

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
# Disable ipv6  
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

# logout localhost
exit

# Change right on authorized_keys
chmod 600 $HOME/.ssh/authorized_keys

# logout dataiku
exit
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

# Change password
sudo su passwd dataiku

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
# log as dataiku user
sudo su dataiku

# Copy key
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop-m

# Test connection. You should access without password.
ssh hadoop-m

# Log out master node
exit

```

> Step 9: Modify sshd_config file _client node_
Set PermitRootLogin to yes
set PasswordAuthentication yes

```sh

# create a copy of sshd_config file
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit sshd_config file and change current configuration
sudo nano /etc/ssh/sshd_config

# Initialise configuration                            
sudo service ssh restart

```

> Step 10: Add client host into master /etc/hosts file

```sh
# Edit file 
sudo nano /etc/hosts

# Add master host into client /etc/hosts file
edge-server.c.equipe.internal edge-server
``` 

> Step 11: Add ssh key from master node to client node

``` sh
# log as dataiku user
sudo su dataiku
cd

# Generate ssh key
ssh-keygen -t rsa -P ""

# Add ssh key to authorized host
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

# log to localhost
ssh dataiku@localhost

# logout localhost
exit

# Copy key
ssh-copy-id -i ~/.ssh/id_rsa.pub edge-server

# Test connection. You should access without password.
ssh edge-server

# Log out client node
exit

```


> Step 12: Create Hadoop user _Master node_

```sh
# Log as hdfs superuser
sudo su hdfs

# Create a directory structure in HDFS for the new use
hadoop fs -mkdir /user/dataiku/
hadoop fs -chown -R dataiku:dataiku /user/dataiku

# Log out hdfs user
exit

# Create temp dir for dataiku 
sudo su dataiku
mkdir /home/dataiku/tmp
chmod 777 /home/dataiku/tmp

```

> Step 13: Download Hadoop ditribution _Client node_

```sh
# Log to dataiku
sudo su dataiku

# download hadoop distribution
wget http://apache.trisect.eu/hadoop/common/hadoop-2.7.1/hadoop-2.7.1.tar.gz

# Unpack
tar zxvf hadoop-2.7.1.tar.gz

# Rename Hadoop folder
mv hadoop-2.7.1 hadoop

```

> Step 14: Update environment variables for dataiku in its .bashrc _Client node_

```sh
# Edit file
nano .bashrc

# Add the following code
# HADOOP VARIABLES START
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
# HADOOP VARIABLES END

# Run 
source ~/.bashrc

```

> Step 15: Configure HDFS _Client Node_

```sh
# Copy core-site.xml and yarn-site.xml file from master node to client node (comment topology_script proprety in core-site.xml)
scp /etc/hadoop/2.4.3.0-227/0/core-site.xml dataiku@edge-server:/home/dataiku/hadoop/etc/hadoop/
scp /etc/hadoop/2.4.3.0-227/0/yarn-site.xml dataiku@edge-server:/home/dataiku/hadoop/etc/hadoop/

chmod +x /home/dataiku/hadoop/etc/hadoop/yarn-env.sh

# Edit yarn-env.sh
nano /home/dataiku/hadoop/etc/hadoop/yarn-env.sh

# Add JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64

# HDFS 
hadoop fs -ls /user/

```

> Step 16: Configure Mapreduce _Client node_

```sh
# Copy mapred-site.xml from master node to client node
scp /etc/hadoop/2.4.3.0-227/0/mapred-site.xml dataiku@edge-server:/home/dataiku/hadoop/etc/hadoop/

# get hadoop version
ls -l /etc/hadoop/ # on master node

# Replace ${hdp.version} by hadoop version (2.4.3.0-227 in this case)
sed -i -e 's/${hdp.version}/2.4.3.0-227/g' /home/dataiku/hadoop/etc/hadoop/mapred-site.xml

cat /home/dataiku/hadoop/etc/hadoop/mapred-site.xml | grep hdp.version
cat /home/dataiku/hadoop/etc/hadoop/mapred-site.xml | grep 2.4.3.0-227
  
# Edit hadoop-env.sh
nano /home/dataiku/hadoop/etc/hadoop/hadoop-env.sh

# The /etc/hadoop/hadoop-env.sh sets the maximum java heap memory for Hadoop, by Default it is:
# java heap memory can be low. For example: export HADOOP_CLIENT_OPTS="-Xmx128m $HADOOP_CLIENT_OPTS"
# Set this value at least 2G of memory. For example: export HADOOP_CLIENT_OPTS="-Xmx2048m $HADOOP_CLIENT_OPTS"
 
# Run mapreduce
cd hadoop
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar pi 2 4

```

> Step 17: Configure hive

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
# Hive variables start
export HIVE_HOME=/home/dataiku/hive
export PATH=$PATH:$HIVE_HOME/bin
export CLASSPATH=$CLASSPATH:/home/dataiku/hadoop/lib/*:.
export CLASSPATH=$CLASSPATH:/home/dataiku/hive/lib/*:.
export HADOOP_USER_CLASSPATH_FIRST=true
# hive variables end

# Run
source ~/.bashrc

# Copy hive-site.xml form master node to client
scp /etc/hive/conf/hive-site.xml dataiku@edge-server:/home/dataiku/hive/conf
scp /etc/hive/conf/hive-env.sh dataiku@edge-server:/home/dataiku/hive/conf

or 

create new one and append this content

<property>
      <name>hive.metastore.authorization.storage.checks</name>
      <value>false</value>
    </property>
    
    <property>
      <name>hive.metastore.cache.pinobjtypes</name>
      <value>Table,Database,Type,FieldSchema,Order</value>
    </property>
    
    <property>
      <name>hive.metastore.client.connect.retry.delay</name>
      <value>5s</value>
    </property>
    
    <property>
      <name>hive.metastore.client.socket.timeout</name>
      <value>1800s</value>
    </property>
    
    <property>
      <name>hive.metastore.connect.retries</name>
      <value>24</value>
    </property>
    
    <property>
      <name>hive.metastore.execute.setugi</name>
      <value>true</value>
    </property>
    
    <property>
      <name>hive.metastore.failure.retries</name>
      <value>24</value>
    </property>    
    <property>
      <name>hive.metastore.pre.event.listeners</name>
      <value>org.apache.hadoop.hive.ql.security.authorization.AuthorizationPreEventListener</value>
    </property>
    
    <property>
      <name>hive.metastore.sasl.enabled</name>
      <value>false</value>
    </property>
    
    <property>
      <name>hive.metastore.server.max.threads</name>
      <value>100000</value>
    </property>
    
    <property>
      <name>hive.metastore.uris</name>
      <value>thrift://hadoop-w-0.c.equipe-1314.internal:9083</value>
    </property>    
    <property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/apps/hive/warehouse</value>
    </property>  
      
    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.jdbc.Driver</value>
    </property>
        
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://hadoop-w-0.c.equipe-1314.internal/hive?createDatabaseIfNotExist=true</value>
    </property>
    
    <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>hive</value>
    </property>
    
    
# Edit hive-env.sh
nano /home/dataiku/hive/conf/hive-env.sh

# Replace 
HADOOP_HOME=${HADOOP_HOME:-/usr/hdp/current/hadoop-client}

# By 
HADOOP_HOME=${HADOOP_HOME:-/home/dataiku/hadoop}

# Replace
export HIVE_CONF_DIR=/usr/hdp/current/hive-client/conf

# by
export HIVE_CONF_DIR=/home/dataiku/hive/conf

# Add this line at the end of file
export HADOOP_USER_CLASSPATH_FIRST=true


nano .bashrc
export HADOOP_USER_CLASSPATH_FIRST=true
source ~/.bashrc

# Test hive
hive

show database;

# From master note
hadoop fs -copyFromLocal /home/dataiku/hive/lib/* /home/dataiku/hive/lib/
```

> Step 17: Configure Spark _Client node_

```sh
# Download spark (http://spark.apache.org/downloads.html)
wget http://d3kbcqa49mib13.cloudfront.net/spark-1.6.1-bin-hadoop2.6.tgz

# Unpack spark
tar zxvf spark-1.6.1-bin-hadoop2.6.tgz

# Remane Spark folder
mv spark-1.6.1-bin-hadoop2.6 spark

# Copy this file from master node
scp /etc/hadoop/conf/topology_script.py dataiku@edge-server:/home/dataiku/spark/

# Create spark configuration file
cp /home/dataiku/spark/conf/spark-env.sh.template /home/dataiku/spark/conf/spark-env.sh

# Edit Spark configuration file
nano /home/dataiku/spark/conf/spark-env.sh

# Add the following lines
export HADOOP_CONF_DIR=/home/dataiku/hadoop/etc/hadoop/

# Edit 
nano .bashrc

# Add the foolwing line
# Spark variables start
export SPARK_HOME=/home/dataiku/spark
export PATH=$PATH:$SPARK_HOME/bin

export PYSPARK_PYTHON=python2.7

# Spark variables end

# Run
source ~/.bashrc

# Test Spark
# Go to spark folder
cd /home/dataiku/spark

# Edit nano /home/dataiku/hadoop/etc/hadoop/core-site.xml
nano /home/dataiku/hadoop/etc/hadoop/core-site.xml

# Comment this line
<property>
  <name>net.topology.script.file.name</name>
  <value><!--/etc/hadoop/conf/topology_script.py--></value>
</property>

# Copy hive-site.xml into spark folder
cp /home/dataiku/hive/conf/hive-site.xml /home/dataiku/spark/conf/

# Run spark shell on yarn
./bin/spark-shell --master yarn-client

# Run scala command
sc.parallelize(Seq(1, 2, 3)).sum()

# Exit spark shell
:quit
```



> Step 16: Configure R


unset HADOOP_HDFS_HOME
$HADOOP_HDFS_HOME
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4


```sh

```

