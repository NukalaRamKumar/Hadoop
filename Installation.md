# How to Install Hadoop Single Node Cluster in Ubuntu
The following process is step by step process to install Hadoop Single Node Cluster in Ubuntu.

### 1. Installing Java
You can install OpenJDK 11 from the default apt repositories:

```
sudo apt update 
sudo apt install openjdk-11-jdk
```

Once installed, verify the installed version of Java with the following command:
```
java -version
```
### 2. Configure SSH Key-based Authentication
Run the following command to generate Public and Private Key Pairs:
```
ssh-keygen -t rsa 
```

You will be asked to enter the filename. Just press Enter to complete the process:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa):
Created directory '/home/hadoop/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/hadoop/.ssh/id_rsa
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:QSa2syeISwP0hD+UXxxi0j9MSOrjKDGIbkfbM3ejyIk hadoop@ubuntu20
The key's randomart image is:
+---[RSA 3072]----+
| ..o++=.+        |
|..oo++.O         |
|. oo. B .        |
|o..+ o * .       |
|= ++o o S        |
|.++o+  o         |
|.+.+ + . o       |
|o . o * o .      |
|   E + .         |
+----[SHA256]-----+
```

Next, append the generated public keys from id_rsa.pub to authorized_keys and set proper permission:

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
chmod 640 ~/.ssh/authorized_keys 
```

Next, verify the passwordless SSH authentication with the following command:

```
ssh localhost
```

You will be asked to authenticate hosts by adding RSA keys to known hosts. Type yes and hit Enter to authenticate the localhost:

```
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:JFqDVbM3zTPhUPgD5oxJ4ClviH6tzxVXf2GD3BdNqQPQ.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

### 3. Installing Hadoop
Download the latest version of [Hadoop](https://hadoop.apache.org/releases.html) using the wget command:
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.1/hadoop-3.3.1.tar.gz
```

Once downloaded, extract the downloaded file:
```
tar -xvzf hadoop-3.3.1.tar.gz 
```

Next, Rename the extracted directory to hadoop and change the directory to **/usr/opt**:
```
mv hadoop-3.3.1 hadoop
sudo mv hadoop //opt
```

Next, you will need to configure Hadoop and Java Environment Variables on your system.

Open the **~/.bashrc** file in your favorite text editor:
```
gedit ~/.bashrc
```

Append the below lines to file. You can find JAVA_HOME location by running dirname ```whereis java``` command on terminal:

```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export HADOOP_HOME=/opt/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

Next, open the Hadoop environment variable file:
```
gedit $HADOOP_HOME/etc/hadoop/hadoop-env.sh 
```

Set the JAVA_HOME in hadoop environemnt:
```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

### 4. Configuring Hadoop
First, you will need to create the namenode and datanode directories inside Hadoop home directory:
```
mkdir -p $HADOOP_HOME/dfs/name 
mkdir -p $HADOOP_HOME/dfs/data 
```

Next, edit the **core-site.xml** file and update with your system hostname:
```
gedit $HADOOP_HOME/etc/hadoop/core-site.xml 
```

Change the following configuration as per your system hostname:
```
<configuration>
    <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost:9000/</value>
    </property>
</configuration>
```

Save and close the file. Then, edit the **hdfs-site.xml** file:
```
gedit $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Change the NameNode and DataNode directory path as shown below:
```
<configuration>
    <property>
            <name>dfs.replication</name>
            <value>1</value>
    </property>
    <property>
            <name>dfs.namenode.name.dir</name>
            <value>$HADOOP_HOME/dfs/name</value>
    </property>
    <property>
            <name>dfs.datanode.data.dir</name>
            <value>$HADOOP_HOME/dfs/data</value>
    </property>
</configuration>
```

Save and close the file. Then, edit the **mapred-site.xml** file:
```
<configuration>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/opt/hadoop</value>
    </property>
</configuration>
```

Save and close the file. Then, edit the **yarn-site.xml** file:
```
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>127.0.0.1</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>127.0.0.1:8025</value>
    </property>
    <property>
           <name>yarn.resourcemanager.scheduler.address</name>
        <value>127.0.0.1:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>127.0.0.1:8040</value>
    </property>
</configuration>
```

### 5. Start Hadoop Cluster

Before starting the Hadoop cluster. You will need to format the Namenode as a hadoop user.

Run the following command to format the hadoop Namenode:
```
hdfs namenode -format 
```

You should get the following output:
```
2020-11-23 10:31:51,318 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
2020-11-23 10:31:51,323 INFO namenode.FSImage: FSImageSaver clean checkpoint: txid=0 when meet shutdown.
2020-11-23 10:31:51,323 INFO namenode.NameNode: SHUTDOWN_MSG:
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at localhost
************************************************************/
```

After formatting the Namenode, run the following command to start the hadoop cluster:
```
start-dfs.sh 
```

Once the HDFS started successfully, you should get the following output:
```

Starting namenodes on [localhost]
hadoop.tecadmin.com: Warning: Permanently added 'localhost,fe80::200:2dff:fe3a:26ca%eth0' (ECDSA) to the list of known hosts.
Starting datanodes
Starting secondary namenodes [localhost]
```

Next, start the YARN service as shown below:
```
start-yarn.sh 
```

You should get the following output:
```
Starting resourcemanager
Starting nodemanagers
```

You can now check the status of all Hadoop services using the jps command:
```
jps 
```

You should see all the running services in the following output:
```
18194 NameNode
18822 NodeManager
17911 SecondaryNameNode
17720 DataNode
18669 ResourceManager
19151 Jps
```

### 6. Adjust Firewall
Hadoop is now started and listening on port 9870 and 8088. Next, you will need to allow these ports through the firewall.

Run the following command to allow Hadoop connections through the firewall:
```
firewall-cmd --permanent --add-port=9870/tcp 
firewall-cmd --permanent --add-port=8088/tcp 
```

Next, reload the firewalld service to apply the changes:
```
firewall-cmd --reload
```

### 7. Acess Hadoop
To access the Namenode, open your web browser and visit the URL **`http://your-server-ip:9870`**. You should see the following screen:
```
http://localhost:9870
```

To access the Resource Manage, open your web browser and visit the URL **`http://your-server-ip:8088``**. You should see the following screen:
```
http://localhost:8088
```
## Note
If you want to stop the Hadoop Cluster follow the steps. [ *If not followed you will face issues to start the datanode* ]
```
stop-yarn.sh
```
After Yarn is stopped. Stop the dfs:
```
stop-dfs.sh
```