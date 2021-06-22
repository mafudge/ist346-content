# Information Security

In this lab, we will demonstrate the basics of applied information security.

## Learning Objectives

In this lab you will learn how to

- Configure a host-based firewall to deny access to a running service, then modify the rule to block specific IP's
- Harden a service by locking down and/or disabling the unused features.
- Follow well know security practices with respect to accounts and passwords.

## Before you begin 

### Prep your lab environment. 

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. IMPORTANT: This lab requires access to Docker's internals, you must enter this command:  
`PS ist346-labs> $Env:COMPOSE_CONVERT_WINDOWS_PATHS=1`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-J` folder:  
`PS ist346-labs> cd lab-J`

## Part 1: Utilizing a host based firewall

The first line of defense for any system is a firewall on the host. Firewalls limit traffic to ports defined by the system administrator. So while the services might be running, the firewall might contain a rule which blocks certain hosts from accessing the service. Potentially, this stops malicious users from accessing the system through ports opened by our services. On Linux many system administrators utilize IPTables to create a firewall. Due to the complexity and flexibility of Iptables we will be using a simplified version called UFW or Uncomplicated Firewall. UFW really does live up to its name it is much easier than IPTables.

Lets begin by bring up our first docker environment. It a network consisting of a simple webserver, `nginx` and two clients. On client is legitimate and the other is a hacker!

```
+--------------+
| client       |---+
| 172.44.10.103|   |
+--------------+   |
                   |
+--------------+   |
| hacker       |---+
| 172.44.10.104|   |
+--------------+   |
                   |   +---------------+     ++++++++++++++
                   +---| host computer |-----|  Internet  |
                   |   +---------------+     ++++++++++++++
+--------------+   |
|   webserver  |---+
| 172.44.10.102|
+--------------+
```

1. Let's bring up the environment:  
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml up -d`
2. Now let's make sure the environment is up and running:  
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml ps`  
You should see the `client`, `hacker`, and `webserver` containers running, all in the `Up` status.

### A hacker is among us!

1. There have been reports to our helpdesk of the website responding slowly. Now lets see what is going on with our server by checking out the logs.  
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml logs webserver`   
You should see logs from `webserver_1` if you do not, wait a minute and try again.  
Whoa, What is going on! Our weblogs are showing the attacker making attempts at user and admin logins ONE EVERY SECOND!. The error message in the logs indicate that the attack is not successful, which is a good thing, but we don't want them to succeed. We need to stop this user. Let's block them with a firewall!
2. Open and new terminal and login to the server. IMPORTANT: Leave the window with the logs open!
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml exec webserver bash`  
This should bring you to the command prompt on the webserver. Once logged in we need to setup a firewall to stop malicious users. UFW is already installed on most Linux distributions. The best way to guarantee security at this point in the hack is to stop everything then only let in what we need to, a practice called successive refinement.
3. Let's set the default rule to deny access to all services to all clients. Type:
`root@webserver:/# ufw default deny`
4. Once that is set lets we turn the firewall on.
`root@webserver:/# ufw enable`
5. Let's go back to the original terminal window, still running PowerShell:
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml logs webserver`  
Great! the attack was stopped. How do you know? Look for the date and time in the bottom of the logs. Notice how the last entry stays the name? That's because the hacker can no longer access the service and thus no new entries are logged. Sure he's tring at this point but definately not succeeding!

### Oops. What did we do?

1. Before we celebrate, let's think about what we did. We've denied access to all clients! That means the people who need to access our webserver cannot. Let's demonstrate, by logging into the client:
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml exec client bash`  
2. From the `root@client:/#` command prompt, use the `curl` command to attempt to access the website. `curl` is a command line web browser of sorts, allow us to issue HTTP requests from the command line. Try this:
`root@client:/# curl -m 3 http://webserver`
This command attempts to connect to our webserver on the well-known port of TCP/80, timing out after 3 seconds.  You get this error message:
`curl: (28) Connection timed out a 3000 milliseconds.` 
3. Uh Oh, that's not good, it looks like our clients can't access the site either. This happened because we stopped all traffic to the server, lets fix this by first opening up port TCP/80, allowing http traffic to reach our server. 
4. Switch back to the server command prompt `root@webserver:/#` let's create the rule: 
`root@webserver:/# ufw allow 80`
5. And then reload the firewall:
`root@webserver:/# ufw reload`
6. Now check your client again. go back to the `root@client:/#` command prompt, and try our curl command again:
`root@client:/# curl -m 3 http://webserver`  
This time it works! We see the HTML which would normally display as a webpage in our browser.
7. Exit from the client to return to the PowerShell prompt:
`root@client:/# exit`  

