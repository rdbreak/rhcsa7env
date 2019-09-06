VAGRANTFILE_API_VERSION = "2"
VAGRANT_DISABLE_VBOXSYMLINKCREATE = "1"
file_to_disk1 = './disk-0-1.vdi'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
# Use same SSH key for each machine
config.ssh.insert_key = false
config.vm.box_check_update = false

config.vm.define "repo" do |repo|
  repo.vm.box = "rdbreak/pracrepo"
#  repo.vm.hostname = "repo.example.com"
  repo.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/"
  repo.vm.network "private_network", ip: "192.168.55.4"

  repo.vm.provider "virtualbox" do |repo|
    repo.memory = "1024"
  end
end

config.vm.define "ipa" do |ipa|
  ipa.vm.box = "centos/7"
  ipa.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "always"
  ipa.vm.provision :shell, :inline => "yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm -y; sudo yum install -y sshpass python36-pip python36-devel httpd sshpass vsftpd createrepo pki-ca", run: "always"
  ipa.vm.provision :shell, :inline => "alternatives --set python /usr/bin/python36 ; python36 -m pip install -U pip ; python36 -m pip install pexpect; python36 -m pip install ansible", run: "always"
  ipa.vm.provision :shell, :inline => "echo \'vagrant\' | sudo passwd vagrant --stdin", run: "always"
  ipa.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/"
#  ipa.vm.hostname = "ipa.example.com"
  ipa.vm.network "private_network", ip: "192.168.55.5"
  ipa.vm.provider :virtualbox do |ipa|
    ipa.customize ['modifyvm', :id,'--memory', '2048']
    end
    ipa.vm.provision :ansible_local do |ansible|
      ansible.playbook = "/vagrant/playbooks/master.yml"
      ansible.install = false
      ansible.compatibility_mode = "2.0"
      ansible.inventory_path = "/vagrant/inventory"
      ansible.config_file = "/vagrant/ansible.cfg"
      ansible.limit = "all"
     end
end
config.vm.define "system1" do |system1|
  system1.vm.box = "rdbreak/pracenvs"
#  system1.vm.hostname = "system1.example.com"
  system1.vm.network "private_network", ip: "192.168.55.6"
  system1.vm.network "private_network", ip: "192.168.55.101"
  system1.vm.network "private_network", ip: "192.168.55.102"
  system1.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "always"
  system1.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/"

  system1.vm.provider "virtualbox" do |system1|
    system1.memory = "1024"

    if not File.exist?(file_to_disk1)
      system1.customize ['createhd', '--filename', file_to_disk1, '--variant', 'Fixed', '--size', 10 * 1024]
    end
    system1.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 2]
    system1.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk1]
  end
  
    system1.vm.provision "shell", inline: <<-SHELL
    yes|  mkfs.ext4 -L extradisk /dev/sdb
    SHELL
    system1.vm.provision "shell", inline: <<-SHELL
    mkdir /extradisk ; echo \'LABEL=extradisk /extradisk ext4 defaults 0 0\' >> /etc/fstab
    SHELL
    system1.vm.provision :ansible_local do |ansible|
      ansible.playbook = "/vagrant/playbooks/system.yml"
      ansible.install = false
      ansible.compatibility_mode = "2.0"
      ansible.inventory_path = "/vagrant/inventory"
      ansible.config_file = "/vagrant/ansible.cfg"
      ansible.limit = "all"
     end
    system1.vm.provision :shell, :inline => "reboot", run: "always"
end
end