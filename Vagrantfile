VAGRANTFILE_API_VERSION = "2"
VAGRANT_DISABLE_VBOXSYMLINKCREATE = "1"
file_to_disk1 = './disk-0-1.vdi'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
# Use same SSH key for each machine
config.ssh.insert_key = false
config.vm.box_check_update = false
config.vm.define "ipa" do |ipa|
  ipa.vm.box = "centos/7"
  ipa.vm.provision :shell, :inline => " sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;  systemctl restart sshd;", run: "always"
  ipa.vm.provision :shell, :inline => " yum install -y wget createrepo httpd ipa-server ipa-server-dns vsftpd createrepo;", run: "always"
  ipa.vm.provision :shell, :inline => " yum -y install python36;  curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py ; python get-pip.py ; pip install -U pip ; pip install pexpect;", run: "always"
  ipa.vm.provision :shell, :inline => " mkdir -p /var/www/html/rpms;", run: "always"
#  ipa.vm.provision :shell, :inline => "for i in \"Development Tools\" \"Server with GUI\" \"Web Server\"; do yum group install \"$i\" --downloadonly --downloaddir=/var/www/html/rpms;done;", run: "always"
#  ipa.vm.provision :shell, :inline => "createrepo /var/www/html/rpms", run: "always"
  ipa.vm.hostname = "ipa.example.com"
  ipa.vm.network "private_network", ip: "192.168.55.5"
  ipa.vm.provider :virtualbox do |ipa|
    ipa.customize ['modifyvm', :id,'--memory', '2048']
    end
  ipa.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbooks/ipa.yml"
#    ansible.verbose = true
  end
end
  
config.vm.define "repo" do |repo|
  repo.vm.box = "centos/7"
  repo.vm.hostname = "repo.example.com"
  repo.vm.network "private_network", ip: "192.168.55.4"
  repo.vm.network "private_network", ip: "192.168.55.101"
  repo.vm.network "private_network", ip: "192.168.55.102"
  repo.vm.provision :shell, :inline => " sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;  systemctl restart sshd; echo vagrant |  passwd vagrant --stdin;", run: "always"
  repo.vm.provision :shell, :inline => " sudo systemctl stop packagekit; sudo systemctl mask packagekit", run: "always"
  repo.vm.provision :shell, :inline => " mkdir -p /var/www/html/rpms;", run: "always"
  repo.vm.provider "virtualbox" do |repo|
    repo.memory = "1024"
  end
  repo.vm.provision :shell, :inline => " yum -y install createrepo httpd ipa-client", run: "always"
  repo.vm.provision :shell, :inline => " yum group install -y \"Development Tools\" ;  yum install -y python-devel curl ; curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py ; python get-pip.py ;  pip install -U pip ;  pip install pexpect;", run: "always"
  repo.vm.provision :shell, :inline => "pip install ansible", run: "always"
  repo.vm.provision "ansible_local" do |ansible|
    ansible.playbook = 'playbooks/repo.yml'
#    ansible.verbose = true
  end
end

config.vm.define "system1" do |system1|
  system1.vm.box = "puppetlabs/centos-7.0-64-nocm"
  system1.vm.hostname = "system1.example.com"
  system1.vm.network "private_network", ip: "192.168.55.6"
  system1.vm.network "private_network", ip: "192.168.55.101"
  system1.vm.network "private_network", ip: "192.168.55.102"
  system1.vm.provision :shell, :inline => " yum -y install httpd ipa-client createrepo", run: "always"
  system1.vm.provision :shell, :inline => " sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config;  systemctl restart sshd; echo vagrant |  passwd vagrant --stdin;", run: "always"
  system1.vm.provision :shell, :inline => " sudo systemctl stop packagekit; sudo systemctl mask packagekit", run: "always"
  system1.vm.provider "virtualbox" do |system1|
    system1.memory = "1024"

    if not File.exist?(file_to_disk1)
      system1.customize ['createhd', '--filename', file_to_disk1, '--variant', 'Fixed', '--size', 10 * 1024]
    end
    system1.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 2]
    system1.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk1]
  end
  
    system1.vm.provision "shell", inline: <<-SHELL
    yes|  mkfs.ext4 /dev/sdb
    SHELL
  system1.vm.provision :shell, :inline => " yum group install -y \"Development Tools\" ;  yum install -y python-devel curl ; curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py ; python get-pip.py ;  pip install -U pip ;  pip install pexpect;", run: "always"
  system1.vm.provision :shell, :inline => "pip install ansible", run: "always"
  system1.vm.provision "ansible_local" do |ansible|
    ansible.playbook = 'playbooks/system.yml'
#    ansible.verbose = true
  system1.vm.provision :shell, :inline => "reboot", run: "always"
  end
end
end