### The hacker is back!

1. If you check your server logs you see that the hacker attackes are getting through the firewall again!  
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml logs webserver`  
(Again look for the time in the bottom of the logs. If you need to run this command more than once.)
2. That's because we allow ALL IP ADDRESSES to access our server over TCP/80. We need to stop this by by blocking traffic from the hacker's IP address. If we look at the logs being created on the server you can find the IP address of the attacker.
```
172.44.1.104 - - [15/Oct/2018:14:50:34 +0000] "POST /admin HTTP/1.1" 404 153 "-" "curl/7.47.0" "-"
```
3. The address is `172.44.10.104`, which is visible at the beginning of the line on the logs. Lets block that person by inserting a new rule at position number 1, UFW evaluates rules in order, so we need to make sure our block rule is before the allow all rule for port 80. The status command shows the current rules. Back at the `root@webserver:/#` prompt, type:
`root@webserver:/# ufw insert 1 deny from 172.44.10.104 to any`  
To insert the rule as the first in the chain.
4. Next., type:  
`root@webserver:/# ufw reload`  
to reload the firewall rules.
5. finally type this command to see the status of the numbered rules:  
`root@webserver:/# ufw status numbered`
You should now see this output:

```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] Anywhere                   DENY IN     172.44.1.104
[ 2] 80                         ALLOW IN    Anywhere
[ 3] 80 (v6)                    ALLOW IN    Anywhere (v6)
```

6. And now check your server logs again from the PowerShell prompt:  
`PS C:\ist346\Lab-J> docker-compose -f ufw-example.yml logs webserver`  
and you will see the attacker has been stopped. This is a very simple example, UFW has many options I encourage you to check the documentation for many other ways to secure your server.

### Cleaning up!

Let's clean up from this part:

1. From `root@webserver:/#` Type:  
`root@webserver:/# exit`
2. Tear down this part of the lab:  
`PS C:\ist346\Lab-J> docker-compose -f .\ufw-example.yml down`

## Part 2: Hardening a Service

Firewalls on the host are just one layer of security. Another thing to consider is disabling the individual features of a service that we do not require as part of our application. This is referred to as service hardening.  When you do this to every service on the server, its server hardening.

One of the most used and exposed services on a server is the webserver, it's the most used service on the Internet and is utilized by almost every organization. Being so popular and widely used means, webservers are popular attack targets primary But regardless of the webserver they all need to be configured to protect against attacks.

Cross-site scripting (XSS) is a type of computer security vulnerability typically found in web applications. XSS enables attackers to inject client-side scripts into web pages viewed by other users. A cross-site scripting vulnerability may be used by attackers to bypass access controls such as the same-origin policy. XSS attacks are performed by an attacker inputting a script into a form field on a website such as a comment field on a blog or forum.

These types of attacks can be mitigated by the webserver, by configuring it to informing the client browser to not download resources from external sources that you do not control.

### Getting Started

Let's spin up the lab:
`PS C:\ist346\Lab-J> docker-compose -f vulnerable-website.yml up -d`

Two webservices are started, one is our website named `webserver` which is available to host computer on port `80` and the other is a server that a malicious user controls called `badsite` available to the host on port `8080`. Normally these would be different domains, but for this example they will use different ports.

```
+-------------+
| badsite:8080|---+
+-------------+   |
                  |   +---------------+     ++++++++++++++
                  +---| host computer |-----|  Internet  |
                  |   +---------------+     ++++++++++++++
+-------------+   |
| webserver:80|---+
+-------------+
```

