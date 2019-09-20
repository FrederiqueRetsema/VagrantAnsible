# VagrantAnsible
Setup and scripts for Ansible with both Windows and Linux servers

This repository contains scripts to install VM's using Vagrant and to setup an environment using Ansible. By following along, you will first install a very basic environment, and by using Ansible this environment will grow. There are two ways to learn:
1) To go along with this description, see what is happening, enhance the scripts that are provided.
2) To read the solution and figure out how this is done and why it is done this way.
It's op to you how you want to learn. When you want to follow along, don't look at the solution but try to follow the guidance in this README.md file.

Table of contents:  
1. Installation and configuration  
1.1 Installation of Vagrant and VirtualBox  
1.2 Installation of the VM's  
1.3 Windows licence expired  
1.4 Loging on to the Linux machines using Vagrant  
1.5 Using Vagrant to stop and start the VM's   
1.6 Install and configure Ansible on Linux  
2. Using Ansible  
2.1 Using Ansible ad-hoc commands on Linux  
2.2 Ansible playbooks on Linux  
2.2.1 file module 
2.2.2 lineinfile module  
2.2.3 Exercise: Time to create a playbook of your own
2.2.4 Using multiple commands (modules group, lineinfile, user)  
2.2.5 Excercise: add a user in the sudoers group  
2.2.6 Excercise: yum update  
2.2.7 Excercise: use yum to install snmp  
2.2.8 Excercise: Sending and executing a script on other nodes  
3. Windows
3.1 Installation
4. Useful links

# 1. Installation and configuration 
## 1.1 Installation of Vagrant and VirtualBox
First, we have to install Vagrant and VirtualBox: use a laptop where you have full administrator rights. 

Install VirtualBox and Vagrant:
- VirtualBox: https://www.virtualbox.org/wiki/Downloads
- Vagrant: https://www.vagrantup.com/
- VirtualBox Guest Editions: https://download.virtualbox.org/virtualbox/6.0.12/VBoxGuestAdditions_6.0.12.iso

## 1.2 Installation of the VM's
Clone this repository:

`git clone https://github.com/FrederiqueRetsema/VagrantAnsible.git`

Go to the directory VagrantAnsible, and type:

`vagrant up`

This will bring up two CentOS7 VM's (generic/CentOS7) and one Windows 2016 VM (jacqinthebox/windowsserver2016). The CentOS7 box is provided by https://roboxes.org/, a project that distributes many open source OS'es for Vagrant and Docker. The Windows 2016 box is the evaluation version of Windows 2016 Server edition, provided by jacqinthebox. This  is an end-user of Vagrant. The Windows-box is downloaded more than 6.500 times, the box is released about two years ago.

There are no post installation scripts, we will use Ansible for the configuration of (all) the nodes.

## 1.3 Windows licence expired
You can log on in the Windows system by selecting the Windows-VM-window and choosing input > keyboard > Insert ctrl-alt-del. You will see two possibilities: Administrator and Vagrant User. Choose for Vagrant User and login with the password vagrant.  

Before we can start to do nice things with Ansible, we first have to get rid of a nasty little thing in Windows: we have an evaluation license, but the license is expired when we log in the first time. On the desktop of the Vagrant-user is an icon with a commandfile that is called "extend-trial.cmd". Right-click on this icon and run this script as administrator. After rebooting the VM, you will have a trial period of 180 days.

## 1.4 Loging on to the Linux machines
You can logon to the Linux machines by going to a command prompt and typing `vagrant ssh control` or `vagrant ssh linuxSlave`. There are two users by default: vagrant (password: vagrant) and root (which you can sudo into).


## 1.5 Using Vagrant to stop and start the VM's 
Both on Windows and on Linux, there is a root directory vagrant that is a reference to the directory where the Vagrant file that we just used resides. 

Though it is possible to stop and start the VM's from VirtualBox, it is faster to stop the machines by using

