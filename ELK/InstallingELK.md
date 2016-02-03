# Installing ELK
Java is required for Elasticsearch and Logstash to work. You can install OpenJDK or Oracle Java. In the following example, Oracle Java is installed. You can use the sudo add-apt-repository -y ppa:webupd8team/java command to add the Oracle Java Personal Package Archive (PPA) to the Ubuntu Server, as shown in Example 4-6.
Example 4-6 Adding the Oracle Java PPA

    omar@elk-srv1:~$ sudo add-apt-repository -y ppa:webupd8team/java
    gpg: keyring `/tmp/tmpe2orytku/secring.gpg' created
    gpg: keyring `/tmp/tmpe2orytku/pubring.gpg' created
    gpg: requesting key EEA14886 from hkp server keyserver.ubuntu.com
    gpg: /tmp/tmpe2orytku/trustdb.gpg: trustdb created1
    gpg: key EEA14886: public key "Launchpad VLC" imported
    gpg: Total number processed: 1
    gpg:               imported: 1  (RSA: 1)
    OK
    omar@omar-srv1:~$

After you add the PPA, update the apt package database with the sudo apt-get update command, as shown below:

    omar@elk-srv1:~$ sudo apt-get update
    sudo: unable to resolve host omar-srv1
    Ign http://us.archive.ubuntu.com trusty InRelease
    Ign http://us.archive.ubuntu.com trusty-updates InRelease
    Ign http://us.archive.ubuntu.com trusty-backports InRelease
    Hit http://us.archive.ubuntu.com trusty Release.gpg
    Ign http://security.ubuntu.com trusty-security InRelease
    Get:1 http://us.archive.ubuntu.com trusty-updates Release.gpg [933 B]
    Get:2 http://us.archive.ubuntu.com trusty-backports Release.gpg [933 B]
    Ign http://ppa.launchpad.net trusty InRelease
    Hit http://us.archive.ubuntu.com trusty Release
    Get:3 http://security.ubuntu.com trusty-security Release.gpg [933 B]
    <...output omited>
    Ign http://us.archive.ubuntu.com trusty/multiverse Translation-en_US
    Ign http://us.archive.ubuntu.com trusty/restricted Translation-en_US
    Ign http://us.archive.ubuntu.com trusty/universe Translation-en_US
    Fetched 4,014 kB in 6s (590 kB/s)
    Reading package lists... Done
    omar@omar-srv1:~$

Install the latest stable version of Oracle Java with the sudo apt-get -y install oracle-javaXX-installer command (XX is the version of Java). Oracle Java 8 is installed as follows:
    omar@elk-srv1:~$ sudo apt-get -y install oracle-java8-installer

## Installing Elasticsearch
The following are the steps necessary in order to install Elasticsearch in Ubuntu Server:
Step 1. Add the Elasticsearch public GPG key into apt with the following command:

    wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -

Step 2. Create the Elasticsearch source list with the following command:

    echo 'deb http://packages.elasticsearch.org/elasticsearch/1.4/debianstable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list

Step 3. Use the sudo apt-get update command to update your apt package database.
Step 4. Install Elasticsearch with the sudo apt-get -y install elasticsearch=1.4.4 command. In this example, Elasticsearch Version 1.4.4 is installed. For the latest version of Elasticsearch, visit https://www.elastic.co.
Step 5. The Elasticsearch configuration resides in the /etc/elasticsearch/elasticsearch.yml file. To edit the configuration file, use your favorite editor. (Vi is used in this example: sudo vi /etc/elasticsearch/elasticsearch.yml.) It is a best practice to restrict access to the Elasticsearch installation on port 9200 to only local host traffic. Uncomment the line that specifies network.host in the elasticsearch.yml file and replace its value with localhost. T

Step 6. Restart the Elasticsearch service with the sudo service elasticsearch restart command. You can also use the sudo update-rc.d elasticsearch defaults 95 10 command to start Elasticsearch automatically upon boot.

## Install Kibana
The following are the steps necessary to install Kibana in Ubuntu Server:
Step 1. Download Kibana from https://www.elastic.co/downloads/kibana.
Step 2. The Kibana configuration file is config/kibana.yml. Edit the file, as shown in the following example:

    omar@elk-srv1:~$ vi ~/kibana-4*/config/kibana.yml

