# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  #config.vm.provision "shell", inline: "developping stuff ........."

  config.vm.define "master" do |master|
    config.vm.box = "ubuntu/trusty64"
    config.vm.network "private_network", ip: "192.168.33.10"

    # get vagrantfile's path
    vagrantfile_path = File.dirname(__FILE__)

    # download hadoop package
    config.vm.provision "file", source: "#{vagrantfile_path}/hadoop-2.9.2.tar.gz", destination: "hadoop-2.9.2.tar.gz"

    # provision to be executed at VM startup
    config.vm.provision "shell", inline: <<-SHELL

      # hduser setting
      sudo addgroup hadoop_group
      sudo adduser --ingroup hadoop_group hduser --disabled-password
      echo "hduser:hadoop" | sudo chpasswd
      sudo adduser hduser sudo

      # unpack hadoop
      tar -xvzf /home/vagrant/hadoop-2.9.2.tar.gz

      # rename archive
      mv /home/vagrant/hadoop-2.9.2 /home/vagrant/hadoop

      # move to hadoop place
      sudo mv /home/vagrant/hadoop /usr/local

      # install java
      sudo apt-get install -y openjdk-7-jre

      # rename java folder
      sudo mv /usr/lib/jvm/java-7-openjdk-amd64 /usr/lib/jvm/java

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

      # reload hduser's .bashrc
      source /home/hduser/.bashrc

      # Hard code JAVA_HOME into "hadoop-env.sh"
      echo "" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
      echo "export JAVA_HOME=/usr/lib/jvm/java" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

      # setup core-site.xml:
      rm $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <name>fs.defaultFS</name>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "        <value>hdfs://localhost:9000</value>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/core-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/core-site.xml

      # setup hdfs-site.xml:
      rm $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml version="1.0" encoding="UTF-8"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo '<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>' >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "<configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    <property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <name>dfs.replication</name>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "        <value>1</value>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "    </property>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml
      echo "</configuration>" >> $HADOOP_HOME/etc/hadoop/hdfs-site.xml

      # set up ssh localhost connections
      su hduser -c "ssh-keygen -t rsa -P '' -f /home/hduser/.ssh/id_rsa"
      su hduser -c "cat /home/hduser/.ssh/id_rsa.pub >> /home/hduser/.ssh/authorized_keys"
      su hduser -c "chmod 0600 /home/hduser/.ssh/authorized_keys"
      su hduser -c "ssh localhost"
      su hduser -c "exit"

      # grant rights for hduser
      su hduser -c 'sudo chmod 777 -R /usr/local/hadoop/'

      # format HDFS
      su hduser -c '$HADOOP_HOME/bin/hdfs namenode -format'

      # Start NameNode daemon and DataNode daemon
      su hduser -c '$HADOOP_HOME/sbin/start-dfs.sh'

    SHELL

  end


end