`vagrant halt`

to stop the VM's and  

`vagrant up` 

to start them again.

## 1.6 Install and configure Ansible on Linux
Ansible has excellent documentation, you can see the latest documentation on https://docs.ansible.com/ansible/latest/ . 

Please mind, that when you use google to find a solution for Ansible and you come to this website, the version number in the upper left might not be the latest/right version.

From the documentation, you can go to the installation guide (left menu). The commands below are all copied from that page. 

Use the following command on the control node:

`sudo -i`  
`yum install ansible -y`

You don't need to install ansible on the linux slave node: ansible uses ssh to connect to other linux machines.

By default, the CentOS7 machines that we use, don't allow ssh connections with passwords. So let's change that to make it easier to send the key from the control node. Log on to `both` the linuxSlave and control and type:  
`sudo -i`  
`vi /etc/ssh/sshd_config` (change the line "`PasswordAuthentication no`" to "`PasswordAuthentication yes`"). After that, reboot the service:  
`systemctl restart sshd`

Let's create a user on both the control and linuxSlave nodes and make it possible for the control node to log on to the slave node. Login to the slave node and type:

`sudo -i`  
`useradd -m -G wheel ansible`  
`passwd ansible` (create a password that you can remember)  

Log on to the slave node, and look at it's IP address:   

`ip addr show`

Mind, that the 10.x.x.x-network address is always 10.0.2.15, so don't use this address
(the control node will send packets to itself instead of sending them to the linuxSlave node). Use the 172 network-address instead.

On the control node, add this address in the /etc/hosts table:  
`sudo vi /etc/hosts`  and go to the end of the file,
- add (for example) `172.28.128.4 linuxSlave` 
- add `127.0.0.1 control`

Logon to the control node and type:

`sudo -i`  
`useradd -m -G wheel ansible`  
`passwd ansible` (create a password that you can remember)  

On the control node, continue with creating a key for the ansible node:

`su ansible`   
`cd`  
`ssh-keygen` (defaults are fine)

Now copy the key to both the linuxSlave and control (to make it possible to change the control node itself using ansible). On the control node:  
`su ansible`  
`ssh-copy-id ansible@linuxSlave` (type yes to add the key of the linuxSlave to the known hosts, type the password of ansible on linuxSlave after that)  
`ssh-copy-id ansible@control` (type yes to add the key of control to the known hosts, type the password of ansible on control after that)

Try to log on to the slave node, you shouldn't be asked for a password:    
`ssh ansible@linuxSlave`  
`exit` (to go back to control)

Try to log on with ssh to the control node, you shouldn't be asked for a password:  
`ssh ansible@control`  
`exit` (to come back)

`exit` (to go back to the root user on the control node)

Now the ssh connection works, we can add the linuxSlave to the hosts file of ansible:
`cd /etc/ansible`  
`vi hosts` 

Add the following lines:  
`[linux]`  
`control`  
`linuxSlave`  

If you want to, change the line `PasswordAuthentication` back to `no` in the file /etc/ssh/sshd_config on the linuxSlave and control machines.

Windows will not work (yet), we will first start with Ansible on Linux.

# 2. Using Ansible

## 2.1 Using Ansible ad-hoc commands on Linux
The first test is to see if Ansible is installed and configured correctly:  

`su ansible`  
`ansible linux -m ping`  

This will send a simple ping (via ssh, not via ICMP) to all the nodes that are part of the linux group. You will get the following result:  

    control | SUCCESS => {  
        "ansible_facts": {  
            "discovered_interpreter_python": "/usr/bin/python"  
        },  
        "changed": false,  
        "ping": "pong"  
    }  
    linuxSlave | SUCCESS => {  
        "ansible_facts": {  
            "discovered_interpreter_python": "/usr/bin/python"  
        },  
        "changed": false,  
        "ping": "pong"  
    }  

