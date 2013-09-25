DevStack is a set of scripts and utilities to quickly deploy an OpenStack cloud.

Intro
=====

This is a clone of devstack (https://github.com/openstack-dev/devstack) that allows users to quickly build evaluation and development OpenStack environments (Grizzly release) on Ubuntu or Fedora bases systems.  See the devstack github page or http://devstack.org/ for additional information on configuration and setup.

This repo allows for additional configuration setup of the Brocade VCS plugin for OpenStack networking (aka quan^H^H^Hneutron).  The following additional parameters can be added to the localrc to help configure the neutron plugin:

    * VCS_USERNAME: user login for the Brocade VCS fabric(defaults to admin)
    * VCS_PASSWORD: password for the Brocade VCS fabric (defaults to password)
    * VCS_IPADDR:   management IP address for the fabric (you must set this option)
    * BRCD_OSTYPE:  OS type (defaults to NOS)  
    * PHYSICAL_NETWORK:  name of the neutron physical network (defaults to physnet1)
    * TENANT_VLAN_RANGE: accpetable tenant VLAN range (default 2:999)
    * BRCD_PHYSICAL_INTERFACE: physical server NIC attached to fabric (defaults to eth2)

    
An example localrc might look like this:


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


Setup
=====

    * install ncclient 
    
    git clone https://code.grnet.gr/git/ncclient
    
    
    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # The primary network interface
    auto eth0
    iface eth0 inet dhcp
    
    # Tenant networks
    auto eth1
    iface eth1 inet manual
    up ifconfig eth1 up promisc
    
