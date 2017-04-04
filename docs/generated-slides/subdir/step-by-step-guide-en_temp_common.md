= Step-by-step guide to Linux security for beginners
Clément Levallois <clementlevallois@gmail.com>
2017-04-03

last modified: {docdate}

:icons!:
:asciimath:
:iconsfont:   font-awesome
:revnumber: 1.0
:example-caption!:

//ST: 'Escape' or 'o' to see all sides, F11 for full screen, 's' for speaker notes

== Ordering the server
//ST: Ordering the server

- Server ordered on Hetzner.de (based in Germany, dirt cheap, but without management.)
- I use Debian, version 8.7 (http://www.pontikis.net/blog/five-reasons-to-use-debian-as-a-server[why?])
- Vi is used as a text editor in the following
- we are logged as root first

== Make sure you have the latest version of all packages:

Do:

 sudo apt-get update && sudo apt-get upgrade

Because:

//ST: !

 apt-get update

-> refreshes the repositories and fetches information about packages that are available online.

 apt-get upgrade

-> downloads and installs updates for all installed packages - as long as it doesn't bother dependencies (install new packages, remove old ones or crosses a repo source (switch a package from one repo to another)).

(http://askubuntu.com/questions/639822/is-apt-get-upgrade-a-dangerous-command/639838[source])

//ST: !

Install this package to get the clock of your server right:

 aptitude install ntp

Then define your time zone (the one where your server is located).

This step helps when your server needs to be synchronized with other servers.


//ST: !
==== Changing the SSH port

By default, loggging to the server via SSH is done on the port 22. Knowing that, attackers scan the port 22. Changing the port to a different one makes the attacker's job more difficult. To do that:

 vi /etc/ssh/sshd_config

Text to change in the file: change port SSH 22 by a new port (*let's say 1234*), write the new port down somewhere

 service sshd restart


== Creating a user and disabling logging for root
//ST: Creating users and disabling SSH connections for root

We should now disable root login via SSH. Why? Because attackers would know that a "root" user is available to log in, and it just remains to attack its password.

With the root user disabled at the SSH login step, the attackers must guess *both* the username and its password to access the connection, and that's much harder.

Of course, an attacker who aims at you or your server specifically (a "targeted" attack) would expect a series of usernames (in my case "seinecle", the name I use on all social media), so don't use it either.

//ST: !

So the logic is the following: we will create a user with much less priviledges than the root user. Only this weak user will have the right to connect to the server.

The user will be "enough" for regular tasks on the server. If we need the admin rights of root to install stuff or else, we will *temporarily* switch from this weak user to root to execute what we need, but then revert back to this weak user.

This way, we limit greatly the exposure of root privileges to the outside.

The steps:

//ST: !
1. making sure we have installed the "sudo" command that will allow us to switch from a weak user to root.
2. creating a weak user
3. giving rights to this user to establish a connection to the server (not just act on it once logged)
4. removing the rights of root to connect to the server.


//ST: !
==== 1. Installing the sudo command:

apt-get install sudo


//ST: !
[start = 2]
==== 2. Adding a new user (let's call it "myUser")

 adduser myUser

Have a strong password ready. Other questions (email, phone...) can be left empty by just pressing enter.

[start = 3]
==== 3. Enabling server connections via myUser
*text to add* still in the file sshd_config:

AllowUsers myUser

//ST: !

Then restart the SSH service:

 service sshd restart

//ST: !
[start = 4]
====  4. Disabling connection through root

  vi /etc/ssh/sshd_config

Text to change in the file:

PermitRootLogin no

From there on, you cannot login to the server from root, only from myUser!


//ST:!
Let's try it. Create a new SSH session with myUser. Then:

Switch to root privileges:

 su -

(you must enter the root password at this step)

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

Logging through SSH rather than passwords can be hair rising because there are so many tiny details that can go wrong. There is a good chance that if you do it for the first time you will lock yourself outside the server.

So, do this before you can erase the server, of if you are confortable waiting that your provider will unlock it for you.

Steps:

//ST: !
1. Parameters to change in `/etc/ssh/sshd_config`:

ChallengeResponseAuthentication no

X11Forwarding no

UsePAM no

LogLevel DEBUG3 (this should be added, the parameter is not listed by default)

Save the file, then:

 service sshd restart

//ST: !
[start= 2]
2. Add your public key to `/home/myUser/.ssd/authorized_keys`

- make sure you have put the keys in /home/myUser/.ssd/authorized_keys (not just in the root user folder)
- make sure your key starts with "the "ssh-rsa" (the first "s" might be missing ...)
- triple check the key doesn't break in several lines
- do `chmod 700 ~/.ssh` on the home folder

 //ST: !
[start= 3]
3. What will probably happen:

Your private key will probably not be recognized the first time because of some problems above not completely fixed.

Keep trying to log with your SSH key. To find the cause of your issues, inspect the log for auth operations:

 tail -f /var/log/auth.log

//ST: !
Some useful answers to questions from developers lost in making SSH keys works:

- A recap of the steps: http://askubuntu.com/a/306832
- On debugging (saved my life): http://stackoverflow.com/a/20923212/798502

//ST: !
[start= 4]
4. Finally, when the login via SSH keys work, only then can you disable login via passwords:

In `/etc/ssh/sshd_config`, you can disable password authentification:

PasswordAuthentication no

Do again: `service sshd restart`

Now only connecions via a public / private key is possible.

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

//ST: !

== use Psad

INFO:: this part builds on: http://www.pontikis.net/blog/psad-install-config-debian-wheezy

Psad is an app which bans users which scan ports. Before installing it, we need to make sure the firewall logs traffic:

 iptables -A INPUT -j LOG
 iptables -A FORWARD -j LOG

Then we install Psad:

 apt-get install psad

//ST: !

Now we configure Psad by modifying this file:

 vi /etc/psad/psad.conf

Possible values for some interesting parameters (and the source for this section), are here:

http://www.pontikis.net/blog/psad-install-config-debian-wheezy

//ST: !
Then we must edit this file to add the address of the server to the whitelist:

 vi /etc/psad/auto_dl

where I put just 2 values:

 127.0.0.1    0;  # localhost
 xx.xx.xxx.xxx    0; # Server IP (replace xx.xx.xxx.xxx by your actual server IP)



== use fail2ban

This is an app which bans users which fail to login after a number of times - typically bots trying to break in.

fail2ban can actually scan logs from a list of apps you decide (MongoDB, Apache server, GlassFish, etc.) and ban ips mentioned in logs showing a failed access. You need to setup a regex rule specific for each log format, though.

I'll cover it later, when I'll have MongoDB and GlassFish installed.

Documentation on failtoban: http://www.pontikis.net/blog/fail2ban-install-config-debian-wheezy

== the end
//ST: The end!

//ST: !

Author of this tutorial: https://twitter.com/seinecle[Clement Levallois]

All resources on linux security: https://seinecle.github.io/linux-security-tutorials/