ansible is the name of the tool that can simply send one command to many nodes. You can also use the shell:  

`ansible linux -m shell -a "ls -a"`    

    control | CHANGED | rc=0 >>  
    .  
    ..  
    .ansible  
    .bash_history  
    .bash_logout  
    .bash_profile  
    .bashrc  
    .ssh  
    .viminfo  
  
    linuxSlave | CHANGED | rc=0 >>  
    .  
    ..  
    .ansible  
    .bash_history  
    .bash_logout  
    .bash_profile  
    .bashrc  
    .ssh  

When you follow along, you see that the letters of the output have different colors: the ping command cannot change anything, so it will always return green output, because nothing on the host is changed. Ansible cannot determine if the shell has changed something - or not. That depends on the command that is given as a parameter (the -a switch). So Ansible will
be careful and assume that something might have changed.

It is also possible to use ansible to send a command to one host:  

`ansible control -m shell -a "tail .viminfo"`  

    control | CHANGED | rc=0 >>  
            "       6       0  
  
    > /etc/host  
            "       1       0  
  
    > /etc/ansible/hosts  
            "       46      0  
            ^       46      1  
            .       46      0  
            +       46      0  

There is a nice module to get information from other nodes. Try this:  

`ansible linuxSlave -m setup`  

You will get a lot of information about that node. You can use a filter to get less, use the labels you saw in the output of the previous command, for example:  

`ansible linuxSlave -m setup -a "filter=*ansible_memory_mb*"`

## 2.2 Ansible playbooks on Linux

### 2.2.1 file module
It is nice to be able to send one command to one node, or even to two, as we have seen before. It is much better when we can send multiple commands to multiple nodes. This can be done using ansible playbooks. Playbooks are yaml files, so the extension of playbooks is yml.

You will find examples of playbooks in the script directory in the repository. Go, in the clone VM, to /vagrant/scripts and look at the file file.yml:

        ---  
        - hosts: linuxSlave  
          tasks:  
          - name: touch file in homedir    
            file:   
              name: ~/file_in_homedir   
              state: touch

The three dashes on the first line are mandatory, they indicate that this is a yml-file. On the second line, we indicate the hosts to which this code should be send. In this case, we choose to send it only to the linuxSlave.  

Under the hosts are the tasks that should be executed. In this case, we just do one thing: we touch a file. After the name (which is mandatory), the name of the module is given. In this case, the modulename is file. Under the module name are the parameters, in this case the filename and the command to touch the file. 

Now it's time to execute this playbook:  

- Type  
`ansible-playbook file.yml`  (and press Enter)

- Check, with one of the previous commands, that this file has been created on the linuxSlave node.  
- When it works, change the playbook to touch both the linuxSlave node and the control node (hint: use the group name).  
- The file module can do much more. Change the yml-file to delete the file_in_homedir file. See the documentation for attributes: https://docs.ansible.com/ansible/latest/modules/file_module.html#file-module .
- You can use this module also for changing the owner of a file, or the permissions.

### 2.2.2 lineinfile module 
Let's add an alias for the ansible user (file: lineinfile.yml):

    ---
    - hosts: linuxSlave
      tasks:
        - name: Add alias to .bash_profile 
          lineinfile:
             path: /home/ansible/.bash_profile
             state: present
             line: "alias ch='cat /etc/hosts'"

Use this script to add a line to the .bash_profile of the Ansible-user. Like last time, the first thing the playbook does is gather facts about the nodes. This line is in green, so nothing has been changed by the playbook. Under our comment "Add alias to .bash_profile", there is a yellow line: there was no alias to cat /etc/hosts yet, so this line has been added.

- Execute the playbook a second time: you will see that the line under "Add alias to .bash_profile" is green: the alias was already present, so the playbook didn't have do anything. This is an important rule: every line in the playbook should be made in a way that it can be run multiple times, where only the first time something is changed. This makes it possible to run playbooks for example every hour, or every 15 minutes to enforce the changes that are in the playbook.
- Logon to the linuxSlave host as the ansible user, and try `ch`. Does the alias work?
- With `state: absent` the line can also be deleted. The lineinfile can do much more: see the documentation: https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html#lineinfile-module .

