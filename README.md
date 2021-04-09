# Creo Virtualis

This project Creo Virtualis offers way to enable Vagrant to use data from a configuration file to create Virtual Machines.  This way it may be more simple to get Vagrant to setup multiple VMs at the same time. 

Ansible is used to perform minimal provising to the VMs. The file would be typically located on Ansible Inventory directory.

Vargrant uses VirtualBox v6.1 to host the VMs.

## Environment

One needs to install following software in order to use Creo Virtualis:
* Vagrant (>= v2.2.14)
* VirtualBox (>= v6.1.x)
* Ansible (2.9.x/2.10.x)

## Usage

Following commands should normally be used.
* vagrant --hostfile=<file_path> up
* vagrant --hostfile=<file_path> provision
* vagrant --hostfile=<file_path> destroy
* vagrant --hostfile=<file_path> destroy -f
* vagrant --hostfile=<file_path> ssh <hostname>

Note: use relative path like '../somedir/vagrant_hosts.yml' 

When using following commands then default inventory/vagrant_hosts.yml is use. 
* vagrant up
* vagrant provision
* vagrant destroy
* vagrant destroy -f
* vagrant ssh <hostname>

So, all other normal Vagrant commands should also work. 

If --hostfile=.. is not given then 'inventory/vagrant_hosts.yml' path is assumed to be used.

If not configuration file given (--hostfile) or found (default), then hard coded default values are used.


## Inventory

When using this project please consider using external inventory with real variable values.

Inventory or some other place needs to contain vagrant_hosts.yml type of file with file the content structure in order to Vagrant to be able to create the VMS.

The default hostfile file path 'inventory/vagrant_hosts.yml' could then be changed in the Vagrant file to point to the correct inventory directly.

## Password

The password for Ansible Vault is 'changeme'.

When using this project please consider using external inventory with real variable values.

## The Config File - vagrant_hosts.yml

The default location for vagrant_hosts.yml file is inventory/vagrant_hosts.yml

This configuration file contains few variables to enable Vagrant to setup the VMs.

host specific settings are defined by 'nodes' variable.

The 'nodes' must contain at least 'hostname' parameter in order to function.

```
nodes:
  - hostname:

```

Following parameter are support for the 'nodes'
```
nodes:
  - hostname:
    ip:
    ip2:
    ip3:
    ip4:
    cpu:
    mem:
    box:
```

When the parameter is missing default value(s) are used. Following default values are used:
* If ip/ip2/ip3/ip4 is missing then that network is not configured
* box: 'debian/buster64'
* cpu: '1'
* mem: '512'

Following general setting are supported and must be defined

```
# Ansible user name to Vagrant to use
ansible_user:
ansible_user_password:
# Dir name under home dir to store ssh keys
ssh_key_dir:
# Actual private key file name to be created
ssh_key:
# Ansible intentory to use
ansible_inventory_path:
# Ansible vault password file
ansible_vault_password_file:
# Ansible cfg file path
ansible_config_file:
# Ansible play to play
ansible_playbook:
# Ansible limit to use
ansible_limit:
```

## Credit

I had looked into creation of VMs more dynamically for a while but https://ctrlnotes.com/vagrant-advanced-examples/ article gave me the idea to try to do it this way.

## ToDo

* Make script to us ansible-inventory command and generated the file automatically from Ansible hosts file based on some way to indicate resource amount per VM.
