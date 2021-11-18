# Infrastructure as Code with Ansible

## Controller Setup
SSH in box
```
vagrant ssh (box name)
```

```
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install software-properties-common -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get install ansible -y
```
### Connecting to Servers with Ansible
To make viewing files easier
```
sudo apt-get install tree -y
```
To SSH into the different boxes from ansible from /etc/ansible
 ```
ssh vagrant@boxIP(in Vagrantfile)
 ```

Add new /etc/ansible/hosts file
```
sudo rm -rf hosts
sudo nano hosts
```
add into the hosts file
```
[web]
192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant 

[db]
192.168.33.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant

```
To ping all servers from ansible (/etc/ansible)
```
# replace all with name of server to ping individually
ansible all -m ping
```
### Ansible adhoc commands
Allow you to run commands on the agent nodes from the controller
e.g.
```
ansible web -a "hostnamectl"
ansible web -a "uname -a"
ansible db -a "date"
ansible db -a "free"
ansible web -m shell -a "ls -a"
```
checking uptime of servers
```
ansible all -m shell -a uptime
```
installing nginx on web server
```
ansible web -a "sudo apt-get install nginx -y
```

## Ansible Playbook
- Create new file `nginx.yml` in /etc/ansible
```
# define the agent node's name host name

- hosts: web

# see logs
  gather_facts: yes

# sudo permision with become = true
  become: true


# install nginx on web server
  tasks:
  - name: Install nginx
    apt: pkg=nginx state=present
```
- Run the playbook
  ```
  ansible-playbook nginx.yml
  ```

### Ansible Controller and Agent nodes set up with Vagrant
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what

# MULTI SERVER/VMs environment
#
Vagrant.configure("2") do |config|

# creating first VM called web
  config.vm.define "web" do |web|

    web.vm.box = "bento/ubuntu-18.04"
   # downloading ubuntu 18.04 image

    web.vm.hostname = 'web'
    # assigning host name to the VM

    web.vm.network :private_network, ip: "192.168.33.10"
    #   assigning private IP

    config.hostsupdater.aliases = ["development.web"]
    # creating a link called development.web so we can access web page with this link instread of an IP

    end

# creating second VM called db
  config.vm.define "db" do |db|

    db.vm.box = "bento/ubuntu-18.04"

    db.vm.hostname = 'db'

    db.vm.network :private_network, ip: "192.168.33.11"

    config.hostsupdater.aliases = ["development.db"]
    end

# creating are Ansible controller
  config.vm.define "controller" do |controller|
    
    controller.vm.box = "bento/ubuntu-18.04"
    
    controller.vm.hostname = 'controller'
    
    controller.vm.network :private_network, ip: "192.168.33.12"
    
      
  end
end
```