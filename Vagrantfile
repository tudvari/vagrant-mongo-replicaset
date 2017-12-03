# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
$rsStartupScript = <<-"SCRIPT"

mongo --host 192.168.42.100 << 'EOF'
config = { _id: "rs-example", members:[
          { _id : 0, host : "192.168.42.100:27017"},
          { _id : 1, host : "192.168.42.110:27017"},
          { _id : 2, host : "192.168.42.120:27017"} ]
         };
rs.initiate(config);
rs.status() ;
EOF
SCRIPT

$mongoInitScript = <<-"SCRIPT"
  YUM_REPO_CONFIG_PATH="/etc/yum.repos.d/mongodb-org-3.4.repo"

  tee $YUM_REPO_CONFIG_PATH <<-"EOF"
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=0
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
EOF

  yum -y install openssl openssl-devel
  yum -y install nano mongodb-org --nogpgcheck

  MONGOD_CONF_FILE="/etc/mongod.conf"

  tee -a $MONGOD_CONF_FILE <<-"EOF"
replication:
  oplogSizeMB: 64
  replSetName: rs-example
EOF

  iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 27017 -j ACCEPT
  iptables-save > /etc/sysconfig/iptables
  service iptables restart
  perl -pi -e 's/bindIp: 127.0.0.1/#bind_ip=127.0.0.1/g' /etc/mongod.conf
  service mongod restart
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "centos64"
  config.vm.provider "VirtualBox"
  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130427.box"

  config.vm.provider "VirtualBox" do |v|
    v.customize ["modifyvm", :id, "--cpus", "2"]
  end

  config.vm.define :mongo2 do |mongo2|
    mongo2.vm.network :private_network, ip: "192.168.42.110"
    mongo2.vm.provision "shell", inline: $mongoInitScript
  end

  config.vm.define :mongo3 do |mongo3|
    mongo3.vm.network :private_network, ip: "192.168.42.120"
    mongo3.vm.provision "shell", inline: $mongoInitScript
  end

  config.vm.define :mongo1 do |mongo1|
    mongo1.vm.network :private_network, ip: "192.168.42.100"
    mongo1.vm.provision "shell", inline: $mongoInitScript
    mongo1.vm.provision "shell", inline: $rsStartupScript
  end

end
