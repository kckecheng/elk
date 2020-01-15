Tips
=====

How to add tags based on filed content with pipelines?
--------------------------------------------------------

If the filed's name is known, it can be used directly. If not, use "message" which holds everything.

::

  filter {
    if [message] =~ /regexp/ {
      mutate {
        add_tag => [ "tag1", "tag2" ]
      }
    }
  }

Integrate Kafka
----------------

Kafka can be integrated into the middle of an Elastic Stack. The simplest implementation is leveraging the kafka input/output plugin of logstash directly. With that, the data flow looks as below:

- data source -> logstash -> kafka
- kafka -> logstash -> elasticsearch

More information on scaling Logstash can be found from `Deploying and Scaling Logstash <https://www.elastic.co/guide/en/logstash/current/deploying-and-scaling.html>`_

Send into Kafka
~~~~~~~~~~~~~~~~~

1. Create a logstash pipeline as below:

   ::

     input {
       tcp {
         port => 5000
         tags => ["syslog", "topic1"]
       }
     }

     output {
       kafka {
         topic_id => "topic1"
         codec => json
         bootstrap_servers => "kafka_server1:9092,kafka_server2:9092,kafka_server3:9092"
       }
     }

#. Send a test information:

   ::

     telnet logstash_server 5000
     message1
     message2

#. Verify the messages have been sent to Kafka successfully:

   ::

     bin/kafka-console-consumer.sh --bootstrap-server "kafka_server1:9092,kafka_server2:9092,kafka_server3:9092" --topic topic1 --from-beginning

Read from Kafka and Send to Elasticsearch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Create a logstash pipeline as below:

   ::

     input {
       kafka {
         client_id => "logstash_server"
         # group_id => "logstash_server"
         topics => ["topic1"]
         codec => "json"
         bootstrap_servers => "kafka_server1:9092,kafka_server2:9092,kafka_server3:9092"
       }
     }

     output {
       elasticsearch {
         hosts => ["http://elasticsearch1:9200", "http://elasticsearch2:9200", "http://elasticsearch3:9200"]
         index => "topic1-%{+YYYY.MM.dd}"
       }

     }

#. From Kibana, the informaiton should be able to be seen

Add Tags to Different Kafka Topics
------------------------------------

**Notes:** [@metadata][kafka][topic] will be empty sometimes due to unknown issues. Hence this tip is listed here for reference.

::

  input {
    kafka {
      client_id => "logstash_server"
      group_id => "logstash"
      topics => ["unity", "xio"]
      codec => "json"
      bootstrap_servers => "kafka_server1:9092,kafka_server2:9092,kafka_server3:9092"
    }
  }

  filter {
    if [@metadata][kafka][topic] == "unity" {
      mutate { add_tag => ["unity"] }
    }
    if [@metadata][kafka][topic] == "xio" {
      mutate { add_tag => ["xio"] }
    }
  }

  output {
    elasticsearch {
      hosts => ["http://elasticsearch1:9200", "http://elasticsearch2:9200", "http://elasticsearch3:9200"]
      index => "storagebox-%{+YYYY.MM.dd}"
    }
  }
