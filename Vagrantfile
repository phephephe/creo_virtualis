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
nodes = []
ansible_user = ''
ansible_user_password = ''
ssh_key_dir = ''
ssh_key = ''
public_key_path = ''
ansible_inventory_path = ''
ansible_vault_password_file = ''
ansible_hostfile = ''
ansible_playbook = ''
ansible_limit = ''

opts = GetoptLong.new(
  [ '--hostfile', GetoptLong::REQUIRED_ARGUMENT ]
)
opts.ordering=(GetoptLong::REQUIRE_ORDER) 

opts.each do |opt, arg|
  case opt
    when '--hostfile'
      hostfile=arg
  end
end

# Read the conf file
if File.exists?(File.join(File.dirname(__FILE__), hostfile))
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
end
puts "*** Nodes found ***"
puts nodes
puts "******"
# All Vagrant configuration is done below.
vagrant_version = "2"

# Generate test ssh keys to be used by the Ansible user.
# Will not overwrite keys
# Only in case of vagrant up or varant provision commands
ARGV.each do|a|
  if a == 'provision' or a == 'up'
    if !nodes || nodes.empty?
      puts "No hosts defined in " + hostfile
      abort
    end
    system "echo 'Generating Test SSH Keys, will not overwrite keys'"
    system "mkdir ~/#{ssh_key_dir}/"
    system "chmod 0700 #{ssh_key_dir}"
    system "echo 'Do not select y or n, just wait'"
    system "ssh-keygen -t rsa -b 4096 -q -f ~/#{ssh_key_dir}/#{ssh_key} -P '' << N"
    system "echo "
  end # end if
end # end each loop

# May work with older then 2.2.14 but not tested
Vagrant.require_version ">= 2.2.14"

Vagrant.configure(vagrant_version) do |config|
  config.vm.box_check_update = false
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  config.vm.synced_folder ".", "/vagrant", disabled: true

  nodes.each_with_index do |nodes, index|
    config.vm.define nodes['hostname'] do |cfg|
      cfg.vm.provider :virtualbox do |vb, override|
        if nodes['box'].to_s.empty?
          nodes['box'] = 'debian/buster64'
        end
        if nodes['cpu'].to_s.empty?
          nodes['cpu'] = '1'
        end
        if nodes['mem'].to_s.empty?
          nodes['mem'] = '512'
        end
        config.vm.box = nodes['box']
        if !nodes['ip'].to_s.empty?
          override.vm.network :private_network, ip: nodes['ip']
        end
        if !nodes['ip2'].to_s.empty?
          override.vm.network :private_network, ip: nodes['ip2']
        end
        if !nodes['ip3'].to_s.empty?
          override.vm.network :private_network, ip: nodes['ip3']
        end
        if !nodes['ip4'].to_s.empty?
          override.vm.network :private_network, ip: nodes['ip4']
        end
        override.vm.hostname = nodes['hostname']
        vb.name = nodes['hostname']
        vb.customize ['modifyvm', :id, '--memory', nodes['mem'], '--cpus', nodes['cpu'], '--hwvirtex', 'on']
      end # end provider

    # Lets copy our generated ssh public key as authorized key to the Virtualbox VMs
    file_pub = File.open("#{ENV['HOME']}/#{ssh_key_dir}/#{ssh_key}.pub")
    public_key = file_pub.read
    config.vm.provision "shell", :inline =>"
        echo 'Copying the public SSH key to the VMs.'
        mkdir -p /home/#{ansible_user}/.ssh
        chmod 700 /home/#{ansible_user}/.ssh
        echo '#{public_key}' >> /home/#{ansible_user}/.ssh/authorized_keys
        chmod -R 600 /home/#{ansible_user}/.ssh/authorized_keys
        echo 'Host 10.0.*.*' >> /home/#{ansible_user}/.ssh/config
        echo 'StrictHostKeyChecking no' >> /home/#{ansible_user}/.ssh/config
        echo 'UserKnownHostsFile /dev/null' >> /home/#{ansible_user}/.ssh/config
        chmod -R 600 /home/#{ansible_user}/.ssh/config
        ", privileged: false

    # Ansible is run after last node is created to all (test hodst group) the nodes
    if index == nodes.size - 1
      # code for the last node member
      cfg.vm.provision :ansible do |ansible|
        # Disable default limit to connect to all the machines
        ansible.limit = "#{ansible_limit}"
        ansible.inventory_path = "#{ansible_inventory_path}"
        ansible.vault_password_file = "#{ansible_vault_password_file}"
        ansible.hostfile = "#{ansible_hostfile}"
        ansible.playbook = "#{ansible_playbook}"
        ansible.compatibility_mode = "2.0"
        ansible.verbose = true
        #ansible.verbose = "-vvvv"
        ansible.become = true
        ansible.extra_vars = {
          ansible_ssh_user: "#{ansible_user}",
          ansible_connection: "ssh"
        }
        end # end of ansible
      end # end last node member
    end # end config
  end # end nodes
end # end of end
