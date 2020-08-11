## Lab 03 - Configure Tenants and Tenant Constructs

## Context

Having completed the Fabric Policy configuration so that we have an operational Underlay (confusingly called "Overlay-1" as you will see when executing CLI commands) and set our Switch Policies (which define our switches) and Interface Policies, Profiles, and AEPs (which define interface behaviors and *allowed* encapsulations or Vlans) we now start to define the logical components that will make it very easy to secure hosts and services uniformly at scale without consideration of their IP addresses, rather via their EPG association.

This logical separation begins with one or more Tenants.  The fabric comes "out of the box" with three Tenants.

- infra
  - A Tenant container for the ACI Fabric Infrastructure
- mgmt
  - Dedicated tenant for managing the ACI Fabric 
- common
  - A tenant with special properties so that any construct configured within this tenant is automatically available to any other Tenant created in the fabric.
  - This tenant is very useful for shared services and you will often find an L3Out , VRF, and Bridge Domains which are shared across other Tenants configured in the common tenant.




## Lab Goals


Tenant portion of the ACI Management Information Tree (MIT)

![Tenant Objects](https://www.cisco.com/c/dam/en/us/td/i/300001-400000/340001-350000/348001-349000/348505.jpg)









- Understand the relationships between the ACI Logical Constructs
- Step 1: Create a Tenant and VRF
- Step 2: Create a Bridge Domain and Subnets
- Step 3: Create an Application Profile and EPGs
- Step 4: Create Filters and Contracts
- Step 5: Apply Contracts to EPGs

This is the final "set up only" lab.  At the completion of this lab, the fabric will be ready for actual traffic and connectivity.

The following constructs or objects will be configured in this lab:

| Object Type         | Object Name                | Function                                                     |
| ------------------- | -------------------------- | ------------------------------------------------------------ |
| Tenant              | POD##_Tenant               | Administrative domain containing all subsequent objects (VRFs, BDs, EPGs, Subnets, Application Profiles) |
| VRF                 | POD##_VRF                  | Layer 3 routing and forwarding domain within a tenant        |
| Bridge Domain       | POD##_BD                   | Logical container defining flooding behavior. Always associated with a single VRF and often associated with one ore more subnets. |
| Subnet              | 10.0.1.254/24              | IP Subnet                                                    |
| Application Profile | Tiered_AppProfile          | Container for EPGs and the policies which define associations, encapsulations, and interactions |
| EGP                 | - Web<br />- App<br />- DB | Logical grouping of endpoints which have similar requirements, often security requirements. |

Reminder:

- \## = Your Pod Number.  For example, if your Pod number is 11, then replace \## with 11 (POD11_Tenant)

It is important to think about how to consistently name your constructs.  You have some flexibility in the lab but please keep in mind that the lab is shared and so you should always make sure your constructs can be associated to your tenant and your work.  



#### Step 1 - Create a Tenant and VRF

A Tenant in ACI represents a management domain.    Common tenants in actual deployments include tenants such as:

- Production
- Dev
- QA
- DMZ

As you can see from the MIT diagram, a Tenant contains one or more VRFs and so you often find that the Production tenant has a Production VRF associated with it.   A VRF in ACI is a VRF.  You cannot have overlapping IP Address space within an VRF and so the same is true for a VRF in ACI.

In the lab, each lab participant will create a tenant based on their Pod assignment (Pod number).

Navigate to **Tenants > Add Tenant **

![Add Tenant](images\03_addTenant.jpg)

The **Create Tenant** dialog box will appear.  The only required field is the Tenant name.

Enter your Tenant name in the **Name** field and click Submit.

Notice that here you can also associate a Monitoring Policy as well as Security Domains.  If this Tenant had special access requirements, then here is where you can associate a specific Security Domain policy with the Tenant.  This can be done after the Tenant is created.

![03_addTenantDialog](images\03_addTenantDialog.jpg)



Because we left the "Take me to this tenant when I click finish" option checked, we will be taken to the new tenant page once Submit is clicked.  

From here navigate to **Networking > VRFs ** and right click on the VRF Folder icon.  Select **Create VRF** and the Create VRF dialog will appear.

![Create VRF Options](images\03_createVRF.jpg)



In the **Create VRF** dialog, enter the VRF name, uncheck the "Create A Bridge Domain" check box, and Click "Finish".  Once you uncheck the "Create A Bridge Domain" check box the **Next** option becomes **Finish** as there is no next step.
All other values remain unchanged and at their default settings.

As you know, ACI behaves in a default-deny,  "white list" security model, so that only the required access is defined for connectivity.   In a "black list" model, only the "bad" traffic is denied which assumes you know what the "bad" traffic is.   The default-deny, only allow what is needed ("white list"), is by far the more secure.   ACI does give you the option of disabling this behavior in several ways, one of which is here a the VRF level.  Note that the **Policy Control Enforcement Preference** is **Enforced** by default.   Should you have a need to disable this behavior (NOT RECOMMENDED), you can do so by setting  **Policy Control Enforcement Preference** to **Unenforced**.  This is rarely done in production but may be a useful troubleshooting tool.

 

![03_addTenantVRF](images\03_addTenantVRF.jpg)

Because this VRF was created within the Tenant context it is now associated with the Tenant.  



#### Step 2: Create a Bridge Domain and Subnets

In traditional networking a Vlan inherently defines a broadcast domain, an encapsulation, and optionally a Layer 3 subnet.  We don't typically think of all of those items as separate functions but in ACI you must.  In its basic form, a Bridge Domain defines a broadcast domain as well as flooding behavior (we don't call this out specifically in traditional network because we don't have any other options, but in ACI we do).  ACI can optimize flooding behavior and reduce broadcasts and so a Bridge Domain allows you to define how you want flooding to behave for a particular Bridge Domain (BD).   It is also common to define a subnet within the Bridge Domain thereby creating the foundation for a Layer 3 "Vlan".  Notice that you don't define the encapsulation within the Bridge Domain. That takes place within an EPG either statically or dynamically depending on the need.  

A Bridge Domain can have multiple subnets and contains settings which instantiate the subnet's gateway on the ACI Fabric.

Navigate to **Tenants > PODXX_Tenant > Networking > Bridge Domains ** and right click on the **Bridge Domain Folder** icon.  Select **Create Bridge Domain** and the Create Bridge Domain dialog will appear.



![03_BD_CreateDialog](images\03_BD_CreateDialog.jpg)



![03_BD_CreateDialog2_Forwarding](images\03_BD_CreateDialog2_Forwarding.jpg)



![03_BD_CreateDialog3_Forwarding](images\03_BD_CreateDialog3_Forwarding.jpg)





![03_BD_CreateDialog4_Layer3](images\03_BD_CreateDialog4_Layer3.jpg)





![03_BD_CreateDialog5_SubnetDialog](images\03_BD_CreateDialog5_SubnetDialog.jpg)



![03_BD_CreateDialog6_SubnetDialog](images\03_BD_CreateDialog6_SubnetDialog.jpg)



![03_BD_CreateDialog7_Advanced](images\03_BD_CreateDialog7_Advanced.jpg)



![03_BD_CreateDialog8_Completed](images\03_BD_CreateDialog8_Completed.jpg)



#### Step 3: Create an Application Profile and EPGs

An Application Profile is on organizational or grouping construct only.  It is a container or folder for EPGs and their associations.

An EndPoint Group (EPG) is a container for endpoints that share some commonality.  In many cases, these are endpoints that share a common security profile.  This is an important distinction to make because Contracts (think of these as ACLs which act on EPGs rather than subnets and IP addresses) are applied to EPGs.

Navigate to **Tenants > PODXX_Tenant > Application Profiles ** and right click on the Application Profiles Folder icon.  

![03_AP_CreateAppProfile](images\03_AP_CreateAppProfile.jpg)

Select **Create Application Profile** and the Create Application Profile dialog will appear.  Enter the Application Profile Name in the **Name:** field and click **Submit**.

You now have a container for your EPGs and their associated constructs.

From here, expand the Application Profile folder by clicking on the ">" symbol.

![03_AP_CreateAppProfileDialog](images\03_AP_AppProfileExpand.jpg)



Expand your Application Profile ***Tiered\_AppProfile*** and right click on the **Application EPG**s folder icon for the "**Create Application EPG**" option.

![03_EPG_CreateEPG](images\03_EPG_CreateEPG.jpg)



Remember the Tenant portion of the ACI Management Information Tree (MIT).  Notice that an EPG must be associated to 

- one Application Profile (which is associated to one Tenant)
- one Bridge Domain (which is associated to one VRF)

At a minimum those objects must be defined in order to create an EPG.  Other EPG attributes such as Contracts and VM Domain profiles can be configured at a later time.

![Tenant Objects](https://www.cisco.com/c/dam/en/us/td/i/300001-400000/340001-350000/348001-349000/348505.jpg)



![03_EPG_CreateEPGDialog](images\03_EPG_CreateEPGDialog.jpg)

Enter the EPG name in the **Name:** field.  Select the Bridge Domain in the Bridge Domain: field and click **Finish**.

Accept all the default values.

![03_EPG_CreateEPGDialogCompleted](images\03_EPG_CreateEPGDialogCompleted.jpg)



Using this same process, create two additional EPGs within this Application Profile, mapped to your PODXX_BD Bridge Domain.

- App
- DB



![03_AP_with_all_EPGs](images\03_AP_with_all_EPGs.jpg)





![03_AP_Topology](images\03_AP_Topology.jpg)



#### Step 4: Create Filters and Contracts

Contracts, and how they behave, make ACI one of the easiest fabrics to secure.  Think of them as Access Control Lists which can be applied anywhere across the fabric (across all of your top of rack Leaf switches) and which are independent of a vlan or IP Subnet/Address.

 A Cisco ACI filter is a TCP/IP header field, such as a Layer 3 protocol type or Layer 4 ports, that are used to allow inbound or outbound communications between EPGs.

A Cisco ACI contract defines the rules that specify what and how communication in a network is allowed. In Cisco ACI, contracts specify how communications between EPGs take place. Contract scope can be limited to the EPGs in an application profile, a tenant, a VRF, or the entire fabric.  It is important to understand the impact scope can have on access.

Contracts are wholly logical constructs and so there is no Physical Layer configuration.

Contracts defined in the common tenant can be used across any Tenant like other constructs defined in the common tenant.

You will configure three filters and use them as building blocks for defining contracts between EPGs.

1. Basic Filter
   - Allow ICMP and SSH
2. HTTP Filter
   - Allow HTTP 
3. FTP Filter
   - Allow FTP

A few naming Tips:

Start the name with protocol/port as that is what typically appears to reduce ambiguity

If you are going to do Taboo (not recommended) or Deny contracts, consider adding the action to the Subject name. 

For example: TCP_22_Permit_SSH_Subject. If you will not use these then establish the convention that all Subjects are "Permit".Consider adding Source or Destination (For example: TCP_9004_DST_Entry)

Example Object Names:

- TCP_9004_Filter
- TCP_9004_Entry
- PKI_BIDi_Contract
- TCP_22_SSH_Subject

For the lab we will use a simplified naming convention.

###### Filter Table

| Filter Name                | Entry Name | EtherType | IP Protocol | Source Port_Range<br />From - To | Destination Port_Range<br />From - To |
| -------------------------- | ---------- | --------- | ----------- | -------------------------------- | ------------------------------------- |
| BASIC-POD##                | ICMP       | IP        | icmp        | Leave blank                      | Leave blank                           |
| BASIC-POD## (second entry) | SSH        | IP        | tcp         | 1024 - 65535                     | 22 - 22                               |
| HTTP-POD##                 | HTTP       | IP        | tcp         | 1024 - 65535                     | 80 - 80 (http - http also works)      |
| FTP-POD##                  | FTP        | IP        | tcp         | 1024 - 65535                     | 21 - 21                               |



##### Step 4.1 Basic Filter for Ping (ICMP) and SSH

Within your Tenant, navigate to  **Contracts > Filters** 

Right-click **Filters** and choose **Create Filters**

![03_04-1-FiltersCreate](images\03_04-1-FiltersCreate.jpg)



1. Create a filter named **BASIC-POD##**
2. Click the plus sign (**+**) to add an Entry (again, think of this as a single Access List Entry).
3. Complete the first entry which you will name ICMP per the table above
4. Click the **Update** button for your entry row.  

![03_04-1-FiltersCreateDialog](images\03_04-1-FiltersCreateDialog.jpg)

This filter will have two entries, ICMP and SSH

Click the plus sign again to add the second SSH entry.

Click **Update** and **Submit**

![03_04-1-FiltersCreateDialog2ndEntry](images\03_04-1-FiltersCreateDialog2ndEntry.jpg)



##### Step 4.2 HTTP Filter for HTTP 

Using the procedure above and the information in the Filter Table, create an HTTP filter.



##### Step 4.3 FTP Filter for FTP

Using the procedure above and the information in the Filter Table, create an FTP filter.



Once you complete all of the filters in the Filter Table, you should be able to expand all the objects in the **Filter** folder and should resemble this:

![03_04-1-FiltersCreateCompleted](images\03_04-1-FiltersCreateCompleted.jpg)



##### Step 4.4 Contracts

Contracts provide a way to control traffic flow within the ACI fabric between endpoint groups. Because ACI operates by default in a white list mode, without contracts there is no communication between EPGs. These contracts are built using a provider-consumer model where one endpoint group provides the services it wants to offer (servers) and another endpoint group consumes them (servers or clients). Contracts are assigned a scope of Global, Tenant, VRF, or Application Profile, which limit the accessibility of the contract.  The default scope is VRF.Exter

Contracts consist of:

- Subjects - One or more subjects can be associated with a Contract. A subject has a name, a filter, and an action (Permit or Deny). A Subject also has a shortcut to support bidirectional (Apply Both Directions and Reverse Filter Ports). 
- Filters - a Subject can have 1 or more filters.  Think of the Filter as an ACE or Entry within an ACL.

A Contract in ACI will define the permitted (white list) communication between its associated EPGs.  Again, think of the Filters as Access Control Entries (ACE) and the Contract as an Access Control List (ACL). The biggest difference you will see, is that there are no vlans or IP Subnets or Host IP Addresses in these configurations.   

With the Filters you have defined, you have port/protocol objects which you can now bundle together and apply in whichever direction is appropriate to EPGs.POD

You will create two Contracts Using these FIlter Objects.

###### Contract Table

| Contract Name    | Subject Name               | Apply Both Directions Checkbox | Reverse Filters Ports Checkbox | Filter Name (Select Existing form Drop Down) | Action |
| ---------------- | -------------------------- | ------------------------------ | ------------------------------ | -------------------------------------------- | ------ |
| WebAccess-POD##  | WEB-ACCESS                 | Checked (Default)              | Checked (Default)              | PODXX/BASIC-PODXX                            | Permit |
| WebAccess-POD##  | WEB-ACCESS (Second Entry)  | Checked (Default)              | Checked (Default)              | PODXX/HTTP-PODXX                             | Permit |
| FileAccess-POD## | FILE-ACCESS                | Checked (Default)              | Checked (Default)              | PODXX/BASIC-PODX                             | Permit |
| FileAccess-POD## | FILE-ACCESS (Second Entry) | Checked (Default)              | Checked (Default)              | PODXX/FTP-PODXX                              | Permit |

Directives Leave blank for all (Default)

Priority- Leave blank for all (Default)

###### Step 4.4.1 Web Access Contract

Within your Tenant, navigate to  **Contracts** 

Right-click **Contracts** folder icon and choose **Create Contract**

![03_04-4-ContractCreate](images\03_04-4-ContractCreate.jpg)



1. Create a contract named **WebAccess-POD##**.  Accept all the defaults including the Scope (VRF).
2. Click the plus sign (**+**) in the **Subjects:** section to create a subject and add Filters.

![03_04-4-ContractCreateName](images\03_04-4-ContractCreateName.jpg)

3. Add Filters using the previously created filters per the table above
4. Click the **Update** button for your entry row.  

![03_04-4-ContractCreateDialog-Subject](D:\Dropbox (Indigo Wire Networks)\scripts\training\2020\aci4_labguide\docs\images\03_04-4-ContractCreateDialog-Subject.jpg)



5. Click **OK** to return to the Contract dialog

![03_04-4-ContractCreateDialog-SubjectOK](images\03_04-4-ContractCreateDialog-SubjectOK.jpg)



6. Click **Submit** to complete the contract

![03_04-4-ContractCreateSubmit](images\03_04-4-ContractCreateSubmit.jpg)



###### Step 4.4.2 File Access Contract

Using the procedure above and the information in the Contract Table, create the File Access Contract.



Upon completion of the two Contracts, your Contracts section should look like below:

![03_04-4-ContractCreateCompleteExpanded](images\03_04-4-ContractCreateCompleteExpanded.jpg)



#### Step 5: Apply Contracts to EPGs

Just as with ACLs its a good idea to make sure you understand the application flows.

In ACI the terminology may add confusion at first.  Just remember that 'Providing' a Contract means providing a service (like TCP/80 inbound) and 'Consuming' a contract allows an EPG to connect to a provided service.

Without contracts the endpoints in our EPGs have no connectivity outside their EPG.

When we apply our new Contracts we want the endpoints in the App EPG to provide HTTP, ICMP, and SSH to the endpoints in the Web EPG.  We also want the endpoints (databases and file servers) in the DB EPG to provide FTP, ICMP, and SSH services to the endpoints in the App EPG.

![CloudMyLab_ACI4LabGuide_Diagrams_Contracts](images\CloudMyLab_ACI4LabGuide_Diagrams_Contracts.jpeg)



##### Step 5.1 Apply the WebAccess-PODXX contract as a Provided Contract to the App EPG

Navigate to **PODXX_Tenant >** and expand **Tiered-App > Application EPGs > App  > Contracts** 

Right click on the **Contracts** Folder icon and select **Add Provided Contract**

Select **WebAccess-POD##** contract. No other changes are necessary

Click **Submit**

Examine the visual relationship by clicking on your **Tiered-App** Application Profile







##### Step 5.2 Apply the FileAccess-PODXX contract as a Provided Contract to the DB EPG

 Use the procedure above to apply the **FileAccess-POD##** contract as a **Provided Contract** of the **DB** EPG.






## Check in

This is quite a long lab so lets check in to see what we have accomplished and what functionality we should have.

Once this Lab is complete you should have:

- A Tenant and associated VRF
- A Bridge Domain containing three subnets associated with the VRF
- An Application Profile "container" with three EPGs all mapped to the Bridge Domain (one BD but three subnets)
- Filters describing
  - ICMP
  - SSH
  - HTTP
  - FTP
- Two contracts describing 
  - the allowed connectivity between the App EPG and the Web EPG utilizing the appropriate Filters
  - the allowed connectivity between the APP EPG and the DB EPG utilizing the appropriate Filters
- The contracts applied to the appropriate EPGs in the appropriate direction ( Provider - Server and Server Services and Consumer - Clients)

The necessary Tenant-level logical components have now been created however a few steps remain to achieve connectivity external to the fabric and to Virtualized Hosts and Virtual Machines.