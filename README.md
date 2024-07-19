https://medium.com/@mahinshanazeer/creating-vms-using-vagrant-for-testing-f9e2b05526d5


Vagrant is an open-source tool that helps developers create, configure, and manage virtual machine environments. It’s designed to be simple and fast and can help improve development setup time. Vagrant can also help with:

1. Creating consistent development environments
2. Working with complex software stacks
3. Managing multiple projects
4. Improving collaboration
5. Simulating complex network topologies
6. Sharing pre-configured environments

In this repository, we have two files; (I learned vagrant from https://techiescamp.com/, I have added the same example file)
1. Vagrantfile
2. settings.yml

The primary function of the Vagrantfile is to describe the type of machine required for a project, and how to configure and provision these machines. Vagrantfiles are called Vagrantfiles because the actual literal filename for the file is Vagrantfile (casing does not matter unless your file system is running in a strict case sensitive mode).

Vagrant is meant to run with one Vagrantfile per project, and the Vagrantfile is supposed to be committed to version control. This allows other developers involved in the project to check out the code, run vagrant up, and be on their way. Vagrantfiles are portable across every platform Vagrant supports.

The syntax of Vagrantfiles is Ruby, but knowledge of the Ruby programming language is not necessary to make modifications to the Vagrantfile, since it is mostly simple variable assignment. In fact, Ruby is not even the most popular community Vagrant is used within, which should help show you that despite not having Ruby knowledge, people are very successful with Vagrant.

The settings.yml file is typically a YAML configuration file used to define settings and parameters for various applications and development environments. It is often used in conjunction with tools like Vagrant, Ansible, and others to provide an easy and human-readable way to specify configuration options. The exact content and structure of the settings.yml file depend on the specific application or tool it is used with.

How They Work Together?
~~~
When using Vagrant and tools like Ansible or Puppet, the settings.yml file often works in conjunction with the Vagrantfile to manage configuration and provisioning. The Vagrantfile sets up the virtual environment, while the settings.yml provides detailed configuration parameters that are used by provisioners to configure the VM's software and services.

Summary
~~~
settings.yml: A YAML file for configuring application-specific settings.
Vagrantfile: A Ruby-based configuration file for defining and managing virtual machine environments with Vagrant.
Integration: settings.yml can be used to provide detailed configuration options that are referenced in the Vagrantfile for a more dynamic and flexible VM setup.


Lab:
~~~
Vagrant is used to create VMs for test purposes. Make sure Virtualbox is installed and active on your machine. Clone the repository > Use the command ‘vagrant up’ inside the repository. I was getting the following error when I tried to run this command:
~~~
root@ms:/home/master/projects/vagrant_data# sudo systemctl status libvirtd
Unit libvirtd.service could not be found.
root@ms:/home/master/projects/vagrant_data# sudo systemctl start libvirtd
Failed to start libvirtd.service: Unit libvirtd.service not found.
root@ms:/home/master/projects/vagrant_data#
~~~

The error indicates that the Libvirt daemon (libvirtd.service) is not installed on your system. To fix this, you’ll need to install Libvirt. Here’s how to do it:

For Debian/Ubuntu-based Systems
~~~
sudo apt-get update
sudo apt-get install libvirt-daemon libvirt-daemon-system libvirt-clients
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
sudo systemctl status libvirtd
~~~~

Prerequisites:
*********************
Make sure all these services are installed and configured:
~~~
sudo apt-get update
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
~~~

Verify your machine support Virtualization:
~~~
lscpu
ls /dev/kvm
sudo modprobe kvm
sudo modprobe kvm_intel # For Intel-based systems
sudo modprobe kvm_amd # For AMD-based systems
sudo modprobe kvm_arm # For ARM-based systems like Raspberry Pi
lsmod | grep kvm
sudo apt-get install libvirt-clients
sudo virt-host-validate
~~~

This will give a detailed output. I got the following error:
~~~
QEMU: Checking if device /dev/kvm exists : PASS
QEMU: Checking if device /dev/kvm is accessible : PASS
QEMU: Checking if device /dev/vhost-net exists : PASS
QEMU: Checking if device /dev/net/tun exists : PASS
QEMU: Checking for cgroup ‘cpu’ controller support : PASS
QEMU: Checking for cgroup ‘cpuacct’ controller support : PASS
QEMU: Checking for cgroup ‘cpuset’ controller support : PASS
QEMU: Checking for cgroup ‘memory’ controller support : WARN (Enable ‘memory’ in kernel Kconfig file or mount/enable cgroup controller in your system)
QEMU: Checking for cgroup ‘devices’ controller support : PASS
QEMU: Checking for cgroup ‘blkio’ controller support : PASS
QEMU: Checking for device assignment IOMMU support : WARN (Unknown if this platform has IOMMU support)
QEMU: Checking for secure guest support : WARN (Unknown if this platform has Secure Guest support)
LXC: Checking for Linux >= 2.6.26 : PASS
LXC: Checking for namespace ipc : PASS
LXC: Checking for namespace mnt : PASS
LXC: Checking for namespace pid : PASS
LXC: Checking for namespace uts : PASS
LXC: Checking for namespace net : PASS
LXC: Checking for namespace user : PASS
LXC: Checking for cgroup ‘cpu’ controller support : PASS
LXC: Checking for cgroup ‘cpuacct’ controller support : PASS
LXC: Checking for cgroup ‘cpuset’ controller support : PASS
LXC: Checking for cgroup ‘memory’ controller support : FAIL (Enable ‘memory’ in kernel Kconfig file or mount/enable cgroup controller in your system)
LXC: Checking for cgroup ‘devices’ controller support : PASS
LXC: Checking for cgroup ‘freezer’ controller support : FAIL (Enable ‘freezer’ in kernel Kconfig file or mount/enable cgroup controller in your system)
LXC: Checking for cgroup ‘blkio’ controller support : PASS
LXC: Checking if device /sys/fs/fuse/connections exists : PASS
~~

Solution:
~~~
Basically virtualization requires few things;

cgroups
namespaces.
Here, we have to resolve each ‘FAIL’ cases.

LXC: Checking for cgroup ‘memory’ controller support : FAIL (Enable ‘memory’ in kernel Kconfig file or mount/enable cgroup controller in your system)
LXC: Checking for cgroup ‘freezer’ controller support : FAIL (Enable ‘freezer’ in kernel Kconfig file or mount/enable cgroup controller in your system)
Verify group is enabled for memory: cat /proc/cgroups

Here the issue is caused due to not enabling cgroup controller for memory. To do that, we have to edit bootline:
~~~
vi /boot/firmware/cmdline.txt or vi /boot/cmdline.txt

#add the following lines

cgroup_enable=memiory swapaccount=1
~~~

Once it is done, reboot the machine and make sure everything is working fine ‘cat /proc/cgroups’

Vagrant tweaks:
***************
vagrant status - To see the status of vagrant enviroments
vagrant ssh master-node
vagrant destroy -f
vagrant global-status
vagrant global-status - prune
vagrant box remove box/name
vagrant box list
vagrant destroy vm-name
vagrant global-status - prune

Initialize a Vagrant environment in the current directory : vagrant init
Start and provision the Vagrant environment : vagrant up
SSH into the running Vagrant machine : vagrant ssh
Stop the running Vagrant machine : vagrant halt
Suspend the Vagrant machine (save its current state) : vagrant suspend
Resume a suspended Vagrant machine : vagrant resume
Destroy the Vagrant environment and its resources : vagrant destroy
Reload the Vagrant environment, applying any new Vagrantfile changes : vagrant reload
Display the status of the Vagrant machine : vagrant status
SSH into a specific machine if multiple 
machines are defined in the Vagrantfile : vagrant ssh [machine_name]
List all Vagrant environments (boxes) 
currently installed on your system : vagrant box list
Add a box to be used in the Vagrant environment : vagrant box add [box_name]

vagrant destroy -f : Destroy all nodes
vagrant init [box]: Initializes a new Vagrant environment with the specified box.
vagrant up: Creates and provisions the virtual machine according to the Vagrantfile.
vagrant ssh: Connects to the virtual machine via SSH.
vagrant halt: Gracefully shuts down the virtual machine.
vagrant destroy: Stops and deletes all traces of the virtual machine.
vagrant status: Shows the current status of the virtual machine.
vagrant suspend: Suspends the virtual machine, saving its current state.
vagrant resume: Resumes a suspended virtual machine.
