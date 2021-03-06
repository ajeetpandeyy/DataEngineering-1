# Server Login Tutorial
CDS has several servers, the most important of which are `master` and `gpu1`. This document explains how to log into them. For first time setup, be sure to read the corresponding section.

Reminder: exercise caution when modifying system files. If you require help, please contact Haram Kim.

## Using the VPN

In order to ensure security as well as enable off-campus access of servers, Cornell requires you to use the Cornell VPN to connect to our server - *even on campus* (at least for `gpu1`)! To install and use the Cornell VPN, follow Cornell's instructions [here](https://it.cornell.edu/cuvpn).

## Login Procedure
As a member of CDS your work on the servers will fall under either an account associated with your project/subteam, or an individual account, depending on if you are using the CPU (`master`) or GPU (`gpu1`) servers. For  access to `gpu1`, contact Haram Kim or Nolan Gray to get an account. 

The following are the accounts to use for `master` and the related CPU servers.

The team-username mappings are given below:
* Data Engineering: serverteam_1
* Insights: datavis_1
* Intelligent Systems: deeplearning_1
* Algorithmic Trading: algo_1

**NOTE: You will need to follow the "First Time Setup" instructions before the following is likely to work.**

The syntax for logging in is as follows, with \[username\] replaced by your username and \[servername\] replaced with the ip address, domain name, or alias for the server you want to connect to:
```
ssh [username]@[servername]
```
Note that providing a username is optional if you already specified one in your SSH config file (see first time setup).
For example, an Intelligent Systems member connecting to `gpu1` would issue the following command:
```
ssh rjb392@gpu1
```

