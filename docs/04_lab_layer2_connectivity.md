## Lab 04 Configure Host Connectivity Constructs (L2) and Layer 2 Links

## Context

So far we have a working fabric (underlay), working individual interface behaviors and characteristics, and our logical overlay (Tenants, EPGs, etc.).

We have no "Data Plane" connectivity.  That  is, while we can manage our fabric and the fabric itself is operational, there is no connectivity to external network devices or hosts.  As with any network, this is where you will spend most of your time once the fabric is operational.  

## Lab Goals

The primary goal of this lab is to understand the physical and logical steps necessary to achieve connectivity from an ACI fabric to hosts and external networking devices.

As with all things in ACI, it is important to understand the relationship between the constructs which abstract the physical layer and the logical.

Lets begin with our topology.

![04_CloudMyLab_ACI4LabGuide_Diagrams - L2Topology](images\04_CloudMyLab_ACI4LabGuide_Diagrams - L2Topology.jpeg)



- Understand Interface Constructs and Relationships

- Step 1:  Create an Access Port

- Step 2:  Create a Trunk Port

- Step 3:  Create a L2 vPC Access & Trunk

  

## Putting It All Together

### Access Policies and Tenant objects to achieve connectivity





---

#### Step 1: Create an Access Port

##### **High Level Workflow for Access Port**

###### Configure the Physical Layer

Configuring Interface to External interface (in this case it will be a switch interface)

1. *Configure the port behavior (Port Policy Group)*
2. *Apply the port behavior to one or more interfaces (Interface Profile and Interface Selector)*
3. *Associate those interfaces to one or more switches as called for by the design*



###### Configure the Logical Layer

“Configure” the EPG

- *Associate the Physical Domain*
- *Associate the Static Ports**
- *Confirm default or more specific contracts are in place* 



Step 1.1: **Configure the port behavior (Port Policy Group):**

**Fabric > Access Policies** > Interfaces > Leaf Interfaces > Policy Groups > Leaf Access Port 

Create Leaf Access Port Policy Group

POD##-PROD-10GAccess-PortPolGrp

- 10G
- CDP enabled
- LLD enabled
- AEP PODXX



Step 1.2: **Apply the behavior defined by the port policy group "ENJ-PROD-10GAccess-PortPolGrp” to one or more interfaces:**

**Fabric > Access Policies** > Interfaces > Leaf Interfaces > Profiles 

Create Leaf Interface Profile
POD##-PROD-DB-10GAcc-205-206-P03-IntProf

POD##-PROD-DB-10GAcc-205-206-P14-IntProf

POD##-PROD-DB-10GAcc-207-208-P16-IntProf

POD##-PROD-DB-10GAcc-207-208-P04-IntProf




##### Step 1 - 

## Check in