Step 3. Find the line that specifies host. By default, 0.0.0.0 is configured. Replace it with localhost. This way Kibana will be accessible only to the local host. In this example, Nginx will be installed, and it will reverse proxy to allow external access.
Step 4. Copy the Kibana files to a more suitable directory. The /opt directory is used in the following example:

    omar@elk-srv1:~$ sudo mkdir -p /opt/kibana
    omar@elk-srv1:~$ sudo cp -R ~/kibana-4*/* /opt/kibana/

Step 5. To run Kibana as a service, download a init script with the following command:

    cd /etc/init.d && sudo wget https://gist.githubusercontent.com/thisismitch/8b15ac909aed214ad04a/raw/bce61d85643c2dcdfbc2728c55a41dab444dca20/kibana4

Step 6. Enable and start the Kibana service with the following commands:

    omar@elk-srv1:~$ sudo chmod +x /etc/init.d/kibana4
    omar@elk-srv1:~$ sudo update-rc.d kibana4 defaults 96 9
    omar@elk-srv1:~$ sudo service kibana4 start

## Installing Nginx
In this example, Nginx is used as a reverse proxy to allow external access to Kibana. Complete the following steps to install Nginx:
Step 1. Use the sudo apt-get install nginx apache2-utils command to install Ngnix.
Step 2. You can use htpasswd to create an admin user. In this example, secadmin is the admin user:

    sudo htpasswd -c /etc/nginx/htpasswd.users secadmin

Step 3. Edit the Nginx default server block (/etc/nginx/sites-available/default) and update the server_name to match your server’s name. The following is the /etc/nginx/sites-available/default file contents used in this example:

    server {
        listen 80;
    
        server_name elk-srv1.example.com;
    
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/htpasswd.users;
    
        location / {
            proxy_pass http://localhost:5601;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }

Step 4. Restart Nginx with the sudo service nginx restart command.
You should now be able to access Kibana in your browser via the fully qualified domain name (FQDN) or its IP address.

## Install Logstash
Complete the following steps to install Logstash in the Ubuntu Server:
Step 1. Download Logstash from https://www.elastic.co/downloads/logstash. Alternatively, you can add the Logstash repository in Ubuntu and update the package database, as shown here:

    omar@elk-srv1:~$ echo 'deb http://packages.elasticsearch.org/logstash/1.5/debian stable main' | sudo tee /etc/apt/sources.list.d/    logstash.list 
    omar@elk-srv1:~$ sudo apt-get update

Step 2. Install Logstash with the sudo apt-get install logstash command, as shown here:
    omar@elk-srv1:~$ sudo apt-get install logstash

Step 3. The Logstash configuration files are under /etc/logstash/conf.d. The configuration files are in JSON format. The configuration consists of three sections: inputs, filters, and outputs:

    input {
        udp {
          port => 9996
          codec => netflow {
            definitions => "/opt/logstash/codecs/netflow/netflow.yaml"
            versions => 9
          }
        }
      }
      output {
        stdout { codec => rubydebug }
        if ( [host] =~ "172.18.104.1" ) {
          elasticsearch {
            index => "logstash_netflow-%{+YYYY.MM.dd}"
            host => "localhost"
          }
        } else {
          elasticsearch {
            index => "logstash-%{+YYYY.MM.dd}"
            host => "localhost"
          }
        }
      }
    
In this example, Logstash is configured to accept NetFlow records from R1 (172.18.104.1). The NetFlow data is exported to Elasticsearch with the logstash_netflow-YYYY.MM.dd index named; where YYYY.MM.dd is the date when the NetFlow data was received. The server is configured to listen on UDP port 9996.
Note: You can find additional examples and resources at https://github.com/santosomar/netflow. You can also contribute with your own examples and code there.
The following template is used for the server to be able to parse the fields from NetFlow:

    curl -XPUT localhost:9200/_template/logstash_netflow9 -d '{
      "template" : "logstash_netflow9-*",
      "settings": {
        "index.refresh_interval": "5s"
      },
      "mappings" : {
        "_default_" : {
          "_all" : {"enabled" : false},
          "properties" : {
            "@version": { "index": "analyzed", "type": "integer" },
            "@timestamp": { "index": "analyzed", "type": "date" },
            "netflow": {
              "dynamic": true,
              "type": "object",
              "properties": {
                "version": { "index": "analyzed", "type": "integer" },
                "flow_seq_num": { "index": "not_analyzed", "type": "long" },
                "engine_type": { "index": "not_analyzed", "type":
                  "integer" },
                "engine_id": { "index": "not_analyzed", "type":
                  "integer" },
                "sampling_algorithm": { "index": "not_analyzed", "type":
                  "integer" },
                "sampling_interval": { "index": "not_analyzed", "type":
                  "integer" },
                "flow_records": { "index": "not_analyzed", "type":
                  "integer" },
                "ipv4_src_addr": { "index": "analyzed", "type": "ip" },
                "ipv4_dst_addr": { "index": "analyzed", "type": "ip" },
                "ipv4_next_hop": { "index": "analyzed", "type": "ip" },
                "input_snmp": { "index": "not_analyzed", "type": "long" },
                "output_snmp": { "index": "not_analyzed", "type": "long" },
                "in_pkts": { "index": "analyzed", "type": "long" },
                "in_bytes": { "index": "analyzed", "type": "long" },
                "first_switched": { "index": "not_analyzed", "type": "date" },
                "last_switched": { "index": "not_analyzed", "type": "date" },
                "l4_src_port": { "index": "analyzed", "type": "long" },
                "l4_dst_port": { "index": "analyzed", "type": "long" },
                "tcp_flags": { "index": "analyzed", "type": "integer" },
                "protocol": { "index": "analyzed", "type": "integer" },
                "src_tos": { "index": "analyzed", "type": "integer" },
                "src_as": { "index": "analyzed", "type": "integer" },
                "dst_as": { "index": "analyzed", "type": "integer" },
                "src_mask": { "index": "analyzed", "type": "integer" },
                "dst_mask": { "index": "analyzed", "type": "integer" }
              }
            }
          }
        }
      }
    }'
    
The preceding template is used for Elasticsearch to be able to process all indices that start with logstash_netflow9.
Note: Refer to the Elasticsearch documentation for more information at https://www.elastic.co/guide/index.html 
