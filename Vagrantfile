# -*- mode: ruby -*-
# vi: set ft=ruby :
iCSN = 3  # // NUMBER OF CONSUL SERVER INSTANCES UP TO 9 <= iN > 0
iCCN = 2  # // NUMBER OF CONSUL CLIENT INSTANCES UP TO 9 <= iN > 0
sIP_CLASS_D='192.168.77'  # // NETWORK CIDR for Consul configs.
sIPS=''  # // IPs we'll construct based on IP D class + instance number
sVUSER='vagrant'  # // vagrant user
sHOME="/home/#{sVUSER}"  # // home path for vagrant user
sNET='en0: Wi-Fi (Wireless)'  # // network adaptor to use for bridged mdoe
#sNET='en7: USB 10/100/1000 LAN'  # // network adaptor to use for bridged mdoe `networksetup -listallhardwareports ; # on mac`
sIP_C1 = "#{sIP_CLASS_D}.1"  # // first node for copying .pem files

(1..iCSN).each do |iY|  # // CONSUL Server Nodes IP's for join (concatenation)
  if iY < iCSN
     sIPS+="\"#{sIP_CLASS_D}.#{iY}\", "
  else
     sIPS+="\"#{sIP_CLASS_D}.#{iY}\""
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"  # // OS
  # config.vm.box = "debian/buster64"  # // OS

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024  # // RAM / Memory
    v.cpus = 1  # // CPU Cores / Threads
  end

  # // OS packages
  config.vm.provision "file", source: "1.install_commons.sh", destination: "#{sHOME}/install_commons.sh"
  config.vm.provision "shell", inline: "/bin/bash #{sHOME}/install_commons.sh"

  # // allow for SSHD on all interfaces & setup default identity files for ssh
  config.vm.provision "shell", inline: 'sed -i "s/#ListenAddress/ListenAddress/g" /etc/ssh/sshd_config'
  config.vm.provision "shell", inline: 'sed -i "s/#.*IdentityFile ~\/\.ssh\/id_rsa/    IdentityFile ~\/\.ssh\/id_rsa/g" /etc/ssh/ssh_config'
  config.vm.provision "shell", inline: 'sed -i "s/.*IdentityFile ~\/\.ssh\/id_rsa/    IdentityFile ~\/\.ssh\/id_rsa\n    IdentityFile ~\/\.ssh\/id_rsa2/g" /etc/ssh/ssh_config'

  # // CONSUL Script for instaling & setup
  config.vm.provision "file", source: "2.install_consul.sh", destination: "#{sHOME}/install_consul.sh"
  config.vm.provision "shell", inline: "sed -i 's/\"__IPS-SET__\"/#{sIPS}/g' #{sHOME}/install_consul.sh"

  # // CONSUL Server Nodes
  (1..iCSN).each do |iY|
    config.vm.define vm_name="consul-server#{iY}" do |consul_server_node|
      consul_server_node.vm.hostname = vm_name
      consul_server_node.vm.network "public_network", bridge: "#{sNET}", ip: "#{sIP_CLASS_D}.#{iY}"
      if iY > 1
        consul_server_node.vm.provision "file", source: ".vagrant/machines/consul-server1/virtualbox/private_key", destination: "#{sHOME}/.ssh/id_rsa"
        consul_server_node.vm.provision "shell", inline: "ssh-keyscan #{sIP_C1} 2>/dev/null >> ~/.ssh/known_hosts"
        consul_server_node.vm.provision "shell", inline: "cp -r #{sHOME}/.ssh /root/."
        consul_server_node.vm.provision "shell", inline: "rsync -q vagrant@#{sIP_C1}:~/*.pem #{sHOME}/. && chown -R #{sVUSER} #{sHOME}/"
        consul_server_node.vm.provision "shell", inline: "rsync -q vagrant@#{sIP_C1}:~/gossip.txt #{sHOME}/. && chown -R #{sVUSER} #{sHOME}/"
      end
#      consul_server_node.vm.network "forwarded_port", guest: 80, host: "5818#{iY}", id: "consul#{iY}"
      consul_server_node.vm.provision "shell", inline: "/bin/bash #{sHOME}/install_consul.sh"
    end
  end

  # // CONSUL Client Nodes
  (1..iCCN).each do |iX|
    config.vm.define vm_name="consul-client#{iX}" do |consul_client_node|
      consul_client_node.vm.hostname = vm_name
      consul_client_node.vm.network "public_network", bridge: "#{sNET}", ip: "#{sIP_CLASS_D}.#{254-iX}"
      consul_client_node.vm.provision "file", source: ".vagrant/machines/consul-server1/virtualbox/private_key", destination: "#{sHOME}/.ssh/id_rsa"
      consul_client_node.vm.provision "shell", inline: "ssh-keyscan #{sIP_C1} 2>/dev/null >> ~/.ssh/known_hosts"
      consul_client_node.vm.provision "shell", inline: "cp -r #{sHOME}/.ssh /root/."
      consul_client_node.vm.provision "shell", inline: "rsync -q vagrant@#{sIP_C1}:~/*.pem #{sHOME}/. && chown -R #{sVUSER} #{sHOME}/"
      consul_client_node.vm.provision "shell", inline: "rsync -q vagrant@#{sIP_C1}:~/gossip.txt #{sHOME}/. && chown -R #{sVUSER} #{sHOME}/"
#      consul_client_node.vm.network "forwarded_port", guest: 80, host: "5818#{iX}", id: "vault#{iX}"
      consul_client_node.vm.provision "shell", inline: "/bin/bash -c 'SETUP=client #{sHOME}/install_consul.sh'"
    end
  end

end
