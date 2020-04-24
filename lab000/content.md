# Accessing Labs Environment #

In this chapter, we will connect to the labs environment and make sure we are ready for the hands-on.
Your environment is hosted on Oracle Cloud Infrastructure with the following virtual machines as instances on IaaS:

- **dbclient**: a simple client application with both 11gR2, 12cR2 and 18c instant clients. We can use it for some of the labs (e.g. when setting up network encryption)
- **secdb**: a Linux box hosting an Oracle Database 19c with a pluggable database PDB1 and most of the labs scripts: they are designed to be easily re-usable
- **av**: an Audit Vault Server 12.2 which will be configured as part of the labs
- **emcc**: an Enterprise Manager Cloud Control 13cR2 whose agent is deployed on secdb

The virtual machines can be used accessing with a SSH Client (Putty, MobaXterm) or with `ssh` on the command line (bash, mac or linux).
For secdb, the Virtual Cloud Network (**VNC**) is also configured to access a graphical environment.

## Disclaimer ##
The following is intended to outline our general product direction. It is intended for information purposes only, and may not be incorporated into any contract. It is not a commitment to deliver any material, code, or functionality, and should not be relied upon in making purchasing decisions. The development, release, and timing of any features or functionality described for Oracle’s products remains at the sole discretion of Oracle.

## Requirements ##

Access to the OCI tenancy, provided by the instructor.
Private key to access via SSH with the client.
Optionally for secdb, a VNC client.

## Step 1: Connect to OCI Console ##
Connect to the OCI using **OraclePartnerSAS** tenant:

https://console.us-phoenix-1.oraclecloud.com/?tenant=OraclePartnerSAS#/a/

![](./images/Lab000_Step1_1.png)

The user name and password will be provided by your instructor.

### Get the Instance listing

Once your are connected into the OCI console, select from the Menu->Core Infrastructure->Compute->Instances

![Compute Instance](./images/Lab000_Step1_2.png "")

On the **List Scope** section, choose the **COMPARTMENT** assigned to you by the instructor.

### Copy the Public Address

Here you can see the public IP Address assigned to each of your instances.

![Copy IP Addresses](./images/Lab000_Step1_3.png "")

These IP Addresses will be used to connect with SSH client.

## Step 2: Connect to the instances

### From Windows

Using the Public Address, open a ssh client like Putty and connect to the instance

![Connect to the instance](./images/Lab000_Step2_1.png "")

Set the **oracle** user in Connection->Data->Auto-login username for **dbclient** and **secdb** connections

![Connect with oracle User](./images/Lab000_Step2_2.png "")

Choose the private key on Connection->SSH->Auth menu and save it to the session 

![Choose private key](./images/Lab000_Step2_3.png "Private Key")

Additionally you can specify a keepalive of 10 seconds to prevent disconnections 

![Keepalive](./images/Lab000_Step2_4.png "")

### Connect from Linux or Mac

If you are using Linux, Mac or some bash terminal, you will need to use the private key in Open SSH format.

The file containing the private key to use in that case is dbseckey (without extension).

To connect to the dbclient or secdb servers from command line, use the following syntax (change the path to the directory holding the labkey file):

    $ cd /<path-to-keys-folder>/

Use the actual IP address for each server, to open 2 terminals: 

    $ ssh -i dbseckey oracle@ip.address

You now can start the workshop labs.


## Acknowledgments

**Authors** 

- Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - April 2020.