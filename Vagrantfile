# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "1024"
  end

  config.ssh.insert_key = false

  config.vm.provision "shell", inline: <<-SHELL
    USER=vagrant
    HOME=/home/${USER}
    apt-get update
    # http://apache.javapipe.com/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
    cp /vagrant/hadoop-2.8.0.tar.gz /opt

    apt-get install -y openjdk-7-jdk
    cd /opt
    if ! [ -d "/opt/hadoop-2.8.0" ]; then
      tar -zxvf /opt/hadoop-2.8.0.tar.gz
      ln -s /opt/hadoop-2.8.0 /opt/hadoop
    fi;

    cp /vagrant/keys/hadoop ${HOME}/.ssh/id_rsa
    cp /vagrant/keys/hadoop.pub ${HOME}/.ssh/id_rsa.pub
    if [ $(wc -l ${HOME}/.ssh/authorized_keys | cut -d " " -f1) == 2 ]; then
      cat ${HOME}/.ssh/id_rsa.pub >> ${HOME}/.ssh/authorized_keys
    else
      wc -l ${HOME}/.ssh/authorized_keys | cut -d " " -f1
    fi;
    ssh-keyscan -H namenode > .ssh/known_hosts
    ssh-keyscan -H datanode1 >> .ssh/known_hosts
    ssh-keyscan -H datanode2 >> .ssh/known_hosts

    chown -R ${USER}:${USER} ${HOME}/.ssh/
    chmod 700 ${HOME}/.ssh/*

    cat << FILE > /opt/hadoop/etc/hadoop/slaves
datanode1
datanode2
FILE

    cat << FILE > /etc/profile.d/hadoop.sh
export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop/
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
export PATH=${PATH}:/opt/hadoop/bin:/opt/hadoop/sbin

FILE
    source /etc/profile.d/hadoop.sh

    rm -rf /hadoop/
    mkdir -p /hadoop/hdfs/nn /hadoop/hdfs/data
    cp /vagrant/conf/* /opt/hadoop/etc/hadoop/

    chown -R ${USER}:${USER} /hadoop
    chown -R ${USER}:${USER} /opt/hadoop*
  SHELL

  config.vm.define "namenode" do |nn|
  	nn.vm.hostname = "namenode"
  	nn.vm.network :private_network, :ip => '10.0.3.4'
  	nn.vm.provision :hosts do |provisioner|
      provisioner.autoconfigure = true
      provisioner.sync_hosts = true

    end
    nn.vm.network "forwarded_port", guest: 9000, host: 9000
    nn.vm.network "forwarded_port", guest: 50070, host: 50070
    nn.vm.network "forwarded_port", guest: 10020, host: 10020
    nn.vm.network "forwarded_port", guest: 19888, host: 19888
    nn.vm.network "forwarded_port", guest: 10033, host: 10033

    nn.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
    end

  	nn.vm.provision "shell", inline: <<-SHELL
      echo "nn"
      sed -i.bak '/127\.0\.1\.1/d' /etc/hosts
      su - vagrant
      source /etc/profile.d/hadoop.sh
      source /opt/hadoop/etc/hadoop/hadoop-env.sh

      hdfs namenode -format -force hadoop
      hadoop-daemon.sh --config $HADOOP_CONF_DIR --script hdfs start namenode
      yarn-daemon.sh --config $HADOOP_CONF_DIR start resourcemanager
      yarn-daemon.sh --config $HADOOP_CONF_DIR start proxyserver
      mr-jobhistory-daemon.sh --config $HADOOP_CONF_DIR start historyserver
  	SHELL
  end

  config.vm.define "datanode1" do |dn1|
  	dn1.vm.hostname = "datanode1"
  	dn1.vm.network :private_network, :ip => '10.0.3.2'
    dn1.vm.provision :hosts do |provisioner|
      provisioner.autoconfigure = true
      provisioner.sync_hosts = true
            #provisioner.add_host '172.16.3.11', ['apt.mirror.local']
    end

    dn1.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
    end

  	dn1.vm.provision "shell", inline: <<-SHELL
      echo "dn1"
      sed -i.bak '/127\.0\.1\.1/d' /etc/hosts

      su - vagrant
      source /etc/profile.d/hadoop.sh
      source /opt/hadoop/etc/hadoop/hadoop-env.sh

      hadoop-daemons.sh --config $HADOOP_CONF_DIR --script hdfs start datanode
      yarn-daemons.sh --config $HADOOP_CONF_DIR start nodemanager
  	SHELL
  end

  config.vm.define "datanode2" do |dn2|
  	dn2.vm.hostname = "datanode2"
  	dn2.vm.network :private_network, :ip => '10.0.3.3'
    dn2.vm.provision :hosts do |provisioner|
      provisioner.autoconfigure = true
      provisioner.sync_hosts = true
          #provisioner.add_host '172.16.3.11', ['apt.mirror.local']
    end

    dn2.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
    end

  	dn2.vm.provision "shell", inline: <<-SHELL
      echo "dn2"
      sed -i.bak '/127\.0\.1\.1/d' /etc/hosts

      su - vagrant
      source /etc/profile.d/hadoop.sh
      source /opt/hadoop/etc/hadoop/hadoop-env.sh

      hadoop-daemons.sh --config $HADOOP_CONF_DIR --script hdfs start datanode
      yarn-daemons.sh --config $HADOOP_CONF_DIR start nodemanager
  	SHELL
  end

end
