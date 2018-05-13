## A Vagrant based Mesos cluster

** This is purely for educational purpose.

Original Source from [krestijaninoff](https://github.com/krestjaninoff/vagrant-mesos-cluster) <br>
I have replaced the docker role with [geerlingguy](https://github.com/geerlingguy/ansible-role-docker) due to a problem with the selinux. <br>
Also changed the vagrant box image to 'centos/7' <br>

This includes the following technologies:

  * Zookeeper
  * Mesos (+ MesosDns)
  * Marathon (+ HaProxy based LoadBalancer)
  * Docker

### Dependencies

Note, you have to install

  * vagrant (>= 1.7.4, tested on Virtualbox=5.0-5.0.10 as a provider)
  * ansible (>= 1.9.4)

on your local machine before you can start.


### Getting started

#### Cluster

First of all, open the "Vagrantfile" and study it :) As you can see, we are going to create 3 groups of servers:

  * Mesos masters
  * Mesos nodes/slaves

Each group is provided by its own Ansible playbook (see "ansible" folder). The most intersting part here is
"role" folder, where you can find all the installation logic for the specific components.

Anyway, all you need to do to start the party is

```
vagrant up
```

It takes some time to download and install all the stuff... so be patient :)


##### Web UIs

After the job is done you can try to check the following UIs:

 * Mesos    - http://192.168.99.11:5050
 * Marathon - http://192.168.99.11:8080

Update your `/etc/hosts` file according to what you see in the head of the Vagrantfile,
if you want to use short names like `master1`.


#### Launch application
```
sh run.sh
```

#### Install and setup dcos cli

```
cd /usr/local/bin/
curl -O https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.8/dcos
sudo -i
cd
dcos config set core.dcos_url http://192.168.99.11:5050
dcos config set marathon.url http://192.168.99.11:8080
dcos marathon app list
dcos package install --cli cassandra
```


### Troubleshooting

#### Proxy

If you need a proxy support, perform the following:

```
sudo yum install libvirt-devel
vagrant plugin install vagrant-proxyconf
```

Then update $HOME/.vagrant.d/Vagrantfile to apply your settings for all VMs:

```
Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://10.0.0.1:3128"
    config.proxy.https    = "http://10.0.0.1:3128"
    config.proxy.no_proxy = "localhost,127.0.0.1"
  end
end