This will provide you with remote terminal access to the server. From there you can [issue commands](https://www.codecademy.com/courses/learn-the-command-line/lessons/navigation) to manipulate files and run programs.

# First time setup
The first time you want to log into the server, there are several steps to take to ensure that it will be as easy as possible to connect in the future. You will only have to do this once, so follow the instructions carefully and things will be easy in the future.

## Getting SSH
You will first need to install SSH. On linux this is easy, on mac its pretty easy, and on windows its kinda hard. You should google how to do this.

## Server IP addresses
The CDS Servers are hosted at the following locations:
* 128.84.48.178: master
* 128.84.48.177: slave-1
* 128.84.48.179: slave-2
* 128.84.48.180: slave-3
* gpu-server-1.cds.engineering.cornell.edu: gpu1

Note that `gpu1` does not have an ip address! This is because the cornell network controls the server's ip address, and has its own address for it: `gpu-server-1.cds.engineering.cornell.edu`.

## The hosts file
When you type `www.google.com` into your browser's URL, that request gets sent to a "Domain Name System" (DNS) server, which turns it into an ip address, like `172.217.19.238`.  That ip address is the real address of google's server, and is what the internet actually uses to communicate. Remembering it is hard for humans, which is why DNS lookups are nice.

The [hosts](https://en.wikipedia.org/wiki/Hosts_(file)) file is a file on your computer that can override such DNS lookups or provide custom ones. It is what will allow us to avoid having to remember the ip addresses of the CDS servers. It is located at `/etc/hosts` on Linux and Mac, and `C:\Windows\System32\drivers\etc\hosts` on Windows. The format is the ip address, followed by spaces or tabs, and then the domain name to map to that ip address.

You will need to add the mappings for `master`, `slave-1`, `slave-2`, and `slave-3` to your hosts file, which will allow you to use the domain names instead of IP addresses while logging in. Note that the `gpu1` cannot be added to the hosts file because it is not a mapping from domain name ip address. We will get around this later.

Your hosts file may look something like this:
```
# This is a comment, localhost is the name that refers to your own computer
127.0.0.1       localhost

# The following are CDS servers
128.84.48.178   master
128.84.48.177   slave-1
128.84.48.179   slave-2
128.84.48.180   slave-3
```

## Configuring your ssh config file
To further ease your ability to connect with the servers and also enable you to connect with `gpu1` without typing out the full address, you must configure your ssh settings further. Your [config file](https://linux.die.net/man/5/ssh_config) is located at `~/.ssh/config` (or wherever the ssh folder is on Windows).

First lets set up the necessary settings to connect to `gpu1`. We want to be able to connect to `gpu1` rather than the actual address which is `gpu-server-1.cds.engineering.cornell.edu` for convenience purposes, so to do so we will use the `HostName` option:
```ssh_config
Host gpu1
  HostName gpu-server-1.cds.engineering.cornell.edu
```
Now, lets set up some ease of life settings. Whenever using programs that open GUI windows, you can actually get those on your own computer through [X11 Forwarding](https://kb.iu.edu/d/bdnt). We also want to set up SSH agent forwarding so that you don't need to unnecessarily re-enter your ssh private-key password. Finally, we can avoid typing in a username by providing a default username to connect with. The following is what an Intelligent Systems user might set their `~/.ssh/config` file up as:
```ssh_config
Host gpu1
  HostName gpu-server-1.cds.engineering.cornell.edu
  User my_individual_username

Host gpu1 master slave-1 slave-2 slave-3
  User my_team_username
  ForwardAgent yes
  ForwardX11 yes
  ForwardX11Trusted yes
```
That will make things as easy as possible - connecting to gpu1 will only require you to run `ssh gpu1` as the command.

Note that in the above example, the `User` option is set for `gpu1` twice. Because this conflicts, the more specific rule (the one that only affects `gpu1`) overrides the less specific one (the one that affects all the servers). This lets us set up options that apply to most/all the servers but override the behavior for individual servers. For example, most of the servers use the team username logins, but `gpu1` uses individual accounts.

## SSH Keys
Normally when you ssh into a server, you have to enter in the password associated with the account you are connecting with. This is both insecure and cumbersome. A better system is to use SSH keys. SSH keys use a public-private key exchange system called [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) to securely connect. An RSA key consists of a public key and a private key. The public key, usually stored in `~/.ssh/id_rsa.pub` on your computer, is what you place in servers' `~/.ssh/authorized_keys` file. It provides a way for anyone to identify you, and your private key remains secure even when distributing your public key across the internet. Your private key, usually stored in `~/.ssh/id_rsa` on your computer, will be used to actually verify you are who you say you are. You should never share the private key with anyone else, and it is usually encrypted by its own password to keep hackers from using it even if they copy it off of your computer.

SSH keys mean that as long as you can access the unencrypted private key on your computer, you can easily log into any server that has your public key. This will be useful for us because it will allow us to avoid having to enter our password every time we try to do something. It also means you can use your public key for multiple things at once - for example, you can [put your public on github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/) and on the server too. The only password you will need to remember will be password encrypting your private key, if it exists.

### Generating an SSH key if you don't already have one
Your SSH keys will be stored in `~/.ssh`. If you see `id_rsa` and `id_rsa.pub` there already, skip to "Putting keys on the server".
To make your first ssh key, follow [these instructions](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-windows). Be sure to select a secure password! Once you finish, you should see both the public and private keys in `~/.ssh`. 

As an additional optional step to take if you're not on Linux, you might want to [configure SSH-agent](https://help.github.com/articles/working-with-ssh-key-passphrases/) to remember your private key after you enter your password once. SSH-agent is a process that runs and will safely store a secure version of your private ssh key after you have unencrypted it at least once with your password. If you don't have SSH-agent set up, you will still have to enter your password every time you commit in git or connect to the server, which is annoying. The way you enable ssh-agent depends a lot on whether you're on mac or windows and if you are using the built in command line or another application. This might take some googling to figure out.

### Copying public key to the server(s)
Any server that you anticipate that you will need to log in to, you must put your public key into the server's `~/.ssh/authorized_keys` file. To easily do this, first ensure you have the ssh keys generated. Then run the following:
```
ssh-copy-id [username]@[servername]
```
Where \[username\] is your team's username and \[servername\] is the server you want to put the key on. You can test it by trying to ssh into the server again with the `-v` option and seeing if it uses your key or not.

# Starting the server remotely
Under normal conditions, the server will stay on forever except in the instance of a superuser invoking `restart`. Generally speaking, this ensures that we can always ssh into the server. However, in the event of power loss server crash, or accidentally running `shutdown -h`, the server will not be accessible via ssh as it will be completely powered off. Luckily there is a mechanism in place to allow remotely powering on the system. The system should (in theory) automatically turn on within 24 hours, but if you need it you probably don't want to wait that long. To start it up without physical access, use Wake-on-LAN. Note that this might be the only option, as the physical power button on `gpu1` has been disabled to avoid passerby disabling the server.

## Wake-on-LAN
[Wake-on-LAN](https://en.wikipedia.org/wiki/Wake-on-LAN) is a system that enables computers equipped with capable motherboards to power themselves on when detecting a "magic packet", which is a special type of packet that tells the motherboard to power on. It is frequently sent as a [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) packet to either port 0, 7, or 9. When the motherboard receives the packet, it will then power on the computer. Typically, magic packets can only be sent on the same [LAN](https://en.wikipedia.org/wiki/Local_area_network) as the computer. However because IT has opened all ports to the server, we can actually send the packet remotely.

## How to send the magic packet
There is a unix utility called `wakeonlan` that will send a magic packet to a given IP and [MAC address](https://en.wikipedia.org/wiki/MAC_address). First, install the program with `sudo apt install wakeonlan`, or the equivalent method on your OS. Then, identify the MAC address of the server. You can get the ip address and MAC address for each of your network adapters by running `ip addr`. Here is a list of the MAC addresses of the different servers:
```
00:25:90:69:c4:2a    master
00:1e:67:06:a4:f8    slave-1
00:1e:67:06:a4:fc    slave-2
00:25:90:69:c4:7a    slave-3
10:7b:44:94:7c:fa    gpu1
```
Once you have identified the ip address and MAC address of the target, run the following command:
```
wakeonlan -i [ip_address] [mac_address]
```
That will send a magic packet to port 9 for the given IP and MAC address. Give the server a minute to start up, then try ssh-ing. It should (hopefully) greet you with a successful ssh login!
