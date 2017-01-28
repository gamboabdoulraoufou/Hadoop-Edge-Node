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

> Step 11: Download Hadoop ditribution _Client node_

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

> Step 11: Update environment variables for dataiku in its .bashrc _Client node_

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

> Step 12: Configure HDFS _Client Node_

```sh
# Copy core-site.xml and yarn-site.xml file from master node to client node (comment topology_script proprety in core-site.xml)
scp /etc/hadoop/2.4.3.0-227/0/core-site.xml dataiku@edge-server:/home/dataiku/hadoop/etc/hadoop/
scp /etc/hadoop/2.4.3.0-227/0/yarn-site.xml dataiku@edge-server:/home/dataiku/hadoop/etc/hadoop/

# Edit yarn-env.sh
nano /home/dataiku/hadoop/etc/hadoop/yarn-env.sh

# Add JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64

# HDFS 
hadoop fs -ls /user/

```

> Step 13: Configure Mapreduce _Client node_

```sh
# Copy mapred-site.xml from master node to client node
scp /etc/hadoop/2.4.3.0-227/0/mapred-site.xml dataiku@edge-server:/home/dataiku/hadoop/etc/hadoop/

sudo mkdir -p -m 751 /usr/hdp/2.4.3.0-227/hadoop/lib/native 
sudo mkdir -p -m 751 /usr/local/lib/hadoop/lib/
sudo mkdir -p -m 751 /etc/hadoop/conf/secure
sudo mkdir -p -m 751 /hdp/apps/2.4.3.0-227/

scp /usr/hdp/2.4.3.0-227/hadoop/lib/native/Linux-amd64-64 dataiku@edge-server:/usr/hdp/2.4.3.0-227/hadoop/lib/native/
scp /usr/local/lib/hadoop/lib/* dataiku@edge-server:/usr/local/lib/hadoop/lib/
scp /usr/hdp/2.4.3.0-227/hadoop/lib/hadoop-lzo-0.6.0.2.4.3.0-227.jar dataiku@edge-server:/usr/hdp/2.4.3.0-227/hadoop/lib/
scp /etc/hadoop/conf/secure/* dataiku@edge-server:/etc/hadoop/conf/secure/
scp /hdp/apps/2.4.3.0-227/* dataiku@edge-server:/hdp/apps/2.4.3.0-227/

replace ${hdp.version} by 2.4.3.0-227


OR

# Create mapred-site.xml file
cp /home/dataiku/hadoop/etc/hadoop/mapred-site.xml.template /home/dataiku/hadoop/etc/hadoop/mapred-site.xml

# Edit
nano /home/dataiku/hadoop/etc/hadoop/mapred-site.xml

# Add the following content
<property>
    <name>mapreduce.application.framework.path</name>
    <value>/hdp/apps/2.4.3.0-227/mapreduce/mapreduce.tar.gz#mr-framework</value>
  </property>
  <property>
      <name>mapreduce.admin.user.env</name>
      <value>LD_LIBRARY_PATH=/home/dataiku/hadoop/lib/native:/home/dataiku/hadoop/lib/native/$
    </property>
  <property>
    <name>mapreduce.application.classpath</name>
    <value></value>
  </property>
<property>
 <name>mapreduce.admin.map.child.java.opts</name>
 <value>-server -Djava.net.preferIPv4Stack=true
   -Dhdp.version=2.4.3.0-227
   </value>
 <final>true</final>
</property>
 <property>
  <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
 </property>
<property>
 <name>mapreduce.admin.reduce.child.java.opts</name>
 <value>-server -Djava.net.preferIPv4Stack=true -Dhdp.version=2.4.3.0-227</value>
 <final>true</final>
</property>
  <property>
    <name>mapreduce.framwork.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.map.memory.mb</name>
    <value>1536</value>
  </property>
  <property>
    <name>mapreduce.map.java.opts</name>
    <value>-Xmx1228m</value>
  </property>
  <property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>2048</value>
    </property>
  <property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>2048</value>
  </property>
  <property>
    <name>mapreduce.reduce.java.opts</name>
    <value>-Xmx1638m</value>
  </property>
  <property>
    <name>mapreduce.task.io.sort.mb</name>
    <value>859</value>
  </property>
  <property>
    <name>mapreduce.task.io.sort.factor</name>
    <value>100</value>
  </property>
<property>
    <name>mapreduce.reduce.shuffle.parallelcopies</name>
    <value>30</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
      <value>hadoop-w-0.c.equipe-1314.internal:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop-w-0.c.equipe-1314.internal:19888</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.intermediate-done-dir</name>
    <value>/mr-history/tmp</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.done-dir</name>
    <value>/mr-history/done</value>
  </property>
  
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
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar pi 2 4

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
export CLASSPATH=$CLASSPATH:/home/dataiku/hive/lib/*:.
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


nano .bashrc
export HADOOP_USER_CLASSPATH_FIRST=true
source ~/.bashrc

cat /home/dataiku/hive/conf/hive-site.xml | grep system:user.name


# Test hive
hive

show database;


hadoop fs -copyFromLocal /home/dataiku/hive/lib/* /home/dataiku/hive/lib/
```

> Step 16: Configure R


unset HADOOP_HDFS_HOME
$HADOOP_HDFS_HOME
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar pi 2 4


```sh

```

