# The ideas from 
# https://gist.github.com/roblayton/c629683ca74658412487
# https://blog.scottlowe.org/2016/01/14/improved-way-yaml-vagrant/
# https://ctrlnotes.com/vagrant-advanced-examples/#-config-yml-based-file-shared-between-vagrantfile-and-ansible-playbook-holds-variables-used-by-both
# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'getoptlong'
require 'yaml'

# Variable, Default conf file
hostfile = "inventory/vagrant_hosts.yml"

opts = GetoptLong.new(
  [ '--hostfile', GetoptLong::OPTIONAL_ARGUMENT ]
)

opts.each do |opt, arg|
  case opt
    when '--hostfile'
      hostfile=arg
  end
end

# Check the file status
if !File.exists?(File.join(File.dirname(__FILE__), hostfile))
  puts "The vagrant host file config is missing"
  abort
end

# Read the conf file
conf_vars = YAML.load_file(File.join(File.dirname(__FILE__), hostfile))

# Variables from the config file
nodes = conf_vars['nodes']
ansible_user = conf_vars['ansible_user']
ansible_user_password = conf_vars['ansible_user_password']
ssh_key_dir = conf_vars['ssh_key_dir']
ssh_key = conf_vars['ssh_key']
public_key_path = conf_vars['public_key_path']
ansible_inventory_path = conf_vars['ansible_inventory_path']
ansible_vault_password_file = conf_vars['ansible_vault_password_file']
ansible_hostfile = conf_vars['ansible_hostfile']
ansible_playbook = conf_vars['ansible_playbook']
ansible_limit = conf_vars['ansible_limit']

# Error if nodes are missing
if !nodes || nodes.empty?
  puts "No hosts defined in " + hostfile
  abort
end

# Debug
puts nodes
puts ssh_key_dir
puts ssh_key

# All Vagrant configuration is done below.
vagrant_version = "2"

# Generate test ssh keys to be used by the Ansible user.
# Will not overwrite keys
# Only in case of vagrant up or varant provision commands
opts.each do|a|
 
  if a == 'up'
    #system "echo Local virtual box image will be used: #{localbox}"
    #system "echo nodes from file: #{node_file}"
  end
  if a == 'provision' or a == 'up'
    system "echo 'Generating Test SSH Keys, will not overwrite keys'"
    system "mkdir #{ssh_key_dir}"
    system "chmod 0700 #{ssh_key_dir}"
    system "echo 'Do not select y or n'"
    system "ssh-keygen -t rsa -b 4096 -q -f #{ssh_key} -P '' << N"
    system "echo "
  end # end if
end # end each loop

# VM nodes to create
#nodes = {
#  "tester1" => { :ip => "192.170.1.20", :ip2 => "192.171.1.20", :cpus => 1, :mem => 1024 },
#  "tester2" => { :ip => "192.170.1.21", :ip2 => "192.171.1.21", :cpus => 1, :mem => 1024 },
#  "ca" => { :ip => "192.170.1.10", :ip2 => "192.171.1.10", :cpus => 1, :mem => 512 }
#  # add, and more similar config lines here for additional VMs
#}

# May work with older then 2.2.7 but not tested
Vagrant.require_version ">= 2.2.7"

Vagrant.configure(vagrant_version) do |config|
  config.vm.box_check_update = false
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true
  puts "here debug"

  nodes.each_with_index do |nodes, index|
  #nodes.each_with_index do |(hostname, info), index|
  puts "#{nodes['hostname']}"
    config.vm.define "#{nodes['hostname']}" do |cfg|
      cfg.vm.provider :virtualbox do |vb, override|
        config.vm.box = "#{nodes['box']}"
        override.vm.network :private_network, ip: "#{nodes['ip']}"
        override.vm.network :private_network, ip: "#{nodes['ip2']}"
        override.vm.hostname = "#{nodes['hostname']}"
        vb.name = "#{nodes['hostname']}"
        vb.customize "['modifyvm', :id, '--memory', #{nodes['mem']}, '--cpus', #{nodes['cpu']}, '--hwvirtex', 'on']"
      end # end provider

    # Lets copy our generated ssh public key as authorized key to the Virtualbox VMs
    public_key = File.read("#{ENV['HOME']}/test_ssh_keys/test_id_rsa.pub")
    config.vm.provision "shell", :inline =>"
        echo 'Copying the public SSH key to the VMs.'
        mkdir -p /home/vagrant/.ssh
        chmod 700 /home/vagrant/.ssh
        echo '#{public_key}' >> /home/vagrant/.ssh/authorized_keys
        chmod -R 600 /home/vagrant/.ssh/authorized_keys
        echo 'Host 10.0.*.*' >> /home/vagrant/.ssh/config
        echo 'StrictHostKeyChecking no' >> /home/vagrant/.ssh/config
        echo 'UserKnownHostsFile /dev/null' >> /home/vagrant/.ssh/config
        chmod -R 600 /home/vagrant/.ssh/config
        ", privileged: false

    # Ansible is run after last node is created to all (test hodst group) the nodes
    if index == nodes.size - 1
      # code for the last node member
      cfg.vm.provision :ansible do |ansible|
        # Disable default limit to connect to all the machines
        ansible.limit = "all"
        ansible.inventory_path = "../MyFoE_inventory/development/hosts"
        ansible.vault_password_file = "~/.ansible-vault.sh"
        ansible.hostfile = "ansible.cfg"
        ansible.playbook = "vagarant_init_play.yml"
        ansible.compatibility_mode = "2.0"
        ansible.verbose = true
        #ansible.verbose = "-vvvv"
        ansible.become = true
        ansible.extra_vars = {
          ansible_ssh_user: "vagrant",
          ansible_connection: "ssh"
        }
        end # end of ansible
      end # end last node member
    end # end config
  end # end nodes
end # end of end
