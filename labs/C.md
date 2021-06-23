# Lab C - Networking Fundamentals

## Learning Objective

In this lab you will:

- Learn basic networking commands for PowerShell and Linux
- Understand the basics of how the Docker network environment works.
- Learn the basics of the Linux command line. 

### Lab Setup At A Glance

In this lab, we will have two computers setup on their own network (as docker containers). They share this network with your Windows 10 host computer. Access to the Internet is obtained through the host computer.

The server computer exposes two services (a web server and secure shell) to the host computer and workstation.
```
+-------------+
| workstation |----+
+-------------+    |
                   |   +---------------+     ++++++++++++++
                   +---| host computer |-----|  Internet  |
                   |   +---------------+     ++++++++++++++
+-------------+    |
|   server    |----+
+-------------+
```

### Commands we will use in this lab

- `ipconfig / ifconfig` For retrieving IP configuration of a device
- `ping` To test communications with another device on a network.
- `nslookup` To resolve an IP Address to a DNS Name
- `nmap` To test open TCP ports on a network endpoint
- `tracert` (traceroute) For tracing the entire path a packet traverses to reach and endpoint
- `netstat` For viewing all active communications (inbound and outbound).
- `ssh` For connecting to another computer over the network
- `lynx` A command line web browser

## Before You Begin

### Prep your lab environment ###

This should be done before every lab! On the computer where your labs are located:

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-C` folder:  
`PS ist346-labs> cd lab-C`
5. Start the lab environment in Docker:  
`PS ist346-labs\lab-C> docker-compose up -d`
6. Verify your server and workstation are up and running:
`PS ist346-labs\lab-C> docker-compose ps`  
You should see labC_server and labC_workstation up and running.


## Part 1: Hosts and IP Addresses

First let's try some network commands from the PowerShell prompt, which is your Windows 10 host computer.

### IP Addresses

Every computer on the internet must have an IP address, but this is not usually how we go about finding services. For example you don't need to know Facebook's IP address to use it. We use the name facebook.com instead, but rest assured there's an IP address behind the communication, so... 

1. What is my ip? Type:
`PS ist346-labs\lab-C> ipconfig`   
You're going to see a few IP addresses in your output. Focus on the one for `Ethernet0` as that is the primary interface for this computer. What is your IP address? 

### DNS: What's my Name? 

 You probably don't call your friends using their phone number - you use a contact list and dial by name. DNS serves this purpose on the Internet - looking up  names to IP addresses and vice-versa. Imagine a system where if you ask to dial someone not in your contact list it would look up the phone number for you in someone else's list. That, in essence is how DNS works. There are several DNS servers on the internet each is the authority over one or more domains, like syr.edu or google.com.
 
1. What is the IP address for a given hostname? To find the IP address for a given hostname, we use `nslookup`. Type:
`PS ist346-labs\lab-C> nslookup ischool.syr.edu`   
to get the IP address for the host name `ischool.syr.edu`. Make a note of the IP address as you will need it for the next part. (At the time this )
1. Let's perform the reverse. Lookup a hostname for a given IP address: 
`PS ist346-labs\lab-C> nslookup 128.230.146.100`   
If all goes to plan this should be `ischool.syr.edu` 
1. Which DNS server are you using? By default you're using the DNS server set by your network configuration. You can ask another server to resolve the name for you. Let's ask google's public DNS server to do it:  
`PS ist346-labs\lab-C> nslookup ischool.syr.edu 8.8.8.8`   
The IP address `8.8.8.8` is the DNS server we wish to use to resolve the name for us.   
You'll notice this time it says `Non-authoratative answer` the DNS server is telling you that it does not contain the actual record for this name - it got it from another server.

### Ping: Are you there, God? It's me, Margaret.

Looking up names and numbers is fine, but how to we know the computer we want to access is online? This is the purpose of the `ping` utility. It will let you know if a host is 'alive'.

1. Is the ischool website host online? Type this to ping the server 4 times:  
`PS ist346-labs\lab-C> ping -n 4 ischool.syr.edu`    
In the response you will each reply. In the reply it gives you a response time in milliseconds. It also gives you a summary of how long the request and response took round-trip. This is a good indicator of how fast and reliable the network is between you and the host. 
1. Let's ping the IP address `1.2.3.4`, which I'm certain is not around:    
`PS ist346-labs\lab-C> ping -n 2 1.2.3.4`  
You will see that the requests time out. 
1. Let's ping something with a longer response time, for example this host is not on the Syracuse University network:  
`PS ist346-labs\lab-C> ping -n 2 www.apphammer.co`  
Notice how the response time was a bit longer (60ms). 60 milliseconds is still pretty fast but in the world of the Internet, every millisecond counts!

IMPORTANT: It should be noted that just because the computer responds, it does not imply the service running on the computer actually works. Ping only verifies network connectivity between you and the host in question. Also some network administrators block ping requests via a firewall so the host might actually function but not respond to ping requests.

### Tracert: Ya can't get there from here.

The trace route `tracert` utility shows the route over the Internet your data packets take from source to destination. For each router along the way (called a hop) it calculates the round-trip time (RTT) from source to that hop. In general the lower the number of hops and the lower the round trip time (RTT) the fast the communication between you and the host will be.

1. This should go quickly:  
`PS ist346-labs\lab-C> tracert ischool.syr.edu`  
The output should have 4 hops and a response time of less than 1ms (same as the `ping` BTW).
1. Let's perform a longer tracert:  
`PS ist346-labs\lab-C> tracert www.apphammer.co`   
This output used 20 hops and required 61 milliseconds. Again, consistent with the `ping`

1. Let's tracert a problem. In this example we will use an IP address not on the Internet. To make the example go faster we will restrict the RTT to 10 milliseconds and the number of hops to 8.   
`PS ist346-labs\lab-C> tracert -w 10 -h 8 1.2.3.4`   
You'll notice that as soon as we route off campus and on to the public internet, we can not find the host, and receive a series of timeouts until we reach the maximum number of hops. This type of output is indicative of situations when hosts are not available.

## Part 2: Ports

As we said before just because a host responds to a ping it does not necessarily imply the service running on the host is available. For instance you might be able to ping a webserver but for some reason none of the web pages on that server load.

The applications which run on these servers communicate over a port. The port consists of a number between 0 and 65535 and a protocol of either TCP or UDP. Common services we use everyday such as HTTP are setup to run over the same port. This ensures consistency from and makes it easer for clients to consume the service. 

A list of the well-known ports can be found here: 
[https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)

### Getting started

1. Let's connect to our workstation running Ubuntu linux:  
`PS ist346-labs\lab-C> docker-compose exec workstation bash`   
1. You will now see a bash command prompt like this:  
`root@workstation:/#`   
Indicating that you are no longer in PowerShell but inside your workstation linux container.
1. Let's check the ip address of the workstation, type:  
`root@workstation:/# ifconfig`   
Notice that on Linux it is `ifconfig` not `ipconfig`. Also the interface it called `eth0`. Make a note of the IP Address of your workstation.

