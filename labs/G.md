# Lab G - Scalability

## Learning Objectives

In this lab you will:

- Learn about horizontal scalability.
- Understand how a load balancer works.
- Learn to consolidate logging in a horizontally scaled environment.

### Lab Setup At A Glance

In this lab we will demonstrate how applications are scaled horizontally. Specifically we will run a web application built using the [Flask Web Framework](http://flask.pocoo.org) through a load balancer. We will use the very versatile [Nginx](https://docs.nginx.com/nginx/) as a load balancer. It is a very fast, reliable load balancer which comes with a variety of configurable algorithms for balancing load across servers on a network. We will spin up multiple instances of the same Flask web application behind the load balancer in so that we can get a clear picture of how traffic is distributed by a load-balancing application like Nginx.

```
          +----LOAD BALANCER---+   +---CLIENT:BROWSER----+   ++++++++++++++
          | Docker: nginx      |---| host computer       |---|  Internet  |
   +------|http://nginx:80     |   | http://localhost:80 |   ++++++++++++++
   |      +--------------------+   +---------------------+  
   |
+----SERVER:Flask----+
| Docker: webapp     |--+
| http://webapp:8080 |  | 
+--------------------+  |
   | 1 or more instances|  
   +--------------------+
```
## Before you begin 

### Prep your lab environment. 

1. Open the PowerShell Prompt
2. Change the working directory folder to `ist346-labs`  
`PS > cd ist346-labs`
3. IMPORTANT: This lab requires access to Docker's internals, you must enter this command:
`PS ist346-labs> $Env:COMPOSE_CONVERT_WINDOWS_PATHS=1`
3. Update your git repository to the latest version:  
`PS ist346-labs> git pull origin master`
4. Change the working directory to the `lab-G` folder:  
`PS ist346-labs> cd lab-G`
5. Start the lab environment in Docker. This version is just a single instance of the web app:  
`PS ist346-labs\lab-G> docker-compose -f one.yml up -d`
6. Verify your `nginx` load balancer and three instances of your `webapp` are up and running:
`PS ist346-labs\lab-G> docker-compose -f one.yml ps`  
You should see `nginx` has tcp port `80`. The `webapp` is running on tcp port `8080`, but only exposes itself to the proxy.
8. Open a web browser like Chrome on your host. Enter the address `http://localhost:80` and you will see the sample web application:  
![sample web application screenshot](images/lab-g-webapp.png)  

**NOTE:** The webpage is designed to reload every 3 seconds so that you can see what happens on subsequent HTTP requests. No need to hit the refresh button in your browser!

### Understanding the Sample Web Applications

There's some really important information page of the **Sample Web Application**. It's designed to help you understand the behavior of the load balancer.

- **HOSTNAME** This indicates the host name of the running `webapp` docker container which served the page. Currently on page refresh we get the same host name each time. That's because there is only one `webapp` container running on the backend of the load balancer.
- **IP Address** This displays the IP Address of the docker container which served the page. Again you will see the same IP address on each page refresh because we only have one running instance of the `webapp` container.
- **Current Date/Time** This displays the current date/time on the server. This information should change with each request. Its primary purpose is to help you see that new content is being loaded with each request.

### Scaling the app

Let's scale our app to 3 instances and then observe what happens.

1. To scale the `webapp` service so there are 3 instances (instead of 1), we  must bring down the current application:
`PS ist346-labs\lab-G> docker-compose -f one.yml down` 
1. Then we bring up a different configuration that supports load-balancing, type:
`PS ist346-labs\lab-G> docker-compose -f roundrobin.yml up -d  --scale webapp=3` 
1. Go back to your browser running the **Statistics Report** and click the `Refresh Now` link. You should now see 3 instances of the `lab-g-_webapp` container. We have scaled the app.
1. Go back to your browser running the **Sample Web Application** on `http://localhost:80`. Notice now when the page refreshes, on each request you get one of three different HOSTNAMEs and IP Addresses. 
1. The load balancer cycles through each of the 3 docker containers in a predictable pattern. That's because the load balancer is configured to distribute the load using the `roundrobin` algorithm. You can learn more about it here: [https://en.wikipedia.org/wiki/Round-robin_scheduling](https://en.wikipedia.org/wiki/Round-robin_scheduling).

### Turning it up to 11

If your app is designed to scale horizontally, then you should be able to scale it infinitely. Large Hadoop clusters, a system designed to scale horizontally, have thousands of nodes. The problem, of course, is writing an app to scale horizontally is non a trivial task. The biggest issue around making data available to each node participating on the back end and dealing with updates to that data.
## Different Load Balancing Algorithms

As we explained in the previous section the default load balancing algorithm uses **round robin**. Let's explore two other load balancing algorithms `leastconn` and `ip_hash`.

### leastconn

The leastconn algorithm selects the instance with the least number of connections. If a node is busy serving a client, the next request will not use that node but instead select another available node. Let's demonstrate this.

1. First bring down your existing roundrobin setup, type:  
`PS ist346-labs\lab-G> docker-compose -f roundrobin.yml down`
1. Then start the environment using the leastconn configuration:  
`PS ist346-labs\lab-G> docker-compose -f leastconn.yml up -d --scale webapp=3`
1. Now let's return to the browser where we have **Sample Web Application** running on `http://localhost:80` at first glance, it seems to work the same as before, rotating evenly through each instance.
1. Let's put that to the test. We are going to access another url which keeps the instance busy so that the other browser cannot use that connection. Open another browser in a new window (`CTRL` + `n` does this). And arrange the windows so you can see both at the same time. In the new window let's request the **Sample Web Application** but instead use this url: `http://localhost/slow/10` 
1. You'll see the browser accessing `http://localhost` now only uses ONE or TWO of the three instances. The other instance is busy fulfilling the `http://localhost/slow/10` request! When that request finishes, the other browser will once again use all three instances to fulfill requests.

## uri hash

The `uri` algorithm selects an instance based on a hash of the Uri (Uniform Resource Indicator). This algorithm differs from the `leastconn` or `roundrobin`, in that a given Uri will always map to the same instance. Let's see a demo

1. First bring down your existing least conn environment, type:  
`PS ist346-labs\lab-G> docker-compose -f leastconn.yml down`
1. Then start the environment using the hash configuration:  
`PS ist346-labs\lab-G> docker-compose -f hash.yml up -d --scale webapp=3`
1. Now let's return to the browser where we have **Sample Web Application** running on `http://localhost` notice how with every page load we get the same instance. This is because that Uri maps to the instance you see.
1. Let's try another Uri in the browser, type `http://localhost/a` you will get a different instance. If you re-load the page (you must do this manually) you will still get the same instance for this `uri`.
1. Let's try a 3rd Uri in the browser, type `http://localhost/b` you will get yet another different instance. If you re-load the page (you must do this manually) you will still get the same instance for this `uri`.
1. Going back to either `http://localhost`, `http://localhost/a`, or `http://localhost/b` will always yield a response from the same instance. That's how uri mapping works!

There are other algorithms which in this manner. Consider their applications. Imagine distributing load based on geographical location, browser type, operating system, user attributes, etc... This offers a greater degree of flexibility for how we balance load.

## Tear down

This concludes our lab. Time for a tear down!

1. From the PowerShell prompt, type:
`PS ist346-labs\lab-G> docker-compose -f hash.yml down`  
to tear down the docker containers used in this lab.

## Questions

1. How is horizontal scalability as demonstrated in this lab different from the vertical scalability of the previous lab?
1. What do we call multiple copies of the service we scale horizontally?
1. What are the two things Nginx does as explained in this lab?
1. Why is scaling out to more nodes/instances easier than scaling back to fewer nodes/instances?
1. Type the docker compose command to scale the service `myservice` to `7` instances.
1. How does the `leastconn` algorithm differ from the `roundrobin` algorithm? How are they similar?
1. What are some potential uses of the `uri` load balancer algorithm?
1. In spite of being load balanced, does our environment still have a single point of failure? If so, what is it? Do you have thoughts as to how can this be remedied?