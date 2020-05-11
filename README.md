[![Build Status](https://travis-ci.org/ksator/lab_management.svg?branch=master)](https://travis-ci.org/ksator/lab_management)

# Repository documentation structure

- [**What to find in this repository**](README.md#what-to-find-in-this-repository)
- [**Requirements to use this repository**](README.md#requirements-to-use-this-repository)
- [**Source of truth**](README.md#source-of-truth) 
- [**How to locate a mac address in the network using Python**](README.md#how-to-locate-a-mac-address-in-the-network-using-python)
- [**How to collect data from the network using Ansible**](README.md#how-to-collect-data-from-the-network-using-ansible) 
- [**How to configure the network using Ansible**](README.md#how-to-configure-the-network-using-ansible)
- [**How to audit the network using Ansible**](README.md#how-to-audit-the-network-using-ansible)
- [**How to audit the network using JSNAPy**](README.md#how-to-audit-the-network-using-jsnapy)
- [**Repository structure**](README.md#repository-structure)  
- [**Continuous integration with Travis CI**](README.md#continuous-integration-with-travis-ci)  
- [**Looking for help**](README.md#looking-for-help)

# What to find in this repository

Junos automation content for data center network fabric.  
It is used to manage a lab.  
It is based on:    
- Junos devices
- Ansible, PyEZ, JSNAPy

# Requirements to use this repository

### Get the content of the remote repository locally

```
sudo -s
git clone https://github.com/ksator/lab_management
ls lab_management
```

### Move to the local copy of the remote repo

```
cd lab_management
sudo -s
```

### Install PyEZ, Ansible, JSNAPy

This repository has been tested using Ansible 2.4.2.0  

Run these commands on Ubuntu 16.04 to install these tools:
```
sudo -s
apt-get update
apt-get install -y python-dev libxml2-dev python-pip libxslt1-dev build-essential libssl-dev libffi-dev git
pip install junos-eznc jxmlease wget jsnapy ansible==2.4.2.0 requests ipaddress cryptography 
ansible-galaxy install Juniper.junos,1.4.3
```
Check the Ansible version:
```
ansible --version
```
Verify you have the Juniper.junos role: 
```
ls /etc/ansible/roles/
```
This repository has been tested using the version 1.4.3 of the Juniper.junos role available on Galaxy.  
Use this command to see the name and version of each role installed:
```
ansible-galaxy list
```

You can now use the local copy of this remote repository.  
You need to run the below commands within the root of the project tree.

# Source of truth

This repository uses Ansible, PyEZ and JSNAPy.  
Ansible is the source of truth for the inventory and the credentials, so: 
- [**JSNAPy inventory file**](jsnapy/testfiles/devices.yml) is created automatically (using this [**playbook**](pb.generate.jsnapy.inventory.yml) and [**template**](/templates/jsnapy_inventory.j2)), based on the [**Ansible inventory file**](hosts) and based on the [**Ansible variables for devices credentials**](/group_vars/JUNOS/credentials.yml)
- Devices list for PyEZ is created automatically based on the [**Ansible inventory file**](hosts) (using this [**python script**](/python/inventory.py)). In addition, using this [**python script**](/python/credentials.py), PyEZ is able to reuse the [**Ansible variables for devices credentials**](/group_vars/JUNOS/credentials.yml).


# How to locate a mac address in the network using Python
Execute this [**python script**](python/locate.mac.address.py) to locate a mac address in the network
```
python ./python/locate.mac.address.py 38:4f:49:f2:5f:fc
```

# How to collect data from the network using Ansible

### How to pass show commands on junos devices and collect the commands output
Edit the [**cli.yml**](cli.yml) file to indicate the list of junos show commands you want to use
```
vi cli.yml
```

Run this command to execute the [**pb.collect.cli.output.yml**](pb.collect.cli.output.yml) playbook.  
It runs the junos show commands from the [**cli.yml**](cli.yml) file and saves the output on the Ansible server in the [**cli**](cli) directory. 
```
ansible-playbook pb.collect.cli.output.yml
```

The junos show commands output is available in the [**cli**](cli) directory 
```
ls cli
```

### How to pass show commands on junos devices and collect the output (alternative automation content)
Edit the [**pb.collect.commands.output.yml**](pb.collect.commands.output.yml) file to indicate the list of junos show commands you want to use
```
vi pb.collect.commands.output.yml
```

Run this command to execute the [**pb.collect.commands.output.yml**](pb.collect.commands.output.yml) playbook.  
It runs the junos show commands and saves the output on the Ansible server in the [**command**](command) directory. 
```
ansible-playbook pb.collect.commands.output.yml
```

The commands output is available in the [**command**](command) directory. 
```
ls command
```

### How to collect the facts from junos devices 

The playbook [**pb.collect.facts.yml**](pb.collect.facts.yml) collects the facts on junos devices and saves them on Ansible in the directory [**facts**](facts)

Run this command to collect the facts from the junos devices
```
ansible-playbook pb.collect.facts.yml
```

The facts are available in the directory [**facts**](facts)
```
ls facts/
```


### How to collect the junos configuration file from junos devices 

The playbook [**pb.collect.configuration.yml**](pb.collect.configuration.yml) collects the Junos configuration in set, xml, json and text formats, and saves the configuration files in the [**directory configuration**](configuration)  

Run this command to collect the junos configuration files for a device/group.  
```
ansible-playbook pb.collect.configuration.yml --limit DC1
```

Run this command to collect the junos configuration files for the whole network
```
ansible-playbook pb.collect.configuration.yml
```

The configuration files are available in the directory [**configuration**](configuration)
```
ls configuration/
```

### How to collect the running configuration on the junos devices and update the golden configuration files

The golden configuration files are the configuration files that will be loaded at the beginning of each demo. 

The playbook [**pb.collect.golden.yml**](pb.collect.golden.yml) collects the running configuration on the junos devices and updates the directory [**golden_configuration**](golden_configuration) with these files.

Run this command to do it for a device/group
```
ansible-playbook pb.collect.golden.yml --limit QFX5110
```

Run this command to do it for the whole network
```
ansible-playbook pb.collect.golden.yml 
```

The golden configuration files are available in the directory [**golden_configuration**](golden_configuration)
```
ls golden_configuration
```


# How to configure the network using Ansible

### How to overwrite the running configuration on junos devices with their golden configuration

The playbook [**pb.configure.golden.yml**](pb.configure.golden.yml) overwrites the running configuration on the junos devices with the files in the directory [**golden_configuration**](golden_configuration).  
You can use it at the beginning of each demo to restore the golden configuration files on the Junos devices.   
Run this command to do it for a device/group
```
ansible-playbook pb.configure.golden.yml --limit QFX10K2-176
```
Run this command to do it for the whole network
```
ansible-playbook pb.configure.golden.yml
```

The playbook [**pb.configure.golden.yml**](pb.configure.golden.yml) backs-up the current running configuration from the remote devices in the directory [**backup**](backup) before applying the golden configuration. 
```	
ls backup/
```

### How to configure junos devices with set/delete commands

The playbook [**pb.configure.lines.yml**](pb.configure.lines.yml) configures the junos devices with set/delete commands. 

Edit the playbook [**pb.configure.lines.yml**](pb.configure.lines.yml) to indicate the list of set and delete commands you want to use:
```
vi pb.configure.lines.yml
```

In order to know which junos devices would have a configuration change if you execute the playbook [**pb.configure.lines.yml**](pb.configure.lines.yml), execute it in dry run mode.  
This won’t change the junos configuration.  
```
ansible-playbook pb.configure.lines.yml --check
```

In order to know if a junos device will have a configuration change if you execute the playbook [**pb.configure.lines.yml**](pb.configure.lines.yml), and also to know the difference between the desired state described in the playbook [**pb.configure.lines.yml**](pb.configure.lines.yml) and the device actual state, run this command.  
This won’t change the junos configuration.  
```
ansible-playbook pb.configure.lines.yml --check --diff --limit QFX10K2-176
```

Run this command to execute the playbook [**pb.configure.lines.yml**](pb.configure.lines.yml) for one device/group.  
This will configure the device/group with the list of set/delete commands. 
```
ansible-playbook pb.configure.lines.yml --limit DC2
```

Run this command to execute the playbook [**pb.configure.lines.yml**](pb.configure.lines.yml).  
This will configure the whole network with the list of set/delete commands. 
```
ansible-playbook pb.configure.lines.yml
```

The playbook [**pb.configure.lines.yml**](pb.configure.lines.yml) backs-up the current running configuration from the remote devices in the directory [**backup**](backup) before applying the configuration change. 
```
ls backup/
```

### How to configure devices with telemetry

The directory [**templates**](templates) has the jinja templates.  

The template [**telemetry.j2**](/templates/telemetry.j2) is used by the playbook [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml) to generate the junos configuration for streaming telemetry.  
```
more templates/telemetry.j2
```

Run this command to render the telemetry template locally.  
This will generate the junos telemetry configuration files, without actually configuring the junos devices.  
The directory [**render**](render) has the files generated from the jinja templates and variables.  
```
ansible-playbook pb.configure.telemetry.yml --tag render
```
```
ls render/telemetry/
```

In order to know which junos devices will have a configuration change if you execute the playbook [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml), execute it in dry run mode.  
This won’t change the junos configuration.  
```
ansible-playbook pb.configure.telemetry.yml --check
```

In order to know if a junos device will have a configuration change if you execute the playbook [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml), and also to know the difference between the desired state and the device actual state, run this command.  
This won’t change the junos configuration.  
 ```
ansible-playbook pb.configure.telemetry.yml --check --diff --limit QFX10K2-176
```

Run this command to execute the playbook [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml) for one device/group.  
This will configure telemetry on the device/group
```
ansible-playbook pb.configure.telemetry.yml --limit QFX10K2-176
```

Run this command to execute the playbook [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml).  
This will configure telemetry on the whole network.
```
ansible-playbook pb.configure.telemetry.yml
```

The playbook [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml) backs-up the current running configuration from the remote devices in the directory [**backup**](backup) before applying the configuration change. 
```
ls backup/
```

### How to rollback the running configuration to a previous state

The playbook [**pb.rollback.yml**](pb.rollback.yml) playbook performs a configuration rollback on junos devices.

Run this command to rollback 1 the whole network 
```
ansible-playbook pb.rollback.yml --extra-vars rbid=1
```

Run this command to rollback 3 the group DC2 
```
ansible-playbook pb.rollback.yml --extra-vars rbid=3 --limit DC2
```

The directory [**rollback**](rollback) has the Junos configuration differences from rollbacks done with the ansible playbook [**pb.rollback.yml**](pb.rollback.yml) 
```
ls rollback/
```


# How to audit the network using Ansible

### How to validate if some services are reachable on Junos devices

The playbook [**pb.check.ports.availability.yml**](pb.check.ports.availability.yml) checks if Ansible can connect on some ports on Junos devices (ssh, telnet, ftp, netconf)  
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.check.ports.availability.yml
```

### How to validate the status of interfaces on Junos devices
The playbook [**pb.check.interfaces.yml**](pb.check.interfaces.yml) checks from device operational state if the status (admin status and op status) of the interfaces is up. It does it for the interfaces described in YAML in [**host_vars**](host_vars).  
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.check.interfaces.yml
```

### How to validate the physical topology 
The playbook [**pb.check.lldp.yml**](pb.check.lldp.yml) compares the desired LLDP neighbors (described in YAML in [**host_vars**](host_vars)) against the actual LLDP neighbors  
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.check.lldp.yml
```
The playbook [**pb.check.lldp.json.yml**](pb.check.lldp.json.yml) does the same thing but uses a json represention of the LLDP neighbors instead of xml. 
```
ansible-playbook pb.check.lldp.json.yml
```

### How to validate the BGP sessions state 
The playbook [**pb.check.bgp.yml**](pb.check.bgp.yml) checks from the devices operationnal state if the sessions state of the BGP neighbors described in YAML in [**host_vars**](host_vars) is Established.  
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.check.bgp.yml
```

### How to validate if desired vlans are present
The playbook [**pb.check.vlans.yml**](pb.check.vlans.yml) checks from devices operational state if the desired vlans described in YAML in [**host_vars**](host_vars) are present  
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.check.vlans.yml
```

### How to collect the facts on junos devices and print them on Ansible
The playbook [**pb.print.facts.yml**](pb.print.facts.yml) collects the facts on junos devices and prints them on Ansible.  
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.print.facts.yml
```

### How to run all the above tests in one single command
The playbook [**pb.check.all.yml**](pb.check.all) includes these playbooks: 
 - [**pb.check.ports.availability.yml**](pb.check.ports.availability.yml)
 - [**pb.check.interfaces.yml**](pb.check.interfaces.yml)
 - [**pb.check.lldp.yml**](pb.check.lldp.yml)
 - [**pb.check.bgp.yml**](pb.check.bgp.yml)
 - [**pb.check.vlans.yml**](pb.check.vlans.yml)
 - [**pb.print.facts.yml**](pb.print.facts.yml)  
 
Run this command to execute this playbook for the whole network:
```
ansible-playbook pb.check.all.yml
```

### How to check which devices are not running their golden configuration

In order to know which junos devices will have a configuration change if you load the golden configuration files, execute the playbook [**pb.configure.golden.yml**](pb.configure.golden.yml) in dry run mode.  
This won’t load the golden configuration.  
```
ansible-playbook pb.configure.golden.yml --check
```

Run this command to do it for one device/group.  
This won’t load the golden configuration.  
```
ansible-playbook pb.configure.golden.yml --check --limit QFX10K2-176
```

### How to get the difference between the configuration running on devices and their golden configuration

In order to know if a junos device will have a configuration change if you load its golden configuration file, and also to know the difference between its running configuration and its golden configuration, run this command.  
This won’t change the junos configuration.  
```
ansible-playbook pb.configure.golden.yml --check --diff --limit QFX10K2-176
```

Run this command to do it for the whole network.  
This won’t load the golden configuration.  
```
ansible-playbook pb.configure.golden.yml --check --diff 
```

# How to audit the network using JSNAPy

JSNAPy is a tool to take snapshots, store snapshots, compare snapshots.  
There are 2 JSNAPy workflows:
- take snapshots and compare them against pre defined criteria
- take pre snapshots before any modification and then take post snapshots after modification and then compare them based on test cases

JSNAPy is supported in three modes
- a command line tool
- a Python module 
- An Ansible module hosted on the Ansible [**Galaxy**](https://galaxy.ansible.com/Juniper/junos/)

### How to validate there is no active alarm on the devices
The JSNAPy configuration file [**cfg_file_snapcheck_alarms.yml**](jsnapy/cfg_file_snapcheck_alarms.yml) is used to validate there is no active alarm on the devices.  
It uses the JSNAPy test file [**test_file_snapcheck_alarms.yml**](jsnapy/testfiles/test_file_snapcheck_alarms.yml)  
Run this command to validate there is no active alarm on the devices.  
JSNAPy will take snapshots and compare them against criteria described in the JSNAPy test file [**test_file_snapcheck_alarms.yml**](jsnapy/testfiles/test_file_snapcheck_alarms.yml). 
```
jsnapy --snapcheck -f jsnapy/cfg_file_snapcheck_alarms.yml --folder jsnapy
```

The directory [**snapshots**](jsnapy/snapshots) has the snapshots taken by jsnapy. If you want to read the snapshots, run this command:
```
ls jsnapy/snapshots
```
### How to validate there is no interfaces error
The JSNAPy configuration file [**cfg_file_snapcheck_interfaces.yml**](jsnapy/cfg_file_snapcheck_interfaces.yml) is used to validate there is no interfaces error.  
It uses the JSNAPy test file [**test_file_snapcheck_interfaces.yml**](jsnapy/testfiles/test_file_snapcheck_interfaces.yml)  
Run this command to validate there is no interfaces error on the devices.  
JSNAPy will take snapshots and compare them against criteria described in the JSNAPy test file [**test_file_snapcheck_interfaces.yml**](jsnapy/testfiles/test_file_snapcheck_interfaces.yml)
```
jsnapy --snapcheck -f jsnapy/cfg_file_snapcheck_interfaces.yml --folder jsnapy
```

The directory [**snapshots**](jsnapy/snapshots) has the snapshots taken by jsnapy. If you want to read the snapshots, run this command:
```
ls jsnapy/snapshots
```
### How to validate some BGP details
The JSNAPy configuration file [**cfg_file_snapcheck_bgp.yml**](jsnapy/cfg_file_snapcheck_bgp.yml) jsnapy file is used to validate some BGP details  
It uses the JSNAPy test file [**test_file_snapcheck_bgp.yml**](jsnapy/testfiles/test_file_snapcheck_bgp.yml)  
Run this command to validate some BGP details.  
JSNAPy will take snapshots and compare them against criteria described in the JSNAPy test file [**test_file_snapcheck_bgp.yml**](jsnapy/testfiles/test_file_snapcheck_bgp.yml) 

```
jsnapy --snapcheck -f jsnapy/cfg_file_snapcheck_bgp.yml --folder jsnapy
```
The directory [**snapshots**](jsnapy/snapshots) has the snapshots taken by jsnapy. If you want to read the snapshots, run this command:
```
ls jsnapy/snapshots
```

### How to check if the topology changed

Note: As xml output of "show lldp neighbors" is different on QFX and EX, it requires a different parsing. So we are using different JSNAPy files for EX and QFX.  

The JSNAPy configuration file [**cfg_file_check_topology_EX.yml**](jsnapy/cfg_file_check_topology_EX.yml) is used to check if the topology changed. It uses the JSNAPy test file [**test_file_check_topology_EX.yml**](jsnapy/testfiles/test_file_check_topology_EX.yml)  

The JSNAPy configuration file [**cfg_file_check_topology_QFX.yml**](jsnapy/cfg_file_check_topology_QFX.yml) is used to check if the topology changed. It uses the JSNAPy test file [**test_file_check_topology_QFX.yml**](jsnapy/testfiles/test_file_check_topology_QFX.yml)  
  
Take a first snapshot. It will be the source of Truth 
```
jsnapy --snap pre -f jsnapy/cfg_file_check_topology_QFX.yml --folder jsnapy
jsnapy --snap pre -f jsnapy/cfg_file_check_topology_EX.yml --folder jsnapy

```
The directory [**snapshots**](jsnapy/snapshots) has the snapshots taken by jsnapy. If you want to read the snapshots, run this command: 
```
ls jsnapy/snapshots/
```


Later on, if you want to check if the topology changed, take a new snapshot:
```
jsnapy --snap post -f jsnapy/cfg_file_check_topology_QFX.yml --folder jsnapy
jsnapy --snap post -f jsnapy/cfg_file_check_topology_EX.yml --folder jsnapy
```
The directory [**snapshots**](jsnapy/snapshots) has the snapshots taken by jsnapy. If you want to read the snapshots, run this command:   
```
ls jsnapy/snapshots/
```
Then, to actually check if the topology changed, JSNAPy will compare the pre and post snapshots based on test cases described in the JSNAPy test files [**test_file_check_topology_QFX.yml**](jsnapy/testfiles/test_file_check_topology_QFX.yml) and [**test_file_check_topology_EX.yml**](jsnapy/testfiles/test_file_check_topology_EX.yml).  
Run this command: 
```
jsnapy --check pre post -f jsnapy/cfg_file_check_topology_QFX.yml --folder jsnapy
jsnapy --check pre post -f jsnapy/cfg_file_check_topology_EX.yml --folder jsnapy

```
You can also limit this action to one device, and use the verbose mode:  
```
jsnapy --check pre post -f jsnapy/cfg_file_check_topology_QFX.yml --folder jsnapy -v -t  172.25.90.174
```

# Repository structure 

### Ansible inventory file
The ansible inventory file is [**hosts**](hosts) file at the root of the repository.    

### Ansible configuration file
The ansible configuration file is [**ansible.cfg**](ansible.cfg) at the root of the repository.   

### host_vars directory 
The variables are yml files under [**group_vars**](group_vars) and [**host_vars**](host_vars) directories.  
Host specific variables under the directory [**host_vars**](host_vars).   

### group_vars directory 
The variables are yml files under [**group_vars**](group_vars) and [**host_vars**](host_vars) directories.  
Group related variables are yml files under the directory [**group_vars**](group_vars)

### templates directory
The directory [**templates**](templates) has the jinja templates

### render directory
The directory [**render**](render) has the files generated from the jinja templates and variables

### Ansible Playbooks
The ansible playbooks are at the root of the repository.  
All playbooks are named **pb.*.yml**      

##### Ansible Playbooks to configure the network
- [**pb.configure.golden.yml**](pb.configure.golden.yml) playbook overwrites the running configuration on the junos devices with the files in the directory [**golden_configuration**](golden_configuration). 
- [**pb.configure.lines.yml**](pb.configure.lines.yml) playbook configures junos devices with set/delete commands
- [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml) playbook configures junos devices with telemetry
- [**pb.rollback.yml**](pb.rollback.yml) playbook performs a rollback on junos devices.

##### Ansible Playbooks to collect data from the network
- [**pb.collect.configuration.yml**](pb.collect.configuration.yml) playbook performs a configuration backup of the network and saves the configuration files in the directory [**configuration**](configuration)
- [**pb.collect.golden.yml**](pb.collect.golden.yml) playbook collects the running configuration on the junos devices and updates the directory [**golden_configuration**](golden_configuration) with these files.
- [**pb.collect.commands.output.yml**](pb.collect.commands.output.yml) playbook runs junos show commands and saves the output on Ansible 
- [**pb.collect.cli.output.yml**](pb.collect.cli.output.yml) playbook runs junos show commands and saves the output on Ansible. This playbook uses the show commands in the file [**cli.yml**](cli.yml)
- [**pb.collect.facts.yml**](pb.collect.facts.yml) playbook collects the facts on junos devices and saves them on Ansible in the directory [**facts**](facts)

##### Ansible Playbooks to audit the network

- [**pb.check.lldp.json.yml**](pb.check.lldp.json.yml) playbook checks the LLDP topology.  

- [**pb.check.all.yml**](pb.check.all) playbook includes these playbooks:  
  - [**pb.check.ports.availability.yml**](pb.check.ports.availability.yml) playbook checks if Ansible can connect on some ports on Junos devices (ssh, telnet, ftp, netconf)
  - [**pb.check.interfaces.yml**](pb.check.interfaces.yml) playbook checks the status of the interfaces on Junos devices
  - [**pb.check.lldp.yml**](pb.check.lldp.yml) playbook checks the LLDP topology 
  - [**pb.check.bgp.yml**](pb.check.bgp.yml) playbook checks the BGP states 
  - [**pb.check.vlans.yml**](pb.check.vlans.yml) playbook checks if desired vlans are present from devices operational state
  - [**pb.print.facts.yml**](pb.print.facts.yml) playbook collects the facts on junos devices and prints them on Ansible

##### Other Ansible Playbooks
- [**pb.generate.variables.structure.yml**](pb.generate.variables.structure.yml) playbook was used at the beginning of the project to create some of the directories and files used to define yaml variables. 
- [**pb.generate.jsnapy.inventory.yml**](pb.generate.jsnapy.inventory.yml) playbook creates the JSNAPy inventory file [**devices.yml**](devices.yml) based on the Ansible inventory file [**hosts**](hosts)


### cli directory 
The directory [**cli**](cli) has the output of the Junos show commands from the playbook [**pb.collect.cli.output.yml**](pb.collect.cli.output.yml)

### command directory 
The directory [**command**](command) has the output of the Junos show commands from the playbook [**pb.collect.commands.output.yml**](pb.collect.commands.output.yml) 

### facts directory
The directory [**facts**](facts) has the Junos facts collected by the playbook [**pb.collect.facts.yml**](pb.collect.facts.yml) 

### rollback directory
The directory [**rollback**](rollback) has the Junos configuration differences from rollbacks done with ansible playbook [**pb.rollback.yml**](pb.rollback.yml) 

### backup directory
The directory [**backup**](backup) has the junos configuration files automatically backed up by the playbooks: 
  - [**pb.configure.lines.yml**](pb.configure.lines.yml) 
  - [**pb.configure.golden.yml**](pb.configure.golden.yml)
  - [**pb.configure.telemetry.yml**](pb.configure.telemetry.yml)

### configuration directory
The directory [**configuration**](configuration) has the junos configuration files backed up when we ran the playbook [**pb.collect.configuration.yml**](pb.collect.configuration.yml) 

### golden directory
The directory [**golden_configuration**](golden_configuration) has the junos configuration files we need to push on the junos devices before starting the demo. 
- The playbook [**pb.collect.golden.configuration.yml**](pb.collect.golden.configuration.yml) collects the running configuration on the junos devices and updates the directory [**ebgp_underlay_evpn_vxlan_overlay**](/golden_configuration/ebgp_underlay_evpn_vxlan_overlay/) with these files.
- The playbook [**pb.configure.golden.yml**](pb.configure.golden.yml) overwrites the running configuration on the junos devices with the files in the directory [**ebgp_underlay_evpn_vxlan_overlay**](/golden_configuration/ebgp_underlay_evpn_vxlan_overlay/)

### fragments directory
The directory [**fragments**](fragments) is used by the playbook [**pb.generate.jsnapy.inventory.yml**](pb.generate.jsnapy.inventory.yml) to create the JSNAPy inventory file [**devices.yml**](devices.yml) based on the Ansible inventory file [**hosts**](hosts).  
The directory [**fragments**](fragments) doesn’t contain the JSNAPy inventory file [**devices.yml**](devices.yml) itself.

### python directory
The directory [**python**](python) has the python scripts
- The file [**inventory.py**](python/inventory.py) creates a python list of devices ip address based on the ansible inventory file [**hosts**](hosts)
- The file [**credentials.py**](python/credentials.py) gets the devices username and password from the ansible variables file  [**credentials.yml**](/group_vars/JUNOS/credentials.yml)
- The file [**locate.mac.address.py**](python/locate.mac.address.py) indicates where a given mac address in the network is located.

### jsnapy directory
The directory [**jsnapy**](jsnapy) has the jsnapy content:
- The directory [**jsnapy**](jsnapy) has the JSNAPy configuration files. They are named **cfg_file_*.yml**.
    - [**cfg_file_check_topology_EX.yml**](jsnapy/cfg_file_check_topology_EX.yml) jsnapy file checks if the topology changed between 2 snapshots
    - [**cfg_file_check_topology_QFX.yml**](jsnapy/cfg_file_check_topology_QFX.ym) jsnapy file checks if the topology changed between 2 snapshots 
    - [**cfg_file_snapcheck_alarms.yml**](jsnapy/cfg_file_snapcheck_alarms.yml) jsnapy file checks if they are active alarms
    - [**cfg_file_snapcheck_bgp.yml**](jsnapy/cfg_file_snapcheck_bgp.yml) jsnapy file checks some BGP details
    - [**cfg_file_snapcheck_interfaces.yml**](jsnapy/cfg_file_snapcheck_interfaces.yml) jsnapy file checks if there are interfaces errors  
- The directory [**snapshots**](jsnapy/snapshots) has the snapshots taken by jsnapy
- The directory [**testfiles**](jsnapy/testfiles) has the JSNAPy inventory file [**devices.yml**](jsnapy/testfiles/devices.yml). It is created with the playbook [**pb.generate.jsnapy.inventory.yml**](pb.generate.jsnapy.inventory.yml), based on the Ansible inventory file [**hosts**](hosts) and on Ansible variables file for devices credentials  [**credentials.yml**](/group_vars/JUNOS/credentials.yml)
- The directory [**testfiles**](jsnapy/testfiles) also has the test files used by jsnapy. They are named **test_file_*.yml**. 
    - [**test_file_check_topology_EX.yml**](jsnapy/testfiles/test_file_check_topology_EX.yml)
    - [**test_file_check_topology_QFX.yml**](jsnapy/testfiles/test_file_check_topology_QFX.yml)
    - [**test_file_snapcheck_alarms.yml**](jsnapy/testfiles/test_file_snapcheck_alarms.yml)
    - [**test_file_snapcheck_bgp.yml**](jsnapy/testfiles/test_file_snapcheck_bgp.yml)
    - [**test_file_snapcheck_interfaces.yml**](jsnapy/testfiles/test_file_snapcheck_interfaces.yml)
    
# Continuous integration with Travis CI

There is a github webhook with [**Travis CI**](https://travis-ci.org/ksator/lab_management)  
The playbooks in  this repository are tested automatically by Travis CI.  
The files [**.travis.yml**](.travis.yml) and [**requirements.txt**](requirements.txt) at the root of this repository are used for this.  

We are using two types of playbooks in this repository:  
- Some playbooks do not interact with Junos:   
   - Travis CI is executing them.  
- Some playbooks interact with Junos
  - ansible-playbook has a built-in option to check only the playbook's syntax (using the flag ```--syntax-check```). This is how Travis is testing them. If there is any syntax error, Travis will fail the build and output the errors in the log.  
    
Here's the last build status [![Build Status](https://travis-ci.org/ksator/lab_management.svg?branch=master)](https://travis-ci.org/ksator/lab_management)

# Looking for help 

### Looking for help with Junos automation:

https://github.com/ksator?tab=repositories  
https://gitlab.com/users/ksator/projects  
https://gist.github.com/ksator/  

### Looking for more EVPN-VXLAN automation examples

You can refer to these projects:  
https://github.com/JNPRAutomate/ansible-junos-evpn-vxlan  
https://github.com/ksator/EVPN_DCI_automation  
