# creo_virtualis

To create Virtual Machines using Vagrant to Virtualbox using data from a file located on Ansible Inventory.

This way it may be more simple to get Vagrant to setup correct VMs. 

It may be possible to use ansible-inventory and some cli magic to generated file that Vagrant will use to dynamicaly generate correct VMS.

## Usage

vagrant --hostfile <file_path> up

All normal Vagrant commands should also work. If --hostfile is not given then 'inventory/vagrant_hosts.yml' path is assumed to be used.

## Inventory

When using this project please consider using external inventory with real variable values.

Inventory or some other place needs to contain vagrant_hosts.yml type of file with file the content structure in order to Vagrant to be able to create the VMS.

The default hostfile file path 'inventory/vagrant_hosts.yml' could then be changed in the Vagrant file to point to the correct inventory directly.

## Password

The password for Ansible Vault is 'changeme'.

When using this project please consider using external inventory with real variable values.

## Credit

I had looked into making creating VMs more dynamically for a while but https://ctrlnotes.com/vagrant-advanced-examples/ article gave me the idea to try to do it this way.

## Todo
Make script to us ansible-inventory command and generated the file automatically from Ansible hosts file based on some way to indicate resource amount per VM.