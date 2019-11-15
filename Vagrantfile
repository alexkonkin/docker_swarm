VAGRANTFILE_API_VERSION = "2"

$install_misc = <<-SCRIPT
  echo    ""
  echo    "-------------------------------------------------"
  echo    "|   misc section has been selected to install   |"
  echo    "-------------------------------------------------"
  echo    ""

   apt update
   apt install mc sshpass -y
   echo -e "172.16.94.100       docker-master1\n172.16.94.11       docker-worker1\n172.16.94.12       docker-worker2\n172.16.94.13       docker-worker3">> /etc/hosts
   sed -i 's/127.0.1.1/#127.0.1.1/g' /etc/hosts
SCRIPT

$install_docker = <<-SCRIPT
  echo    ""
  echo    "------------------------------------------------------------"
  echo    "|       docker section has been selected to install        |"
  echo    "------------------------------------------------------------"
  echo    ""
  apt update
  apt upgrade
  apt upgrade -y
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update
  apt-get install -y docker-ce -y
  systemctl start docker
  systemctl enable docker
  systemctl status docker
SCRIPT

$init_swarm = <<-SCRIPT
  echo    ""
  echo    "-----------------------------------------------------------------"
  echo    " docker swarm initiaization has been selected to install        |"
  echo    "-----------------------------------------------------------------"
  echo    ""
  docker swarm init --advertise-addr $(hostname -i)
  docker node ls
SCRIPT

$add_workers = <<-SCRIPT
  echo    ""
  echo    "------------------------------------------------------------"
  echo    "|  new workers will be added to the docker swarm           |"
  echo    "------------------------------------------------------------"
  echo    ""
  
  cmd=$(docker swarm join-token worker|grep join)
  for i in {1..3};do
    sshpass -p 'vagrant' ssh -o StrictHostKeyChecking=no vagrant@docker-worker$i sudo $cmd
  done
  #sshpass -p 'vagrant' ssh -o StrictHostKeyChecking=no vagrant@docker-worker1 sudo $cmd
  #sshpass -p 'vagrant' ssh -o StrictHostKeyChecking=no vagrant@docker-worker2 sudo $cmd
  #sshpass -p 'vagrant' ssh -o StrictHostKeyChecking=no vagrant@docker-worker3 sudo $cmd
  docker node ls
SCRIPT

$add_visualizer = <<-SCRIPT
  apt install unzip
  wget https://github.com/dockersamples/docker-swarm-visualizer/archive/master.zip
  mv docker-swarm-visualizer-master dockersamples
  docker run -it -d -p 5000:8080 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define "dw1" do |dw1|
    dw1.vm.box = "bento/ubuntu-18.04"
    dw1.vm.hostname = "docker-worker1"
    dw1.vm.network :private_network, ip: "172.16.94.11"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "docker", type: "shell" do |shell|
       shell.inline = $install_docker
    end
  end

  config.vm.define "dw2" do |dw2|
    dw2.vm.box = "bento/ubuntu-18.04"
    dw2.vm.hostname = "docker-worker2"
    dw2.vm.network :private_network, ip: "172.16.94.12"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "docker", type: "shell" do |shell|
       shell.inline = $install_docker
    end
  end

  config.vm.define "dw3" do |dw3|
    dw3.vm.box = "bento/ubuntu-18.04"
    dw3.vm.hostname = "docker-worker3"
    dw3.vm.network :private_network, ip: "172.16.94.13"
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "docker", type: "shell" do |shell|
       shell.inline = $install_docker
    end
  end

  config.vm.define "dm1" do |dm1|
    dm1.vm.box = "bento/ubuntu-18.04"
    dm1.vm.hostname = "docker-master1"
    dm1.vm.network :private_network, ip: "172.16.94.100"
    dm1.vm.provider "virtualbox" do |dmvm|
      dmvm.memory = 320
    end
    config.vm.provision "misc", type: "shell" do |shell|
       shell.inline = $install_misc
    end
    config.vm.provision "docker", type: "shell" do |shell|
       shell.inline = $install_docker
    end
	config.vm.provision "init_swarm", type: "shell" do |shell|
       shell.inline = $init_swarm
    end
    config.vm.provision "add_workers", type: "shell",run: "never" do |shell|
       shell.inline = $add_workers
    end
	config.vm.provision "add_visualizer", type: "shell",run: "never" do |shell|
       shell.inline = $add_visualizer
    end
  end

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
    vb.customize ["modifyvm", :id, "--cpus", "1"]
  end
end
