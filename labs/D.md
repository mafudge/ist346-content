# Lab D - Virtual machines and containers

## Learning Objective

In this lab you will:

- Learn how how Docker works to create virtual environments.

## Before You Begin

### Prep your lab environment ###

Since this lab uses, Docker the setup will be a little different.  We need to reset the Docker environment so that the images are consistent with the video used in the lab. 

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-D` folder:  
`PS ist346-labs> cd lab-D`
5. Stop all running containers:  
`PS ist346-labs\lab-D> docker kill $(docker ps -q)`
5. Delete all stopped containers:  
`PS ist346-labs\lab-D> docker rm $(docker ps -a -q)`
6. Delete all images:  
`PS ist346-labs\lab-D> docker rmi $(docker images -q)`
7. Get the `ist346/ubnutu` image from Docker Hub:
`PS ist346-labs\lab-D> docker pull ist346/ubuntu:latest`
8. Set the environment variable used in the video. Paste this command into your PowerShell window:
```
$FORMAT="\nID\t{{.ID}}\nIMAGE\t{{.Image}}\nCOMMAND\t{{.Command}}\n
CREATED\t{{.RunningFor}}\nSTATUS\t{{.Status}}\nPORTS\t{{.Ports}}\nNAMES\t{{.Names}}\n"
```

### Log in to Lynda.com with your Syracuse University Account ###

1. Navigate your web browser to https://lynda.syr.edu 
2. This will prompt you with a Syracuse University login screen. Login with your Syracuse University **NetID** and **Password**. 
3. Once your are logged on you will see the Lynda.com home page. You can jump directly to the course here: [https://www.lynda.com/Docker-tutorials/Learning-Docker/485649-2.html?org=syracuse.edu](https://www.lynda.com/Docker-tutorials/Learning-Docker/485649-2.html?org=syracuse.edu) 

The entire course is about 3 hours long. We will only complete part 2 which should take you about 60 minutes.

NOTE: You'll probably want to use a pair of headphones so you can listen to the audio of the video lessons in private.

**Okay! You're now ready to start the Lab!**

## Learning Docker ##

### What is Docker? ###

Start by watching the following video: What is docker?
[https://www.lynda.com/Docker-tutorials/What-Docker/721901/779036-4.html?org=syracuse.edu](https://www.lynda.com/Docker-tutorials/What-Docker/721901/779036-4.html?org=syracuse.edu) 

## Docker Commands ##

Follow along with the Lynda.com video for this chapter:

- **2. Using Docker** -- Full Chapter (Approx 55 minutes) from here:     
[https://www.lynda.com/Docker-tutorials/Learning-Docker/485649-2.html?org=syracuse.edu](https://www.lynda.com/Docker-tutorials/Learning-Docker/485649-2.html?org=syracuse.edu)

Important Tips: 

- The author is using Linux but you are using PowerShell. The lab should work fine.
- As you watch the video, you should be able to type the same commands from your PowerShell command prompt. 
- When you need to open additional windows, like in the video, type `start powershell` from the PowerShell command prompt to open PowerShell in another window.
- At one point you will need to know the IP Address of the computer running docker. In PowerShell you can get this with `ipconfig`  unlike the `ifconfig` command the author uses in the Linux video.

## Tear-Down the Lab Environment ##

To tear down this lab environment when you are finished:

5. Stop all running containers:  
`PS ist346-labs\lab-D> docker kill $(docker ps -q)`
5. Delete all stopped containers:  
`PS ist346-labs\lab-D> docker rm $(docker ps -a -q)`
6. Delete all images:  
`PS ist346-labs\lab-D> docker rmi $(docker images -q)`


## Questions ##

1. What is the difference between an image and a container?
2. What docker command turns an image into a container?
3. What docker command turns a container back into an image?
4. Which command lists running containers?
5. Where do docker images come from?
6. How do you inspect the output of a docker container?
7. Write a docker command to run the command `echo "hi"` on the image ubuntu.
8. `docker run -p 1000:2000 ubuntu my-program` Which port is available to the host? Which port does this application run on? 
9. What are the two types of docker volumes?
10. What are the two ways you can network containers in docker?
