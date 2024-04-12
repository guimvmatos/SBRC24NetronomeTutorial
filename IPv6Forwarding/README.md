# P4 Programming Tutorial on Netronome SmartNICs
This tutorial provides a basic introduction to P4 (Programming Protocol-independent Packet Processors) programming on Netronome SmartNICs. Here you will find all the steps for installing and setting up the development environment, as well as presenting, implementing, and running a simple P4 program on a Netronome SmartNIC.

The tutorial is designed in a simple and straightforward manner, aiming to abstract away the common complexities found in hardware programming. The goal is to provide a quick learning curve by covering only the essential topics up to the implementation of the first program. 

# Prerequisites
- Advanced knowledge of computer networks.
- Familiarity with Linux and command line.
- Basic understanding of the P4 language.
-Access to a Netronome SmartNIC (in this tutorial, we will use the Netronome Agilio CX 2x10Gb).

# Installation and Environment Setup
To install the necessary drivers and modules and configure the environment, follow the instructions below:

- Access the Agilio-P4-SmartNIC directory.
- Follow the presented tutorial to install the required drivers and modules.

By following these steps, you will be ready to start developing and running your P4 programs on Netronome SmartNIC.



## Details about Netronome architecture

The Netronome SmartNIC uses single-root input/output virtualization (SRIOV), which enables virtual functions (VFs) to be created from a physical function (PF). The VFs thus share the resources of a PF, while VFs remain isolated from each other. The isolated VFs are typically assigned to virtual machines (VMs) on the host. In this way, the VFs allow the VMs to directly access the PCI device, thereby bypassing the host kernel. In this tutorial, we have two physical (p0, p1) and four virtual interfaces (Vf0\_1 to Vf0\_5). We will work with a p4 ipv6 forwarding bla bal bal

### Link to a DEMO Presentation published on 2021 P4 Workshop
- https://www.youtube.com/watch?v=0BnOH88fgGU

This work is divided into three repositories:
- P4-INCA: The main contribution. Here you can find the INCA P4 code for Netronome Agilio SmartNIC and its instructions to deploy it and configure.
- [P4-BMv2-RAN-UPF](https://github.com/guimvmatos/P4-BMv2-RAN-UPF): Auxiliary P4 code to build a RAN and UPF on context of 5G simulations for INCA Project.
- [SFC-SRv6-topology](https://github.com/guimvmatos/SFC-SRv6-topology): Repository with instructions to complete the construction of the necessary topology for INCA tests and simulations.


# Deployment

To reproduce these repositories, you'll need the following requirements:

- Netronme Agilio SmartNIC
- Vagrant
- VirtualBox
- Python

Once you already have your netronome installed and configured on you machine, follow the steps:

## Steps

1. INCA
First, you must configure your card to have at least 7 logical interfaces (VFs). You can configure this setting by editing the following file: `/lib/systemd/system/nfp-sdk6-rte.service`. locate and change the following line.

`Environment=NUM_VFS=7`

Once this is done, proceed with the inca code P4. You have to clone this repository inside your source codes of netronome, generally its inside `path_for_agilio/src/p4-16`.

```
git clone git@github.com:guimvmatos/P4-INCA.git
cd P4-INCA
sudo /opt/netronome/p4/bin/./nfp4build --nfp4c_p4_version 16 --no-debug-info -p out -o firmware.nffw -l lithium -4 ipv6_forward.p4
sudo /opt/netronome/p4/bin/./rtecli design-load -f firmware.nffw -p out/pif_design.json
sudo /opt/netronome/p4/bin/rtecli config-reload -c user_config.json
```

If you want, you can check the configured rules.
```
sudo /opt/netronome/p4/bin/./rtecli tables -i 0 list-rules
```

Now your netronome is configured with INCA. Let's proceed with configurations of RAN, UPF and others VMs.

2. RAN / UPF

Follow the instructions of [P4-BMv2-RAN-UPF Repository](https://github.com/guimvmatos/P4-BMv2-RAN-UPF)

After this, you'll have two VMs running and linked with INCA.

3. VFs topology

Follow the instructions of [SFC-SRv6-topology](https://github.com/guimvmatos/SFC-SRv6-topology)


Now, you have all topology running. You can ping from clientVlc to dashServer with `ping6 fc20::2`. You can capture the packets passing through NFVs with tcpdump.

