VAGRANTFILE_API_VERSION = "2"
VAGRANT_DISABLE_VBOXSYMLINKCREATE = "1"
file_to_disk1 = './disk-0-1.vdi'
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
# Use same SSH key for each machine
config.ssh.insert_key = false
config.vm.box_check_update = false
config.vm.define "ipa" do |ipa|
  ipa.vm.box = "centos/7"
  ipa.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "always"
  ipa.vm.provision :shell, :inline => "sudo yum install -y wget | grep -v 'warning\|Error'; sudo cd /etc/pki/rpm-gpg;  sudo wget http://yum.theforeman.org/releases/1.8/RPM-GPG-KEY-foreman; sudo rpm --import RPM-GPG-KEY-foreman; sudo rpm -qa gpg* ; sudo rpm -qi gpg-pubkey-225c9b71-54fda121;", run: "always"
  ipa.vm.provision :shell, :inline => "sudo yum install -y epel-release; sudo yum -y install python36; sudo curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py ; python get-pip.py ; sudo pip install -U pip ; sudo pip install pexpect;", run: "always"
  ipa.vm.provision :shell, :inline => "sudo mkdir -p /var/www/html/rpms;", run: "always"
  ipa.vm.provision :shell, :inline => "for i in \"Development Tools\" \"Server with GUI\" \"File and Print Server\" \"Web Server\" ; do yum group install \"$i\" --downloadonly --downloaddir=/var/www/html/rpms;done;", run: "always"
  ipa.vm.provision :shell, :inline => "yum install -y httpd-manual selinux\* sssd\* --downloadonly --downloaddir=/var/www/html/rpms;", run: "always"
  ipa.vm.hostname = "ipa.example.com"
  ipa.vm.network "private_network", ip: "192.168.55.5"
  ipa.vm.provider :virtualbox do |ipa|
    ipa.customize ['modifyvm', :id,'--memory', '2048']
    end
end
  
config.vm.define "system1" do |system1|
  system1.vm.box = "puppetlabs/centos-7.0-64-nocm"
  system1.vm.hostname = "system1.example.com"
  system1.vm.network "private_network", ip: "192.168.55.6"
  system1.vm.network "private_network", ip: "192.168.55.101"
  system1.vm.network "private_network", ip: "192.168.55.102"
  system1.vm.provision :shell, :inline => "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config; sudo systemctl restart sshd;", run: "always"
#  system1.vm.provision :shell, :inline => "sudo rm -rf /etc/yum.repos.d/* ; touch /etc/yum.repos.d/ipa.repo;", run: "always"
  system1.vm.provider "virtualbox" do |system1|
    system1.memory = "1024"

    if not File.exist?(file_to_disk1)
      system1.customize ['createhd', '--filename', file_to_disk1, '--variant', 'Fixed', '--size', 10 * 1024]
    end
    system1.customize ['storagectl', :id, '--name', 'SATA Controller', '--add', 'sata', '--portcount', 2]
    system1.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk1]
  end
  
    system1.vm.provision "shell", inline: <<-SHELL
    yes| sudo mkfs.ext4 /dev/sdb
    SHELL
  system1.vm.provision "ansible" do |ansible|
    ansible.version = "latest"
    ansible.limit = "all"
    ansible.playbook = 'playbooks/master.yml'
  end
end
end