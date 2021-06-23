# Lab B - Learning the linux command line

## Learning Objective

In this lab you will:

- Learn how to sign in to Lynda.com with your SU account.
- Learn the basics of the linux command line. 

## Before You Begin

### Prep your lab environment ###

This should be done before every lab! On the computer where your labs are located:

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-B` folder:  
`PS ist346-labs> cd lab-B`
5. Start the lab environment in Docker:  
`PS ist346-labs\lab-B> docker-compose up -d`

This will bring up an Ubuntu Linux container for you to use as part of the lab.

### Login to the Linux Container ###

You will need to login to the Linux container as user `scott` with password `IST346`. Once you do that you will be able to follow the videos which are part of the lab. The name of the linux container is `workstation`.

Procedure:

1. Connect the the Linux Container: 
`PS ist346-labs\lab-B> docker-compose exec workstation login`  
NOTE: The name of the Docker container is `linux` and the command we wish to execute on that container is `login`.
2. This will initiate a login prompt on the Linux container:  
`workstation login: `  
At the login prompt, enter the user `scott`
3. At the `password: ` prompt, enter the password `IST346`  
NOTE: All passwords in the labs are `IST346` unless stated otherwise.
4. You will now see a Linux Bash command prompt like this:   `scott@workstation:~$ `

You are ready to follow along with the lab on Lynda.com

### Logging in to Lynda.com with your Syracuse University Account ###

All Syracuse University students have accounts on Lynda.com, a leading technology learning platform for busy professionals. Here's the procedure for logging on: 

1. Nagivate your web browser to https://lynda.syr.edu 
2. This will prompt you with a Syracuse University login screen. Login with your Syracuse University **NetID** and **Password**. 
3. Once your are logged on you will see the Lynda.com home page. You can jump directly to the course here: [https://www.lynda.com/IT-tutorials/Learning-Linux-Command-Line/753913-2.html?org=syracuse.edu](https://www.lynda.com/IT-tutorials/Learning-Linux-Command-Line/753913-2.html?org=syracuse.edu) 

The entire course is about 2 hours long. We will only complete parts 2, 3 and 4 which should take you about 90 minutes. 

NOTE: You'll probably want to use a pair of headphones so you can listen to the audio of the video lessons in private.

**Okay! You're now ready to start the Lab!**

## Learning the Linux Command Line ##  

You should follow along with the Lynda.com video 
[https://www.lynda.com/IT-tutorials/Learning-Linux-Command-Line/753913-2.html?org=syracuse.edu](https://www.lynda.com/IT-tutorials/Learning-Linux-Command-Line/753913-2.html?org=syracuse.edu)
 starting for these chapters:

- **2. Command-Line Basics** -- The Full Chapter (approx. 15 minutes of video
- **3. Files, Folders and Permissions** -- Full Chapter (approx. 35 minutes of video )
- **4. Common Command-Line Tasks and Tools** - Stop After **Edit Text with nano** (approx. 25 minutes of video)  

Tips for getting the most out of the videos!

- As you watch the video, you should be able to type the same commands in your Linux docker container, where you see the `scott@localhost:~$ ` prompt.
- Pause the video and try the commands for yourself. The best way to learn them is to get your hands dirty!
- As you complete the commands and watch the video, write down any questions you have. You are encouraged to ask questions when we go over the lab in our next class session.


### Lab Teardown  / Lab Reset ###

When you are finished, do not forget to tear-down your lab. You can also reset the lab back to its default state if you screw something up or just want to try the lab again!

1. From the Linux prompt, type:
`scott@localhost:~$  exit`  
to quit linux and return to the PowerShell Prompt
1. At this point you've exited the container, but its still running. If you type:  
`PS ist346-labs\lab-B> docker-compose exec linux login`   
You can log back into the container. (Don't do this unless that's what you want!)
1. If you wish to tear down the lab so that you can work on the next lab, or restart this lab, type:
`PS ist346-labs\lab-B> docker-compose down`
This stops the container and deletes all changes made to it.
1. The command:  
`PS ist346-labs\lab-B> docker-compose up -d`
will start the container back up again restoring it to the original state.

## Questions ##

1. For the command `docker-compose exec foo bar` What does `foo` represent? What does `bar` represent?
1. How do you get help in Linux?
1. What is a pipe?
1. For the following file:  
`-rw-r-xr-- 1 scott comedians  1474 Jun 21 20:54 poems.txt`
   1. The file is readable by?
   1. What is the group associated with this file?
   1. Who can execute this file?
   1. How large is the file?
   1. Who can write to the file?
1. How can you determine if you are a `root` user from the command line?
1. Which command allows a user to elevate to run command as the super-user? 
1. What is a link? What are the two types of links?
1. Write the Linux command to move a file named `a.txt` in the `Documents` folder into the `Desktop` folder. Note: Assume both folders are the same parent directory.
1. Write the Linux command to list files in a folder two folder up from the current working directory.
1. Write the Linux command to find the files that begin with the letter `t` in the `/bin` folder.