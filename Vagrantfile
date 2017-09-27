# -*- mode: ruby -*-
# # vi: set ft=ruby :

# Require YAML module
require 'yaml'

# test-uat and prod are current choices.
env = 'public-network'

# Read YAML file with box details
environment = "vagrant/#{env}.yaml"
servers = YAML.load_file(environment)

# Create boxes
Vagrant.configure("2") do |config|

  # Iterate through entries in YAML file
  servers.each do |server|
    config.vm.define server["name"] do |srv|
      srv.vm.box = "ubuntu/trusty64"
      srv.vm.synced_folder ".", "/vagrant", disabled: true
      srv.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
      #srv.vm.network "private_network", ip: server["ip"]
      unless server["port_frwd_guest"].nil?
        srv.vm.network "forwarded_port", guest: server["port_frwd_guest"], host: server["port_frwd_host"]
      end
      srv.vm.provider :virtualbox do |vb|
        vb.name = server["name"]
        vb.memory = 1024
        vb.cpus = 1
      end
      srv.vm.hostname = server["name"]
      unless server["alias"].nil?
        srv.hostsupdater.aliases = [ server["alias"] ]
      end
      srv.vm.provider :libvirt do |domain|
        if server["memory"].nil?
          domain.memory = 1024
        else
          domain.memory = server["memory"]
        end
        domain.cpus = 4
        domain.storage :file, :size => '200G'
      end
    end
    config.vm.provider "libvirt" do |v, override|
    #    override.vm.box = "elastic/ubuntu-14.04-x86_64"
        override.vm.box = "centos/7"
    #    override.vm.box_version = "1601.01"
    end
  end

  config.vm.provision :ansible do |ansible|
    ansible.host_key_checking = false
    ansible.playbook = "vagrant/provision.yml"
    ansible.sudo = true
    ansible.host_vars = {
    }
    ansible.groups = {
      'ucp-primary' => ['node01'],
      'ucp-replica' => ['node02', 'node03'],
      'ucp-worker'  => ['node04', 'node05', 'node06'],
    }
  end
end
