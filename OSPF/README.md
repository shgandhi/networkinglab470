OSPF Lab 2018

### OSPF Configuration
  1. On Router01
     * Configure interfaces
       * net add loopback ip address 10.2.1.1/32
       * net add interface swp1 ip address 192.168.1.1/24
       * net add interface swp2 ip address 192.168.3.1/24
     * Configure OSPF
       * net add ospf network 192.168.1.0/24 area 0.0.0.0
       * net add ospf network 192.168.3.0/24 area 0.0.0.0
     * Commit configuration
       * net pending
       * net commit

### How do RIB and FIB look with OSPF?
  1. OSPF RIB
     Run command _net show ospf database_

  2. OSPF FIB
     Run command _net show route ospf_

### OSPF: Metric Changes

### OSPF: Multiple Areas and LSA Types

### Debugging OSPF
