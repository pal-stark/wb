# -*- mode: ruby -*-
# vi: set ft=ruby :
required_plugins = %w( vagrant-hostsupdater vagrant-share vagrant-vbguest vagrant-reload)
required_plugins.each do |plugin|
  system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end


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
  config.vm.box = "oraclelinux/8"
  config.vm.box_version = "8.6.359"
  
  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "https://oracle.github.io/vagrant-projects/boxes/oraclelinux/8.json"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "172.31.250.10"
  config.vm.hostname = "wb.home.lan"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  config.vm.synced_folder "./services", "/home/vagrant/services"
  config.vm.synced_folder "../projects", "/home/vagrant/projects"
  config.vm.synced_folder "./tpls", "/home/vagrant/tpls"


  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessable to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #   # Customize the amount of memory on the VM:
    vb.memory = "8192"
    vb.cpus = "4"
    vb.name = "WorkBench.home.lab"
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/home//vagrant/projects", "1"]
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
    sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
    setenforce 0
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    systemctl restart sshd
    echo vagrant:vagrant | chpasswd

    dnf update -y
	  dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
	  dnf install -y http://rpms.remirepo.net/enterprise/remi-release-8.rpm
	  dnf -y install dnf-utils git mc  ca-certificates ansible go dotnet-sdk-5.0 python3 haproxy wget unzip kernel-devel tmux nmap telnet -y
	  dnf module install -y php:remi-8.2
	  dnf install php-{mysqlnd,curl,gd,intl,pear,recode,xmlrpc,mbstring,gettext,gmp,json,xml,fpm,amqp,gd,zip,xdebug,pgsql,amqp} -y
    dnf install -y php-devel
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"
    mv composer.phar /usr/bin/composer
    dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    cp /home/vagrant/tpls/daemon.json /etc/docker/
    systemctl start docker
    systemctl enable docker
    docker network create -d bridge work_bench_network
    sudo usermod -aG docker vagrant
    cd /home/services/traefik
    docker compose up -d
    cd ../portainer
    docker compose up -d
  SHELL
end
