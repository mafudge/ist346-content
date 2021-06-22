# Lab K - Email and Messaging

Setting up an email server takes a considerable amount of time and requires intricate knowledge of the many services necessary to download and save email, send email, and provide a user interface for users to interact with their messages. All of these services require knowledge of security best practices. For our example we will be using a prebuilt docker container that has all of these services setup for us.

## Learning Objectives

In this lab you will learn

- To Send SMTP (simple mail transport protocol) email messages through telnet (connecting to the service directly) and retrieve email using IMAP.
- How application-layer TCP protocols like SMTP and IMAP actually work.
- How to configure mail services for sending and receiving emails.
- How to configure a webmail client to communicate with email services.

## Before you begin 

### Prep your lab environment. 

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. IMPORTANT: This lab requires access to Docker's internals, you must enter this command:  
`PS ist346-labs> $Env:COMPOSE_CONVERT_WINDOWS_PATHS=1`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-K` folder:  
`PS ist346-labs> cd lab-K`

## Setup and run the mail server

1. To begin start your environment:  
`PS C:\ist346-labs\lab-K> docker-compose up -d`  
This will start 3 containers. A `mailserver` email server, containing Postfix (email inbox server) and an SMTP server for outgoing mail, a `roundcube` container, which is a webmail client, and a `workstation` container so you can learn to send SMTP mail manually. 
2. Check that your services are running:  
`PS C:\ist346-labs\lab-K> docker-compose ps`  

## Send Mail with SMTP using the Telnet client

We can send and receive mail using the [https://en.wikipedia.org/wiki/Telnet](https://en.wikipedia.org/wiki/Telnet) program. This allows us to interact with the mail server from our terminal at the protocol level. We will Telnet into the SMTP service on mailserver and then issue protocol commands to send an email. It's a great way to see how the SMTP (simple mail transport protocol) actually goes about sending an email.

1. First login to the telnet client.  
`PS C:\ist346-labs\lab-K> docker compose exec workstation bash`
2. Once logged in we can start the telnet interface.  
`root@workstation:/# telnet`
3. From the telnet prompt connect to the SMTP service on the `mailserver` to send an email. This is TCP port 25, the well-known port number for SMTP:  
```
telnet> open mailserver 25
Trying 192.168.144.2...
Connected to mailserver.
Escape character is '^]'.
220-mail.mycompany.com ESMTP Postfix (Debian)
```
4. Once connected to the mailserver you can test the server connection:
`ehlo mycompany.com`  
You should see output similar to this:  
```
250-mail.mycompany.com
250-PIPELINING
250-SIZE 1048576
250-ETRN
250-STARTTLS
250-AUTH PLAIN LOGIN
250-AUTH=PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250 SMTPUTF8
```
5. Now you can send an email, first define who the message is from:  
`mail from: me@mycompany.com`  
The response will say:  
`250 2.1.0 Ok`
6. Second define the recipient:  
`rcpt to: myuser@mycompany.com`  
Again you will see the response:  
`250 2.1.0 Ok`
7. Now set the data:   
`data`  
And you will see the message:  
`354 End data with <CR><LF>.<CR><LF>`
8. Then enter the following:
```
Subject: Test Email
From: Me a name I call myself
Date: 7/4/2021

My Test Email from me!
.
```
Don't forget the . at the end!   
If everything worked the you should see something similiar to:  
`250 2.0.0 Ok: queued as E0684127C`
9. Then quit out of the smtp server:  
`quit`
That should have delivered our message to our `myuser` mailbox at our fake company `mycompany.com`.

## Retrieve Mail with IMAP using the Telnet client

IMAP (internet message access protocol) allows for the recieving of mail from a mailbox. SMTP does the sending to a mailbox, and IMAP allows a user to retieve it. In this part we will again use Telnet to connect to IMAP running on our mailserver and read the email we just sent. Again, this is not commonly how one would use email, but it does help you understand how application-layer protocols like SMTP and IMAP work under the covers!

Any real mail client we use whether it be outlook or a web client must implement these SMTP and IMAP commands to actually send and retrieve emails respectively.

