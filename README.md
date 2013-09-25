DevStack is a set of scripts and utilities to quickly deploy an OpenStack cloud.

Intro
=====

This is a clone of devstack (https://github.com/openstack-dev/devstack) that allows users
to quickly build evaluation and development OpenStack environments (Grizzly release) on 
Ubuntu or Fedora bases systems.  See the devstack github page or http://devstack.org/ 
for additional information on configuration and setup.

This repo allows for additional configuration setup of the Brocade VCS plugin for 
OpenStack networking (aka qua^H^H^Hneutron).  The following additional parameters can 
be added to the localrc to help configure the neutron plugin:

    * VCS_USERNAME: user login for the Brocade VCS fabric(defaults to admin)
    * VCS_PASSWORD: password for the Brocade VCS fabric (defaults to password)
    * VCS_IPADDR:   management IP address for the fabric (you must set this option)
    * BRCD_OSTYPE:  OS type (defaults to NOS)  
    * PHYSICAL_NETWORK:  name of the neutron physical network (defaults to physnet1)
    * TENANT_VLAN_RANGE: accpetable tenant VLAN range (default 2:999)
    * BRCD_PHYSICAL_INTERFACE: physical server NIC attached to fabric (defaults to eth2_

Setup
=====
These instructions describe how to install and configure devstack on a single Ubuntu 12.04
(Precise) server with the option of adding one or more compute nodes to create a
multi-node cluster
    
VCS Fabric Configuration
------------------------
For a multi-node Brocade VCS fabric, VDX switches should be running NOS v4.0 or above with
distributed cluster mode enabled so that port-profiles can be synchronized across all
switches.  Server ports connected to Brocade switches should be configured as port-profile
ports.

    VDX1# show running-config interface te 3/0/1
    interface TenGigabitEthernet 3/0/1
     fabric isl enable
     fabric trunk enable
     port-profile-port
     no shutdown
    !

Server Configuration
--------------------
Network interfaces on each server node should be configured similar to below.  Promiscuous
mode should be enabled on interfaces connected to the VCS fabric.

    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface
    auto eth0
    iface eth0 inet static
    address 10.17.88.129
    netmask 255.255.240.0
    gateway 10.17.80.1
    dns-nameservers 10.17.80.21
    
    # private tenant networks (connected to VCS fabric)
    auto eth1
    iface eth1 inet manual
    up ifconfig eth1 up promisc

Download and install the netconf client

    # git clone https://code.grnet.gr/git/ncclien
    # cd ncclient && python ./setup.py install

OpenStack runs as a non-root user that has sudo privileges.  Use this user for further
configuration

    # adduser stack
    # echo "stack ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

As stack user, install this devstack repo in the home directory

    # git clone http://github.com/openstack-dev/devstack.git
    # cd devstack

Edit the localrc in the devstack directory.  An example might look like this:

    NOVA_BRANCH=stable/grizzly
    CINDER_BRANCH=stable/grizzly
    GLANCE_BRANCH=stable/grizzly
    HORIZON_BRANCH=stable/grizzly
    KEYSTONE_BRANCH=stable/grizzly
    QUANTUM_BRANCH=stable/grizzly

    disable_service n-net
    enable_service q-svc
    enable_service q-agt
    enable_service q-dhcp
    enable_service q-l3
    enable_service q-meta
    enable_service quantum


    DATABASE_PASSWORD=openstack
    ADMIN_PASSWORD=openstack
    RABBIT_PASSWORD=openstack
    SERVICE_PASSWORD=openstack
    SERVICE_TOKEN=openstack
    
    # enable brocade plugin
    Q_PLUGIN=brocade
    VCS_IPADDR=10.17.87.158
    BRCD_PHYSICAL_INTERFACE=eth1

    LOG=True
    LOGFILE=stack.sh.log
    LOGDAYS=1
    
    # if you don't like screen
    # DEST=/opt/stack
    # SCREEN_LOGDIR=$DEST/logs/screen
    
    HOST_IP=10.17.95.35

    SCREEN_HARDSTATUS="%{= Kw}%-w%{BW}%n %t%{-}%+w"

Run the stack.sh script.  It will take some as OpenStack software is downloaded,
installed, and services are enabled.

Testing things out
------------------
Source the openrc in the devstack directory to obtain credentials and use the CLI to have
a look around, create networks, and launch new virtual machine instances.  Alternatively,
login to the Horizon dashboard at http://controlerNodeIP and use the GUI (user: admin or
demo, password: openstack).

Within the VCS fabric, check that new port-profiles are created for every tenant network that is
created.  Two new port-profiles should exist after running stack.sh for the first time.

    VDX1# show port-profile
    port-profile default
    ppid 0
    vlan-profile
      switchport
      switchport mode trunk
      switchport trunk allowed vlan all
      switchport trunk native-vlan 1
    port-profile openstack-profile-2
    ppid 1
    vlan-profile
      switchport
      switchport mode trunk
      switchport trunk allowed vlan add 2
    port-profile openstack-profile-3
    ppid 2
    vlan-profile
      switchport
      switchport mode trunk
      switchport trunk allowed vlan add 3

As new instances are launched, they should be tied to the port-profile corresponding the
network they belong to.  Any instances on the same network should be able to communicate
with each other.

    VDX1# show port-profile status
    Port-Profile                        PPID        Activated        Associated MAC Interface
    openstack-profile-2                 1           Yes              fa16.3e1b.95d0 None
                                                                     fa16.3e64.fce8        Gi 2/0/28
                                                                     fa16.3e85.5b2f        Gi 2/0/28
                                                                     fa16.3ea6.3741        Gi 2/0/5
                                                                     fa16.3ecd.bfc1        Gi 2/0/5
                                                                     fa16.3eeb.87f7        Gi 2/0/28
    openstack-profile-3                 2           Yes              fa16.3e2c.0baf        None

References
----------
Brocade OpenStack Networking Plugin Wiki
https://wiki.openstack.org/wiki/Brocade-quantum-plugin

OpenStack Documentation
http://docs.openstack.org

DevStack
http://devstack.org

