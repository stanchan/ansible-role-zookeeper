# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.7.0"

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

unless Vagrant.has_plugin?("vagrant-hostmanager")
  raise 'vagrant-hostmanager plugin is not installed! https://github.com/devopsgroup-io/vagrant-hostmanager'
end

boxes = [
    {
        :name => "zookeeper.shift-labs.com",
        :eth1 => "172.16.1.100",
        :mem => "2048",
        :cpu => "1",
        :box => "debian/stretch64"
    },
    {
        :name => "zookeeper2.shift-labs.com",
        :eth1 => "172.16.1.101",
        :mem => "2048",
        :cpu => "1",
        :box => "debian/stretch64"
    },
    {
        :name => "zookeeper3.shift-labs.com",
        :eth1 => "172.16.1.102",
        :mem => "2048",
        :cpu => "1",
        :box => "debian/stretch64"
    }
]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false

  boxes.each_with_index do |options, index|

    config.vm.define options[:name] do |config|
      config.vm.box = options[:box]
      config.ssh.insert_key = false
      config.vm.hostname = options[:name]
      config.vm.provider "virtualbox" do |v|
        v.name = options[:name]
        v.customize ["modifyvm", :id, "--memory", options[:mem]]
        v.customize ["modifyvm", :id, "--cpus", options[:cpu]]

        v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        v.customize ["modifyvm", :id, "--ioapic", "on"]
      end
      config.vm.network :private_network, ip: options[:eth1]

      if index == 0
        config.vm.network "forwarded_port", guest: 2181, host: 2181
        config.vm.network "forwarded_port", guest: 9081, host: 9081
        config.vm.network "forwarded_port", guest: 10081, host: 10081
      end

      if index == boxes.size - 1
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "zookeeper.yml"
          ansible.limit = "all"
          ansible.groups = {
            "all:vars" => {
              "ansible_become": "true",
              "zookeeper_netif": "ansible_eth1",
              "zookeeper_clean": "false"
            }
          }
          ansible.host_key_checking = false
          ansible.verbose = "v"
        end
      end
    end
  end
end