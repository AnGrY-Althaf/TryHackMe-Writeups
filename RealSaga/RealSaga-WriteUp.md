
# The Real Saga

## Introduction

Welcome to Real Saga, a linux machine to test out web exploitation, linux privilege and even docker breakouts also.

## Info for HTB

### Access

Passwords:

| User  | Password                                        |
| ----- | ------------------------------------------------|
| angry | angry@ctf                                       |
| dev   | Need to switch from www-data(no need actually   |
| root  | No password need to do the privilege escalation |

### Key Processes

So basically the box is completely built in docker and uses a ubuntu to host it, it does have docker service running and inside the container a apache web server is running with wordpress cms.
For owning this machine players need to breakout from the docker conatiner to the host machine.

### Automation / Crons

Nothing on the automation

### Firewall Rules

No firewalls configured

### Docker

Have created a complete docker container to run the entire idea behind the machine, the container is the core of the Machine which hosts the web server.

### Other

### !!!!! If you guys need the complete docker files let me know. !!!!!! Otherwise you could get that from machine itself



# Writeup

The Real Saga is a Linux machine which is completely build for testing out Skills in web recon, CMS exploitation, Linux privilege escalation techniques and also Docker concept and docker breakouts. 


# Enumeration

## Nmap

```` nmap -sCV machine-ip ````

![](/1.png)

On the nmap result, We can see that port 25 and 80 is open.
Port 80 -> Means there is a Web Server running.


We can see there is a internal domain running which is saga.local
Add this domain to our hosts file.

After that we can visit the site http://saga.local

By using wappalyzer we can see that its running on Wordpress cms. 
Else you use whatweb to find out that ```` whatweb http://saga.local ````
You can tryout wpscan also here.


While doing directory enumeration we do get some directories like wp-content, wp-admin.
wp-admin which is login page for getting into the cms. Without username and password nothing can be done there.
Enumerating again we'll get sub directory inside the wp-content which is plugins.
From the there we can see all the plugins used in the wordpress.

After looking into that we can see a plugin easy-wp-smtp
Which is actually a vulnerble plugin.

Sensitive Data Exposure.
We could see SMTP logs through this plugin
https://www.wordfence.com/threat-intel/vulnerabilities/wordpress-plugins/easy-wp-smtp/easy-wp-smtp-by-sendlayer-230-exposure-of-sensitive-information-via-the-ui

On the Wp login page there function for resetting the password of the wp users. So if have the username we could actually send password reset link to the email of the user. With the help of the vulnerable plugin we can see the complete mail log, we can access the reset link from there. So now we have to find username. 

Doing some enumeration we get a username 'user4dave'.
So reset the password of the user4dave, we can get into the wordpress as user4dave.

But unfortunately he is not the admin user. But simply looking inside the wordpress we get another username 'wpadmin', which is possbily the wordpress admin user.

Do the same process again reset the password of the wpadmin and login into the cms.

# Foothold

After getting into the cms as admin we can actually edit files inside the CMS which will reflect the files in the server also.

Edit the plugin file which is actually php and we could access edited file from the ' /wp-content/plugins ' directory.
So remove complete content of the php file and input a php reverse shell there and access the file which will execute the reverse shell.


By using netcat we could get the shell

````nc -lvp port````


We Got into the server as www-data


# Privilege Escalation

Now next is escalating into other user or root user directly.

By finding out the suid binary. 
We'll get the 'find' binary.

With the help of GTFObins, we could easily escalate our privilege to root.
````find . -exec /bin/sh -p \; -quit````

We rooted the server.

Now its time for hunting flags

````cat /home/dev/user.txt````  Got the user Flag !!

````cat /root/root.txt```` Instead of root flag we got a message,  

#### Great Work .... But This Is'nt the REALSAGA Jump Out And TryHarder !!!

So This is'nt yet. By doing more enumeration inside shell as root. We can see file named docker.sock mounted. From there we could identify that we are inside a dcoker container. By going through message from the root.txt "Jump out and tryharder".

So maybe we need breakout the container and get into host.
By researching about the docker.sock, we can see some privilege escalation.

By just executing the command ````docker ps```` 
We can interact with docker daemon running on the host now.

https://book.hacktricks.xyz/linux-hardening/privilege-escalation/docker-security/docker-breakout-privilege-escalation

With the help of this article, we can actually create new privileged container   
````docker run --privileged --network host -v /:/mnt --rm -it ubuntu:18.04 chroot /mnt bash````
By executing this command we will jump into a new privileged container, Inside this we have the complete access to host file and and shell.

Find the final flag now
````cat /root/root.txt````

Congrats !!!