### 2.2.3 Exercise: Time to create a playbook of your own
The nodes in our environment are delivered out-of-the-box. When you look with history to the previous commands, there is no date/time on which moment these commands were given. Wouldn't it be nice if this information would be available? See this webpage for an information how to set this up: https://www.cyberciti.biz/faq/unix-linux-bash-history-display-date-time/ 

- Hint: Try this out on the vagrant user first, so you know what the commands should be
- Hint: when you see the line in ~/.bash_profile, but logging of and logging on doesn't change anything, then execute the .bash-profile file by hand (`. ./.bash_profile`) 
- The solution is in the solution folder (exercise.yml)

### 2.2.4 Using multiple commands (modules group, lineinfile, user)
You can also give multiple commands in one playbook. Let's give an example: let's take care that people cannot log on as root anymore. But first, let's show that it is (still) possible to logon as root:

- Login on linuxSlave
- `sudo -i`
- `passwd root`  (change the password into something you can remember)
- `exit`
- `su root` (give the password -> this is succesful)

Let's fix this. First, we will create an extra group: sudoers. We will add the new group sudoers to the /etc/sudoers file with the right permissions. And, last but not least, we will disable the root user account. But wait: when we would do it directly like this, we might screw up our node because no-one might be able to log in as root again. So let's do one thing at a time and test each step before creating the whole script.

First of all, let's create the group (group.yml):

    ---
    - hosts: linuxSlave
      become: yes
      tasks:
        - name: "Create group sudoers"
          group:
            name: sudoers
            state: present

Did you notice the line under hosts? The `become: yes` line will elevate our account to root. Execute this playbook and check if the group has been created on the linuxSlave-node:

`ansible-playbook group.yml -K`  
`ssh ansible@linuxSlave`  
`cat /etc/group`

You need the -K parameter on the commandline to ask for the ansible password on the remote node to become a priviledged user.

We know that we have to add a line to the sudoers file. When we screw this up, no-one will be able to use sudo. There is a solution for this: let's add the line we want to add to the sudoers file to a temporary file (disable_root_step_1.yml):

    ---
    - hosts: linuxSlave
      become: yes
      tasks:
        - name: "Create group sudoers"
          group:
            name: sudoers
            state: present
        - name: "Add line to the sudoers file, allow all commands, require a password"
          lineinfile:
             path: /home/ansible/extra_sudoers_line.txt
             state: present
             create: yes
             line: "%sudoers\tALL=(ALL)\tALL"

Try this out, this should create a file in the Ansible home directory. Use the sudoers command to read the extra line in the sudoers file:  
`ssh ansible@linuxSlave`  
`cat extra_sudoers_line.txt`  
`sudo -i`  
`visudo`

Within the /etc/sudoers file, go to the last line (press `G`) and type `:r /home/ansible/extra_sudoers_line.txt` . This will add the content of the file /home/ansible/extra_sudoers_line.txt to the sudoers file. Save the file (\<esc\> :wq ). 

When there is an error on this line, you will get a warning. Go back to the sudoers file, change the last line (`G`, `i`) and change the line so that the sudoers file is correct. Save the sudoers-file (\<esc\> :wq ) until the file is correct. When that is the case, change the line: "%sudoers\tALL=(ALL)\tALL" in the original playbook to the correct line. Remove the file /home/ansible/extra_sudoers_line.txt and start the checks again (see above), it shouldn't be a problem now.

Remove the last line (the line we added ourselves) from the sudoers file: use the command:  
`visudo`

Then press `G` to go to the last line, press `dd` to remove the last line. Then save and quit (\<esc\> :wq).
`exit` (to leave the root account)  
`exit` (to leave the ansible account on the linuxSlave node)

