1.	Introduction 

This Lab in an introduction to Ansible for PowerVC / AIX .

In this lab, we will launch Ansible tasks from the control node running on Linux on Power, performing actions with Ansible and Openstack/PowerVC for creating a new AIX VM first. The VM will be visible on PowerVC. 

The second part of the lab is about automation on the AIX operating system: 
-	Modifying the root user characteristics
-	Setting filesystem size
-	Installing some prereq 
-	Installing Websphere Application Server

1.1.	Objectives 
The objectives of this lab are : 

•	Deploy an AIX  partition,  
•	Perfom operation on filesystems 
•	Install and configure Websphere Application server

Below is an architecture diagram describing the environment:  
  
  ![alt text](./img/Slide3.JPG)


2. Installing ansible openstack python modules

For ansible to be able to interact with PowerVC, you need to install openstacksdk.

Run this command to do so :

> pip3 install --user "openstacksdk==0.51.0" "python-openstackclient==5.4.0" dnspython dig

2.1. Create the openstack configuration file in your home directory

> vi clouds.yaml


``` yaml
clouds: 
   youcloudname: 
     auth: 
       auth_url: https://PowerVCIPaddress:5000/v3/ 
       domain_name: Default 
       username: YourUser 
       password: YourPassword 
       project_name: MyProject  
     verify: false
```
Of course you need to update PowerVC IP Address or hostname, user name , password and project name accordingly with you current setup. 

``` yaml
--- 
- hosts: localhost 
  gather_facts: false 
  tasks: 
  - name: adding ssh key to powervc 
    os_keypair: 
      cloud: idXXXXXXXX  # must refer to what is in clouds.yaml
      state: present 
      name: ansible_key_idXXXXXXXX 
      public_key_file: /home/idXXXXXXXX/.ssh/id_rsa.pub 
  - name: Create a new VM instance 
    os_server: 
      cloud: idXXXXXXXX   # must refer to what is in clouds.yaml
      name: vmidXXXXXXXX  # The VM Name you want to create 
      image: AIX72_40GB   # The image you want to use to create the VM/LPAR
      flavor: was         # AKA Compute Template in PowerVC
      key_name: ansible_key_idXXXXXXXX  # the key name ( Not mandatory )
      availability_zone: "Default Group" 
      nics: 
        - net-name: VL344 # The VLAN Name as in PowerVC
      timeout: 900 
      state: present 
    register: vm 
    
  - name: Showing newly assigned IP address 
    debug: 
      msg: the IP address is "{{ vm.server.public_v4 }}" 
        
  - name: Waits for SSH port 22 to open 
    wait_for: 
      host: "{{ vm.server.public_v4 }}" 
      delay: 5 
       port: 22
``` 

Let’s explain what we are doing here :

> os_keypair: let you upload your public ssh key to PowerVC, it will be used at deployment time so that you won’t need a password authentication.

> public_key_file: /home/idXXXXXXXX/.ssh/id_rsa.pub represents the ssh public key file. It has been created for you during the demo setup using ssh-keygen command. It is located in your home directory.

> os_server: Creates the VM on the cloud name found in clouds.yaml previously created file.

Note: We provided VMname, image name to use and VLAN to connect to

Once the VM is created, it is a good idea the get the IP address that PowerVC has automatically set. This is done with : debug:
wait_for: will wait for the VM to boot up and the ssh port to be open



> ansible-playbook mkvm.yaml -v

