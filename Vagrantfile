VAGRANTFILE_API_VERSION = "2"
VAGRANT_DISABLE_VBOXSYMLINKCREATE = "1"
file_to_disk = './disk-0-0.vdi'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
# Use same SSH key for each machine
config.ssh.insert_key = false
config.vm.box_check_update = false
config.vm.define "ipa" do |ipa|
  ipa.vm.box = "centos/7"
  ipa.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "always"
#  ipa.vm.network "forwarded_port", guest: 80, host: 8089
#  ipa.vm.network "forwarded_port", guest: 443, host: 8450
  ipa.vm.hostname = "ipa.example.com"
  
  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  ipa.vm.network "private_network", ip: "192.168.55.5"
  ipa.vm.network "private_network", ip: "192.168.55.100"
  ipa.vm.network "private_network", ip: "192.168.55.101"

  ipa.vm.provider :virtualbox do |ipa|
    ipa.customize ['modifyvm', :id,'--memory', '2048']
    end
end
config.vm.define "system" do |system|
  system.vm.box = "puppetlabs/centos-7.0-64-nocm"
#  system.vm.network "forwarded_port", guest: 80, host: 8090
#  system.vm.network "forwarded_port", guest: 443, host: 8451
  system.vm.hostname = "system1.example.com"
  system.vm.network "private_network", ip: "192.168.55.6"
  system.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "always"
  system.vm.provision :shell, :inline => "sudo rm -rf /etc/yum.repos.d/* ; touch /etc/yum.repos.d/ipa.repo;", run: "always"
  system.vm.provider "virtualbox" do |system|
    system.memory = "1024"

    if not File.exist?(file_to_disk)
      system.customize ['createhd', '--filename', file_to_disk, '--variant', 'Fixed', '--size', 10 * 1024]
    end
    system.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 1]
    system.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
  end
  
    config.vm.provision "shell", inline: <<-SHELL
    yes| sudo mkfs.ext4 /dev/sdb
    SHELL

  system.vm.provision "ansible" do |ansible|
    ansible.version = "latest"
    ansible.limit = "all"
    ansible.playbook = 'playbooks/master.yml'
    end
  end
end
