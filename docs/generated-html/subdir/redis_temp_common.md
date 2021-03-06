= Setup of redis
Clément Levallois <clementlevallois@gmail.com>
2017-04-26

last modified: {docdate}

:icons!:
:asciimath:
:iconsfont:   font-awesome
:revnumber: 1.0
:example-caption!:
:sourcedir: ../../../main/java

//ST: 'Escape' or 'o' to see all sides, F11 for full screen, 's' for speaker notes
//ST: !

== System
//ST: !

- I use Debian, version 8.7 (http://www.pontikis.net/blog/five-reasons-to-use-debian-as-a-server[why?])
- Vi is used as a text editor in the following

== Why Redis?
//ST: !
- I need a data store of the kind: key -> multiple values. Millions of keys, with hundreds or thousands of values for each key.
- If in RAM, these queries on key-values than scanning tables in a db - I think.
- Redis offers join / intersection / ... operations on sets of keys, which are useful in my use case.

== Installing Redis

 sudo apt-get update && sudo apt-get upgrade
 sudo apt-get install software-properties-common

DO
(source: https://www.linode.com/docs/databases/redis/deploy-redis-on-ubuntu-or-debian)

 sudo vi /etc/apt/sources.list.d/dotdeb.list

 in this file, write:
 deb http://packages.dotdeb.org stable all
 deb-src http://packages.dotdeb.org stable all

Then:
wget https://www.dotdeb.org/dotdeb.gpg
sudo apt-key add dotdeb.gpg
sudo apt-get update
sudo apt-get install redis-server

Stop / start / restart redis:

sudo service redis-server restart


OR
 sudo wget http://download.redis.io/redis-stable.tar.gz
 sudo tar xvzf redis-stable.tar.gz
 cd redis-stable
 make
 sudo cp src/redis-server /usr/local/bin/
 sudo cp src/redis-cli /usr/local/bin/

 sudo mkdir /etc/redis
 sudo mkdir /var/redis

 sudo cp utils/redis_init_script /etc/init.d/redis_6379

 sudo vi /etc/init.d/redis_6379

Replace the beginning by:

(source: https://github.com/antirez/redis/issues/804#issuecomment-234132188)

#!/bin/sh
### BEGIN INIT INFO
# Provides:          redis
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the redis server
# Description:       starts redis using...
### END INIT INFO


sudo cp redis.conf /etc/redis/6379.conf
sudo mkdir /var/redis/6379

sudo vi /etc/redis/6379.conf

- Edit the configuration file, making sure to perform the following changes:
- Set daemonize to yes (by default it is set to no).
- Set the pidfile to /var/run/redis_6379.pid (modify the port if needed).
- Change the port accordingly. In our example it is not needed as the default port is already 6379.
- Set your preferred loglevel.
- Set the logfile to /var/log/redis_6379.log
- Set the dir to /var/redis/6379 (very important step!)

sudo update-rc.d redis_6379 defaults

Starting reddis:

sudo /etc/init.d/redis_6379 start


== moving redis from one server to another
Do scp as described in the tutorial on ssh

Make sure sure redis is stopped and appendonly mode is OFF in the config file.

Move dump.rdb to /var/lib/redis
Change ownership and permission:
 chown redis:redis dump.rdb
 chmod 644 dump.rdb
Start redis.

Change back appendonly to yes when redis has launched.

== The end
//ST: The end
//ST: !

image:round_portrait_mini_150.png[align="center", role="right"]
Tutorial by Clement Levallois.

Discover other tutorials and courses in data / tech for business: http://www.clementlevallois.net

Or get in touch via Twitter: https://www.twitter.com/seinecle[@seinecle]
