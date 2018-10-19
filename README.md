Check Point SM SG end-to-end demo using ansible
=========

Local Ansible demo using Vmware Workstations on windows 10
------------
Ansible playbook that provisions Check Point Security Management servers and Security Gateways as well as providing control over a Check Point Management server using Check Point's web-services APIs.
Note: This demo has been tested and known to be working on on Ubuntu WSL

Requirements
------------
- Windows 10
- VMware Workstation
- 16 Gig of Ram
- At least 50 Gb of avaliable diskspace
- Check_Point_R80.20_T101_Security_Gateway.iso downloaded to the local Windows 10 host
- Check_Point_R80.20_T101_Security_Management.iso downloaded to the local Windows 10 host

Installation instructions
--------------
1. Enable access to Windows Store on Domain joined machines
 - Usually the WSUS Policy prevents Windows 10 users to connect to Microsoft Store as organizations have configured local  Windows Server Update Services (WSUS) preventing these machines to connect to Microsoft Store in order to download apps for Windows 10.
 - To be able to download a Linux distribution as a app from Microsoft Store to use as Windows Subsystem for Linux (WSL). Open the registry by tuping regedit in te start menu and edit this value:
```
Hive: HKEY_LOCAL_MACHINE 
Key path: Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate 
Value name: DoNotConnectToWindowsUpdateInternetLocations 
Value type: REG_DWORD 
Value data: 0
```
This should allow you to download programs from the Windows store for the next 90 minutes until Group Policy checks in again.
I recommend that you set the value back to 1 in the registry to be compliant witht the company group policy.

2. Enable Windows Subsystem for Linux (WSL) and install the Ubuntu Linux distro for WSL as a app.
Follow the instructions on Microsofts Website: https://docs.microsoft.com/en-us/windows/wsl/install-win10. Do not forget to follow the steps to initilize the Linux distro instance.

3. Add required packages to your Ubuntu Linux distro for WSL
 - Open a command promt and type "wsl" to get into Ubuntu Linux distro for WSL.
 - Make sure you are in /home/sysadmin (cd ~)
 - Install Ansible Repo, python netaddr and mtools
```
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get update
sudo apt-get install -y ansible python-pip
sudo apt install python-pip
pip install netaddr
sudo apt install mtools
```

4. Clone and copy the Ansible Module - check_point_mgmt by Check Point® to the correct location
```
git clone --recursive https://github.com/CheckPointSW/cpAnsible
sudo cp -r cpAnsible/check_point_mgmt/ /usr/lib/python2.7/dist-packages/ansible/
sudo cp -r cpAnsible/check_point_mgmt/cp_mgmt_api_python_sdk/ /usr/lib/python2.7/dist-packages/
sudo mkdir -p $HOME/.ansible/plugins/modules
sudo cp -r cpAnsible/check_point_mgmt/ $HOME/.ansible/plugins/modules/
```

5. Clone the cpAnsibleDemo repository with this command:
```git
git clone --recursive https://github.com/jimoq/cpAnsibleDemo
```

6. initialise the demo enviroment
This step will install Gaia on a virtual machine called template, The playbook will ask the user for some information. The installation process will run twice and two snapshots will be taken, one for security management only installations using Linux gaia kernel 3.10 (this snapshot is called sm-pre-ftw) and one for Security Gateway and Stand Alone (mgmt+gw on same machine) installations using Linux gaia kernel 2.6.18 (this snapshot is called pre-ftw). 
 - The ansible plabooks must be executen from local location.
 - Thw installation requires user interaction twice when the Gaia installation boot process starts, (as it starts twice).
 - The VM template ip is default configured to 172.27.254.3 this can be changed by editing the tempate_ip line var in cpAnsibleDemo/group_vars/all
```
cd cpcpAnsibleDemo
ansible-plabook initialise_demo_env.yml
```

Dependencies
------------
The demo is currently using vmnat8 ip-range 172.27.254.0/24 this can be adjusted by changeing the ip addresses in thefollowing files
```
vi cpAnsibleDemo/group_vars/all
vi cpAnsibleDemo/group_vars/mds_var.yml
vi cpAnsibleDemo/group_vars/sg_var.yml
vi cpAnsibleDemo/group_vars/smsg_var.yml
vi cpAnsibleDemo/group_vars/sm_var.yml
```

Execution of Paybooks
----------------
```
ansible-playbook sm.yml                 # Build Security Management with deploy a gateway, configure policy in nordics and install that on the security Gateway (execution tine ~20 min)
ansible-playbook smsg.yml               # Build Security Gmanagement and Gateway (Stand alone), configure policy in nordics and install that (execution tine ~15 min)
ansible-playbook mds.yml                # Build MDS with 4 domanis and deploy a gateway, configure policy in nordics domain and install that on the security Gateway (execution tine ~30 min)
ansible-playbook SecurityCheckup.yml    # Build Securoty Checkup congfiguration on physical appliance (WIP not there yet)
```

Admin users
-------
- VM template:    uid: admin / pwd: admin
- VM sm:          uid: admin / pwd: vpn123
- VM smsg         uid: admin / pwd: vpn123
- VM mds          uid: admin / pwd: vpn123
- VM WebServer    uid: root / pwd: Cpwins1! (WIP not there yet)

License
-------

Apache License 2.0

Author Information
------------------
Jim Öqvist