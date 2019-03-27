## ECE 470 LAB 
### Spring 2019

## Getting Started

### Download and Install the Hypervisor/Developer Environment
#### MAC OS SETUP
Refer to guide [here](https://github.com/CumulusNetworks/cldemo-vagrant/blob/master/documentation/macos/README.md). Follow only steps 1-5 i.e.
  - Installing XCode & XCode Tools
  - Installing Homebrew
  - Installing Virtualbox
  - Installing VirtualBox Extension Pack
  - Installing Vagrant
 
#### UBUNTU 16.04 SETUP
Refer to guide [here](https://github.com/CumulusNetworks/cldemo-vagrant/blob/master/documentation/linux/README.md). Follow only steps 1-3 i.e.
  - Installing Virtualbox
  - Installing Git
  - Installing Vagrant
  
#### WINDOWS SETUP
Refer to guide [here](https://github.com/CumulusNetworks/cldemo-vagrant/blob/master/documentation/windows/README.md). Follow only steps 1-3 i.e.
  - Installing Virtualbox
  - Installing Git
  - Installing Vagrant
  
### SET UP THE TOPOLOGY
Executing the last command should take ~15-20 minutes.

    git clone https://github.com/shgandhi/networkinglab470.git
    cd networkinglab470/OSPF
    vagrant status
    vagrant up
    
### WORKING WITH VAGRANT


These have been taken from [Vagrant Documentation](https://www.vagrantup.com/docs/cli/). 

| Command | Outcome |
|---|---|
| vagrant up | This command creates and configures guest machines according to your Vagrantfile. |
| vagrant destroy | This command stops the running machine Vagrant is managing and destroys all resources that were created during the machine creation process. |
| vagrant suspend | This resumes a Vagrant managed machine that was previously suspended |
| vagrant resume | This suspends the guest machine Vagrant is managing, rather than fully shutting it down or destroying it. |
| vagrant status | This will tell you the state of the machines Vagrant is managing. |
| vagrant ssh [name/id] | This will SSH into a running Vagrant machine and give you access to a shell. |
