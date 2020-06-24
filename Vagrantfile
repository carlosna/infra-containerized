# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
current_dir = File.dirname(File.expand_path(__FILE__))
config_file = ENV['VAGRANT_CONFIG_FILE'] || "vagrant-config.yml"
cfg = YAML.load_file("#{current_dir}/" + config_file)

ansible_config_file = ENV['ANSIBLE_CONFIG_FILE'] || "vars.yml"
puts "Config files\n\tVagrant: " + config_file + "\n\tAnsible: " + ansible_config_file

resources_config = cfg['resources']
ansible_config = cfg['ansible']

Vagrant.require_version ">= 1.7.0"

Vagrant.configure(2) do |config|
  # With bionic we sometimes got a timeout when waiting for dns pod
  config.vm.box = "hashicorp/bionic64"

  config.vm.network "forwarded_port", guest: 80, host: 80, host_ip: "0.0.0.0"
  config.vm.network "forwarded_port", guest: 443, host: 443, host_ip: "0.0.0.0"
  # Disable the new default behavior introduced in Vagrant 1.7, to ensure that all Vagrant machines will use the same SSH key pair.
  # See https://github.com/hashicorp/vagrant/issues/5005
  config.ssh.insert_key = cfg['insert_ssh_key']

  # Requirements for ansible
  config.vm.provision "shell", inline: <<-SHELL
    test -e /usr/bin/python3 || (apt-get update && apt-get install -y python3-minimal build-essential python3-pip python3-apt)
  SHELL

  config.vm.synced_folder ".", "/vagrant", :mount_options => ["dmode=777","fmode=777"]

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = ansible_config['verbose_args']
    ansible.playbook = "k3s.yml"
    ansible.extra_vars = "@" + ansible_config_file
  end

  config.vm.define 'master' do |server|
    server.vm.hostname = "master.localdomain"
    server.vm.network "private_network", ip: "10.0.3.21"
    config.vm.provider :virtualbox do |vb|
      vb.memory = resources_config['memory']
      vb.cpus = resources_config['cpus']
    end
  end

end
