## NAT-PAT Lab 2018

This lab has been adapted from [Cumulus Linux User Guide](https://docs.cumulusnetworks.com/display/DOCS/Open+Shortest+Path+First+-+OSPF+-+Protocol) 

### Getting Started
  Before starting this exercise, install VirtualBox, Vagrant and Cumulus VX refer the documentation [here]().

### Setting up the Topology
    git clone https://github.com/shgandhi/networkinglab470.git
    cd networkinglab470/NAT-PAT
    vagrant up

### Enable Routing
  Cumulus uses Free Range Routing or FRRouting, the routing software package that provides a suite of routing protocols so we can configure routing on our switches. FRRouting does not start by default in Cumulus Linux. Before we run FRRouting, we must enable all the relevant daemons, i.e. for this exercise ```zebra and ospfd```. ```Zebra``` is the core  daemon which acts as an abstraction layer to underlying the Unix kernel. 
  
      1. vagrant ssh leaf01
      2. sudo nano /etc/frr/daemons

Set to ```yes``` the daemons zebra and ospf as below.

     zebra=yes (* this one is mandatory to bring the others up)
     bgpd=no
     ospfd=yes
     ospf6d=no
     ripd=no
     ripngd=no
     isisd=no
     babeld=no
     pimd=no
    
Enter ```Ctrl+X``` to quit and enter ```Y``` to save the file. Enter the following commands the restart the the FRRouting service.
   
    3. sudo systemctl enable frr.service
    4. sudo systemctl start frr.service
   
Execute steps 1. - 4. for leaf02, leaf03, leaf04, spine01 and spine02.
     
### Configure Network Interfaces

  Refer IP Address Management Schema [here](https://github.com/shgandhi/networkinglab470/blob/master/NAT-PAT/IPAM.md) to identify ip addresses allocated to each interface in the topology below.
  
  ![OSPF Topology](https://github.com/shgandhi/networkinglab470/blob/master/NAT-PAT/nat_pat_topology.jpg)
  
  1. On leaf01
     
    vagrant ssh leaf01
    net add interface swp1 ip address 10.10.1.2/24
    net add interface swp2 ip address 10.10.2.2/24
    net add interface swp24 ip address 10.10.9.2/24
    net pending
    net commit
    net show interface swp1
    net show interface swp2
    net show interface swp24
    exit
   
  2. On leaf02
    
    vagrant ssh leaf02
    net add interface swp1 ip address 10.10.3.2/24
    net add interface swp2 ip address 10.10.4.2/24
    net add interface swp24 ip address 10.10.10.2/24
    net pending
    net commit
    net show interace swp1
    net show interface swp2
    net show interface swp24
    exit
    
  3. On leaf03
    
    vagrant ssh leaf03
    net add interface swp1 ip address 10.10.5.2/24
    net add interface swp2 ip address 10.10.6.2/24
    net add interface swp24 ip address 10.10.11.2/24
    net pending
    net commit
    net show interace swp1
    net show interface swp2
    net show interface swp24
    exit
    
  4. On leaf04
    
    vagrant ssh leaf04
    net add interface swp1 ip address 10.10.7.2/24
    net add interface swp2 ip address 10.10.8.2/24
    net add interface swp24 ip address 10.10.12.2/24
    net pending
    net commit
    net show interace swp1
    net show interface swp2
    net show interface swp24
    exit
    
  5. On spine01
    
    vagrant ssh spine01
    net add interface swp1 ip address 10.10.1.1/24
    net add interface swp2 ip address 10.10.3.1/24
    net add interface swp3 ip address 10.10.5.1/24
    net add interface swp4 ip address 10.10.7.1/24
    net pending
    net commit
    net show interace swp1
    net show interface swp2
    net show interface swp3
    net show interface swp4
    exit
    
  6. On spine02
    
    vagrant ssh spine02
    net add interface swp1 ip address 10.10.2.1/24
    net add interface swp2 ip address 10.10.4.1/24
    net add interface swp3 ip address 10.10.6.1/24
    net add interface swp4 ip address 10.10.8.1/24
    net pending
    net commit
    net show interace swp1
    net show interface swp2
    net show interface swp3
    net show interface swp4
    exit
    
    
  7. On Server01
     
    vagrant ssh server01
    sudo ifconfig eth1 10.10.9.1/24 up
    sudo route add default gw 10.10.9.2
    ifconfig eth1
    ping 10.10.9.2
    exit 
    
  8. On Server02
     
    vagrant ssh server03
    sudo ifconfig eth1 10.10.10.1/24 up
    sudo route add default gw 10.10.10.2
    ifconfig eth1
    ping 10.10.10.2
    exit 
    
  9. On Server03
     
    vagrant ssh server03
    sudo ifconfig eth1 10.10.11.1/24 up
    sudo route add default gw 10.10.11.2
    ifconfig eth1
    ping 10.10.11.2
    exit 
    
  10. On Server04
     
    vagrant ssh server04
    sudo ifconfig eth1 10.10.12.1/24 up
    sudo route add default gw 10.10.12.2
    ifconfig eth1
    ping 10.10.12.2
    exit 
     
### Verify network connectivity
   1. From leaf01 ping directly connected interfaces of spine01 and spine02. The ping should work. 
   
     ping 10.10.1.1
     ping 10.10.2.1
     
   2. View the routing table of leaf01. Ignore entries for the management network 10.0.2.0/24 and ipv6 routes for now. After seeing the routing table, do you find any missing routes in the table?  
     
     net show route
     
   3. Repeat steps 1. - 2. for router02 and router03.
   4. Ping from server01 (10.10.9.1) to server04 (10.10.12.1). Does the ping work? Why not?
   

### Configure OSPF
We will put all interfaces on all the links between leaf and spine switches in same area i.e., area 0 (or area 0.0.0.0), and the links from the leaf switches to the servers in area 1 (0.0.0.1) and verify end to end connectivity. 

   1. On leaf01
    
    net add ospf router-id 1.1.1.1
    net add ospf network 1.1.1.1/32 area 0.0.0.0
    net add ospf network 10.10.1.0/24 area 0.0.0.0
    net add ospf network 10.10.2.0/24 area 0.0.0.0
    net add ospf network 10.10.9.0/24 area 0.0.0.1
    
   2. On leaf02
    
    net add ospf router-id 2.2.2.2
    net add ospf network 2.2.2.2/32 area 0.0.0.0
    net add ospf network 10.10.3.0/24 area 0.0.0.0
    net add ospf network 10.10.4.0/24 area 0.0.0.0
    net add ospf network 10.10.10.0/24 area 0.0.0.1
    
   3. On leaf03
    
    net add ospf router-id 3.3.3.3
    net add ospf network 3.3.3.3/32 area 0.0.0.0
    net add ospf network 10.10.5.0/24 area 0.0.0.0
    net add ospf network 10.10.6.0/24 area 0.0.0.0
    net add ospf network 10.10.11.0/24 area 0.0.0.1
    
   4. On leaf03
    
    net add ospf router-id 4.4.4.4
    net add ospf network 4.4.4.4/32 area 0.0.0.0
    net add ospf network 10.10.7.0/24 area 0.0.0.0
    net add ospf network 10.10.8.0/24 area 0.0.0.0
    net add ospf network 10.10.12.0/24 area 0.0.0.1
    
   5. On spine01
    
    net add ospf router-id 5.5.5.5
    net add ospf network 5.5.5.5/32 area 0.0.0.0
    net add ospf network 10.10.1.0/24 area 0.0.0.0
    net add ospf network 10.10.3.0/24 area 0.0.0.0
    net add ospf network 10.10.5.0/24 area 0.0.0.0
    net add ospf network 10.10.7.0/24 area 0.0.0.0
    
   6. On spine02
    
    net add ospf router-id 6.6.6.6
    net add ospf network 6.6.6.6/32 area 0.0.0.0
    net add ospf network 10.10.2.0/24 area 0.0.0.0
    net add ospf network 10.10.4.0/24 area 0.0.0.0
    net add ospf network 10.10.6.0/24 area 0.0.0.0
    net add ospf network 10.10.8.0/24 area 0.0.0.0
    
   7. Verification
     * Verify Connectivity
       Let's ping now again from server01 (10.10.9.1) to server04 (10.10.12.1). Does the ping work? Why?
