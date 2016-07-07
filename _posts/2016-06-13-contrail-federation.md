---
layout: post
title: Multi-OpenStack networks with OpenContrail
published: false
---

<!--more-->

OpenContrail is a network solution for OpenStack. This post will show
how to configure L2 and L3 network spanning across multiple DevStack
environments using OpenContrail. This post will go through the
configuration of 2 DevStack deployments with OpenContrail.

# Installing OpenContrail and DevStack
OpenContrail currently supports only up to OpenStack Liberty on Ubuntu
14.04. Create two VMs running Ubuntu 14.04. Make sure the VMs are able
to communicate with each other via IP.

Clone the git repository for the `contrail-installer` and copy the
sample configuration file, then run the commands to build, install and
start OpenContrail:
```bash
git clone https://github.com/Juniper/contrail-installer.git
cd contrail-installer
cp samples/localrc-? localrc
./contrail.sh build
./contrail.sh install
./contrail.sh configure
./contrail.sh start
```

Install DevStack using the same configuration file and the
`stable/liberty` branch:
```bash
git clone https://git.openstack.org/openstack-dev/devstack.git
git checkout stable/liberty
cp ../contrail-installer/localrc .
./stack.sh
```

If everything went well, you should now have a working installation of
DevStack configured with Neutron and OpenContrail.

# Setting it up
To access the Dashboard for OpenContrail visit `http://<IP>:8080` and
use `user: admin`, `password: contrail123`, and `domain: default` to log
in.

We should give each of our DevStacks unique [ASN]
(https://www.apnic.net/get-ip/faqs/asn). To do that, go to
`Infrastructure -> Global Config -> BGP Options`.

Now go to `Infrastructure -> Networks -> BGP Peering` and register the
nodes as BGP peers of each other.

To be continued.
