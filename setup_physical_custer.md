# Tuto Install Hadoop

## Create hduser

## Download 2.9.2 version:
wget http://apache.mediamirrors.org/hadoop/common/stable/hadoop-2.9.2.tar.gz

## Download verification file 
wget https://www-eu.apache.org/dist/hadoop/common/current2/hadoop-2.9.2.tar.gz.asc

## Unpack archive
tar -xvzf hadoop-2.9.2.tar.gz

## Rename hadoop folder
mv hadoop-2.9.2 hadoop

## Move hadoop folder 
sudo mv hadoop /usr/local

## Add java repositories (ONLY UBUNTU SERVER)
sudo add-apt-repository ppa:webupd8team/java

## Install Java on UBUNTU SERVER
sudo apt-get install openjdk-7-jre

## Install Java on RASPBIAN
sudo apt-get install oracle-java8-jdk

## Go to java home
/usr/lib/jvm

## Rename java folder to java
mv java-8-oracle java

## Update .BASHRC file for hduser
nano $HOME/.bashrc

## Copy the following at the end:

	# Set Hadoop-related environment variables
	export HADOOP_HOME=/usr/local/hadoop
	 
	# Set JAVA_HOME (we will also configure JAVA_HOME directly for Hadoop later on)
	export JAVA_HOME=/usr/lib/jvm/java
	 
	# Some convenient aliases and functions for running Hadoop-related commands
	unalias fs &> /dev/null
	alias fs="hadoop fs"
	#unalias hls &> /dev/null
	alias hls="fs -ls"

	# Add Hadoop bin/ directory to PATH
	export PATH=$PATH:$HADOOP_HOME/bin

	# Add Java bin/ directory to PATH
	export PATH=$PATH:$JAVA_HOME/bin

## Hard code JAVA_HOME into "hadoop-env.sh"
nano $HADOOP_HOME/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/java

## Update files:
etc/hadoop/core-site.xml:
etc/hadoop/hdfs-site.xml

## Format the filesystem
cd $HADOOP_HOME/bin
hdfs namenode -format

## Start NameNode
cd $HADOOP_HOME/sbin
./start-dfs.sh

## Update files:
mapred-site.xml
yarn-site.xml

## SETUP DATA NODE--------------------

## Create hduser on SLAVE
sudo addgroup hadoop_group
sudo adduser --ingroup hadoop_group hduser

## Grant hduser rights
sudo adduser hduser sudo

## From Master, enable password-less SSH connection to SLAVE
ssh-keygen -t rsa -P ""
ssh-copy-id -i $HOME/.ssh/id_rsa.pub hduser@<SLAVE>

## From master, modify host aliases:
nano /etc/hosts
add lines:
<MASTER IP> master
<SLAVE 1 IP> slave_1
<SLAVE 2 IP> slave_2

192.168.1.58 master
192.168.1.63 slave_1
192.168.1.99 slave_2


## Copy stuff
fs -copyFromLocal * /home/input

hadoop-streaming-2.9.2.jar
hadoop jar /usr/local/hadoop/share/hadoop/tools/lib/hadoop-streaming-2.9.2.jar \
    -input /home/input \
    -output /home/output \
    -mapper /home/hduser/mapper.py
