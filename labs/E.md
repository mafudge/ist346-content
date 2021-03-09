# Lab E - Workstations and Clients

## Learning Objectives

In this lab you will:

- Learn how to install and configure software on a Ubuntu Linux system with `apt`  
- Learn how to automate tasks to multiple computers using `Ansible` as part of systems management. 

### Lab Setup At A Glance

In this lab, we will have one server computer, running Ansible, and 5 identical workstation computers. We will use the docker-compose `scale` option to create 5  containers from the same workstation image. The IP Addresses of the workstations and server are included in the diagram for reference. 

```
+-------------+
| workstations|----+
| 172.44.1.2  |    |
| through     |    |
| 172.44.1.6  |    |
+-------------+    |
                   |   +---------------+     ++++++++++++++
                   +---| host computer |-----|  Internet  |
                   |   +---------------+     ++++++++++++++
+--------------+   |
|   server     |---+
| 172.44.1.254 |
+--------------+
```

## Before you begin 

### Prep your lab environment. 

This lab prep's a little different as we will be scaling the workstation image to use 5 containers.

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-E` folder:  
`PS ist346-labs> cd lab-E`
5. Start the lab environment in Docker:  
`PS ist346-labs\lab-E> docker-compose up -d --scale workstation=5`
6. Verify your server and 5 workstation are up and running:
`PS ist346-labs\lab-E> docker-compose ps`  
You should see labE_server and labE_workstation_1 through _5 up and running.

## Part 1: Package management with apt

The `apt` packaging system allows a user to manage software packages on a Linux system. Apt is a wrapper around the `dpkg` packaging system which is used for managing software on Debian Linux based distributions such as Ubuntu Linux.

Let's issue some commands to see how the `apt` package system works.

1. First let's connect to the Linux console on  our `server` container:  
`PS ist346-labs\lab-E> docker-compose exec server bash`  
You will now see the bash command prompt `root@server:/#`
1. From the bash prompt, let's update the package database:  
`root@server:/# apt-get update`  
This updates the list of packages from the available repositories.
1. Let's install the package `nethack-console` which is a text-based dungeon crawl game. To install, type:  
`root@server:/# apt-get install -y nethack-console`  
The `-y` will confirm the installation so that you don't have to type `Y` to continue.
1. With the game installed, let's run it!  
`root@server:/# nethack-console`  
Sadly, it does not run... displaying a `bash: nethack-console: command not found` message. We know it was installed so where is it?
1. So how can we find out what was installed and where it was installed? We need to fall back to the `dpkg` utility for this:  
`root@server:/# dpkg -L nethack-console`  
This command will list `-L` all of the files in the `nethack-console` package.  
There's a few files in the package but the last one is the actual program itself:  
`/usr/games/nethack-console` we know this because by *convention* Linux installs games to `/usr/games`.
1. Armed with this new knowledge, let's run our game by qualifying the full path to it:  
`root@server:/# dpkg -L /usr/games/nethack-console `   
This time the game runs! You see the prompt:   
`Who are you?`  
Press `CTRL` + `c` to exit the game. (Hey, play games on your own time! hehe)
1. What's installed? Want to see a list of packages which are installed? Try:  
`root@server:/# apt list --installed `
1. Too much information too soon? Let's pipe the output to `grep`. For example, let's look for all packages with `net` in them:  
`root@server:/# apt list --installed  | grep "net"`  
You should see our new favorite package, nethack console among some other packages.
1. What if you want information about a package? Type:  
`root@server:/# apt show nethack-console`   
In the output you'll see the description:  
`Description: dungeon crawl game - text-based interface`
1. How do we list all of the available packages? Easy. Type:  
`root@server:/# apt list`  
Ooh. Too much information! 
1. Let's pipe this to `less` so we can scroll through the output with our arrow keys.  
`root@server:/# apt list | less`  
Yikes! the `less` command is not found? 
1. Well we know how to fix that! Install it with `apt`!!!:   
`root@server:/# apt-get install -y less`
1. Now, let's pipe it to `less`:  
`root@server:/# apt list | less`   
Scroll down until you find `alpine` then press `q` to quit `less`.
1. Let's learn about `alpine`, try:  
`root@server:/# apt show alpine`   
Ooh that's interesting. It's an email client! 
1. Finally let's uninstall our game. (No worries you can always install it again later)   
`root@server:/# apt-get remove nethack-console`   

