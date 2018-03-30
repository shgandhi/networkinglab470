## OSPF Lab 2018

This lab has been adapted from [Cumulus Linux User Guide](https://docs.cumulusnetworks.com/display/DOCS/Open+Shortest+Path+First+-+OSPF+-+Protocol) 

### Getting Started
  Before starting this exercise, install VirtualBox, Vagrant and Cumulus VX refer the documentation [here]().

### Setting up the Topology
     ```
     git clone https://github.com/shgandhi/networkinglab470.git
     cd networkinglab470/OSPF
     vagrant up
     ```

### Enable Routing
     Cumulus uses Free Range Routing or FRRouting, the routing software package that provides a suite of routing protocols so we can configure routing on our switches. FRRouting does not start by default in Cumulus Linux. Before we run FRRouting, we must enable all the relevant daemons, i.e. for this exercise ```zebra and ```ospfd```. ```Zebra``` is the core  daemon which acts as an abstraction layer to underlying the Unix kernel. 
     ```
     1. vagrant ssh router01
     2. sudo nano /etc/frr/daemons
     ```
Set to ```yes``` the daemons zebra and ospf as below.
     ```
     zebra=yes (* this one is mandatory to bring the others up)
     bgpd=no
     ospfd=yes
     ospf6d=no
     ripd=no
     ripngd=no
     isisd=no
     babeld=no
     pimd=no
     ```
Enter ```Ctrl+X``` to quit and enter ```Y``` to save the file. Enter the following commands the restart the the FRRouting service.
    ```
    3. sudo systemctl enable frr.service
    4. sudo systemctl start frr.service
    ```
Execute steps 1. - 4. for router02 and router03.
     
### Configure Network Interfaces
  1. On Router01
     ```
     vagrant ssh router01
     net add interface swp1 ip address 10.12.5.1/24
     net add interface swp2 ip address 10.12.1.1/24
     net pending
     net commit
     net show interface swp1
     net show interface swp2
     exit
     ```
  2. On Router02
     ```
     vagrant ssh router02
     net add interface swp1 ip address 10.12.5.2/24
     net add interface swp2 ip address 10.12.3.1/24
     net add interface swp24 ip address 10.12.2.1/24
     net pending
     net commit
     net show interace swp1
     net show interface swp2
     net show interface swp24
     exit
     ```
  3. On Router03
     ```
     vagrant ssh router03
     net add interface swp1 ip address 10.12.3.2/24
     net add interface swp2 ip address 10.12.1.2/24
     net add interface swp24 ip address 10.12.4.1/24
     net pending
     net commit
     net show interace swp1
     net show interface swp2
     net show interface swp24
     exit
     ```
   4. On Server01
     ```
     vagrant ssh server01
     sudo ifconfig eth1 ip address 10.12.2.2/24 up
     sudo route add default gw 10.12.2.1
     ifconfig eth1
     ping 10.12.2.1
     exit 
     ```
   5. On Server02
     ```
     vagrant ssh server02
     sudo ifconfig eth1 ip address 10.12.4.2/24 up
     sudo route add default gw 10.12.4.1
     ifconfig eth1
     ping 10.12.4.1
     exit
     ```
### Verify network connectivity
   1. From router01 ping directly connected interfaces of router02 and router03. The ping should work. 
     ```
     ping 10.12.5.2
     ping 10.12.1.2
     ```
   2. View the routing table of router01. Ignore entries for the management network 10.0.2.0/24 and ipv6 routes for now. After seeing the routing table, do you find any missing routes in the table?  
     ```
     net show ip route
     ```
   3. Repeat steps 1. - 2. for router02 and router03.
   4. Ping from server01 (10.12.2.2) to server02 (10.12.4.2). Does the ping work? Why not?
   

### Configure OSPF
Initially we will put all interfaces on all the routers in same area i.e., area 0 (or area 0.0.0.0), and verify end to end connectivity.

   1. On router01
    ```
    net add ospf router-id 101.1.1.1
    net add ospf network 101.1.1.1/32 area 0.0.0.0
    net add ospf network 10.12.5.0/24 area 0.0.0.0
    net add ospf network 10.12.1.0/24 area 0.0.0.0
    ```
   2. On router02
    ```
    net add ospf router-id 102.1.1.1
    net add ospf network 102.1.1.1/32 area 0.0.0.0
    net add ospf network 10.12.5.0/24 area 0.0.0.0
    net add ospf network 10.12.3.0/24 area 0.0.0.0
    ```
   3. On router03
   ```
    net add ospf router-id 103.1.1.1
    net add ospf network 103.1.1.1/32 area 0.0.0.0
    net add ospf network 10.12.1.0/24 area 0.0.0.0
    net add ospf network 10.12.3.0/24 area 0.0.0.0
    ```
   4. Verification
     * Verify OSPF interfaces
       On each router, verify that all the interfaces are OSPF-enabled. You can use the command: ```net show ospf interface <interface-name>```
     * Verify OSPF neighbor
       On each router, verify its OSPF neighbors using the command: ```net show ospf neighbor```
     * Verify Connectivity
       Let's ping now again from server01 (10.12.2.2) to server02 (10.12.4.2). Does the ping work? Why?


### How do RIB and FIB look with OSPF?
  1. OSPF RIB or OSPF Topology Table
     In OSPF each router needs all topology information. This information gets stored in Link State Data Base(LSDB). On each router, verify Link State Database (LSDB) using the command:
    ```net show ospf database```
     How many entries do you see in the table? List 5 important pieces of information about a particular entry.

  2. OSPF FIB
     Let’s see now the router’s routing (FIB) table. You can use the command:
     ```net show route```
     How many entries do you see in the table? Are all of them learnt using OSPF protocol?
     List 5 important pieces of information about a particular entry learnt using the OSPF protocol.
     Subnets learnt via OSPF should have a prefix "O".
     If you want to display only the routing table entries learnt via OSPF, use the command:
     ```net show route ospf```

### OSPF: Metric Changes

We will now control entries in the OSPF FIB by manipulating cost for the OSPF interfaces. By default all OSPF enabled interfaces have a cost of 100. To reach server02 from router02 there are two paths, one through router01 (router02:swp1 -- router01 -- router03 -- server02) with a cost of 200, and one through a direct path (router02:swp2 -- router03 -- server02) with cost 100. We will increase the cost of router2 -- router03 link to manipulate the entry in the FIB.

   1. On router02, find out the next hop for server02's network i.e. 10.12.4.0/24 using the command: 
      ```net show route ospf```
   2. Observe the current cost of the interfaces:
      ```
      net show ospf interface swp1 | grep Cost
      net show ospf interface swp2 | grep Cost
      ```
   3. Change the cost of interface swp2 on router02:
      ```
      net add interface swp2 ospf cost 300
      net pending
      net commit
      ```
   4. Verify the updated FIB entry on router02:
      ```net show route ospf```
      Do you see the change in the next hop for the route to 10.12.4.0/24 network?

### OSPF: Multiple Areas and LSA Types

### Debugging OSPF
