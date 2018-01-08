---
title: Vagrant, VM and Development
subtitle: 
date: '2016-08-14'
slug: vagrant-vm-dev
---

# Creating production environment locally

Currently the workflow that I use for local development involves using Vagrant
to create a virtual machine and then run Ansible to provision the machine with
environment that will be on the production system. The benefits of this is
that the virtual machine can be easily created and destroyed (in case you mess
something up) and you will know exactly what happens when you push to
production. This is ideal, since you will most likely be setting up a server
that will not be the exact same as your local machine.

## Local development

The gist goes Vagrantfile -> Ansible -> development. The `Vagrantfile` file I
generally have looks like this:

```
    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    
    # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
    VAGRANTFILE_API_VERSION = "2"
    
    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
        config.vm.box = "bento/ubuntu-16.04"
    
        config.vm.define "machine"
        config.vm.hostname = "machine"
        config.vm.network "private_network", ip: "192.168.100.200"
        config.vm.network :forwarded_port, guest: 5432, host: 5432
        config.vm.network :forwarded_port, guest: 8000, host: 8000
    
        config.vm.provision :ansible do |ansible|
            ansible.playbook = "ansible/site.yml"
            ansible.groups = {
                "vagrant" => ["machine",]
            }
        end
    
        config.vm.provider "virtualbox" do |v|
            v.memory = 2048
            v.cpus = 2
        end
    
    end
    
```

This Vagrantfile will create an Ubuntu OS, and then it will set up the
networking, namely the private IP and ports forwarded for your usage. This
part is very important, because opening the ports will allow your machine to
talk to other applications that might need to use it. Vagrant will also open
up a SSH port by default.

## Provisioning

The set after the VM configuration is to provision the VM, which in this case
is done with Ansible. Provisioning is essentially the procedure after you're
acquired all the ingredients. Ansible uses a concept called a playbook, which
consists of a set of "roles" that are run in order to install packages and
other tasks. A simple `ansible/site.yml` playbook will look like:

```
    - name: Provision project
      hosts: all
      become: true
    
      roles:
        - common
        - db
        - project
```

This shows us what roles will need to be looked at, and the host that the
playbook is pointed at. Within the roles folder, there will be tasks,
handlers, and conf. The [Ansible docs][2] are fairly well written, and I found
them to be very useful. A quick example is that within the common role, you
would have a task that does all the `apt-get` installations:

```
    - name: Install basic utilities
      apt: name={{ item }} state=present
      become: yes
      with_items:
        - vim
        - git
        - curl
        - tree
        - zip
        - unzip
        - screen
        - iotop
        - htop
```

With the concept of playbooks, it will allow you to have specific recipes for
dev (`dev-site.yml`), and for prod (`prod-site.yml`).

### Port Forwarding

In order to be able to communicate between the VM, and your computer that you
used to make the VM, is to have the ports configured properly, and no firewall
conflicts.

For example, if you have a PostgreSQL database, and a Flask app, you might
want to be able to run both services and be able to communicate with them in
another VM, or just locally. This means that the port must be opened in the
VM, but there also needs to be a firewall rule in a `task/main.yml` file:

```
    - name: Allow SSH
      ufw: rule=allow port=22
    
    - name: Allow Postgres
      ufw: rule=allow port=5432
    
    - name: Set firewall default policy
      ufw: state=enabled policy=reject
    
    - name: Allow HTTP-development
      ufw: rule=allow port=8000
```

Doing so means you can run your Flask app on the VM, and go to your local
browser and be able to view it. This is pretty much required for any web
development since you'll want to be able to go to your browser and visit
`localhost:8000` to view what your webpage looks like, instead of being
limited to running curl within your VM.

[2]: http://docs.ansible.com/ansible/playbooks_roles.html
