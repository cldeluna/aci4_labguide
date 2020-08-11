# 05 Lab Integrate ACI with VMware Using Native Distributed Virtual Switch (DVS)



## Overview

One of the most powerful features of ACI is its native integration with the Virtualization Environment.

The steps in this lab will integrate ACI APICs with the VMware environment (already configured) via vCenter so that VMs on ESXi hosts will be able to connect to the Logical ACI environment and take advantage of the security we built in Lab 3.  In ACI this configuration construct is called a Virtual Machine Manager (VMM) Domain.

This integration allows the ACI APIC to communicate with vCenter as though it were an Administrator and automatically set up the required networking the Virtual Machines will need to communicate securely on the ACI fabric.



Required Services and Information 

- Administrator level account on vCenter
- Admin Username
- Admin Password
- The exact name of an existing Data Center configured in vCenter to which we want to integreate

In a production environment it is best practice to create a dedicated account for this integration, but in the lab we use the already existing admin account and credentials.

This lab specifically calls out the default VMware DVS which is part of ESXi.   That is because ACI supports integration with VMware vCenter using:

- Distributed Virtual Switch (DVS)	
  - Native to VMware
- Cisco Application Virtual Switch AVS
- Cisco ACI Virtual Edge (AVE) (From ACI 3.1(1) and VMware vCenter 6.0 and later)
  - Next generation virtual switch from Cisco

In this lab we will focus on native VDS integration as it is part of VMware.



Goals

Step 1 - Configure a Dynamic Vlan Pool (Move this to Lab 2)

Step 2 - Configure a vCenter VMM Domain

Step 3 - Verify Cisco APIC Connection to VMware vCenter Server

Step 4 - Verify that the APIC has Provisioned a DVS in vCenter



Configure VMM Domain Integration

Step 1 - Configure a Dynamic Vlan Pool

Navigate to Fabric >  Access Policies > VLAN and right-click on the VLAN folder icon and select **Create VLAN Pool**

Allocation Mode: Dynamic Allocation

| VLAN Range:  | Value | Allocation Mode |
| ------------ | ----- | --------------- |
| Range (From) | 3##1  | Dynamic         |
| Range (To)   | 3##9  | Dynamic         |

\* Replace ## with your assigned 2-digit Pod number

Note: Vlan 3##0 is part of the Static Pool

Click the plus sign (+) in the Encap Blocks table to configure the range.

Allocation Mode: Inherit allocMode from parent

Role: External or On the wire encapsulations

You will recall creating a static Vlan Pool in Lab 2.  With dynamic allocation, the APIC will automatically assign VLANS as needed within the range you define.  This facilitates automation as well as eases the burden of configuration.  

The role defines the use of the VLAN range.  External or On the wire encapsulation is used for allocating VLANS for each EPG associated to the VMM domain.  The VLANs are used when packets are sent to or from Leaf switches.  The internal role is used for private VLAN allocations in the internal vSwitch by the Cisco ACI Virtual Edge (AVE).  With the Intenral role, the VLANS are not seen outside of the ESXi host or on the wire.

Click OK to return to the pool configuration dialog and click Submit.



Step 2 - Configure a vCenter VMM Domain

Its important to remember that what you are actually configuring with a VMM Domain is a virtual switch on the Hypervisor(s) (ESXi) via the hypervisor manager (vCenter in our case).

Step 2.1 - vCenter Domain

Navigate to Virtual **Networking > VMM Domains > VMware** and right-click on the VMware folder and select **Create vCenter Domain**.  A Create vCenter Domain dialog will pop up.

Notice the various Hypervisors which can be integrated into ACI. 



| Setting:                              | Value                             | Comments                                      |
| ------------------------------------- | --------------------------------- | --------------------------------------------- |
| Virtual Switch Name:                  | POD11-vCenter-VDS                 |                                               |
| Virtual Switch:                       | VMware vSphere Distributed Switch | Unchanged<br />This is the default value      |
| Associated Attachable Entity Profile: | Leave Blank                       |                                               |
| Access Mode:                          | Read Write Mode                   | Unchanged<br />Read Write Mode is the default |
| Endpoint Retention Time (seconds)     | 0                                 | Unchanged<br />This is the default            |
| VLAN Pool:                            | POD##-Dynamic-VLAN-Pool           | Configured in Lab 2                           |



Enter the name: POD##-vCenter-VDS

Make sure that **VMware vSphere Distributed Switch** is selected 

