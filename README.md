# Ansible LAMP Stack

This is a demo **Ansible** project showing the basics of setting up a multi-server **LAMP** stack + **HA-Proxy** loadbalancer.

### Requirements

- [Vagrant](http://vagrantup.com)

### The Stack

- Ubuntu 14.04
- HA-Proxy v1.4.24
- Apache v2.4
- PHP v5.5
- Composer
- NodeJS + NPM
    + Grunt
    + Gulp
    + Bower
    + Node Sass
- MySQL v5.5

### Boxes

- 1x HA-Proxy Loadbalancer
- 2x Apache + PHP
- 1x MySQL

The LAMP stack operates as follows: the LB forwards port **80** to port **8000** on the host and balances traffic via round-robin. Each web server is also directly available on host ports **8001** and **8002**. The MySQL box forwards port **3306** to port **33060** on the host for easy access via client apps like Sequel Pro. Each box has an **admin** and **deploy** user. Their passwords are protected via **ansible-vault**. The vault file can be found in ```roles/users/vars/main.yml``` and the password is **admin**.

### Setup

This project contains 2 Vagrant files in separate folders. ```ansible-ctrl``` has the "control" machine. It does the provisioning of the boxes in ```ansible-workers```. I suppose it goes without saying, but you will need to have both sets of machines running. There are 5 in all @ ```512 MB``` each, so make sure you have enough memory on your host machine.

To get it all running, open 2 tabs in terminal, one for ```ansible-ctrl/``` and one for ```ansible-workers/```:

**ansible-workers/**

Note: Make sure this finishes before moving on.

```bash
$ vagrant up
```

**ansible-ctrl/**

```bash
$ vagrant up
$ vagrant ssh
$ cd ~/.ssh
$ ssh-keygen -t rsa -b 4096 -C "your-email@domain.com"
```
    
Repeat the next step for 16 - 19. When prompted the password is **vagrant**. This will allow ansible to ssh into each box it's provisioning via public key.

```bash    
$ ssh-copy-id vagrant@192.168.22.xx
```

And now to provision the stack. The vault password is **admin**:

```bash
$ cd /etc/ansible/playbooks
$ ansible-playbook main.yml --ask-vault-pass
```

Assuming all went well, you should be able to visit ```http://localhost:8000``` in your browser and see a welcome message from **ansible-site.dev**. You can also visit ```/info.php``` to see info about the PHP installation.

### Where to go from here

Ansible is fairly simple to understand. Everything is configured in **YAML**. Start with **ansible-ctrl/main.yml** and move on from there. main.yml includes the bits and pieces that we want to provision. The **roles** in each of the included files represent the sub-folders in the **roles** dir. For each role, **tasks/main.yml** is loaded up first. It can perform tasks, or include other files for better organization. **vars/main.yml** contains, you guessed it, variables to be used in your tasks and templates. **handlers/main.yml** contains commands to be run when they're "*notified*" by a task, ie. restarting Apache.

Try provisioning your own site. Take a look at ```ansible-ctrl/webservers.yml```. You can add more sites there. Also be sure to look at ```ansible-ctrl/roles/webserver/templates/``` to see how the site's ```vhost```, ```index.html```, and ```index.php``` are being created. You can also add databases in ```ansible-ctrl/dbservers.yml```.