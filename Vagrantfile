# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/jammy64"
  config.vm.network 'forwarded_port', guest: 80, host: 40080, protocol: 'tcp'
	config.vm.network 'forwarded_port', guest: 8086, host: 48886, protocol: 'tcp'
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
		vb.cpus = "2"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
	
	config.vbguest.auto_update = false
	
	config.vm.provision "shell", inline: <<-SHELL
		#apt-get update
		#apt-get upgrade -y
		#apt-get -y install apt-transport-https wget gnupg
		. /etc/os-release; if [ ! -z ${UBUNTU_CODENAME+x} ]; then 
				DIST="${UBUNTU_CODENAME}"
			else 
				DIST="$(lsb_release -c| awk '{print $2}')"
		fi
		wget -O- https://packages.icinga.com/icinga.key | apt-key add -
		echo "deb https://packages.icinga.com/ubuntu icinga-${DIST} main" | sudo tee /etc/apt/sources.list.d/${DIST}-icinga.list
		echo "deb-src https://packages.icinga.com/ubuntu icinga-${DIST} main" | sudo tee -a /etc/apt/sources.list.d/${DIST}-icinga.list
		
		wget -O- https://repos.influxdata.com/influxdb.key | apt-key add -
		echo "deb https://repos.influxdata.com/ubuntu ${DIST} stable" | sudo tee /etc/apt/sources.list.d/${DIST}-influxdb.list
		
		#Package Installation
		apt-get update
		apt-get -y install icinga2 icingacli icingadb icingadb-web icingadb-redis monitoring-plugins icingaweb2 libapache2-mod-php mariadb-serverinfluxdb2 icinga-director
		
		#InfluxDB Setup
		systemctl start influxdb
		systemctl enable influxdb
		influx setup --force --username admin --password admin  --org influx --bucket icinga2 --token test --name icinga2
		
		
		icinga2 api setup
		
		#set mariadb root password
		mysql -u root -e "SET PASSWORD FOR root@localhost = PASSWORD('Passw0rt'); flush privileges;"
		#create icingadb user and db
		mysql -u root -e "CREATE DATABASE icingadb; GRANT ALL ON icingadb.* TO 'icingadb'@'localhost' IDENTIFIED BY 'Passw0rt'; flush privileges;"
		mysql -u root icingadb </usr/share/icingadb/schema/mysql/schema.sql
		cp /etc/icingadb/config.yml /etc/icingadb/config.yml.dist
		sed -i 's/CHANGEME/"Passw0rt"/g' /etc/icingadb/config.yml
		
		#create director db
		mysql -u root -e "CREATE DATABASE director CHARACTER SET 'utf8'; GRANT ALL ON director.* TO 'director'@'localhost' IDENTIFIED BY 'Passw0rt'; flush privileges;"
		
		systemctl enable icingadb-redis-server && systemctl start icingadb-redis-server
		systemctl enable icingadb && systemctl start icingadb
		icinga2 feature enable icingadb
		systemctl restart icinga2
		
		mysql -u root -e "CREATE DATABASE icingaweb2; GRANT ALL ON icingaweb2.* TO 'icingaweb2'@'localhost' IDENTIFIED BY 'Passw0rt'; flush privileges;"
		icingacli module enable icingadb
		icingacli setup token create
		
		echo 'Use "sudo influx setup" to setup InfluxDB if this is an initial setup. Also make shure to configure Icinga 2 via its websetup with provided setup token.'
	SHELL
end