Okay we learned how to install packages on Linux systems, but how would you do this on 200 Linux systems without placing your hands on 200 keyboards? Read on to find out!

## Part 2: Using ansible to manage your systems

In this next part, we will use `Ansible` to manage the 5 workstations on our network. What is Ansible? Simply put, it is a systems management automation engine. It allows you to easily perform tasks on remote computers such as changing configuration files, installing software and running programs.

I highly recommend watching this 3 1/2 minute video overview of ansible, from Lynda.com    
[https://www.linkedin.com/learning/learning-ansible-2020/an-introduction-to-ansible?u=76816226](https://www.linkedin.com/learning/learning-ansible-2020/an-introduction-to-ansible?u=76816226) 

NOTE: You will need to log-in with your NetID and Password.

### Setting up Ansible

To make the lab run smoothly we've setup most of ansible for you. Typically to do this you will need:

- Ansible installed on at least one computer (in this lab, it's been setup for you on the `server`)
- python 2.7 or higher + ssh setup on each computer to be managed by ansible (each `workstation` has that configured for you in this lab). As usual, the password to ssh into the hosts is `IST346`

To help you fully understand the power and flexibility of Ansible we will pretend our 5 workstations are divided up amongst 2 departments:

- workstations 1-3 are in the `it` department
- workstations 4-5 are in the `sales` department

We can configure this through Ansible's `hosts` file located at `/etc/ansible/hosts`, let's do this now:

1. First we need to accesss the Linux console of the `server`, type:  
`PS ist346-labs\lab-E> docker-compose exec server bash`  
1. You should now see the familar Linux Bash prompt: `root@server:/#` from this prompt, let's edit the Ansible hosts file:  
`root@server:/# nano /etc/ansible/hosts`
1. This will bring up the file in the `nano` text editor. Add the following lines to the bottom of the file:

```
[it]
lab-e_workstation_[1:3]

[sales]
lab-e_workstation_[4:5]
```

When you are finished editing the file press `CTRL` + `x` and when asked to save modified buffer press `y`, and press `ENTER` to keep the name file name.

### Testing our setup

Let's test our setup by pinging machines:

1. Type the following to make the workstations in the `sales` department ping the `ischool.syr.edu` server two times.  
`root@server:/# ansible sales -k -m shell -a 'ping -c 2 ischool.syr.edu' `  
NOTE: you will be prompted for the `root` password for the workstations, as you may recall, its `IST346`. There is a way to execute these commands without the password, but that's for another lab and another time. ;-)
1. When the command completes you should see output like this:  
`lab-e_workstation_5 | SUCCESS | rc=0 >>`   
`lab-e_workstation_4 | SUCCESS | rc=0 >>`   
Along with the output of the ping itself. Let's break down the `ansible` command:  
`-k ` prompts for the root password (root because that is who we are currently logged in as)  
`-m shell` uses the shell module. Ansible has many modules to perform a variety of tasks.  
`-a` allows us to specify the specific module arguments. This case, the `ping` command.  

### The Ansible ping module

Ansible includes a ping module. This is not the same as the `ping` command. This module verifies that the host is capable of being managed by Ansible.  

1. Let's ping all the workstations:  
`root@server:/# ansible all -k -m ping `   
Again, enter the root password of `IST346`
1. This time every workstation replies back `pong` letting you know that it can respond to ansible commands.

### Installing software with Ansible the wrong way.

Combining what we learned in the previous part of the lab, you'd probably figure you can use the `shell` module to install software with ansible. 

1. For example, to install `mc` on the `sales` computers, type:
`root@server:/# ansible sales -k -m shell -a 'apt-get install -y mc' `  
1. Boy that's a lot of output. Did it work? I assume so, but one cannot be sure. In addition if you repeat the command:  
`root@server:/# ansible sales -k -m shell -a 'apt-get install -y mc' `   
You can clearly see in the output that it didn't install it again, but it would be difficult to process that via a computer.

### Idempodentency is the name of Ansible's game

One advantage of Ansible modules is they ensure **idempotence** - that we can run the same tasks again and again without changing the final results. This is so important in systems management where you are often changing the files on a computer or the contents of a single file. 

Let's see this in action.

1. Let's perform the same install with the `apt` module:  
`root@server:/# ansible sales -k -m apt -a 'pkg=mc state=present update_cache=yes' `  
First what does this command do?  
`state=present` requests that the `pkg=mc` be installed on the workstation.  
`update_cache=yes` requests that an `apt-get update` be performed before the install (should the install need to take place to begin with)  
More importantly, the output is a lot different this time and much easier to read and understand 'what happened'. In fact you can see: `"changed": false` to indicate that no action was taken.
1. Now let's perform the same operation on `all` workstations:  
`root@server:/# ansible all -k -m apt -a 'pkg=mc state=present update_cache=yes' `   
This time you'll get a lot more output. If you scroll through the output you'll notice that `"changed": false` for workstations 4 and 5 (sales department), but `"changed": true` for workstations 1 and 3 (the it department). Idempotency!

### Ansible Playbooks

This command line stuff is great, but what if the single "change" you need to make requires several steps? We could just issue each Ansible sequentially but I'm sure there's a better way, correct? Well, this is the purpose of an Ansible Playbook. The playbook is a file which can run multiple Ansible tasks in addition to providing some additional configuration common across all the commands.

For example let's assume we need to run a Ruby program on all the computers in the `it` department. We need to install the ruby programming language to run the program, but we don't want to leave it on the system after the program runs (let's say for security purposes.) 

Our playbook would look something like this:


```
- hosts: it
  tasks:
    - name: Install Ruby
      apt:
        name : ruby
        state: present
        update_cache: yes
    - name: Run The Ruby Program
      shell: "ruby -e 'puts \"hello from ruby\"'"
    - name: Uninstall Ruby
      apt:
        name: ruby
        state: absent
        update_cache: yes

```

Let's use this playbook.

1. First download the playbook file to the server:  
`root@server:/# curl -o ruby.yml -L https://raw.githubusercontent.com/mafudge/ist346-labs/master/lab-E/ruby.yml`  
This will download the `ruby.yml` file from github to your computer.
1. Check to make sure the file is there:  
`root@server:/# cat ruby.yml`  
The playbook file output should be similar to code you see above.
1. Let's execute the playbook!  
`root@server:/# ansible-playbook -k ruby.yml`   
The playbook output look something like this:  

```
PLAY [it] **********************************************

TASK [Gathering Facts] **********************************************
ok: [lab-e_workstation_1]
ok: [lab-e_workstation_2]
ok: [lab-e_workstation_3]

TASK [Install Ruby] **********************************************
changed: [lab-e_workstation_2]
changed: [lab-e_workstation_3]
changed: [lab-e_workstation_1]

TASK [Run The Ruby Program] **********************************************
changed: [lab-e_workstation_2]
changed: [lab-e_workstation_3]
changed: [lab-e_workstation_1]

TASK [Uninstall Ruby] **********************************************
changed: [lab-e_workstation_1]
changed: [lab-e_workstation_2]
changed: [lab-e_workstation_3]

PLAY RECAP **********************************************
lab-e_workstation_1        : ok=4    changed=3    unreachable=0    failed=0
lab-e_workstation_2        : ok=4    changed=3    unreachable=0    failed=0
lab-e_workstation_3        : ok=4    changed=3    unreachable=0    failed=0
```

## Tear down

This concludes our lab. Time for a tear down!

1. If you are at the `root@server:/#` prompt, type:  
`root@server:/# exit`   
To return to the PowerShell prompt.
1. From the PowerShell prompt, type:
`PS ist346-labs\lab-E> docker-compose down`  
to tear down the docker containers used in this lab.


## Questions 

1. What is apt?
1. What is the purpose of the command `apt-get update` ?
1. Write a command to uninstall the package `bar` ?
1. What is the command to list the files installed by the package `chicken` ?
1. What can you do when the list of files is too large for your screen?
1. What is Ansible?
1. Explain idempotence and why is it important in systems management.
1. Explain the purpose of a playbook file. What are its key advantages?
1. Write a command to use `apt` to install `peachtree` onto computers in the `accounting` department (an Ansible hosts label).
1. What is the purpose of the Anisble ping command? 
