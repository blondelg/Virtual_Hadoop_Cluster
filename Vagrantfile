# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "master" do |master|

    config.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
      v.name = "master"

    end

    config.vm.box = "ubuntu/xenial64"
    config.disksize.size = '10GB'
    config.vm.network "private_network", type: "dhcp"

    # get vagrantfile's path
    vagrantfile_path = File.dirname(__FILE__)

    # download hadoop package
    config.vm.provision "file", source: "#{vagrantfile_path}/hadoop-2.9.2.tar.gz", destination: "hadoop-2.9.2.tar.gz"

    # provision to be executed at VM startup
    config.vm.provision "shell", inline: <<-SHELL

      # enp0s8 ip address
      IP_AD=$(/sbin/ifconfig enp0s8 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}')
      echo $IP_AD

      # hduser setting
      sudo addgroup hadoop_group
      sudo adduser --ingroup hadoop_group hduser --disabled-password
      echo "hduser:hadoop" | sudo chpasswd
      sudo adduser hduser sudo

      # automate hduser connection when loggin though ssh
      echo "" >> /home/vagrant/.bashrc
      echo "# automate hduser loggin when ssh" >> /home/hduser/.bashrc
      echo "sudo su hduser" >> /home/vagrant/.bashrc

      # unpack hadoop
      tar -xvzf /home/vagrant/hadoop-2.9.2.tar.gz

      # rename archive
      mv /home/vagrant/hadoop-2.9.2 /home/vagrant/hadoop

      # move to hadoop place
      sudo mv /home/vagrant/hadoop /usr/local

      # install java
      sudo apt-get install -y default-jdk

      # rename java folder
      sudo mv /usr/lib/jvm/java-8-openjdk-amd64 /usr/lib/jvm/java

      # make used directories
      su hduser -c "mkdir /home/hduser/data"
      su hduser -c "mkdir /home/hduser/data/nameNode"
      su hduser -c "mkdir /home/hduser/data"
      su hduser -c "mkdir /home/hduser/data/dataNode"

      # create HADOOP_HOME and JAVA_HOME variable for setting
      export HADOOP_HOME=/usr/local/hadoop
      export JAVA_HOME=/usr/lib/jvm/java

      # append hduser's .bashrc file
      echo "" >> /home/hduser/.bashrc
      echo "# Set HADOOP_HOME variable" >> /home/hduser/.bashrc
      echo "export HADOOP_HOME=/usr/local/hadoop" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Set JAVA_HOME variable" >> /home/hduser/.bashrc
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Set PATH variable with hadoop and java pathes" >> /home/hduser/.bashrc
      echo "export PATH="$PATH":"$HADOOP_HOME"/bin:"$HADOOP_HOME"/sbin:"$JAVA_HOME"/bin" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Convenient alias" >> /home/hduser/.bashrc
      echo "unalias fs &> /dev/null" >> /home/hduser/.bashrc
      echo 'alias fs="hadoop fs"' >> /home/hduser/.bashrc
      echo "unalias hls &> /dev/null" >> /home/hduser/.bashrc
      echo 'alias hls="fs -ls"' >> /home/hduser/.bashrc
      echo 'echo hadoop | sudo -S chmod 777 -R /usr/local/hadoop/' >> /home/hduser/.bashrc
      echo 'echo  " " ' >> /home/hduser/.bashrc

      # reload hduser's .bashrc
      source /home/hduser/.bashrc

      # Hard code JAVA_HOME into "hadoop-env.sh"
      echo "" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

      # setup /etc/hostname file
      rm /etc/hostname
      echo "master" >> /etc/hostname

      # setup /etc/hosts file
      rm /etc/hosts
      echo "127.0.1.1        master" >> /etc/hosts
      echo "127.0.0.1        localhost" >> /etc/hosts
      echo "        master" >> /etc/hosts
      echo "        slave1" >> /etc/hosts
      echo "        slave2" >> /etc/hosts

      # setup core-site.xml:
      rm $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>fs.defaultFS</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>hdfs://master:9000</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>hadoop.tmp.dir</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>/home/hduser/data/tmp</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      # setup hdfs-site.xml:
      rm $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <name>dfs.namenode.name.dir</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <value>/home/hduser/data/nameNode</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <name>dfs.datanode.data.dir</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <value>/home/hduser/data/dataNode</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.replication</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>2</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.namenode.datanode.registration.ip-hostname-check</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>false</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.datanode.dns.interface</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>enp0s8</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml

      # setup mapred-site.xml:
      rm $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.framework.name</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>yarn</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>yarn.app.mapreduce.am.resource.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>256</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.map.memory.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>128</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.reduce.memory.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>128</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml

      # setup yarn-site.xml:
      rm $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <name>yarn.acl.enable</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <value>0</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <name>yarn.resourcemanager.hostname</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <value>master</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.aux-services</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>mapreduce_shuffle</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.resource.memory-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>768</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.scheduler.maximum-allocation-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>768</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.scheduler.minimum-allocation-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>64</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.vmem-check-enabled</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>false</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml

      # setup /usr/local/hadoop/etc/hadoop/slaves file
      rm $HADOOP_HOME/etc/hadoop/slaves
      echo "slave1" >> $HADOOP_HOME/etc/hadoop/slaves
      echo "slave2" >> $HADOOP_HOME/etc/hadoop/slaves

      # update ssh configfile /etc/ssh/sshd_config
      sudo sed -i -e 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
      sudo sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      sudo service ssh restart

      # set up ssh localhost connections
      su hduser -c "ssh-keygen -t rsa -P '' -f /home/hduser/.ssh/id_rsa"
      su hduser -c "cat /home/hduser/.ssh/id_rsa.pub >> /home/hduser/.ssh/authorized_keys"
      su hduser -c "chmod 0600 /home/hduser/.ssh/authorized_keys"

      # # format HDFS
      # su hduser -c '$HADOOP_HOME/bin/hdfs namenode -format'

      # # Start NameNode daemon and DataNode daemon
      # su hduser -c '$HADOOP_HOME/sbin/start-dfs.sh'

    SHELL

  end

  config.vm.define "slave1" do |slave1|

    config.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
      v.name = "slave1"

    end

    config.vm.box = "ubuntu/xenial64"
    config.disksize.size = '10GB'
    config.vm.network "private_network", type: "dhcp"

    # get vagrantfile's path
    vagrantfile_path = File.dirname(__FILE__)

    # download hadoop package
    config.vm.provision "file", source: "#{vagrantfile_path}/hadoop-2.9.2.tar.gz", destination: "hadoop-2.9.2.tar.gz"

    # provision to be executed at VM startup
    config.vm.provision "shell", inline: <<-SHELL

      # enp0s8 ip address
      IP_AD=$(/sbin/ifconfig enp0s8 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}')
      echo $IP_AD

      # hduser setting
      sudo addgroup hadoop_group
      sudo adduser --ingroup hadoop_group hduser --disabled-password
      echo "hduser:hadoop" | sudo chpasswd
      sudo adduser hduser sudo

      # automate hduser connection when loggin though ssh
      echo "" >> /home/vagrant/.bashrc
      echo "# automate hduser loggin when ssh" >> /home/hduser/.bashrc
      echo "sudo su hduser" >> /home/vagrant/.bashrc

      # unpack hadoop
      tar -xvzf /home/vagrant/hadoop-2.9.2.tar.gz

      # rename archive
      mv /home/vagrant/hadoop-2.9.2 /home/vagrant/hadoop

      # move to hadoop place
      sudo mv /home/vagrant/hadoop /usr/local

      # install java
      sudo apt-get install -y default-jdk

      # rename java folder
      sudo mv /usr/lib/jvm/java-8-openjdk-amd64 /usr/lib/jvm/java

      # make used directories
      su hduser -c "mkdir /home/hduser/data"
      su hduser -c "mkdir /home/hduser/data/nameNode"
      su hduser -c "mkdir /home/hduser/data"
      su hduser -c "mkdir /home/hduser/data/dataNode"

      # create HADOOP_HOME and JAVA_HOME variable for setting
      export HADOOP_HOME=/usr/local/hadoop
      export JAVA_HOME=/usr/lib/jvm/java

      # append hduser's .bashrc file
      echo "" >> /home/hduser/.bashrc
      echo "# Set HADOOP_HOME variable" >> /home/hduser/.bashrc
      echo "export HADOOP_HOME=/usr/local/hadoop" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Set JAVA_HOME variable" >> /home/hduser/.bashrc
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Set PATH variable with hadoop and java pathes" >> /home/hduser/.bashrc
      echo "export PATH="$PATH":"$HADOOP_HOME"/bin:"$HADOOP_HOME"/sbin:"$JAVA_HOME"/bin" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Convenient alias" >> /home/hduser/.bashrc
      echo "unalias fs &> /dev/null" >> /home/hduser/.bashrc
      echo 'alias fs="hadoop fs"' >> /home/hduser/.bashrc
      echo "unalias hls &> /dev/null" >> /home/hduser/.bashrc
      echo 'alias hls="fs -ls"' >> /home/hduser/.bashrc
      echo 'echo hadoop | sudo -S chmod 777 -R /usr/local/hadoop/' >> /home/hduser/.bashrc
      echo 'echo  " " ' >> /home/hduser/.bashrc

      # reload hduser's .bashrc
      source /home/hduser/.bashrc

      # Hard code JAVA_HOME into "hadoop-env.sh"
      echo "" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

      # setup /etc/hostname file
      rm /etc/hostname
      echo "slave1" >> /etc/hostname

      # setup /etc/hosts file
      rm /etc/hosts
      echo "127.0.1.1        slave1" >> /etc/hosts
      echo "127.0.0.1        localhost" >> /etc/hosts
      echo "        master" >> /etc/hosts
      echo "        slave1" >> /etc/hosts
      echo "        slave2" >> /etc/hosts

      # setup core-site.xml:
      rm $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>fs.defaultFS</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>hdfs://master:9000</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>hadoop.tmp.dir</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>/home/hduser/data/tmp</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      # setup hdfs-site.xml:
      rm $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <name>dfs.namenode.name.dir</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <value>/home/hduser/data/nameNode</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <name>dfs.datanode.data.dir</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <value>/home/hduser/data/dataNode</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.replication</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>2</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.namenode.datanode.registration.ip-hostname-check</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>false</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.datanode.dns.interface</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>enp0s8</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml

      # setup mapred-site.xml:
      rm $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.framework.name</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>yarn</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>yarn.app.mapreduce.am.resource.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>256</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.map.memory.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>128</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.reduce.memory.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>128</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml

      # setup yarn-site.xml:
      rm $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <name>yarn.acl.enable</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <value>0</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <name>yarn.resourcemanager.hostname</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <value>master</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.aux-services</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>mapreduce_shuffle</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.resource.memory-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>768</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.scheduler.maximum-allocation-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>768</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.scheduler.minimum-allocation-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>64</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.vmem-check-enabled</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>false</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml

      # setup /usr/local/hadoop/etc/hadoop/slaves file
      rm $HADOOP_HOME/etc/hadoop/slaves
      echo "slave1" >> $HADOOP_HOME/etc/hadoop/slaves
      echo "slave2" >> $HADOOP_HOME/etc/hadoop/slaves

      # update ssh configfile /etc/ssh/sshd_config
      sudo sed -i -e 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
      sudo sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      sudo service ssh restart

      # set up ssh localhost connections
      su hduser -c "ssh-keygen -t rsa -P '' -f /home/hduser/.ssh/id_rsa"
      su hduser -c "cat /home/hduser/.ssh/id_rsa.pub >> /home/hduser/.ssh/authorized_keys"
      su hduser -c "chmod 0600 /home/hduser/.ssh/authorized_keys"

      # # format HDFS
      # su hduser -c '$HADOOP_HOME/bin/hdfs namenode -format'

      # # Start NameNode daemon and DataNode daemon
      # su hduser -c '$HADOOP_HOME/sbin/start-dfs.sh'

    SHELL

    end


  config.vm.define "slave2" do |slave2|

    config.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 1
      v.name = "slave2"

    end

    config.vm.box = "ubuntu/xenial64"
    config.disksize.size = '10GB'
    config.vm.network "private_network", type: "dhcp"

    # get vagrantfile's path
    vagrantfile_path = File.dirname(__FILE__)

    # download hadoop package
    config.vm.provision "file", source: "#{vagrantfile_path}/hadoop-2.9.2.tar.gz", destination: "hadoop-2.9.2.tar.gz"

    # provision to be executed at VM startup
    config.vm.provision "shell", inline: <<-SHELL

      # enp0s8 ip address
      IP_AD=$(/sbin/ifconfig enp0s8 | grep 'inet addr' | cut -d: -f2 | awk '{print $1}')
      echo $IP_AD

      # hduser setting
      sudo addgroup hadoop_group
      sudo adduser --ingroup hadoop_group hduser --disabled-password
      echo "hduser:hadoop" | sudo chpasswd
      sudo adduser hduser sudo

      # automate hduser connection when loggin though ssh
      echo "" >> /home/vagrant/.bashrc
      echo "# automate hduser loggin when ssh" >> /home/hduser/.bashrc
      echo "sudo su hduser" >> /home/vagrant/.bashrc

      # unpack hadoop
      tar -xvzf /home/vagrant/hadoop-2.9.2.tar.gz

      # rename archive
      mv /home/vagrant/hadoop-2.9.2 /home/vagrant/hadoop

      # move to hadoop place
      sudo mv /home/vagrant/hadoop /usr/local

      # install java
      sudo apt-get install -y default-jdk

      # rename java folder
      sudo mv /usr/lib/jvm/java-8-openjdk-amd64 /usr/lib/jvm/java

      # make used directories
      su hduser -c "mkdir /home/hduser/data"
      su hduser -c "mkdir /home/hduser/data/nameNode"
      su hduser -c "mkdir /home/hduser/data"
      su hduser -c "mkdir /home/hduser/data/dataNode"

      # create HADOOP_HOME and JAVA_HOME variable for setting
      export HADOOP_HOME=/usr/local/hadoop
      export JAVA_HOME=/usr/lib/jvm/java

      # append hduser's .bashrc file
      echo "" >> /home/hduser/.bashrc
      echo "# Set HADOOP_HOME variable" >> /home/hduser/.bashrc
      echo "export HADOOP_HOME=/usr/local/hadoop" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Set JAVA_HOME variable" >> /home/hduser/.bashrc
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Set PATH variable with hadoop and java pathes" >> /home/hduser/.bashrc
      echo "export PATH="$PATH":"$HADOOP_HOME"/bin:"$HADOOP_HOME"/sbin:"$JAVA_HOME"/bin" >> /home/hduser/.bashrc
      echo "" >> /home/hduser/.bashrc
      echo "# Convenient alias" >> /home/hduser/.bashrc
      echo "unalias fs &> /dev/null" >> /home/hduser/.bashrc
      echo 'alias fs="hadoop fs"' >> /home/hduser/.bashrc
      echo "unalias hls &> /dev/null" >> /home/hduser/.bashrc
      echo 'alias hls="fs -ls"' >> /home/hduser/.bashrc
      echo 'echo hadoop | sudo -S chmod 777 -R /usr/local/hadoop/' >> /home/hduser/.bashrc
      echo 'echo  " " ' >> /home/hduser/.bashrc

      # reload hduser's .bashrc
      source /home/hduser/.bashrc

      # Hard code JAVA_HOME into "hadoop-env.sh"
      echo "" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

      # setup /etc/hostname file
      rm /etc/hostname
      echo "slave2" >> /etc/hostname

      # setup /etc/hosts file
      rm /etc/hosts
      echo "127.0.1.1        slave2" >> /etc/hosts
      echo "127.0.0.1        localhost" >> /etc/hosts
      echo "        master" >> /etc/hosts
      echo "        slave1" >> /etc/hosts
      echo "        slave2" >> /etc/hosts

      # setup core-site.xml:
      rm $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>fs.defaultFS</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>hdfs://master:9000</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>hadoop.tmp.dir</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>/home/hduser/data/tmp</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      # setup hdfs-site.xml:
      rm $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <name>dfs.namenode.name.dir</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <value>/home/hduser/data/nameNode</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <name>dfs.datanode.data.dir</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "            <value>/home/hduser/data/dataNode</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.replication</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>2</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.namenode.datanode.registration.ip-hostname-check</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>false</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.datanode.dns.interface</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>enp0s8</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml

      # setup mapred-site.xml:
      rm $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.framework.name</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>yarn</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>yarn.app.mapreduce.am.resource.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>256</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.map.memory.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>128</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <name>mapreduce.reduce.memory.mb</name>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "        <value>128</value>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/mapred-site.xml

      # setup yarn-site.xml:
      rm $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <name>yarn.acl.enable</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <value>0</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <name>yarn.resourcemanager.hostname</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "            <value>master</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.aux-services</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>mapreduce_shuffle</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.resource.memory-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>768</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.scheduler.maximum-allocation-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>768</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.scheduler.minimum-allocation-mb</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>64</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <name>yarn.nodemanager.vmem-check-enabled</name>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "        <value>false</value>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/yarn-site.xml

      # setup /usr/local/hadoop/etc/hadoop/slaves file
      rm $HADOOP_HOME/etc/hadoop/slaves
      echo "slave1" >> $HADOOP_HOME/etc/hadoop/slaves
      echo "slave2" >> $HADOOP_HOME/etc/hadoop/slaves

      # update ssh configfile /etc/ssh/sshd_config
      sudo sed -i -e 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
      sudo sed -i -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      sudo service ssh restart

      # set up ssh localhost connections
      su hduser -c "ssh-keygen -t rsa -P '' -f /home/hduser/.ssh/id_rsa"
      su hduser -c "cat /home/hduser/.ssh/id_rsa.pub >> /home/hduser/.ssh/authorized_keys"
      su hduser -c "chmod 0600 /home/hduser/.ssh/authorized_keys"

      # # format HDFS
      # su hduser -c '$HADOOP_HOME/bin/hdfs namenode -format'

      # # Start NameNode daemon and DataNode daemon
      # su hduser -c '$HADOOP_HOME/sbin/start-dfs.sh'

    SHELL

  end

end
