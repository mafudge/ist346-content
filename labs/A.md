# Lab A - Understanding the lab environment

## Learning Objective

In this lab you will:

- Become familiar with the ITELL lab environment
- Learn how to download / update the lab files with `git`
- understand how to manage the lab environment with `docker-compose`

## Before You Begin

- You cannot complete these labs without a Docker and Git setup. 
  - If you are working in the ITELL environment (because you are enrolled in the IST346 course at Syracuse University) then you are ready to start this lab.
  - If you are working from your own personal computer, you must complete the software / setup prerequisites outlined here: 
  
  [https://github.com/mafudge/ist346-labs/blob/master/README.md](https://github.com/mafudge/ist346-labs/blob/master/README.md)


## Part 1: The ITELL vLab

NOTE: If you are running the labs from your personal computer, you can skip this part.

In this part you will become familiar with the ITELL (vLab). The ITELL vLab provides a logged in user with a means to access other computers remotely. Unlike remote lab, these computers are configured for very specific tasks, usually associated with a course. Because it would be wasteful to setup a physical computer for each students, ITELL vLab uses *virtual machines* (VM) which are virtual computers running on top of a physical computer. All of the virtual machines running on the host computer share its resources of CPU, RAM and Disk. 

Follow these instructions to connect to the ITELL vLab:  
[https://answers.syr.edu/x/ZwL-Aw](https://answers.syr.edu/x/ZwL-Aw)

Once you are connected, follow these instructions to navigate to your the Virtual Machine used in this course,  and Power It On, and access the Console: [https://answers.syr.edu/x/bQL-Aw](https://answers.syr.edu/x/bQL-Aw) 

For this class, the Virtual Machine (VM) we will use runs the Windows 10 operating system. The previous instructions explained how to power on your virtual machine and access the console (a.k.a. Screen / Keyboard and Mouse) for this virtual machine. 

By now, you're probably at a login window and wondering: How do I log on? Type:

**Username:** `LocalAdmin`
**Password:** `SU44orange!`

This will log you in to your virtual machine and bring you to the desktop. Here's where you'll stay to work on the labs.

NOTE: When you are done with the lab it's a good idea to power off your virtual machine. That way the host computers will reclaim resources for other students and their lab activities! The instructions to power off are here: [https://answers.syr.edu/x/bQL-Aw](https://answers.syr.edu/x/bQL-Aw) 

IMPORTANT: Practice accessing the ITELL environment, powering your VM on and off, opening the console and logging on. You will need to do this a few times each week so its important to get the process down.

## Part 2: The Command Line

Windows 10 has a nice, fancy UI, but we will ignore it in the course. If you really want to learn how to manage computers, automate tasks and orchestrate activities in the Cloud, you're going to need to master the *command line interface* (CLI). The command line is a text-based user interface to your computer. It allows you to issue commands such as "copy this file" or "pull down the latest code from this git repository". The real benefit of the command line is you can batch commands together in a script to automate tasks, which we will do in this course.

In this course we will be at the command line all the time. You might be at the command line in your Windows 10 VM, or on another computer.  All command lines do not look the same and this is a good thing. Your command line will include a *prompt* which helps you to figure out where you are.

### Always Know where you are!

Common command line prompts you will see in this course.

| What is it? | Which Operating System? | What does it look like? |
| ----------- | -------------- | -------------- |
| PowerShell Prompt | Windows Computers  | Begins with `PS` and ends with `>` For example: `PS C:\Users\LocalAdmin>` |
| Bash Prompt (as a user) | Linux Computers | Ends with a `$`, contains user and computer name Example `scott@servera:~$` |
| Bash Prompt (as a root) | Linux Computers | Ends with a `#`, usually contains user and computer name. Example `root@localhost:/#` |

Let's open the PowerShell prompt on our Windows 10 VM: Press `Windows Key + r` to open the run dialog. (Or you can click on the Windows Icon and type run) Type `powershell` and press the `ENTER` key. You should see a window with the PowerShell prompt: `PS C:\Users\LocalAdmin>`

NOTE: If you are on your own computer the prompt will look different but should still begin with `PS`, which is your indicator it's the PowerShell prompt.

IMPORTANT: You should practice accessing the PowerShell command prompt. You're going to be doing that a lot in the course!

## Getting the Lab Files

Let's cover how to get the lab files. These are stored in a *git repository*, which is a form of version control for code. This allows changes to be made and labs to be updated **while** you're doing the labs! That way if there's a mistake or improvement the change can be made and all the students can easily get the corresponding updates. 

### Cloning ###

The git repository for the course labs is `https://github.com/mafudge/ist346-labs.git` Let's copy it to your Windows 10 VM, which is called a *clone* in git nomenclature.

1. Open the PowerShell prompt.
1. Type: `git clone https://github.com/mafudge/ist346-labs.git` and press `ENTER` to submit the command. It will take a few seconds to copy the code locally.

IMPORTANT: You only need to clone the repository once! That's it! It now lives on your computer in a folder named `ist346-labs`

### Accessing the Repository folder

Cloning is a one-time deal, but you'll always need change the working directory to the repository folder.

1. Open the PowerShell prompt (if you haven't already.)
2. Type: `cd ist346-labs` to change the working directory to that folder.
2. You should notice the command prompt has changed. The end of it now looks like this: `ist346-labs>`
3. You can verify you're in the git repository by typing: `git status` it should say something like `on branch master` to let you know you're working in a folder which is being tracked by git. 

### Where are the labs?

The git repository has each lab in its own folder. For example this is `Lab A` and so the corresponding lab folder is `lab-A`. 

1. To see all the lab folders, type: `dir`
1. To change the working directory into one of the lab folders, type `cd lab-*N*` where N is the lab you wish to access. For example to change the working directory to this lab, the command would be: `cd lab-A`.  Type this now: `cd lab-A`

### Moving up a folder

1. When you are in a lab folder, your PowerShell prompt will end with this: `ist346-labs\lab-A>` to move up to the main repository folder, type `cd ..` try this now.
1. When you are finished your PowerShell prompt should once again end with `ist346-labs>`

### I need more labs!!! Updating your repository

Right now you'll notice that not all the labs are not present. That's because your professor is still working on them! Even if all the labs were released, there might be a need to update them to fix issues or make improvements. To do this we update the git repository:

1. From the PowerShell command prompt ending with `ist346-labs>` type `git pull origin master` to download the latest changes. 

## Your Typical Lab Setup Instructions

Ok. You know enough to be dangerous!  Let's walk through the typical setup instructions:

1. Open the PowerShell prompt.
2. Change the working directory folder to `ist346-labs`
3. Update your repository
4. Change the working directory folder to the lab in question e.g. `lab-A` for example.

Do you think you can do each of these commands? Practice them now with the `lab-A` folder. Close your PowerShell prompt and try it now!

## Docker-Compose Commands

Let's talk about the actual labs. Each lab consists of one or more computers networked together to demonstrate a concept. For example when covering web applications we might work with a database, web server and load balancer all within the comfy-confines of your Windows 10 VM.

How is this possible?  The technology which allows us to do this is called *containerization*, it's like virtualization, but more efficient with the resources. The container application we will use is called *Docker*. We will setup and tear-down complex computer scenarios easily as part of the labs in this course. 

First docker will download the images of the operating systems and applications we require for that lab. Then it will run the image allocating network, CPU, disk, and memory to it. At this point it's called a container.  
So containers are a lot like virtual machines, but offer several advantages:

- Setup and tear down are trivial.
- Containers are "stateless." When you tear them down you lose all changes.
- Containers don't depend on each other. So you don't need to complete one lab to start another. 

Let's take it for a spin:

### Lab Container Tour ###

1. Change into the `lab-A` folder from the PowerShell command prompt.
1. For this lab, the environment is just a simple Linux computer. To bring up the lab environment, type: `docker-compose up -d`
1. The **first time** you do this it's gonna take a bit. It needs to download the image files from the internet and set them up. When its done, you should see a green **done** in the output.
1. Let's see what's running: `docker-compose ps`. In the output you'll see the name `lab-a_ubuntu_1`. `ubnutu` is the name of the container and the `1` tells you there is one running instance of it. 
1. We can connect to the container by typing this command: `docker-compose exec ubuntu bash` 
1. This will bring up the Linux Bash prompt `root@ubuntu:/#` , indicating that you are now no longer in PowerShell land, but at the command prompt of your running Linux container!!!
1. Okay that's enough of that. Type `exit` to quit the Linux container and return to the PowerShell prompt.
1.  When you type `docker-compose ps` notice the environment is still running. It will remain running until we tear it down. 
1. To tear-down the lab containers, type: `docker-compose down`. This will remove the containers and their associated networks

IMPORTANT: You should always tear down the environment when you are finished to make sure that you free up resources! If you keep unused containers running your Windows VM will slow down, as there's only a finite number of resources on the host!

Practice these steps. You will be doing these often in future labs.

## Questions 

1. What is a virtual machine? What is a container? How are they different?
2. What is a command line interface? Command Prompt?
1. Typically how many times to you need to clone a git repository?
2. What command to your type to update the contents of your git repository?
3. What is the command to change the working directory to the git repository?
5. What command do you type to bring up the environment for the lab?
6. What command do you type to shut down the environment for a lab?
7. Why is it important to tear down the lab environment when you are done?