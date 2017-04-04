= Step-by-step guide to Linux security
Clément Levallois <clementlevallois@gmail.com>
2017-04-03

last modified: {docdate}

:icons!:
:asciimath:
:iconsfont:   font-awesome
:revnumber: 1.0
:example-caption!:
:sourcedir: ../../../main/java

//ST: 'Escape' or 'o' to see all sides, F11 for full screen, 's' for speaker notes

== Ordering the server
//ST: Ordering the server

- Server ordered on Hetzner.de
- I use Debian, version 8.7
- Vi is used as a text editor in the following
- we are logged as root first

== Disabling password authentication, enabling SSH
//ST: Disabling password authentication, enabling SSH

Password authentication is less secure than SSH public key. A password transits through the Internet for the auhtentication, it can be hacked at this step.

A SSH private key is not transmitted on the wire. So, it can't be hacked this way.

A detailed explanation is https://security.stackexchange.com/questions/69407/why-is-using-an-ssh-key-more-secure-than-using-passwords[available here].


//ST: !
==== How to generate a SSH key?

- On Windows, use https://docs.joyent.com/public-cloud/getting-started/ssh-keys/generating-an-ssh-key-manually/manually-generating-your-ssh-key-in-windows[Puttygen].
- On Mac, use https://docs.joyent.com/public-cloud/getting-started/ssh-keys/generating-an-ssh-key-manually/manually-generating-your-ssh-key-in-mac-os-x[the Terminal]
- On Linux, use the https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html[ssh-keygen command]

//ST: !
==== How to disable password auth and enable SSH?

-> the server I rented on Hetzner asked it from the console right when renting it.
-> otherwise, this can be set up manually https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-mac-disable-ssh-password-usage[this way].

(see also the "all commands in order" tutorial for precise instructions)

//ST: !
==== Changing the SSH port

By default, loggging to the server via SSH is done on the port 22. Knowing that, attackers scan the port 22. Changing the port to a different one makes the attacker's job more difficult. To do that:

 vi /etc/ssh/sshd_config

Text to change in the file: change port SSH 22 by a new port (*let's say 1234*), write the new port down somewhere

 service sshd restart


== Setting up a firewall
//ST: Setting up a firewall

A firewall gives you control on what can enter and leave your server.

//ST: !

==== ip tables

The rules for setting up ip tables are logical https://help.ubuntu.com/community/IptablesHowTo[but quite complicated]. Using an https://www.perturb.org/content/iptables-rules.html[ip tables generator] could help.

But there is an even easier alternative.

//ST: !

==== better: uncomplicated firewall

Following https://twitter.com/mgilbir[@mgilbir]'s advice, I'll use https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29[ufw: a linux package for "uncomplicated firewall"]. To install it:

 apt-get install ufw

The firewall is now installed, but is is not active yet.

//ST: !
We add a rule to block all incoming traffic, except for SSH connections through the port we defined:

 ufw default deny incoming
 ufw allow 1234/tcp

 //ST: !

Now, we can activate the firewall

 ufw enable


== Creating users and disabling SSH connections for root
//ST: Creating users and disabling SSH connections for root

We should now disable root login via SSH. Why? Because attackers would know that a "root" user is available to log in, and it just remains to attack its password.

With the root user disabled at the SSH login step, the attackers must guess *both* the username and its password to access the connection, and that's much harder.

Of course, an attacker who aims at you or your server specifically (a "targeted" attack) would expect a series of usernames (in my case "seinecle", the name I use on all social media), so don't use it either.

//ST: !



== the end
//ST: The end!

//ST: !

Author of this tutorial: https://twitter.com/seinecle[Clement Levallois]

All resources on linux security: https://seinecle.github.io/linux-security-tutorials/