### Scanning for ports

The `nmap` utility allows you to scan a host for open ports. Open ports will give you an indication of which services are available. 

1. Let's scan `ischool.syr.edu` for open ports. Type:  
`root@workstation:/# nmap ischool.syr.edu`   
The command output should show at least these ports open:  
`80/tcp  http`  
`443/tcp https`  
You would expect these to be available because its a web server.
2. You can connect to the web server using a browser to try it out. On your workstation is the `lynx` utility. Type:
`root@workstation:/# lynx ischool.syr.edu`   
You will see the ischool website on your screen. (It's going to be ugly because `lynx` is a text-only browser)   
When you've had your fun, press `q` to quit and press `y` to confirm to return to the bash prompt.
3. Let's port scan our other container, the server:
`root@workstation:/# nmap server`   
You will see two ports open `22/ssh` and `80/http`
4. Again let's use lynx to connect to this web server:  
`root@workstation:/# lynx http://server:80`   
This page is the default nginx web server page. Press `q` to quit and press `y` to confirm to return to the bash prompt.  
NOTE: WE didn't have to put in `:80` for the port. This is assumed by the `lynx` application because `80` is the well-known port for the `http` prococol.

### SSH and Netstat

In this last section we will connect to the server from the workstation using ssh (secure shell) and verify the open ports locally.

1. Let's use ssh to connect to the ssh service on the server. ssh or secure socket shell allows us to access the shell of a remote computer. Type:  
`root@workstation:/# ssh root@server`  
To connect to `server` as user `root`. You will be prompted to confirm you want to connect. Type `yes` and press `ENTER`.   
You will prompted to enter the `root` password on `server`. This password is `IST346`  
Your prompt will now read `root@server:~#`
1. Let's check this IP address, type:  
`root@server:~# ifconfig`  
You'll notice it's different from your workstation IP.
1. let's port scan the server from the server itself:  
`root@server:~# nmap localhost`   
NOTE: You will still see the two open ports.
1. For fun let's port scan the workstation. Should be no open ports:  
`root@server:~# nmap workstation`   
1. When you want to see the ports in use on the current computer, `netstat` is the better choice. It will show you listening connections, which processes are running the connections and which clients have connected to the port. Type:  
`root@server:~# netstat -ap`    
You should see the `*:http` and `*:ssh` services running, as a client `labC_workstation` connected to `ssh`. Yes, that's you!
1. To quit ssh and return to the workstation, type:  
 ``root@server:~# exit`   
 You should see `connection to server closed`
1. Back at the `root@workstation:/#` prompt, type:  
`root@workstation:/# exit` to return to PowerShell

## Tear-Down

To tear down this lab:

1. `PS ist346-labs\lab-C> docker-compose down` 

## Questions 

1. Write a command to lookup the IP address for `michaelfudge.com` using the dns server `1.1.1.1`?
1. What does it mean when you see a non-authorative answer from a DNS query?
1. What can you do if you cannot `tracert` to your destination in the designated number of hops?
1. What do the numbers like 6ms mean in context of the `tracert` command?
1. Which command do you use to find the open ports on a host?
1. What is the purpose of establishing well known ports for services?
1. How does `netstat` differ from `nmap`?
1. Write a command to `ssh` into the server `elephant.com` as user `dumbo`?
1. Write a command to check for open ports on `google.com` ?