We know that the line we want to add is correct, so we can change the path from `/home/ansible/extra_sudoers_line.txt` to `/etc/sudoers`. The `create: yes` can be removed: the sudoers file should exist (and the playbook should fail if it cannot be found). The new playbook is now (disable_root_step_2.yml):

    ---
    - hosts: linuxSlave
      become: yes
      tasks:
        - name: "Create group sudoers"
          group:
            name: sudoers
            state: present
        - name: "Add line to the sudoers file, allow all commands, require a password"
          lineinfile:
             path: /etc/sudoers
             state: present
             line: "%sudoers\tALL=(ALL)\tALL"

Check on the linuxSlave node that the line has been added. Now, we can disable the root account. You see that the state of root is still present, but the password is locked. (disable_root_step_3.yml)

    ---    
    - hosts: linuxSlave  
      become: yes  
      tasks:  
        - name: "Create group sudoers"  
          group:
             name: sudoers
             state: present
        - name: "Add line to the sudoers file, all commands allowed"
          lineinfile:
            path: /etc/sudoers
            state: present
            line: "%sudoers\tALL=(ALL)\tALL"
        - name: "Disable the root user"
          user:
            name: root
            state: present
            password_lock: yes

Execute this playbook, and check via the following commands that it has become impossible to become root:

- Login on linuxSlave
- `sudo -i` (this still works!!)
- `exit`
- `su root` (give the password -> this is NOT succesful ANYMORE)

Documentation:
- for the group module: https://docs.ansible.com/ansible/latest/modules/group_module.html#group-module
- for the user module: https://docs.ansible.com/ansible/latest/modules/user_module.html#user-module

BTW: it would also have been possible to add the users to the existing wheel group. In that case, it should have been sufficient to disable the root account. You might want to distinguish between operators that can become root (in the wheel group) and end-users that can become root (and restrict their access to change the last ALL in the commands that they are allowed to execute).

### 2.2.5 Excercise: add a user in the sudoers group
With the user module you can also add new users. Excercise: add user jill via a playbook to the linuxSlave and give her sudo rights via our newly created group. Change (as root) the password of Jill, su to Jill and try to sudo into the root account.

- You will find the solution in the solutions directory, filename jill.yml

### 2.2.6 Excercise: yum update
You have seen some examples of playbooks. Time to go ahead and search for yourself. Use the main documentation page: https://docs.ansible.com/ansible/latest/index.html, use the search box in the left menu and create a playbook to do a yum update all linux computers.

 - You will find the solution in the solutions directory, filename yum_update.yml

### 2.2.7 Excercise: use yum to install snmp
Search via `yum search snmp` to find the package that you can install to implement SNMP, use the yum module from ansible to install it.

- Hint: don't think too difficult: you just need two parameters for yum to install the snmp package.
- Before running the playbook, check that nothing has been installed for snmp yet (`yum list installed | grep snmp`)
- When the playbook works, check if the right package is installed.
- You will find the answer in the solutions directory (yum_snmp.yml)

### 2.2.8 Excercise: Sending and executing a script on other nodes
Sometimes is can be useful to create a script that you can execute on other nodes. We will create a script on the control node and send and execute it to the linuxSlave node.

Create the following script on the control node:  
   
    echo "Name of this node: $HOSTNAME"
    echo "SSH connection: $SSH_CONNECTION"
    echo "Time: `date`"

Check (on the control node) that this script works as expected. Create a playbook to implement this functionality. Use the `copy` module to send it to the remote nodes. Use the command `script.sh > output.txt` to execute this.

- check on the remote node that the file output.txt exists and has the right content. 
- The solution is in the solution folder (filename script.yml)
- In production environment, you will want to get a remote file back. Look at the fetch module for that. (script_and_fetch.yml) for an implementation.

# 3 Windows

