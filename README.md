Check Point MDS, SM and SG end-to-end demo using Ansible
=========

What is this?
------------

Ansible playbooks that will without any user interaction provision Check Point Security Management server (SM) or Multi-Domain Management Server (MDS) and Security Gateways (SG) as well as providing control over a Check Point Management server using the Ansible Module - check_point_mgmt by Check Point® and Check Point's Security Management web-services APIs.
##### Note: This demo has been tested and known to be working on Ubuntu WSL

Version
------------
2.0

Requirements
------------
- Windows 10 version that supports Windows Subsystem for Linux WSL
- Logged in user in Microsoft Store
- VMware Workstation 10 or later
- 16 Gig of Ram
- At least 50 Gb of available disk space
- Check_Point_R80.20_T101_Security_Gateway.iso downloaded to the local Windows 10 host
- Check_Point_R80.20_T101_Security_Management.iso downloaded to the local Windows 10 host
- ISOs are avaliabe here: https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk122485

Installation instructions
--------------
1. Enable access to Microsoft Store on Domain joined machines
 - Usually the WSUS Policy prevents Windows 10 users to connect to Microsoft Store as organizations have configured local Windows Server Update Services (WSUS) preventing these machines to connect to Microsoft Store in order to download apps for Windows 10.
 - To be able to download a Linux distribution as a app from Microsoft Store to use as Windows Subsystem for Linux (WSL). Open the registry by typing “regedit” in the start menu and edit this value:
```
Hive: HKEY_LOCAL_MACHINE 
Key path: Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate 
Value name: DoNotConnectToWindowsUpdateInternetLocations 
Value type: REG_DWORD 
Value data: 0
```
This should allow you to download programs from the Microsoft Store until Group Policy checks in again.
I recommend that you set the value back to 1 in the registry to be compliant with the company group policy.
##### NOTE: Before continuing. Open Microsoft Store by typing store in start menu and make sure you are logged into Microsoft Store with a user

2. Enable Windows Subsystem for Linux (WSL) and install the Ubuntu Linux distro for WSL as a app.
Follow the instructions on Microsoft’s Website: https://docs.microsoft.com/en-us/windows/wsl/install-win10. Do not forget to follow the steps to initialise the Linux distro instance.

3. Add required packages to your Ubuntu Linux distro for WSL
 - Open a command prompt and type "wsl" or “bash” to get into Ubuntu Linux distro for WSL.
 - Install Ansible Repo, mtools, python pip unzip and python netaddr, 
```
cd ~
sudo apt-add-repository -y ppa:ansible/ansible
sudo apt-get update
sudo apt-get install -y ansible python-pip mtools unzip
pip install netaddr
ansible --version
```
##### NOTE: After this step verify that Ansible has been updated to latest version, ~2.7 or later. 

4. Clone and copy the Ansible Module - check_point_mgmt by Check Point® to the correct location
```
cd ~
git clone --recursive https://github.com/CheckPointSW/cpAnsible
sudo cp -r cpAnsible/check_point_mgmt/ /usr/lib/python2.7/dist-packages/ansible/
sudo cp -r cpAnsible/check_point_mgmt/cp_mgmt_api_python_sdk/ /usr/lib/python2.7/dist-packages/
mkdir -p $HOME/.ansible/plugins/modules
cp -r cpAnsible/check_point_mgmt/ $HOME/.ansible/plugins/modules/
```

5. Clone the cpAnsibleDemo repository and change the permission on all files in that directory:
##### NOTE: The cpAnsibleDemo folder needs to be located in $HOME for example /home/sysadmin
```
cd ~
git clone --recursive https://github.com/jimoq/cpAnsibleDemo
chmod -Rv o-rwx ./cpAnsibleDemo
```

6. Add following lines to end of /etc/ansible/hosts
```
[localhost:vars]
ansible_user=MyDefautlUnixUser
ansible_ssh_pass=PasswdForMyDefautlUnixUser
ansible_python_interpreter=/usr/bin/python

[localhost]
127.0.0.1 
```

7. Add location of vmrun.exe to Widows 10 path
```
Click  "Start Menu" > "This PC" > "Properties" > "Advanced System Settings" > "Environment Variables" > "Path" > "Edit" > "New" > "C:\Program Files (x86)\VMware\VMware Workstation"
```
This will add the path to vmrun.exe. This is most probably "C:\Program Files (x86)\VMware\VMware Workstation". but you should verify that on your computer.

8. Initialise the demo environment
 * This step will install Gaia on a virtual machine called “template”.
 * The playbook will ask the user for some information. 
 * The installation process will run twice and two snapshots will be taken
   *   1st snapshot is called pre-ftw for Security Gateway and Stand Alone installations using Linux kernel 2.6.18. 
   *   2nd snapshot is called sm-pre-ftw for security management only installations using Linux kernel 3.10 
##### NOTE: The Ansible playbooks must be execute from within cpAnsibleDemo directory as there is a ansible.cfg located there.
##### NOTE: The installation requires user interaction twice when the Gaia installation boot process starts, (as it starts twice).
##### NOTE: The VM template ip is default configured to 172.27.254.3 this can be changed by editing the tempate_ip line var in cpAnsibleDemo/group_vars/all


```
##### NOTE: The Ansible playbooks must be execute from within cpAnsibleDemo directory as there is a ansible.cfg located there.
cd cpAnsibleDemo
ansible-playbook initialise_demo_env.yml
# Next step is optional and will download the demo webserver used in demo deployment mode
ansible-playbook download_and_install_demo_web_server.yml
```

Dependencies
------------
The demo is currently using vmnat8 ip-range 172.27.254.0/24 this can be adjusted by changing the IP-addresses in the following files
```
vi cpAnsibleDemo/group_vars/all
vi cpAnsibleDemo/group_vars/mds_var.yml
vi cpAnsibleDemo/group_vars/sg_var.yml
vi cpAnsibleDemo/group_vars/smsg_var.yml
vi cpAnsibleDemo/group_vars/sm_var.yml
```

Execution of Playbooks
------------
##### NOTE: The Ansible playbooks must be execute from within cpAnsibleDemo directory as there is a ansible.cfg located there.

----------------
```
cd cpAnsibleDemo
ansible-playbook sm.yml                 # Build Security Management, deploy a gateway, configure policy and install that on the security Gateway (execution tine ~20 min)
ansible-playbook smsg.yml               # Build Security Management and Gateway (Stand Alone), configure policy in install that on smsg (execution tine ~15 min)
ansible-playbook mds.yml                # Build MDS with 4 domains,  deploy a gateway, configure policy in domain nordics and install that on the security Gateway (execution tine ~30 min)
ansible-playbook SecurityCheckup.yml    # Build Security Checkup configuration on physical appliance (Work In Process - not there yet)
```
All the above playbooks can be executed in demo deployment mode by using "--tags" switch
 * The tag "prepp-demo" will prepare the environment by deploying the management components only.
 * The tag "sg-demo" can be executed after the preparation phase.
 * This tag will demonstrate an unattended Check Point gateway deployment and launch a webserver that is protected behind that gateway
```
cd cpAnsibleDemo
ansible-playbook sm.yml --tags "prepp-demo" 
ansible-playbook sm.yml --tags "sg-demo" 
ansible-playbook smsg.yml --tags "prepp-demo" 
ansible-playbook smsg.yml --tags "sg-demo" 
ansible-playbook mds.yml --tags "prepp-demo" 
ansible-playbook mds.yml --tags "sg-demo" 
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
