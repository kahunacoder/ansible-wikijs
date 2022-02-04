# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'
sconfig = YAML::load_file(File.join(File.dirname(File.expand_path(__FILE__)), 'group_vars/all/server.yml'))

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |v|
    v.name = sconfig["hostname"]
    v.customize ["modifyvm", :id, "--memory", "4096"]
    v.customize ["modifyvm", :id, "--cpus",  "4"]
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "90"]
  end
	config.vm.define sconfig["hostname"]
  config.vm.hostname = sconfig["hostname"]
  config.vm.box = "ubuntu/focal64" #Ubuntu 20.04 LTS 64bit
  config.vm.network  :private_network, :ip =>'0.0.0.0', :auto_network => true
  config.ssh.forward_agent = true
  config.vm.provision :ansible do |ansible|
    # ansible.verbose = "vvv"
    ansible.config_file = "ansible.cfg"
    ansible.playbook = "playbook.yml"
  end
end
