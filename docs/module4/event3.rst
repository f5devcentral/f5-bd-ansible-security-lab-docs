Install & Configure Logstash
============================

Logstash is a light-weight, open-source, server-side data processing pipeline that allows you to collect data from a variety of sources, transform it on the fly, and send it to your desired destination. It is most often used as a data pipeline for Elasticsearch, an open-source analytics and search engine. 

#. Access Node 1 Web Shell
     In UDF -> Ansible Security Lab -> Components -> Node 1 -> Access -> Web Shell

     .. image:: ../images/Event/Picture9.png
#. Install Logstash
     In the Node 1 Web Shell run the following command

     .. code-block::

       sudo /usr/share/logstash/bin/logstash-plugin install logstash-output-exec

     .. image:: ../images/Event/Picture10.png
#. Install Nano
     In the Node 1 Web Shell run the following command 
     
     .. code-block::

       sudo rpm -ihv https://vault.centos.org/centos/8/BaseOS/x86_64/os/Packages/nano-2.9.8-1.el8.x86_64.rpm 
#. Modify the Logstash configuration file
     In the Node 1 Web Shell run the following command
     
     .. code-block::

       sudo nano /etc/logstash/conf.d/logstash.conf
#. Add the following HTTP code to the input section to match below, to paste into web shell by holding <SHIFT> and pressing <INSERT>
     
     .. code-block::
  
        input {
                  tcp {
                           port => 5001
                           type => awaf
                      }
                  udp {
                           port => 5002
                           type => "ipfix-f5"
                           codec => netflow
                      }
                 http {
                           port => 9001
                           type => "watcher-1"
                      }
             }

#. Add the bottom IF statement (watcher-1) code to the output section to match below, to paste into web shell by holding <SHIFT> and pressing <INSERT>)
     
     .. code-block::

        output {
            if [type] == "awaf" {
                elasticsearch {
                        hosts => ["127.0.0.1:9200"]
                        index => "awaf-%{+YYYY.MM.dd}"
                        user => "elastic"
                        password => "password"
                }
            } 
            if [type] == "ipfix-f5" {
                elasticsearch {
                        hosts => ["127.0.0.1:9200"]
                        index => "ipfix-%{+YYYY.MM.dd}"
                        user => "elastic"
                        password => "password"
                }
            }
            if [type] == "watcher-1" {
                exec {
                        command => "/usr/local/bin/ansible-playbook /home/centos/sample-playbook/awaf-blocking.yaml"
                }
            }
        }
#. Save the file and close the editor (**CTRL + X**) and press **Y** to save then press the **Enter** key
     
     .. image:: ../images/Event/Picture11.png
#. Stop the Logstash service to apply the changes
     
     .. code-block::

          sudo systemctl stop logstash
     
     .. image:: ../images/Event/Picture12.png
#. Start the Logstash service with the updated configuration file (can take up to 60 seconds for output to appear)
     Wait until you see UDP Listener Started
     
     .. code-block::
          
          sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash.conf &

     .. image:: ../images/Event/Picture13.png