#
# A Vagrant-based Mesos cluster
#

# List of VM and their roles
HOSTS = {
  "masters" => {
    "master1" => "192.168.99.11",
    "master2" => "192.168.99.12",
    "master3" => "192.168.99.13"
  },
  "nodes" => {
    "node1"   => "192.168.99.21",
    "node2"   => "192.168.99.22",
    "node3"   => "192.168.99.23",
    "node4"   => "192.168.99.24"
  },
  "logs" => {
    "log1"    => "192.168.99.30"
  }
}

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

  HOSTS['masters'].each_with_index do |host, index|
    config.vm.define host[0] do |machine|
      machine.vm.hostname = host[0]

      machine.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = "1"
      end

      # Update host's /etc/host file to access VM by its names
      # Mesos    - http://192.168.99.11:5050
      # Marathon - http://192.168.99.11:8080
      # HaProxy  - http://192.168.99.11:9090/haproxy?stats
      machine.vm.network "private_network", :ip => host[1]

      machine.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/master.yml"
          ansible.groups = { "masters" => HOSTS['masters'].keys }
          ansible.extra_vars = {
            zk_id: "#{index + 1}",
            zk_host1: "master1",
            zk_host2: "master2",
            zk_host3: "master3",
            mesos_quorum: 2,
            marathon_host1: "master1",
            marathon_host2: "master2",
            marathon_host3: "master3",
            hosts: HOSTS['masters'].merge(HOSTS['nodes'])
          }
      end

    end
  end

  HOSTS['nodes'].each_with_index do |host, index|
    config.vm.define host[0] do |machine|
      machine.vm.hostname = host[0]
      machine.vm.network "private_network", :ip => host[1]

      machine.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = "2"
      end

      machine.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/node.yml"
          ansible.groups = { "nodes" => HOSTS['nodes'].keys }
          ansible.extra_vars = {
            zk_host1: "master1",
            zk_host2: "master2",
            zk_host3: "master3",
            es_logs_host: HOSTS['logs'].keys[0],
            hosts: HOSTS['masters'].merge(HOSTS['nodes']).merge(HOSTS['logs'])
          }
      end
    end
  end

  # Kibana to be available at: http://192.168.99.24:5601
  HOSTS['logs'].each_with_index do |host, index|
    config.vm.define host[0] do |machine|
      machine.vm.hostname = host[0]
      machine.vm.network "private_network", :ip => host[1]

      machine.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = "1"
      end

      machine.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/log.yml"
          ansible.groups = { "logs" => HOSTS['logs'].keys }
          ansible.extra_vars = {
            hosts: HOSTS['logs']
          }
      end
    end
  end

end
