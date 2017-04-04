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

==== changing SSH port
 vi /etc/ssh/sshd_config

 Text to change in the file: change port SSH 22 by a new port (*let's say 1234*), write the new port down somewhere

 service sshd restart

==== installing the undifficult firewall

 sudo apt-get update

 apt-get install ufw

==== denying all incoming traffic except for SSH port

 ufw default deny incoming

 sudo ufw allow 1234/tcp

 ufw enable

==== disabling clear password auth

 vi /etc/ssh/sshd_config

Text to change in the file:

ChallengeResponseAuthentication no

PasswordAuthentication no

UsePAM no

 service sshd restart




== the end
//ST: The end!

//ST: !

Author of this tutorial: https://twitter.com/seinecle[Clement Levallois]

All tutorials on linux security: https://seinecle.github.io/linux-security-tutorials/
pass:[    <!-- Start of StatCounter Code for Default Guide -->
    <script type="text/javascript">
        var sc_project = 11304288;
        var sc_invisible = 1;
        var sc_security = "4ace8383";
        var scJsHost = (("https:" == document.location.protocol) ?
            "https://secure." : "http://www.");
        document.write("<sc" + "ript type='text/javascript' src='" +
            scJsHost +
            "statcounter.com/counter/counter.js'></" + "script>");
    </script>
    <noscript><div class="statcounter"><a title="site stats"
    href="http://statcounter.com/" target="_blank"><img
    class="statcounter"
    src="//c.statcounter.com/11304288/0/4ace8383/1/" alt="site
    stats"></a></div></noscript>
    <!-- End of StatCounter Code for Default Guide -->]
