---
title: Be a Task Warrior - Setup Taskserver
author: Yogesh Singh
date: 2023-02-26T11:48:24+05:30
categories: [posts]
tags: [taskwarrior, productivity, taskserver]
---

[Taskwarrior](https://taskwarrior.org/) is a tool to manage your TODO list in a clean and efficient manner from multiple devices, including the command line, android or even web apps. I have been using Taskwarrior on my work machine for some time and has been helping me to keep track of my tasks and TODO list.

Recently I though of trying my hands on Taskserver. It's a server for Taskwarrior to sync data between muliple devices/clients which is helpful if you want to track your list using multiple devices and reduces the effort ot duplicate data everywhere. 

I wanted to be able to sync my data between my android phone and my work machine. So I decided to run the server on a DigitalOcean droplet. Below are the steps I followed to setup Taskserver for my use. 

### Installation
Install Taskserver (`taskd`) using any of the following methods:

 - via the package manager (I went ahead with this - *I am lazy*)
 - install from a tarball
 - build it yourself

### Configuration
After installing taskd, you need to configure it in order to be able to use it.
The first step is to add an environment variable `TASKDDATA`

    export TASKDDATA=/var/taskd
This needs to be available always while running taskd, so it's better to add it to your *.bashrc* or *.zshrc*

    echo "export TASKDDATA=/var/taskd" >> .bashrc

#### Generate Certificates
Next we are going to generate certificates for Taskserver and also for the clients for to be able to connect to the server.

It's better to copy the `pki` directory from your SOURCEDIR to your TASKDDATA directory. If you installed from a package (manager) search for the pki directory, `find / -name pki -type d` (example `/usr/share/taskd/pki/` for Ubuntu - this works for me). You need to edit the *vars* file in the *pki* directory 

    sudo cp -r /usr/share/taskd/pki $TASKDDATA/
    cd $TASKDDATA/pki
    nano vars

Change the following line:

    CN={your_server_ip_goes_here}

Next run the generate certificate script

    ./generate

This is going to create self-signed certificates for your server. Once the certificates are generated copy them to the *TASKDDATA* directory.

    cp ./*.pem $TASKDDATA/
Note that at least the `ca.cert.pem` and `ca.key.pem` must remain in the *pki* folder for the user-certificate generation later on.
Configure *taskd* to read and start with the self-signed ceritificates generated

    taskd config --force client.cert $TASKDDATA/client.cert.pem 
	taskd config --force client.key $TASKDDATA/client.key.pem 
    taskd config --force server.cert $TASKDDATA/server.cert.pem 
    taskd config --force server.key $TASKDDATA/server.key.pem 
    taskd config --force server.crl $TASKDDATA/server.crl.pem 
    taskd config --force ca.cert $TASKDDATA/ca.cert.pem
Add the following basic configuration for taskd to choose the port and keep log files

    cd  $TASKDDATA/..
    taskd config --force log $PWD/taskd.log
    taskd config --force pid.file $PWD/taskd.pid
    taskd config --force server localhost:53589

Now launch the server

    taskdctl start

#### Setup Client
Now you need to add an organisation and user under the organisation to be able to use and connect to the Taskserver.

    taskd add org {org_name_goes_here}
    taskd add user {org_name_goes_here} {user_name_goes_here}
The last command would return a UUID against the user something like 
`New user key: cf31f287-ee9e-43a8-843e-e8bbd5de4294`. You need to keep this user key handy while connecting from the client.

Next we will generate the self-signed certificates for the user to be able to connect to the server.

    cd $TASKDDATA/pki
    ./generate-client {user_name_goes_here}
This again throws some output and generates the certificates for the user.

#### Connect the client
The following files and info is needed to connect the client to the server.

- `ca.cert.pem` generated above for the server.
- `{user_name}.cert.pem` for the client
- `{user_name}.key.pem` for the client
- Organisation name
- User key generated for the client
- Server IP and Port defined in the above config

**Command Line client**
Copy the above certificates to the `.task` config folder in your home directory. 

    cp {user_name}.cert.pem ~/.task
    cp {user_name}.key.pem ~/.task
    cp ca.cert.pem ~/.task
Configure taskwarrior to pick the certificates and server configuration

    task config taskd.certificate -- ~/.task/first_last.cert.pem
	task config taskd.key -- ~/.task/first_last.key.pem
	task config taskd.ca -- ~/.task/ca.cert.pem
	task config taskd.server -- {ip_address}:53589
	task config taskd.credentials -- {org_name}/{user_name}/{user_key}
	# I had to add the below to ignore hostname verification in certificates
	task config taskd.trust=ignore hostname

To verify that the server configuration has been setup correctly run:

    task sync init
Once successfull, you can start managing your TODO lists from anywhere anytime. You can setup cron jobs to run periodically to sync tasks automatically.
This marks one item crossed on my list.

See you until the next note ...