1. Now lets check the email using IMAP. Again connect using telnet:   
`root@telnet:/# telnet`
2. But this time connect to mailserver on port 143, which is the well-known TCP port for IMAP.   `telnet> open mailserver 143`  
If it works, you should see a line of text starting with:  
`* OK [CAPABILITY...`
3. Once connected you can login a user `myuser` with the pre-set password `mypassword`:  
`a login myuser@mycompany.com mypassword`  
You should see another `OK` response.
4. Now list the available mailboxes for this account:  
`a list "" "*"`  
You should see all the mailboxes setup for the *myuser* account:  
```
* LIST (\HasNoChildren \Sent) "." Sent
* LIST (\HasNoChildren \Trash) "." Trash
* LIST (\HasNoChildren \Drafts) "." Drafts
* LIST (\HasNoChildren \Junk) "." Junk
* LIST (\HasNoChildren) "." INBOX
a OK List completed (0.001 + 0.000 secs).
```
5. Lets check the inbox!  
`a examine inbox`
```
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS ()] Read-only mailbox.
* 1 EXISTS
* 1 RECENT
* OK [UNSEEN 1] First unseen.
* OK [UIDVALIDITY 1540413590] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
a OK [READ-ONLY] Examine completed (0.001 + 0.000 secs).
```
There's our one email we sent via the SMTP service!  

6. Let's read that email: 
`a FETCH 1 BODY[]`
You should see your test email that you sent through the server. Not as pretty as GMail! Haha

```
* 1 FETCH (BODY[] {1393}
Return-Path: <me@mycompany.com>
Delivered-To: myuser@mycompany.com
Received: from mail.mycompany.com
        by mail.mycompany.com with LMTP id GD2SE5bY0FtqUAAAScxBiw
        for <myuser@mycompany.com>; Wed, 24 Oct 2018 20:39:50 +0000
Received: from localhost (localhost [127.0.0.1])
        by mail.mycompany.com (Postfix) with ESMTP id 2BF9D12CC
        for <myuser@mycompany.com>; Wed, 24 Oct 2018 20:39:50 +0000 (UTC)
X-Quarantine-ID: <WwSMRRt2wJmJ>
X-Virus-Scanned: Yes
X-Amavis-Alert: BAD HEADER SECTION, Missing required header field: "Date"
X-Spam-Flag: NO
X-Spam-Score: 2.743
X-Spam-Level: **
X-Spam-Status: No, score=2.743 tagged_above=2 required=6.31
        tests=[ALL_TRUSTED=-1, MISSING_DATE=1.396, MISSING_FROM=1,
        MISSING_HEADERS=1.207, MISSING_MID=0.14]
        autolearn=no autolearn_force=no
Received-SPF: None (mailfrom) identity=mailfrom; client-ip=192.168.144.3; helo=mycompany.com; envelope-from=me@mycompany.com; receiver=<UNKNOWN>
Authentication-Results:mail.mycompany.com; dkim=permerror (bad message/signature format)
Received: from mycompany.com (lab-k_telnet_1.lab-k_default [192.168.144.3])
        by mail.mycompany.com (Postfix) with ESMTP id E0684127C
        for <myuser@mycompany.com>; Wed, 24 Oct 2018 20:38:27 +0000 (UTC)
Subject: Test Email
Message-Id: <20181024203950.2BF9D12CC@mail.mycompany.com>
Date: Wed, 24 Oct 2018 20:39:50 +0000 (UTC)
From: me@mycompany.com

My Test Email
)
a OK Fetch completed (0.001 + 0.000 secs).
```
7. A lot of what you see in the *raw* email message are actually **headers**.  They include information like who the message is to/from, either the message was identified as SPAM, the date/time is was sent, and the message id. All this stuff helps the mail client determine what to do with this message and how to display it to us.
8. We are finished reading mail, let's exit IMAP and the telnet client:
`a logout`
9. And exist the container and return to PowerShell:  
`root@telnet:/# exit`

## Closing note on Telnet as a client

