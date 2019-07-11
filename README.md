# Vagrant/Ansible Study/Test Environment.

## Required software before setting up:
- Ansible - (`yum install ansible` or `brew install ansible`)
- Python2 - (`yum install python2`)
- [Vagrant](https://www.vagrantup.com/downloads.html)
- [Virtualbox](https://www.virtualbox.org/wiki/Downloads) 

Once the setup is complete, the ipa server and client for realm EXAMPLE.COM will already been setup and paired. 

The machines will take about 10 minutes to fully set up and then they will reboot at the end.

### It includes two systems:
- ipa.example.com
- system1.example.com (IP address: 192.168.55.6)

### Network Details:
###### ipa
192.168.55.5
###### system1
192.168.55.6