### Your site has been hacked!

Open your browser and go to the demo site. [http://localhost](http://localhost)

You will see the message: `Your site has been hacked!` 

The malicious user has compromised your site by loading an external script in a comment box! The origin of the script is from the site http://localhost:8080. Is that website the hacker's website? If they are a smart hacker, no. More than likely the hacker has hacked that site too and it using `badsite` to launch attacks.

This hacker's script only displays a message, but in reality the hacker could capture and collect any data you share with the legitimate site!

Lets fix our webserver so these attacks do not effect our good users. We are also going to add some other security measures that help harden our webserver.

1. Open a new terminal window and login to the webserver.
`PS C:\ist346\Lab-J> docker-compose -f vulnerable-website.yml exec webserver bash`
2. Once logged into the server, you need to edit the nginx configuration file for the default site.  
`root@webserver:/# nano /etc/nginx/conf.d/default.conf`
3. Copy and paste the code below into your `default.conf` file. Paste it below the line: `server_name localhost;`  
Text to paste:  

```
# don't send the nginx version number in error pages and Server header
server_tokens off;

# config to don't allow the browser to render the page inside an frame or iframe
# and avoid clickjacking http://en.wikipedia.org/wiki/Clickjacking
# if you need to allow [i]frames, you can use SAMEORIGIN or even set an uri with ALLOW-FROM uri
# https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options
add_header X-Frame-Options SAMEORIGIN;


# This header enables the Cross-site scripting (XSS) filter built into most recent web browsers.
# It's usually enabled by default anyway, so the role of this header is to re-enable the filter for 
# this particular website if it was disabled by the user.
# https://www.owasp.org/index.php/List_of_useful_HTTP_headers
add_header X-XSS-Protection "1; mode=block";

# with Content Security Policy (CSP) enabled(and a browser that supports it(http://caniuse.com/#feat=contentsecuritypolicy),
# you can tell the browser that it can only download content from the domains you explicitly allow in our example we are allowing google analytics.
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com";

```

4. When you are finished your `default.conf` file should look like the one below:
```
server {
    listen       80;
    server_name  localhost;

    # don't send the nginx version number in error pages and Server header
    server_tokens off;

    # config to don't allow the browser to render the page inside an frame or iframe
    # and avoid clickjacking http://en.wikipedia.org/wiki/Clickjacking
    # if you need to allow [i]frames, you can use SAMEORIGIN or even set an uri with ALLOW-FROM uri
    # https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options
    add_header X-Frame-Options SAMEORIGIN;


    # This header enables the Cross-site scripting (XSS) filter built into most recent web browsers.
    # It's usually enabled by default anyway, so the role of this header is to re-enable the filter for 
    # this particular website if it was disabled by the user.
    # https://www.owasp.org/index.php/List_of_useful_HTTP_headers
    add_header X-XSS-Protection "1; mode=block";

    # with Content Security Policy (CSP) enabled(and a browser that supports it(http://caniuse.com/#feat=contentsecuritypolicy),
    # you can tell the browser that it can only download content from the domains you explicitly allow in our example we are allowing google analytics.
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com";

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

5. Save the file by pressing `CTRL`+`x` and selecting `Y` to saving over the modified butter, overwriting the current file.
6. Now we must reload the webserver configuration:  
`root@webserver:/# nginx -s reload`  
You should see the message below, otherwise is will tell you that there is an error
```
2018/10/16 10:45:59 [notice] 28#28: signal process started
```
NOTE: If you get an error you may have to edit the contents of `default.conf` to fix the error.   

6. Now check your [http://localhost](http://localhost). The message should no longer load because we told the browser not to load external scripts!

NOTE: If you still see the error message, it is because the hacker's malicious script is still in your browser cache. You should clear your browser's history to remove the script from the cache. 

This is just the beginning of what needs to be done to harden a webserver, check the NGINX documentation for other recommended security practices.

https://docs.nginx.com/nginx/admin-guide/security-controls/

### Part 2 Tear-Down
1. Exit the webserver's console:  
`root@webserver:/# exit`
2. Tear down the containers for part 2:  
`PS C:\ist346\Lab-J> docker-compose -f vulnerable-website.yml down`

## Part 3: Utilizing private/public ssh keys for SSH.

Key pairs allow you to login to a remote host via SSH (Secure Shell) without using your password.  This is generally seen as a more secure practice than using a password because users have a tendency to use password that are too simple or to use the same password on multiple hosts. In this part of the lab we will demonstrate how to set this up.

1. Let;'s bring up the environment for this part of the lab:  
`PS C:\ist346\Lab-J> docker-compose -f ssh-keys.yml up -d`
2. When the build is complete, check the status of your containers:  
`PS C:\ist346\Lab-J> docker-compose -f ssh-keys.yml ps`  
both the `ssh-client` and `ssh-server` should be `Up`
3. First we need to create a user and keys for that user. Login to the `ssh-server`:  
`PS C:\ist346\Lab-J> docker-compose -f ssh-keys.yml exec ssh-server bash`  
4. You are now logged into the server and should see the `root@ssh-server:/#` prompt. Let's create a new user called `myuser`:   
`root@ssh-server:/# adduser myuser`
5.  Follow the on screen prompts, setting the password for `myuser` to `testing123` and making up values for the rest of the prompts.
6. Now we must test that we can connect to the `ssh-server` as `myuser` for this we need to open another powershell window and make `Lab-J` the current directory.
7. Let's open the shell on the `ssh-client` container:
`PS C:\ist346\Lab-J> docker-compose -f ssh-keys.yml exec ssh-client bash`  
8. From the `root@ssh-client:/#` prompt, let's try to `ssh` into the server:  
`root@ssh-client:/# ssh myuser@ssh-server`  
You should see output asking to save the server's key fingerprint. type `yes` and press `ENTER`. 
```
The authenticity of host 'ssh-server (172.20.0.2)' can't be established.
ECDSA key fingerprint is SHA256:iV9FBVXcbAK0fFuM2ecIOuExbnpP1apuejmsNXuPcvk.
Are you sure you want to continue connecting (yes/no)? yes
```
9. Next you will see a warning, and then be asked to enter the password, which is `testing123`:

```
Warning: Permanently added 'ssh-server,172.20.0.2' (ECDSA) to the list of known hosts.
myuser@ssh-server's password:
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.9.93-linuxkit-aufs x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

myuser@ssh-server:~$
```
10. Yes! it worked. If it didn't work, go back to the server and try setting the password again. Type:  
`myuser@ssh-server:~$ exit`   
to exit out of the `ssh-server` and return to the client. 

### Generating the keys to login without a password

Now that we have a user account and we know it can log in to `ssh-server` its time to generate a keypair so that we can login without using the password. When you generate a key pair, one is public the other is private. The private key is the important one and must be kept safe. How does this work? An analogy with real locks and keys is the private key creates unique padlocks every time which can only be unlocked with the public key.

We must keep the private key safe for if a hacker gets a copy of the private key she too can create unique padlocks on other systems which will be naively unlocked with any system that has the public key. At this point your encryption is useless!

1. Let's generate the key pair, from the `ssh-client`:  
`root@ssh-client:/# ssh-keygen -t rsa`  
You will be asked for a location to store the file and a passphrase. Press `ENTER` three times. We don't want a password to login! The output:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:qzIS1bPB+Yf+nnNHgXmbp8N00C6W+cMpVj4ox3dU6/Q root@ssh-client
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|     o .     o . |
|    . *     o + o|
|   .   =S.   . Oo|
|  .   . o..   X+=|
|   .   ...  .=BBo|
|  . o  .. .o.*=OE|
|   . o.  o+o+.o.+|
+----[SHA256]-----+
```

2. Notice that your randomart image will look different. Again, keys are generated in pairs, a private key and a public key. Your private key should always be kept secret and never distributed to other system. The public key is what other systems use to authenticate and must be on any remote system for which we want to login without a password. We will upload the public key to the server under the user account we created. 
3. First make a `.ssh` directory on the server from the client:   
`root@ssh-client:/# ssh myuser@ssh-server mkdir -p .ssh`   
NOTE: you will have to enter the password `testing123` to execute this command on the remote server.
4. Then we upload the key to a special file called `authorized_keys`, type:  
`root@ssh-client:/# cat /root/.ssh/id_rsa.pub | ssh myuser@ssh-server 'cat >> .ssh/authorized_keys'`   
Once again you will need to enter your `testing123` password.
5. If you go back to the server terminal window,  you will now see a new file under `/home/myuser/.ssh`, type:
`root@ssh-server:/# ls /home/myuser/.ssh`  
You will see:  
`authorized_keys`
6. If we look at the contents of this file you will see the generated public key (NOTE: yours should be different:)   
`root@ssh-server:/# cat /home/myuser/.ssh/authorized_keys`  
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuYmc6toOzLNga95izpfQBNhn3psUoYFpVa4wHPDBwLMQchsp3DjXdVDbBn6clpZB0zCA8M28gKSqKPutc1KAXjDVqoC8/GYE9hlnwMmLgICuEzMSPZYuKQNrzQoQ+o47hviQIJkJEXcd+DsuDm8E7Rjw00DxRG0/ohClhJ57
WxHiSYjMB+Tw/0pYLQ0d6kuoI7ClHWaYhxMhLFoN8Qlw66Qy/F7LIDOaWPBM6wJWuB9dV40x+Ehi77JYtExa+RvOUpvlDvZWYVPtJWGxicVt5mWBqV8BJgvMQQhn0f0uFmPml5VqAERM7Hju3b4X+zm5yOveliqJRPDcU4TfLRAD root@ssh-client
```
7. While on the server we should set the permissions of the `authorized_keys` file so only the `myuser` account user can access it. Password-less ssh will work without this step, but its definately more secure when you do it:  
`root@ssh-server:# chmod 700 /home/myuser/.ssh; chmod 640  /home/myuser/.ssh/authorized_keys`
8. We are all set! You should no be able to access the ssh-server without typing a password now, from the `ssh-client` try:  
`root@ssh-client:/# ssh myuser@ssh-server`  
You now login without a password!  
```
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.9.93-linuxkit-aufs x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Tue Oct 16 14:30:58 2018 from 172.20.0.3
```
8. From the `myuser@ssh-server:~$` prompt, return to the client:  
`myuser@ssh-server:~$ exit`
```
logout
Connection to ssh-server closed.
```
9. Where this is really useful is in scenairios where you need to execute remote commands on the server from the client. For example, let's get the IP address of the server:  
`root@ssh-client:/# ssh myuser@ssh-server ifconfig eth0`   
You should see this output:  
```
eth0      Link encap:Ethernet  HWaddr 02:42:ac:1a:00:02
          inet addr:172.26.0.2  Bcast:172.26.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:414 errors:0 dropped:0 overruns:0 frame:0
          TX packets:302 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:67067 (67.0 KB)  TX bytes:68943 (68.9 KB)
```

### Tear down

1. Exit from the client:
`root@ssh-client:/# exit`
2. Exit from the server:
`root@ssh-server:/# exit`
3. Tear down the environment: 
``PS C:\ist346\Lab-J> docker-compose -f ssh-keys.yml down`

## Questions

1. What is a host firewall? UFW is an easier replacement for which linux firewall technology?
2. For a command like this: `docker-compose -f ufw-example.yml exec webserver bash` What is the command we are executing?  What is the host name?
3. Why are the logs of a service like `nginx` for example so important to being able to administer and secure that service?
4. What does it mean to harden a service?
5. What makes web servers such high value targets for hackers?
6. If you encounter a cross site scripting attack, why should you delete your browser's history?
7. Why is password-less SSH using key pairs considered more secure than using SSH with a password?
8. Explain how a public and private key pair work.
9. What is the command to execute the remote command `cat /etc/passwd` as user `hacker` on server `government` ?