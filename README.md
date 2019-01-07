ansible-role-libvirtvms
=========

An Ansible role to create and remove Libvirt KVM based Virtual Machines, attach disks and manage static ip network addresses.  The intention is to work with the standard virt-builder / cloud images provided by distributions without requiring additional changes.  Currently tested with RHEL7, CentOS7 and Ubuntu 16.04.

The role is driven from a single list called machines.  Each item in the list is a dictionary which defines the virtual machine to manage and its attributes.  Attributes are sensibly defaulted in most cases and can be omitted.  Additional values can be added to the machines dictionary for use post VM provisioning.  A list of groups can be provided which successfully created VMs will be added too in order assist with post provisioning.  Two additional groups are created for running hosts (libvirt_running) and destroyed hosts (libvirt_destroyed).    


Requirements
------------

The role uses Libguestfs and virt-builder tools to create the disk images required on the hypervisor.  The RHEL / CentOS packages required are :  

- qemu-kvm
- libvirt
- libvirt-python
- libguestfs-tools
- virt-install
- libguestfs-xfs
    
  
Role Variables
--------------

A list called machines which contains dictionaries to define the machine :
```yaml
machines:
  - name: host1
    state: running
    memory: 2048
    cpu: 4
    distribution: "rhel7.6"
    os_type: "rhel7"
    nics:
      - bridge: "{{ libvirt_ops_bridge }}"
        ipaddress: 192.168.21.20
        network: 192.168.21.0
        netmask: 255.255.255.0
        broadcast: 192.168.21.255
        gateway: 192.168.21.1
        domain: lab.example.com
    boot_disk:
      name: host1.raw
      path: "{{ disk_location }}"
      device: vba
      size: 20G
    disks:
      - name: minio_vdb.qcow2
        device: vdb
        size: 80G
        path: "{{ disk_location }}"
    groups:
      - new_hosts
      - shakey_towers
      - libvirt_bootstrapped_false
    virt_builder_extra_options: "--uninstall cloud-init"
```      

## Basic Parameters

Item | Purpose
----   -------
ssh_key | The ssh key to inject for the root user. 
default_root_password | The password to set for the root user (use ansible-vault to secure).


## Machine list

Item | Purpose
----   -------
name | The name of the virtual host to manage.
state | Can be running, destroyed or shutdown.
memory | RAM in megabytes to assign to the VM.  
cpu | Virtual CPUs to assign to the VM.
distribution | Name of the virt-builder os-version
os_type | Family and major version of the guest OS, used to identify which static network config to apply, currently use 'rhel7' for /etc/sysconfig/network-scripts/ OSs or 'ubuntu16.04' /etc/network/interfaces OSs. 
nics | A list of network interfaces to create.  Note.  nics.0 is assumed to be the primary / management / ansible connection address when creating the ansible groups for successful hosts and as the primary domain name for the VM.     
nics.bridge | The libvirt network bridge to use. 
nics.ipaddress | IPV4 address
nics.network | IPV4 network
nics.netmask | IPV4 netmask
nics.boardcast | IPV4 broadcast
nics.gateway | IPV4 network gateway 
nics.domain | DNS domain name.
boot_disk | A dictionary defining the boot disk
boot_disk.name | The filename to use for the boot disk.  Note the Disk filetype (raw or qcow2) is inferred from the filename extention and must be provided.
boot_disk.path | The filesystem path in which to create the boot_disk image file.
boot_disk.device | The device name to present to the guest of the boot disk. 
boot_disk.size | The size of the boot disk to create (values are passed directly to virt-builder).
disks | A list of additional disks to create and attach to the guest, values are the same as for the boot disk.  
groups | A list of Ansible groups to add any newly created hosts to in order to aide post creation os bootstrapping of the virtual machine.
virt_builder_extra_options | String to pass to the virt-builder command line.  Eg : "--firstboot-command 'dpkg-reconfigure openssh-server'" or "--uninstall cloud-init" or "--edit '/etc/default/grub: s/^GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT=\"console=tty0 console=ttyS0,115200n8\"/' --run-command update-grub"

Dependencies
------------

N/A

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

TODO.

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
