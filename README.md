# UDACITY - Full Stack Web Developer Nanodegree

*Project: Linux Server Configuration*

Marlon Card

## Description
For this project we are to configure & secure a linux server and use it to deploy our previously completed [catalog application](https://github.com/marloncard/udacity-item-catalog). The Udacity recommended provider was [Amazon Lightsail](https://aws.amazon.com/lightsail/).

## Contents
1. [Project Requirements](#project-requirements)
2. [Get Server](#get-server)
3. [Connect to Instance](#connect-to-instance)
4. [Update](#update)
5. [Secure](#secure)
6. [Create User](#create-user)
7. [Deploy Application](#deploy-application)
8. [Finalize and Debug](#finalize-and-debug)
8. [Tips](#tips)
9. [References](#references)
10. [License](#license)

## Project Requirements
1. SSH Key allowing login as `grader`
2. Cannot login as `root`
3. `grader` can run `sudo` commands on root only files
4. Firewall only allows port 2200 (SSH), port 80 (HTTP) and port 123 (NTP)
5. Key based authentication
6. System packages are updated
7. SSH is not hosted on the default port
8. Web server is running on port 80
9. Server is configured to serve data (PostgreSQL)
10. Web server is configured to serve the application as a WSGI app.
11. README file on github

## Get Server
I used the recommended [Amazon Lightsail](https://aws.amazon.com/lightsail/) which provides 1 month free as of this writing. The lowest tier linux plan was sufficient for the project.
* Once you've signed up the first step is to create an instance.
* I used the OS only blueprint with Ubuntu 16.04 LTS

## Connect to Instance
You can initially use the "Connect using SSH" button within your instance to quickly launch Lightsails browser-based SSH client, but that's only temporary since you'll need to change the SSH port, rendering the client inoperable. Long term you'll need to use a third party client.

![SSH Info](https://i.imgur.com/c08TbVn.png)

### Prerequisites
Regardless of Operating System you'll need to have the following ready:
1. Instance IP Address
2. Username (usually "ubuntu")
3. Private & Public Keys
  * The first option is to download the [default private key](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-ssh) from your account profile.
  * Your profile also allows you the option to [generate a new key](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-ssh)
  * Finally you can generate a key pair locally on your [MAC OS/Linux](https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/create-with-openssh/) or [Windows](https://www.digitalocean.com/docs/droplets/how-to/add-ssh-keys/create-with-putty/) system.

### SSH Clients

**MAC OS / LINUX:**
* Launch terminal
* At the prompt run: `ssh -p <port> -i <private_key_file> <user>@<ip_address>` (e.g., `ssh -p 22 -i ~/.ssh/id_rsa ubuntu@35.182.80.55`)

**WINDOWS:**
* A popular option is the [Putty](https://www.putty.org/) client
* Lightsail has pretty good [instructions](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh) on how to use

## Update
The first thing you'll want to do when you've successfully logged into your server is to update
* Run `sudo apt-get update` to update your package lists
* Then perform `sudo apt-get upgrade` to actually update the software

## Secure
The next step which is good to do early on to secure and change your ports if necessary. This was especially good advice as I found out it's very easy to lock yourself out of your instance; if that happens, you will have to delete and create a new one.

1. Edit your SSH Config (I skipped this initially and found out the hard way why it's important).
  * In the terminal run: `sudo nano /etc/ssh/sshd_config`
  * Look for "Port 22" and change to "Port 2200"
  * While you're in the `sshd_config` file there are a couple other things you should do that have nothing to do with ports but are necessary for security. Look for the line `PasswordAuthentication yes` and change to `no` to disable password authentication. Find `PermitRootLogin` which  might have a value of `yes` or `prohibit-password`. I changed to `no`.

2. Check your UFW status: `sudo ufw status` (it should be 'inactive') and then run the following commands:
  ```
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2200
  sudo ufw allow 80
  sudo ufw allow 123
  sudo ufw enable
  ```
3. Configure the Lightsail firewall in your control panel to match the ufw ports you've allowed.
![Lightsail Ports](https://i.imgur.com/fpJRaYR.png)

4. Exit and restart your SSH session, this time using port `2200`

## Create User
Next we need to create a new user, `grader` to give the Udacity grader access to check our server. I used the following commands:
```
sudo adduser grader
```
Copy ubuntu user profile located in `/etc/sudoers.d`; in my case '90-cloud-init-users' (I verified this was for the ubuntu user with `sudo cat /etc/sudoers.d/90-cloud-init-users`; the output was `ubuntu ALL=(ALL) NOPASSWD:ALL`)
```
cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
```

I now needed to generate an SSH key pair for the user `grader`; I did that locally using the following commands:

```
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): grader
```

A private key, `grader` and public `grader.pub` are created in my directory ~/.ssh/ after which I ran: `less grader.pub` and copied it's contents.

I then logged into the server with the `ubuntu` user and ran the following:
```
sudo nano ../grader/.ssh/authorized_keys
```
I then pasted the entire public key, saved, exited and completed with following commands:
```
sudo chmod 700 ../grader/.ssh/
sudo chmod 644 ../grader/.ssh/authorized_keys
sudo chown grader ../grader/.ssh/
sudo chgrp grader ../grader/.ssh/
sudo chown grader ../grader/.ssh/authorized_keys
sudo chgrp grader ../grader/.ssh/authorized_keys
```

## Deploy Application
With the main server configuration complete I now needed to actually deploy the application which consists of installing Python, [Apache](https://www.apache.org/), setup of PostgreSQL, creating the catalog database, modifying  application code to use PostgreSQL (I initially used SQLite3), installing prerequisite packages from the Vagrant VM environment, cloning the app's github repo, setup of WSGI and debugging.

### Install Python

I began with installing Python (Version 2.7 to match my development environment):
```
sudo apt-get install python2.7
```

### Apache

To install Apache and the [mod_wsgi](https://en.wikipedia.org/wiki/Mod_wsgi) module:
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
```
You can confirm it's working correctly by visiting your server's IP Address in a browser where you'll see the Apache confirmation; the file itself is located here: `/var/www/html/index.html`

### PostgreSQL

Next install PostgreSQL:
```
sudo apt-get install postgresql
```
I then ran the following commands to log into the `postgres` user, starting up the psql terminal, and then create the database and user to which I assigned privileges to peform operations on the database:
```
sudo su -postgres
psql
postgres=# create database catalogdb;
postgres=# create user catalog with encrypted password 'password';
postgres=# grant all privileges on database catalogdb to catalog;
postgres=# \q
```

### Install Pip

[Pip](https://pip.pypa.io) is Python's package installer and is needed to install the remaining packages from your development environment. I ran the following:

```
sudo apt-get install python-pip
```

### Install Prerequisites

To install the packages from your development environment (in my case a Vagrant VM running Ubuntu), from the development environment run the following from within your project folder:
```
pip freeze > requirements.txt
```
At this point you can add the generated `requirements.txt` file to your git and github repo then clone it to your server with the rest of the project files (which I'll describe how to do shortly).

Once this file is on your server run:
```
sudo pip install -r requirements.txt
```
A word of warning, I initially left off `sudo` while installing packages which resulted in Python not being able to import them. Once I figured out what was happening I uninstalled all and reinstalled with `sudo` which fixed the issue. It's also important to take note of errors because you'll have to continue installing the remaining packages after the failure.

### Install Git

My git was preinstalled but if not run the following:
```
sudo apt-get install git
```

### Modify Code

In order to prepare my application for the postgreSQL database I made the following changes within my development environment, created a new branch called 'deploy', committed and pushed to github. (I also fixed some column length issues in models.py which SQLite was apparently ignoring).

`data.py`
![data.py changes](https://i.imgur.com/abNloPk.png)

`models.py`
![models.py changes](https://i.imgur.com/ucN46uH.png)

`views.py`
![views.py changes](https://i.imgur.com/f4JYoxW.png)

### Clone Repository

I performed the following commands to clone the app repository onto my server:

```
cd /var/www/
sudo git clone https://github.com/marloncard/udacity-item-catalog.git catalog
cd catalog/
sudo git checkout deploy
```

### WSGI Setup

The Web Server Gateway Interface (WSGI) allows your web server to forward requests to web apps written in python. To start off I create a `myapp.wsgi` file in my catalog directory with the following code:
```
import sys

sys.path.insert(0, "/var/www/catalog")

from views import app as application

application.secret_key='<your_key_here>'
```

Next I created a conf file `sudo nano /etc/apache2/sites-available/catalog.conf` with the following content:

```
<VirtualHost *:80>
    ServerName 35.182.81.85
    ServerAlias catalog.marloncard.com
    WSGIScriptAlias / /var/www/catalog/myapp.wsgi

    <Directory /var/www/catalog>
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

I ran the following to disable the default conf file (for me `000-default.conf`) and enable my newly created `catalog.conf`
```
sudo a2dissite 000-default.conf
sudo a2ensite catalog.conf
```

### SSL Certificate
I decided to add an ssl certificate to make my login functional and I went with the [Let's Encrypt](https://letsencrypt.org) service (because it's free).

Add port 443 to your allowed ports in ufw:

```
sudo ufw allow 443
```

Make the same changes in your Lightsail firewall and then run the following:

```
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot python-certbot-apache
```

Then just run the following which will prompt you for information to complete the certificate creation process.
```
sudo certbot --apache
```

## Finalize and Debug

Reload Apache with `sudo service apache2 reload` and technically the site should now be viewable in a browser at `35.182.81.85` or catalog.marloncard.com but I had an error because there were a few more changes to make which consisted of entering the full path to all secret key & password json files:

`models.py`
![models.py](https://i.imgur.com/LYwXCm9.png)

`views.py`
![views.py](https://i.imgur.com/xkrGDYk.png)

![views.py](https://i.imgur.com/5qNw9aW.png)

I also needed to prevent access to the `.git` folder which I did with an .htaccess file in the project directory:

`.htaccess`
![.htaccess](https://i.imgur.com/VYBvQsK.png)

Success! The application is now visable at https://catalog.marloncard.com or 35.182.81.85


## Tips
* Lightsail allows you to create snapshots of your instance as a backup in case you're concerned about destroying your setup (I located this feature after locking myself out of the instance for the second time while configuring ports)
* Lightsail can generate SSH keys for you in your profile, if you're having trouble with the locally created keys. You just need to download the private, move it to your `~/.ssh` folder and login.
* The Apache error log is indispensable towards the end of your deployment; it's located here: `/var/log/apache2/error.log`

## References

* [Creating user, database and adding access on PostgreSQL](https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [How do I prevent apache from serving the .git directory?](https://serverfault.com/questions/128069/how-do-i-prevent-apache-from-serving-the-git-directory)
* [CertBot](https://certbot.eff.org/lets-encrypt/ubuntuxenial-apache)

## License
This repository and its content was created by Marlon Card and is covered under terms of the [MIT License](LICENSE).
