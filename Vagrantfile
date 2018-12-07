Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config && sudo systemctl restart sshd"
  config.vm.provision "shell", inline: "sudo echo 'PATH=/usr/local/bin/:$PATH' >> /root/.bashrc"

  config.vm.define "management" do |management|
    management.vm.box = "fedora/28-cloud-base"
    management.vm.provision "shell", inline: "sudo hostnamectl set-hostname management"
    management.vm.network :private_network,
      :type => "dhcp",
      :ip => "192.168.122.10"
  end

  config.vm.define "node1" do |node1|
    node1.vm.box = "centos/7"
    node1.vm.provision "shell", inline: "sudo hostnamectl set-hostname node1"
    node1.vm.provision "shell", inline: "sudo yum update -y"
    node1.vm.network :private_network,
      :type => "dhcp",
      :ip => "192.168.122.101"
    node1.vm.network :private_network,
       :type => "dhcp",
       :ip => "192.168.123.101"
    node1.vm.provider :libvirt do |domain|
      domain.cpus = 4
      domain.memory = 4096
      domain.nested = true
    end
  end
end