Leave the Associated Attachable Entity Profile (AAEP) empty. You will define it in a later procedure.  Choose your dynamic VLAN pool (POD##-VLANs).



Step 2.2 - vCenter Credentials

Click the plus sign (+) in the  **vCenter Credentials:** table to define credentials with these settings.

| Name:                     | vCenter Username            | Password  |
| ------------------------- | --------------------------- | --------- |
| POD##-vCenter-Credentials | administrator@vsphere.local | 1234QWer! |

Click OK



Step 2.3 - vCenter Server

Click the plus sign (+) in the **vCenter:** table to define the controller settings (IP, etc.).

Name: POD##-vCenter-Server

The vCenter controller name does not have to match the name of the vCenter domain.    Either the IP or the hostname can be entered.

| Device     | Management IP Address | Username                    | Password  |
| ---------- | --------------------- | --------------------------- | --------- |
| vCenter-P1 | 192.168.10.202        | administrator@vsphere.local | 1234QWer! |
| vCenter-P2 | 192.168.10.204        | administrator@vsphere.local | 1234QWer! |
| vCenter-P3 | 192.168.10.206        | administrator@vsphere.local | 1234QWer! |
| vCenter-P4 | 192.168.10.208        | administrator@vsphere.local | 1234QWer! |
| vCenter-P5 | 192.168.10.210        | administrator@vsphere.local | 1234QWer! |



| Setting:                | Value                     | Comments                                                     |
| ----------------------- | ------------------------- | ------------------------------------------------------------ |
| DVS Version:            | DVS Version 6             | If you choose the default DVS version (6.5) you would not be able to add the hypervisor with version DVS 6.5 due to a VMware bug |
| Stats Collection:       | Disabled                  | This is the default value                                    |
| Data center:            | DC                        | The data center name must exactly match the data center name as it is defined in vCenter |
| Management  EPG:        | <Empty>                   | You do not configure any EPG for managing the VMware vCenter because the connection from the Cisco APIC to the vCenter is out-of-band (OOB) |
| Associated Credentials: | POD##-vCenter-Credentials |                                                              |
|                         |                           |                                                              |



Set vSwitch Policy to CDP, leave all other settings at their default values 

Click Submit



Step 3 - Verify Cisco APIC Connection to VMware vCenter Server



Step 3.1 Verify that the APIC has discovered vCenter

Navigate to Virtual Networking > VMM Domains > VMware and expand your vCenter domain and all of its subelements.

Note: The APIC connects to the vCenter and obtains its inventory, including hypervisors, VMs, and uplinks.  You will see all the VMs that have been installed on your host.

Examine the status of the vmnic interfaces.  You can over over them in the Topology page, click them in the naviation pane, or go to the General tab.

vmnic 0,2, and 3 should be up.

VMNIC Map for POD##

| VMNIC # | Connected To      | Function   |
| ------- | ----------------- | ---------- |
| 0       | Management Switch | Management |
| 2       | LEAF-1            | Data Path  |
| 3       | Leaf-2            | Data Path  |



Step 4 - Verify that the APIC has Provisioned a DVS in vCenter

Step 4.1 Accessing vCenter

In Google Chrome open another tab and connect to vCenter via https://vcenter.  The hostname will resolve to 192.168.10.50

Accept the untrusted certificate security warnings and clikc vSphere Web Client (Flash)

If prompted, enable Adobe Flash Playre by clicking its button and choosing Allow

Accept any security warnings and log in as **administrator@vsphere.local** with password **1234QWer!**



Go to Networking .  Expand the folder that has been created under your data center (DC).  You should see a DVS wit hthe name of the configured vCenter domain (POD##-vCenter-vDS), within a folder of the same name.  Expand the DVS to see two networks have been automatically created.  Click the Summary tab to see details about the DVS.



vCenter can take up to 15 minutes upon bootup to be ready.  When vCenter vecomes active and reachable you will be able to see the elements.



Configure AAEP to Selectively Allow Vlan Traffic.

Attachable Access Entity Profiles (AAEPs) can be considered the "where" of the fabric configuration and are used to group domains with similar requirements.  They allow a one to many relationship between the policy groups and domains.

AEPs are tied to interface policy groups. One ore more domains are added to an AAEP. By grouping domains into AAEPs and associating them, the fabric knows where the various devices in the domain reside.  Cisco APIC can push the Vlans and policy to the required interfaces.



