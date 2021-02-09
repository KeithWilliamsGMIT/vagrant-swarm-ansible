# Vagrant Swarm Ansible

This repository shows how to setup a cluster of virtual machines using Vagrant. These virtual machines will be configured to work with Docker Swarm and Ansible by assigning each node to either a manager or worker group. The aim is to create a multi-node Swarm locally to simulate a cloud environment.

## Getting Started

The first step to getting started is to clone the repository:

```bash
git clone https://github.com/KeithWilliamsGMIT/vagrant-swarm-ansible.git
```

Next we can use Vagrant to create two Ubuntu 18.04 virtual machines with the names `node(n)` and with IPv4 addresses of `192.168.2.(n+1)` on a private network through the `eth1` network interface and `192.168.1.(n+149)` through `wlo1`. These nodes will also have X11 forwarding enabled by default but this can be removed if needed. To create this cluster first install Vagrant and VirtualBox and then run the following command:

```bash
vagrant up
```

This may take some time when first running it as the Vagrant box will need to be downloaded but subsequent runs should be quicker. These virtual machines are provisioned using Ansible to update `pip` which may also take some time. Vagrant will generate an Ansible inventory file which saves us from manually writing one ourselves. We define which hosts are apart of which group in the `Vagrantfile`. The inventory file will look similar to the following for two nodes:

```
node1 ansible_host=127.0.0.1 ansible_port=2222 ansible_user='vagrant' ansible_ssh_private_key_file='/home/ubuntu/repos/vagrant-swarm-ansible/.vagrant/machines/node1/virtualbox/private_key'
node2 ansible_host=127.0.0.1 ansible_port=2203 ansible_user='vagrant' ansible_ssh_private_key_file='/home/ubuntu/repos/vagrant-swarm-ansible/.vagrant/machines/node2/virtualbox/private_key'

[docker_swarm_manager]
node[1:1]

[docker_swarm_worker]
node[2:2]

[vagrant]
node[1:2]
```

There is a manager and worker group which will define the role of the nodes in the Swarm. You can define how many managers and workers you want to create. Note that all hosts are also apart of a group called `vagrant`. This allows you to define group variables when running Ansible against the Vagrant cluster.

To customise the virtual machines you can create a `Customfile`. This file will be ignored by Git. For example, to increase the memory to 4GB you can create a `Customfile` with the below content and then reload the virtual machines using `vagrant reload`.

```ruby
config.vm.provider :virtualbox do |v|
  v.customize ["modifyvm", :id, "--memory", 4096]
end
```

## Does it work?

To check if both virtual machines are running you can use the command `vagrant global-status`:

```bash
$ vagrant global-status
id       name   provider   state   directory
-------------------------------------------------------------------------
92f9f47  node1  virtualbox running /home/ubuntu/repos/vagrant-swarm-ansible
206e274  node2  virtualbox running /home/ubuntu/repos/vagrant-swarm-ansible

...
```

Once they are running you can ssh into any of the nodes. For example, to ssh into `node1` use the command:

```bash
vagrant ssh node1
```

## Conclusion

Vagrant is a very powerful tool which can be quickly used to spin up a cluster of virtual machines. This is extremely useful for simulating a cloud environment and testing Docker Swarm stacks locally on a multi-node architecture. Finally, it integrates well with Ansible. Ansible can be used to provision the virtual machines and the generated inventories file allows you to easily run any other roles on the virtual machines.