Using telnet is no way to send and read your email. Most people use an email client, which is software that takes your actions and formulates the SMTP and IMAP commands for you

For example when you compose a new email and hit send, your email client issues the same SMTP commands we typed into Telnet to your SMTP server on your behalf. This is hidden from us and the email just seems to send... which is a really good thing for us! Same thing is true for listing mail folders and reading messages. The client translates our actions (clicking on a message) into  IMAP commands to fetch the message.

The value of Telnet is it helps you understand how the protocol works. You can use telnet to learn other TCP protocols such as HTTP. Telnet can be used to trouble issues with protocols help you to determine the problem is with the client's implementation of the protocol. Something I've done a couple of times in my career. 

## Using an actual mail client. 

Now let's try an actual mail client. I think most of you understand how to use an email client, so instead we will think about how it works under the hood using SMTP and IMAP. The `roundcube` container is running a web-based email client on port TCP/8080. We should be able to use it to login and send and receive emails.

1. Open [http://localhost:8080](http://localhost:8080) in your browser. 
2. You should see the roundcube login. You can login with **myuser@mycompany.com** and **mypassword**. Sound familiar? They are the same username and password we used to connect to IMAP!
3. Once logged in you should be able to see your test message! The client hides all of the details of the IMAP protocol from you. Notice when you click on the **Test Email** you don't see all of the headers we saw when we used the protocol. Most of those headers are intended for the client to use to present this message to us.
4. If you want to see the headers, in this client you can by clicking on **more ...** then selecting **Show Source** fromt he menu. The mail headers will open in a new browser tab.
5. Let's login as another user so we can send more email! Open an incognito browser window or a completely different browser and navigate to [http://localhost:8080](http://localhost:8080) again.
6. This time, login with the other account **me@mycompany.com** (password: **mypassword**).
7. Try sending this message to **myuser@mycompany.com**.   
Subject: Another email.    
Body: This is another email!  
8. Click **Send**. When you send this email, the client actually does TWO things: 1) It uses SMTP to send the email to **myuser@mycompany.com** and then 2) It uses IMAP to save a copy of the email in the Sent folder. 
9. Open the **Sent** folder to verify. You should see the email you sent in that folder. 
10. Switch back to the other browser where you are logged in as **myuser@mycompany.com** has the new message arrived yet? How does the client know to check for new messages? The answer is it asks the IMAP server every XX seconds. That right to make new email magically arrive in your inbox, the client must connect to the mail server over IMAP and execute an `a examine inbox` command!

## Concluding remarks

That is about the limit of what you can do with our simple mail setup. It should be noted that production ready setups address many other factors suitable for email in production. Just a few issues with this setup:
1. We don't need to authenticate to use SMTP. This is a spammer's dream come true! 
2. We did not use TLS (transport-layer security) encryption for SMTP or IMAP. This is the equivalent to HTTPS for email, encrypting messages over the network.
3. We do not have a junk mail / anti-spam system setup to prevent unwanted or dangerous messages from hitting user's inboxes.
4. There's no directory service integration. If you want to send someone and email, you must know their email address! 
5. We have no scaling strategy. Typically we scale SMTP with a round-robin configuration. We scale IMAP by distributing users mailboxes over multiple servers.

## Tear-down

To tear down this lab:
`PS C:\ist346-labs\lab-K> docker-compose up down`  

## Questions

1. What is SMTP? IMAP? Which one sends mail and which retrieves it?
2. For each of the following commands explain what they do and which protocol they are part of:
   1. EHLO 
   2. A LIST 
   3. MAIL
   4. A FETCH 
   5. telnet
3. What is the value of using Telnet to connect to any TCP protocol?
4. Web browsers implement the client version of the HTTP protocol. Based on what you did in this lab, what do you think the browser is doing with respect to the HTTP protocol when we as it to load a website?
5. Which protocol(s) are used to list messages in a folder?
6. When you send an email from an actual mail client, which protocol(s) are used and why?
7. What are the purpose of the email message headers?
8. What are some concerns which need to be addressed in a production mail setup?