## 3.1 Installation
In Vagrant, there are some problems with the downloaded Windows 2016 box in combination with the c:\vagrant drive. This is caused by a different version of the Virtual Box Guest Editions. Insert the iso via the window of the windowsSlave-machine (Machine > Settings > Storage). Click on the empty Cd-Rom under IDE Controller, and click on the blue disk right of Optical Drive. Now choose "Virtual Optical Disk File" and insert the iso that you downloaded in part 1 of this description. 

Log on on the windows Slave-node, install the VirtualBox Guest Editions (all default options are fine). When you installed the VirtualBox Guest Editions (and rebooted the Windows VM), you will be able to go to c:\vagrant and see the contents.

When you want to use Ansible on Windows, there are many options to setup your environment. Setup will also differ between various versions of Windows, see this link for more detail: https://docs.ansible.com/ansible/latest/user_guide/windows.html . 

In this section, we will install Ansible on Windows 2016, in a very fast (but insecure) way. The reason for this is that we are playing along in a development environment, in a temporary situation. The focus in our excercises is the ansible playbooks for Windows, not on the installation. 

Log on to the Windows 2016 node with the Vagrant user, and start a Powershell window. Use the following code to configure remoting for Ansible (c:\vagrant\windowsScripts\installation_configure_remoting_for_ansible.ps1):

    $url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
    $file = "$env:temp\ConfigureRemotingForAnsible.ps1"

    (New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

    powershell.exe -ExecutionPolicy ByPass -File $file

After that, use the following command to setup the WinRM listener:

`winrm quickconfig`

We will use basic authentication without encryption, so run the following in Powershell:

`Set-Item -Path WSMan:\localhost\Service\Auth\Basic -Value $true`  
`Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true`

Now, we need the IP-Address of the Windows node: type

`ipconfig`  

add the ip address to /etc/hosts on the control node, for example:  
- add `172.28.128.5 windowsSlave`

Now, we will configure the settings for the Windows-node in the /etc/ansible/hosts file. Add the following lines:

`[windows]`  
`windowsSlave`

`[windows:vars]`  
`ansible_user=vagrant`  
`ansible_password=vagrant`  
`ansible_connection=winrm`  
`ansible_winrm_transport=basic`  
`ansible_port=5985`

On the control node, we need to install extra software to allow Ansible to use winrm:

`yum install python2-pip`  
`pip install "pywinrm>=0.3.0"`

Now all is installed, we can do a simple ping, mind that the module has another name on Windows than on Linux:

`ansible windows -m win_ping`

## 3.1 Windows Updates
Let's look at the playbook for Windows Update (source: https://docs.ansible.com/ansible/latest/user_guide/windows_usage.html ):

    ---
    - hosts: windows
      tasks:
        - name: Windows Updates
          win_updates:
            state: installed
          register: update_result

        - name: reboot if necessary
          win_reboot:
          when: update_result.reboot_required

You see that we use two modules: win_updates, to check for (and install) the updates, and win_reboot to reboot the machine if necessary. The result of win_updates is passed to win_reboot via the register keyword and an intermediate variable update_result. 

You can use register to get the results for any module, and show the contents of this variable via the debug module:

    ---
    - hosts: windows
      tasks:
        - name: Windows Updates
          win_updates:
            state: installed
          register: update_result

        - name: show contents of update_result
          debug:
            var: update_result

        - name: reboot if necessary
          win_reboot:
          when: update_result.reboot_required

When you run this playbook, you may see the following output:


    [vagrant@control windowsScripts]$ ansible-playbook windowsUpdatesDebug.yml

    PLAY [windows] *********************************************************************************************************

    TASK [Gathering Facts] *************************************************************************************************
    ok: [windowsSlave]

    TASK [Windows Updates] *************************************************************************************************
    changed: [windowsSlave]

    TASK [show contents of update statement] *******************************************************************************
    ok: [windowsSlave] => {
        "update_result": {
            "changed": true,
            "failed": false,
            "failed_update_count": 0,
            "filtered_updates": {
                "1631a9c2-c045-4def-b6e7-dc206013dc97": {
                    "categories": [
                        "Definition Updates",
                        "Windows Defender"
                    ],
                    "filtered_reason": "category_names",
                    "id": "1631a9c2-c045-4def-b6e7-dc206013dc97",
                    "installed": false,
                    "kb": [
                        "2267602"
                    ],
                    "title": "Security Intelligence Update for Windows Defender Antivirus - KB2267602 (Version 1.301.1787.0)"
                },
                "a711f6a5-5bf3-4392-95d0-686f748789dd": {
                    "categories": [
                        "Updates",
                        "Windows Server 2016"
                    ],
                    "filtered_reason": "category_names",
                    "id": "a711f6a5-5bf3-4392-95d0-686f748789dd",
                    "installed": false,
                    "kb": [
                        "4103720"
                    ],
                    "title": "2018-05 Cumulative Update for Windows Server 2016 for x64-based Systems (KB4103720)"
                }
            },
            "found_update_count": 2,
            "installed_update_count": 2,
            "reboot_required": false,
            "updates": {
                "d0bf3ffb-d01a-4877-b5df-be5d48ff898c": {
                    "categories": [
                        "Update Rollups",
                        "Windows Server 2016"
                    ],
                    "id": "d0bf3ffb-d01a-4877-b5df-be5d48ff898c",
                    "installed": true,
                    "kb": [
                        "890830"
                    ],
                    "title": "Windows Malicious Software Removal Tool x64 - August 2019 (KB890830)"
                },
                "df51b003-58d6-42d6-ad9e-d81adaa1437e": {
                    "categories": [
                        "Security Updates",
                        "Windows Server 2016"
                    ],
                    "id": "df51b003-58d6-42d6-ad9e-d81adaa1437e",
                    "installed": true,
                    "kb": [
                        "4512574"
                    ],
                    "title": "2019-09 Servicing Stack Update for Windows Server 2016 for x64-based Systems (KB4512574)"
                }
            }
        }
    }
    TASK [reboot if necessary] *********************************************************************************************
    skipping: [windowsSlave]

    PLAY RECAP *************************************************************************************************************
    windowsSlave               : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

These playbooks can be found in the c:\vagrant\windowsScripts directory. 
- Run both playbooks. Mind, that WindowsUpdates can take a considerable amount of time.
- When you ran the second playbook, the Windows VM shouldn't be rebooted, due to the when-keyword.

# 3.2 Using Ansible to install software
Let's create a server. We need an IIS server, with specific features. We also make absolutely sure that, whatever happens, one specific feature will NOT be enabled. In Powershell, we would use Install-WindowsFeature, but in Ansable we have a module for that: win_feature.

You can use the Powershell-command Get-WindowsFeature to get the names of the features:

    ---
    - hosts: windows
      become: yes
      tasks:
      - name: Install IIS on this node
        win_feature:
          include_management_tools: yes
          name:
          - Web-Default-Doc
          - Web-Dir-Browsing
          - Web-Http-Errors
          - Web-Static-Content
          - Web-Http-Logging
          - Web-Stat-Compression
          - Web-Dyn-Compression
          - Web-Filtering
          - Web-Windows-Auth
          - Web-Net-Ext45
          - Web-Asp-Net45
          - Web-ISAPI-Ext
          - Web-ISAPI-Filter
          - Web-WebSockets
          - Web-Mgmt-Console
          state: present
      - name: Be sure that WebDAV is disabled
        win-feature:
          name: Web-DAV-Publishing
          state: absent

Let's change some configuration settings. First, we will allow feature delegation for anonymous, forms and windows authentication. You can find this in the IIS-tool in the properties of the IIS-server, under management, delegation, but we will do is script-wise. 

The first place to search for a solution is the Windows module: https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html . There are four iis modules, but these modules cannot be used to delegate features, so we have to use Powershell. I found the solution on the site of stackoverflow: https://stackoverflow.com/questions/35141373/toggle-iis-feature-delegation-with-powershell . When you give the value Allow, the value in the GUI is read/write. When the value is Deny, the value in the GUI is Read only. Mind, that the values Allow and Deny are case sensitive. For some strange reason, it doesn't seem to be possible to change the delegation of Forms authentication via Powershell. For our case this doesn't hurt, as we want to have the default of "Read/Write".

`Set-WebConfiguration //System.WebServer/Security/Authentication/anonymousAuthentication -metadata overrideMode -value Allow -PSPath IIS:/`

`Set-WebConfiguration //System.WebServer/Security/Authentication/windowsAuthentication -metadata overrideMode -value Allow -PSPath IIS:/`

Now, let's assume that we want to change the authentication of our site. The way it looks in the GUI is quite simple: we have four settings that we can either enable or disable:

- Anonymous Authentication
- ASP.NET impersonation
- Forms Authentication
- Windows Authentication

After some searching on the internet, I came to the following Powershell statements to change the Default Web Site: 

`Set-WebConfigurationProperty -filter /system.WebServer/security/authentication/AnonymousAuthentication -name enabled -value true -location "Default Web Site"`  
`Set-WebConfigurationProperty -filter /system.WebServer/security/authentication/WindowsAuthentication -name enabled -value false -location "Default Web Site"`  
`Set-WebConfigurationProperty -filter /system.web/identity -name impersonate -value false -PSPATH "IIS:\Sites\Default Web Site"`  

`# Stackoverflow: disable Forms https://stackoverflow.com/questions/37186386/powershell-script-that-switch-between-foms-authentication-and-windows-authentica`

`#$config=(Get-Webconfiguration system.web/authentication "IIS:\sites\Default Web Site")`  
`#$config.mode="Windows"`  
`#$config | Set-WebConfiguration system.web/authentication`  

`# Stackoverflow: enable Forms https://stackoverflow.com/questions/37186386/powershell-script-that-switch-between-foms-authentication-and-windows-authentica`

`$config=(Get-Webconfiguration system.web/authentication "IIS:\sites\Default Web Site")`  
`$config.mode="Forms"`  
`$config | Set-WebConfiguration system.web/authentication`  

Let's add this configuration to the previous Ansible script. Or even better: let me show how the first part (IIS > Management > Feature Delegation) works, and then you can try to do the same with the second part (IIS > Sites > Default Web Site). This first part is in c:\vagrant\windowsScripts (filename: iis_install_and_configure_1.yml):

      [...]   
      - name: Configure delegation
        win_shell:
          Set-WebConfiguration //System.WebServer/Security/Authentication/anonymousAuthentication -metadata overrideMode -value Allow -PSPath IIS:/
          Set-WebConfiguration //System.WebServer/Security/Authentication/windowsAuthentication -metadata overrideMode -value Allow -PSPath IIS:/

        - name: Configure delegation (IIS > Management > Feature Delegation)
          win_shell: |
            Set-WebConfiguration //System.WebServer/Security/Authentication/anonymousAuthentication -metadata overrideMode -value Allow -PSPath IIS:/
            Set-WebConfiguration //System.WebServer/Security/Authentication/windowsAuthentication -metadata overrideMode -value Allow -PSPath IIS:/

- You will see that the Default Web Site doesn't work when you don't change something to the code. Why would this be?
- Look at the parameters of the Powershell-command, how could we solve this?
- The answer is in the solution directory

# 4 Useful links
- List with all modules: https://docs.ansible.com/ansible/latest/modules/list_of_all_modules.html
- Modules per category:  https://docs.ansible.com/ansible/latest/modules/modules_by_category.html
- List with all Windows modules: https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html 



