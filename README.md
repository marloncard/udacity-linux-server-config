# UDACITY - Full Stack Web Developer Nanodegree
*PROJECT: Linux Server Configuration*
Marlon Card

## Description
For this project we are to configure & secure a linux server and use it to deploy our previously completed [catalog application](https://github.com/marloncard/udacity-item-catalog). The Udacity recommended provider was [Amazon Lightsail](https://aws.amazon.com/lightsail/).

## Contents
[Project Requirements](#project-requirements)
[Get Server](#get-server)
[Connect to Instance](#connect-to-instance)
[Update](#update)
[Secure](#secure)
[Create User](#create-user)

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
You can initially use the "Connect using SSH" button within your instance to quickly launch Lightsails browser-based SSH client, but that's only temporary since you'll need to change the SSH port rendering the client inoperable. Long term you'll need to use a third party client.

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
The next step which is good to do early on for a few reasons is to secure and change your ports if necessary. This was especially good advice as I found out it's very easy to lock yourself out of your instance; if that happens, you will have to delete and create a new one.

### Change SSH Port
1. Edit your SSH Config (I skipped this initially and found out the hard way why it's important).
  * In the terminal run: `sudo nano /etc/ssh/sshd_config`
  * Look for "Port 22" and change to "Port 2200"
  * Look for the line `PasswordAuthentication yes` and change to `no` to disable password authentication.

2. Check your UFW status: `sudo ufw status`; it should be 'inactive'
  * Run the following commands:
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
Copy ubuntu user profile located in `/etc/sudoers.d` named '90-cloud-init-users'. I was under the impression it would be called "ubuntu", like the username so this naming convention might be specific to Lightsail
```
cp 90-cloud-init-users grader
```
