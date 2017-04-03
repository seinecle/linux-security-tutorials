= Step-by-step guide to Linux security
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

- Server ordered on Hetzner.de

== Disabling password authentication, enabling SSH
//ST: Disabling password authentification, enabling SSH

Password authentification is less secure than SSH public key. A password transits through the Internet for the auhtentication, it can be hacked at this step.

A SSH private key is not transmitted on the wire. So, it can't be hacked this way.

A detailed explanation is https://security.stackexchange.com/questions/69407/why-is-using-an-ssh-key-more-secure-than-using-passwords[available here].


//ST: !
How to generate a SSH key?

- On Windows, use https://docs.joyent.com/public-cloud/getting-started/ssh-keys/generating-an-ssh-key-manually/manually-generating-your-ssh-key-in-windows[Puttygen].
- On Mac, use https://docs.joyent.com/public-cloud/getting-started/ssh-keys/generating-an-ssh-key-manually/manually-generating-your-ssh-key-in-mac-os-x[the Terminal]
- On Linux, use the https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html[ssh-keygen command]

//ST: !
How to disable password auth an enable SSH?

-> the server I rented on Hetzner asked it from the console right when renting it.
-> otherwise, this can be set up https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-mac-disable-ssh-password-usage[this way]:




== the end
//ST: The end!

//ST: !

Author of this tutorial: https://twitter.com/seinecle[Clement Levallois]

All tutorials on linux security: https://seinecle.github.io/linux-security-tutorials/
