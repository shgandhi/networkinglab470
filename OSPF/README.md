## OSPF Lab 2018

This lab has been adapted from [Cumulus Linux User Guide](https://docs.cumulusnetworks.com/display/DOCS/Open+Shortest+Path+First+-+OSPF+-+Protocol) 

### Getting Started
  Before starting this exercise, install VirtualBox, Vagrant and Cumulus VX refer the documentation [here]().

### Setting up the Topology
    git clone https://github.com/shgandhi/networkinglab470.git
    cd networkinglab470/OSPF
    vagrant up

### Enable Routing
  Cumulus uses Free Range Routing or FRRouting, the routing software package that provides a suite of routing protocols so we can configure routing on our switches. FRRouting does not start by default in Cumulus Linux. Before we run FRRouting, we must enable all the relevant daemons, i.e. for this exercise ```zebra and ospfd```. ```Zebra``` is the core  daemon which acts as an abstraction layer to underlying the Unix kernel. 
  
      1. vagrant ssh router01
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
   
Execute steps 1. - 4. for router02 and router03.
     
### Configure Network Interfaces

  ![OSPF Topology](https://github.com/shgandhi/networkinglab470/blob/master/OSPF/ospf_topology.jpg)
  
  1. On router01
     
    vagrant ssh router01
    net add interface swp1 ip address 10.12.5.1/24
    net add interface swp2 ip address 10.12.1.1/24
    net pending
    net commit
    net show interface swp1
    net show interface swp2
    exit
   
  2. On router02
    
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
    
  3. On router03
     
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
    
   4. On Server01
     
    vagrant ssh server01
    sudo ifconfig eth1 ip address 10.12.2.2/24 up
    sudo route add default gw 10.12.2.1
    ifconfig eth1
    ping 10.12.2.1
    exit 
     
   5. On Server02
     
    vagrant ssh server02
    sudo ifconfig eth1 ip address 10.12.4.2/24 up
    sudo route add default gw 10.12.4.1
    ifconfig eth1
    ping 10.12.4.1
    exit
   
### Verify network connectivity
   1. From router01 ping directly connected interfaces of router02 and router03. The ping should work. 
   
     ping 10.12.5.2
     ping 10.12.1.2
     
   2. View the routing table of router01. Ignore entries for the management network 10.0.2.0/24 and ipv6 routes for now. After seeing the routing table, do you find any missing routes in the table?  
     
     net show ip route
     
   3. Repeat steps 1. - 2. for router02 and router03.
   4. Ping from server01 (10.12.2.2) to server02 (10.12.4.2). Does the ping work? Why not?
   

### Configure OSPF
Initially we will put all interfaces on all the routers in same area i.e., area 0 (or area 0.0.0.0), and verify end to end connectivity. 

   1. On router01
    
    net add ospf router-id 101.1.1.1
    net add ospf network 101.1.1.1/32 area 0.0.0.0
    net add ospf network 10.12.5.0/24 area 0.0.0.0
    net add ospf network 10.12.1.0/24 area 0.0.0.0
    
   2. On router02
    
    net add ospf router-id 102.1.1.1
    net add ospf network 102.1.1.1/32 area 0.0.0.0
    net add ospf network 10.12.5.0/24 area 0.0.0.0
    net add ospf network 10.12.3.0/24 area 0.0.0.0
    
   3. On router03
   
    net add ospf router-id 103.1.1.1
    net add ospf network 103.1.1.1/32 area 0.0.0.0
    net add ospf network 10.12.1.0/24 area 0.0.0.0
    net add ospf network 10.12.3.0/24 area 0.0.0.0
    
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
      
     net show ospf interface swp1 | grep Cost
     net show ospf interface swp2 | grep Cost
      
   3. Change the cost of interface swp2 on router02:
      
     net add interface swp2 ospf cost 300
     net pending
     net commit
      
   4. Verify the updated FIB entry on router02:
      ```net show route ospf```
      Do you see the change in the next hop for the route to 10.12.4.0/24 network?

### OSPF: Multiple Areas and LSA Types

OSPF is a decentralized routing protocol. Each router in OSPF knows the entire topology (using the Reliable Flooding procedure mentioned in the class lectures). This topology information is stored in the LSDB. This makes LSDB very big. The overhead for both memory (to store such a large database) and processing (to process LSDB for shortest paths) increases significantly with a large topology. To reduce this oevrhead, OSPF has the concept of areas which limits the
LSDB size. In this section, we observe the impact of the area concept on OSPF size. We will use two areas: Area0 (backbone area) and Area1 (Non-backbone area). We can have multiple non-backbone areas in an OSPF process. Recall that the routers participating in multiple areas are termed as Area Border Routers (ABR).
We will configure router01 and router03 as ABR. The router interfaces will belong to the following areas.
router02 all interfaces in area 0.
router01 swp1, router-id in area 0 and swp2 in area 1.
router03 swp1 in area 0 and swp2,swp24 and router-id in area 1.

   1. On router01 change the network 10.12.1.0/24 from area 0 to area 1.
     
     net del ospf network 10.12.1.0/24 area 0.0.0.0
     net add ospf network 10.12.1.0/24 area 0.0.0.1
     net pending
     net commit
     

   2. On router03 change the network 10.12.1.0/24, 10.12.4.0/24 and 103.1.1.1/32 from area 0 to area 1
    
    net del ospf network 10.12.4.0/24 area 0.0.0.0
    net del ospf network 10.12.1.0/24 area 0.0.0.0
    net del ospf network 103.1.1.1/32 area 0.0.0.0
    net add ospf network 103.1.1.1/32 area 0.0.0.1
    net add ospf network 10.12.1.0/24 area 0.0.0.1
    net add ospf network 10.12.4.0/24 area 0.0.0.1
    net pending
    net commit
    
   3. Verification
     * Verify OSPF enable interface
       On each router, verify that all the interfaces are OSPF enabled. You can use the command:`net show ospf interface`
     * Verify OSPF neighbor
       On each router, verify its OSPF neighbor using the command:`net show ospf neighbor`
     * Verify LSDB
       On each router, verify Link State Database (LSDB) using the command:
       ```net show ospf database```
       Note the reduced size of LSDB as compared to the single area configuration. Can you figure out the topology seen by router01 and Router03? (HINT: a network in the other area appears directly connected to the ABR router.)
     * Verify ABR
       On each router, verify Area Border Routers (ABR) using the command:
       ```net show ospf border-router```

   4. OSPF LSA Types
       In OSPF, LSDB is formed using Link State Advertisement (LSA) update messages. The LSA from within the area or outside area need to be differentiated. For this OSPF uses different LSA types. For example LSA type 1 is for advertisement from within the same area. Similarly LSA type 3 is used for summary of router advertisement from one area into other. OSPF has many LSA types for different purposes. In this lab, we will observe the type 1 and type 3 LSA.

       * Observe the LSDB on router03, using the command
         ```net show ospf database```
       * How many net summary link states (give the different LINK ID) you see in the output? How many advertising routers (give ADV router detail) you observe?
Notice that the summary LSA (TYPE3) is from ABR. Router Link states are from the same area router. In this case, these routes are populated by LSA type1.
       * Repeat the same on router01 and router02.

### Debugging OSPF [taken from Cumulus Documentation](https://docs.cumulusnetworks.com/display/DOCS/Open+Shortest+Path+First+-+OSPF+-+Protocol)
   1. The three most important states while troubleshooting the protocol are:

       * Neighbors, with net show ospf neighbor. This is the starting point to debug neighbor states (also see tcpdump below).

       * Database, with net show ospf database. This is the starting point to verify that the LSDB is, in fact, synchronized across all routers in the network. For example, sweeping through the output of show ip ospf database router taken from all routers in an area will ensure if the topology graph building process is complete; that is, every node has seen all the other nodes in the area.

       * Routes, with net show route ospf. This is the outcome of SPF computation that gets downloaded to the forwarding table, and is the starting point to debug, for example, why an OSPF route is not being forwarded correctly.

   2. Use tcpdump to capture OSPF packets on router01 interface swp1:

       ```sudo tcpdump -v -i swp1 ip proto ospf``` Observe the OSPF update messages.
