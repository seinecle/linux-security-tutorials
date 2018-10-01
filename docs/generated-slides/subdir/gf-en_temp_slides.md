= Setup of GlassFish and security measures
Clément Levallois <clementlevallois@gmail.com>
2017-04-09

last modified: {docdate}

:icons!:
:asciimath:
:iconsfont:   font-awesome
:revnumber: 1.0
:example-caption!:

//ST: ! 'Escape' or 'o' to see all sides, F11 for full screen, 's' for speaker notes


== System
== !
- I use Debian, version 8.7 (http://www.pontikis.net/blog/five-reasons-to-use-debian-as-a-server[why?])
- Vi is used as a text editor in the following


== Installing the JDK 8
== !
//ST: Installing the JDK 8

Create a new file:

sudo vim /etc/apt/sources.list.d/java-8-debian.list

add these lines:

deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main

deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main

Close the file.

//ST: !

get the certif key:

 sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886

install the JDK 8:

 sudo apt-get update && sudo apt-get install oracle-java8-installer

//ST: !
Set the env variables straight:

 sudo apt-get install oracle-java8-set-default

== Installing Glassfish 4.1
== !
//ST: Installing Glassfish 4.1

We will install it in /opt/app-server::



//ST: !
create a glassfish user (here the `-m` option creates its home directory if not already present):

INFO:: maybe a good idea to choose a user name different from glassfish - default values are always to be avoided.

adduser glassfish -shell /bin/bash
passwd glassfish
adduser glassfish sudo

 //ST: !
 change to user glassfish

 su - glassfish


//ST: !
mkdir /opt/app-server
make the user owner of /opt/app-server:

 sudo chown -R glassfish /opt/app-server
 sudo chmod -R 700 /opt/app-server


//ST: !

download glassfish from here: https://glassfish.java.net/download.html

 cd /opt/app-server
 wget http://download.java.net/glassfish/4.1.2/release/glassfish-4.1.2.zip

//ST: !
sudo apt-get install zip unzip
 unzip glassfish-4.1.2.zip

Glassfish is now installed.

== Hardening Glassfish 4.1
== !
//ST: Hardening Glassfish 4.1

We can  access glassfish commande line by doing:

cd glassfish4/glassfish/bin

 ./asadmin

//ST: !
Change the masterpassword:

source: http://ufasoli.blogspot.fr/2013/07/glassfish-changing-master-password-and.html

 change-master-password --savemasterpassword

(put "changeit" when the default password is asked, then choose your own)

//ST: !

We will now delete the default domain ("domain1") and create a new one. 2 reasons for this:

- We will create a domain with a custom name, to avoid using the default value.
- Create a domain from scratch allows to make use of the nice "portbase" option:

-> The "portbase" option allows for creating a set of ports different from the default ones.

(source: https://blogs.oracle.com/alexismp/entry/flexible_glassfish_domain_creation_using)

//ST: !
 delete-domain domain1

 create-domain --portbase 17390 yourcustomdomain

NOTE:: just letters and numbers in the domain name.

//ST: !
So, with portbase 17390 (choose your own value instead), the port for the GlassFish GUI console will be 17390+ 48 = 17438

Add a tunnel for port 17438 in your SSH connection. Restart your SSH connection.

Go back to /opt/app-server/glassfish-4/glassfish/bin and ./asadmin

 start-domain

If you tunneled the port 17438 in your SSH, you can now go to `http://localhost:17438`

//ST: !

In the GUI console,

- change the admin password
- change the "Address field" in http-listener-2 sub menu to the IP of the server.

//ST: !

The following tips are directly taken from http://blog.eisele.net/2011/05/securing-your-glassfish-hardening-guide.html

- change the content of /domains/yourcustomdomain/docroot/index.html -> "my server is online"

//ST: !

- create an "errorpages" subdir in your domain root folder and an error page in /domains/yourcustomdomain/errorpages/404.html

Then, in the GUI console, point to this file via:

 Configuration|Virtual Server|server

 name: send-error_1

 value: code=404 path=/errorpages/404.html reason=Resource_not_found

//ST: !
Hide the identity of the server in the headers:

- In Network-config -> Network-listeners -> http-listeners (1 et 2) -> HTTP tab: turn off the "XPowered By:" header with your http-listener
- add a JVM-Option -Dproduct.name=""


== the end
== !
//ST: The end!

//ST: !

Author of this tutorial: https://twitter.com/seinecle[Clement Levallois]

All resources on linux security: https://seinecle.github.io/linux-security-tutorials/
pass:[    <!-- Start of StatCounter Code for Default Guide -->
    <script type="text/javascript">
        var sc_project = 11304288;
        var sc_invisible = 1;
        var sc_security = "11304288";
        var scJsHost = (("https:" == document.location.protocol) ?
            "https://secure." : "http://www.");
        document.write("<sc" + "ript type='text/javascript' src='" +
            scJsHost +
            "statcounter.com/counter/counter.js'></" + "script>");
    </script>
    <noscript><div class="statcounter"><a title="site stats"
    href="http://statcounter.com/" target="_blank"><img
    class="statcounter"
    src="//c.statcounter.com/11304288/0/11304288/1/" alt="site
    stats"></a></div></noscript>
    <!-- End of StatCounter Code for Default Guide -->]
