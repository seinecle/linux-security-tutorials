= Setup of mong and elasticsearch
Clément Levallois <clementlevallois@gmail.com>
2017-04-10

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
- *MongoDB 3.4 and Elasticsearch 5.x*

== Why MongoDB and Elasticsearch?
//ST: !
- MongoDB is a database which stores data without the need for a pre-established model ("strict description") of this data. In practice: I can save something into MongoDB without spending time creating tables and stuff. Just save a JSON doc, that's it.
- MongoDB alone is great, but I will store gigabytes of data, with several text fields and some simple graph logic as well. Elasticsearch is known for managing well the indexes and queries related to these data types.

//ST: !

- A blog post which details how Elasticsearch helped on performances for Mongo: http://blog.quarkslab.com/mongodb-vs-elasticsearch-the-quest-of-the-holy-performances.html
- And Elasticsearch can then integrate with https://www.elastic.co/fr/products/kibana[Kibana], a way to visualize query results. Awesome!

== Installing MongoDB

source: https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/

 sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
 echo "deb http://repo.mongodb.org/apt/debian jessie/mongodb-org/3.4 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list

disable Transparent Huge Pages as per https://docs.mongodb.com/manual/tutorial/transparent-huge-pages/

 sudo vi /etc/init.d/disable-transparent-hugepages

paste this in the text editor:

[source,bash]
-----------------------
#!/bin/bash
### BEGIN INIT INFO
# Provides:          disable-transparent-hugepages
# Required-Start:    $local_fs
# Required-Stop:
# X-Start-Before:    mongod mongodb-mms-automation-agent
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Disable Linux transparent huge pages
# Description:       Disable Linux transparent huge pages, to improve
#                    database performance.
### END INIT INFO

case $1 in
  start)
    if [ -d /sys/kernel/mm/transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/transparent_hugepage
    elif [ -d /sys/kernel/mm/redhat_transparent_hugepage ]; then
      thp_path=/sys/kernel/mm/redhat_transparent_hugepage
    else
      return 0
    fi

    echo 'never' > ${thp_path}/enabled
    echo 'never' > ${thp_path}/defrag

    re='^[0-1]+$'
    if [[ $(cat ${thp_path}/khugepaged/defrag) =~ $re ]]
    then
      # RHEL 7
      echo 0  > ${thp_path}/khugepaged/defrag
    else
      # RHEL 6
      echo 'no' > ${thp_path}/khugepaged/defrag
    fi

    unset re
    unset thp_path
    ;;
esac
-----------------------

Make the file executable:

 sudo chmod 755 /etc/init.d/disable-transparent-hugepages

Make the file to be ran on reboot:

 sudo update-rc.d disable-transparent-hugepages defaults



Start Mongo:

-change for user mongdb:

 sudo -u mongodb bash

Then only actually start mongo:

 sudo service mongod start

Check that it runs fine:

 sudo cat /var/log/mongodb/mongod.log

-> There should be a line "[initandlisten] waiting for connections on port <port>"

== Install Elasticsearch

source: https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

Download the public signing key:

 wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

(can take a while)

Then:

 sudo apt-get install apt-transport-https

 echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list

 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable elasticsearch.service

===== Config

 switch to root user:

  su -
  cd /etc/default/elasticsearch
  vi elasticsearch.yml

-> switch this param to true: bootstrap.memory_lock: true

You then need to make sure the JVM Heap size is no more than half the RAM. First fix a memory param:

 sudo mkdir /etc/systemd/system/elasticsearch.service.d
 sudo vi elasticsearch.conf

Add these lines:
[Service]
LimitMEMLOCK=infinity

Adjust resource limits:

 sudo vi /etc/security/limits.conf

Add line:

elasticsearch  -  nofile  65536

Add a jvm parameter:

sudo /etc/elasticsearch/jvm.options

Add this line:

-Djava.io.tmpdir=/var/tmp


== Install the mongo to elasticsearch connection

==== elastic2-doc-manager

This is a doc manager by mongodb labs.

Source: https://github.com/mongodb-labs/elastic2-doc-manager

 sudo easy_install pip
 sudo pip install 'elastic2-doc-manager[elastic5]'
 sudo pip install 'mongo-connector[elastic5]'

==== run Mongo as a replicaset:

 sudo service mongod stop

 sudo vi /etc/mongodb.conf

 Change dbPath to /data/db

Then:

 sudo chown -R mongodb:mongodb /data/db

 Then launch mongo as a replicaset:

 sudo mongod --port 27017 --dbpath /data/db --replSet rs0 --fork --logpath /var/log/mongodb.mongod.log

== Install kibana

Kibana is the visualization engine for elastic.

 sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
 sudo apt-get install kibana

Configure Kibana to start automatically at boot:

 sudo /bin/systemctl daemon-reload
 sudo /bin/systemctl enable kibana.service

Start Kibana:

sudo systemctl start kibana.service

==== Install X-pack

 https://www.elastic.co/guide/en/x-pack/current/installing-xpack.html

== Connect Mongo to ElasticSearch:

Source: https://blog.jixee.me/how-to-use-mongo-connector-with-elasticsearch/

 sudo mongo-connector -m localhost:27017 -t localhost:9200 -d elastic2_doc_manager  -n database1.collection1,database1.collection2

You can check that the connection is made here, your Mongo collections should be listed on this page:

 http://localhost:9200/_cat/indices?v

== the end

//ST: !

Author of this tutorial: https://twitter.com/seinecle[Clement Levallois]

All resources on linux security: https://seinecle.github.io/linux-security-tutorials/
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
