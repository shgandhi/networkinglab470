## ECE 470 LAB 
### Spring 2018

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
