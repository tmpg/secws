# Lab 0: Accessing the Labs Environment

In this chapter, we will create connections to the lab environment VMs and make sure we are ready for the hands-on.

Your environment is hosted on Oracle Cloud Infrastructure with the following compute instances (virtual machines):

* **secdb**: a Linux box hosting an Oracle 19C Database with a pluggable database PDB1 and most of the lab scripts. They are designed to be easily re-usable and are included in your [**Student's Package**](./files/Package.zip) for your reference or use in future projects.

* **dbclient**: a simple client machine with both 11gR2 and 19c instant clients. We will use it for some of the labs (e.g. when setting up network encryption).

* **av**: an Audit Vault Server 12.2 which will be configured as part of the labs.

* **emcc**: an Enterprise Manager Cloud Control 13cR2 installation.  There is a repository database with an agent that is deployed on secdb.

The virtual machines can be accessed by using an **SSH** client (**PuTTY**, **MobaXterm**) or with `ssh` from a terminal (bash, Mac or Linux).

For secdb, **VNC** has also been configured to provide a GUI to the desktop.

## Requirements

### General
* Access to the Oracle Cloud (OCI) tenancy. 
### Windows
* Download the Windows Access Package **[here](./files/secws_windows_access_pkg.zip)**.
### Mac (or Unix-like machines)
* Private key to access client machines by SSH. **[Download SSH keys](./files/dbsec_keys.zip)**.
* TigerVNC Viewer client software. 
  - Mac users can use this link.  **[Download TigerVNC Viewer for Mac](https://www.macupdate.com/app/mac/60678/tigervnc)**.

## Step 1: Get IP addresses for your workshop VM environment. ##

To get IP addresses to your own workshop environment, visit **[http://holadmin.oraclepts.nl](http://holadmin.oraclepts.nl)** and enter your email and the secret code provided by the instructor.  Then hit *Submit*.

![Get your VM IP addresses](./images/Lab000_Step0_1.png "")

You will see a Welcome page where you click on "Connection Details Download".

![IP addresses Welcome](./images/Lab000_Step0_2.png "")

A PDF file should download where you will see the public IPs for your workshop environment.  

![IP addresses Welcome](./images/Lab000_Step0_3.png "")

- Database server = **secdb**
- Database client = **dbclient**

## Step 2: Create SSH connections to secdb and dbclient ##

### For Windows Users using the Windows Access Package

Unzip and open `secws_windows_access_pkg.zip`

![Open secws_windows_access_pkg.zip](./images/Lab000_Step2_1.png "")

Using the **Public IPs** in **Step 1**: open **PuttyPortable**, load the **dbclient** profile, and add its IP. After, do the same for **secdb**.

![Setting PuttyPortables IPs](./images/Lab000_Step2_2.png "")

Be sure to keep both sessions open during the lab!

#### Troubleshooting
To rule out connectivity problems, use the **Workshop Connectivity Test System** profile in **PuttyPortable** to ensure your machine can connect to Oracle Infrastructure

![Troubleshooting](./images/Lab000_Step2_T.png "")

### Connect from Mac (or Unix-like machines) 

If you are using Linux, Mac or some bash terminal, you will need to use the downloaded private key in Open SSH format (*dbseckey.pub*).

To connect to the dbclient or secdb servers from command line, use the following syntax (change the path to the directory holding the dbseckey.pub file):

    $ cd /<path-to-keys-folder>/

Use the actual IP address for each server to create each terminal session:

    $ ssh -i dbseckey.pub oracle@129.213.112.147

The syntax to create an SSH tunnel to secdb enabling a VNC connection should be (**Remember to use your own IP address**):

  ssh -L 5902:localhost:5902 -i .ssh/dbseckey.pub oracle@129.213.112.147

## Step 3: Create a GUI connection to secdb's desktop

### For Windows Users using the Windows Access Package

Ensure an SSH session is established with **secdb** and drag&drop **secdb.vnc** into **VNC-Viewer.exe** 

![Open VNC-Viewer.exe](images/Lab000_Step3_1.png "")

A VNC session should appear.

![drag&drop secdb.vnc](images/Lab000_Step3_2.png "")

### Connect from Mac (or Unix-like machines)

For some labs, it will be easier to use a VNC connection to secdb. To do this, first connect using PuTTY to create the SSH tunnel and then launch a VNC client such as TigerVNC Viewer and connect to **localhost:2**

![Compression](./images/Lab000_Step1_8.png )

When asked for a password, enter **oracle**.

![Compression](./images/Lab000_Step1_9.png )

*Note: In case you need to work on the command line outside VNC, you can use WinSCP or another tool to copy files to your local computer. Connect as oracle and specify the private key to authenticate.*

### Now, you can start the workshop labs!

## Acknowledgements

- **Authors** - Adrian Galindo, PTS LAD & François Pons, PTS EMEA - Database Product Management - May 2020.
- **Credits** - This lab is based on materials provided by Oracle Database Security Product Management.
