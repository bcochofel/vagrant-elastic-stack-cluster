# -*- mode: ruby -*-
# vi: set ft=ruby :

### configuration parameters ###

# Vagrant variables
VAGRANTFILE_API_VERSION = "2"
DEFAULT_BOX_NAME = "bento/centos-7.6"
DEFAULT_VM_RAM = "1024"
DEFAULT_VM_CPU = "1"

# servers
servers = [
  { :hostname => 'master-1', :ip => '192.168.77.30', :ram => 1024, :cpus => 1 },
  { :hostname => 'data-1', :ip => '192.168.77.40', :ram => 1024, :cpus => 1 },
  { :hostname => 'data-2', :ip => '192.168.77.41', :ram => 1024, :cpus => 1 },
  { :hostname => 'data-3', :ip => '192.168.77.42', :ram => 1024, :cpus => 1 },
  { :hostname => 'logstash-1', :ip => '192.168.77.50', :ram => 2048, :cpus => 1 },
  { :hostname => 'kibana-1', :ip => '192.168.77.60', :ram => 2048, :cpus => 1 }
]

# ansible groups for inventory
ansible_groups = {
  "master_nodes" => [
    "master-1"
  ],
  "data_nodes" => [
    "data-1",
    "data-2",
    "data-3"
  ],
  "logstash_nodes" => [
    "logstash-1"
  ],
  "kibana_nodes" => [
    "kibana-1"
  ],
  "all:vars" => {
    "cluster_initial_master_nodes" => "[\"master-1\"]",
    "discovered_seed_hosts" => "[\"192.168.77.30:9300\"]",
    "elasticsearch_servers" => "[\"master-1:9200\"]",
    "logstash_servers" => "[\"logstash-1:5044\"]"
  }
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # vagrant-hostmanager options
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = false

  # Always use Vagrant's insecure key
  #config.ssh.insert_key = false
  
  # Forward ssh agent to easily ssh into the different machines
  config.ssh.forward_agent = true

  # Provision servers
  servers.each do |server|
    config.vm.define server[:hostname] do |config|
      config.vm.hostname = server[:hostname]

      # Vagrant box
      config.vm.box = server[:box] ? server[:box] : DEFAULT_BOX_NAME;

      config.vm.network :private_network, ip: server[:ip]

      memory = server[:ram] ? server[:ram] : DEFAULT_VM_RAM;
      cpus = server[:cpus] ? server[:cpus] : DEFAULT_VM_CPU;

      config.vm.provider :virtualbox do |vb|
        vb.customize [
          "modifyvm", :id,
          "--memory", memory.to_s,
          "--cpus", cpus.to_s,
          "--ioapic", "on",
          "--natdnshostresolver1", "on",
          "--natdnsproxy1", "on"
        ]
      end

      # check if it's the last VM
      if server[:hostname] == servers[-1][:hostname]
        # preinst configurations
        config.vm.provision "preinst", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/preinst.yml"
          ansible.groups = ansible_groups
          ansible.limit = "all"
        end
        # telegraf configurations
        config.vm.provision "telegraf", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/telegraf.yml"
          ansible.groups = ansible_groups
          ansible.limit = "all"
        end
        # monitoring configurations
        config.vm.provision "monitoring", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/monitoring.yml"
          ansible.groups = ansible_groups
          ansible.limit = "kibana_nodes"
        end
        # install elasticsearch on master nodes
        config.vm.provision "master_nodes", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/master_nodes.yml"
          ansible.groups = ansible_groups
          ansible.limit = "master_nodes"
        end
        # install elasticsearch on data nodes
        config.vm.provision "data_nodes", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/data_nodes.yml"
          ansible.groups = ansible_groups
          ansible.limit = "data_nodes"
        end
        # install kibana and coordination-only node
        config.vm.provision "kibana_nodes", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/kibana_nodes.yml"
          ansible.groups = ansible_groups
          ansible.limit = "kibana_nodes"
        end
        # install logstash
        config.vm.provision "logstash_nodes", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/logstash_nodes.yml"
          ansible.groups = ansible_groups
          ansible.limit = "logstash_nodes"
        end
        # install beats
        config.vm.provision "beats", type: "ansible" do |ansible|
          ansible.compatibility_mode = "auto"
          ansible_config_file = "ansible/ansible.cfg"
          ansible.playbook = "ansible/beats.yml"
          ansible.groups = ansible_groups
          ansible.limit = "all"
        end
      end
    end
  end
end
