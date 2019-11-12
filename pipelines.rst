Logstash Pipelines
====================

After bringing up the ELK stack, the next step is feeding data (logs/metrics) into the setup.

Based on our previous introduction, it is known that Logstash act as the bridge/forwarder to consolidate data from sources and forward it to the Elasticsearch cluster. But how?

ELK Data Flow
--------------

The data flow in ELK is as below:

1. Something happens on the monitored targets/sources:

   - A path down happens on a Linux OS
   - A port flap happens on a switch
   - A LUN is deleted on a storage array
   - A new event is triggered on an application
   - Etc.

2. The event/metric/activity gets recorded by:

   - syslog
   - filebeat
   - metricbeat
   - Etc.

3. Based on the configuration of syslog/filebeat/metricbeat/etc., event(s) are forwarded to Logstash (or to Elasticsearch directly, but we prefer using Logstash in the middle);
4. Logstash:

   a. Get data through its licensing port(s);
   b. Filter/Consolidate/Modify/Enhance data;
   c. Forward data to the Elasticsearch cluster or other supported destinations;

5. Elasticsearch store and index data;
6. Kibana visualize data.

Logstash Pipeline
------------------

Based on the "ELK Data Flow", we can see Logstash sits at the middle of the data process and is responsible for data **gathering (input), filtering/aggregating/etc. (filter), and forwarding (output)**. The process of event processing (**input -> filter -> output**) works as a pipe, hence is called pipeline.

.. image:: images/logstash_pipeline_overview.png

Pipeline is the core of Logstash and is the most important concept we need to understand during the use of ELK stack. Each component of a pipeline (input/filter/output) actually is implemented by using plugins. The most **frequently used plugins** are as below:

- Input:

  - file   : reads from a file directly, working like "tail -f" on Unix like OS;
  - syslog : listens on defined ports (514 by default) for syslog message and parses based on syslog RFC3164 definition;
  - beats  : processes events sent by beats, including filebeat, metricbeat, etc.

- Filter:

  - grok   : parses and structures arbitrary text;
  - mutate : modifies event fields, such as rename/remove/replace/modify;
  - drop   : discards a event;

- Output:

  - elasticsearch : sends event data to Elasticsearch cluster;
  - file          : writes event data to a file;
  - graphite      : sends event data to graphite for graphing and metrics.

**Notes:**

- Multiple pipelines can be defined;
- Multiple input sources, filters, and output targets can be defined within the same pipeline;

For more information, please refer to  `Logstash Processing Pipeline <https://www.elastic.co/guide/en/logstash/2.3/pipeline.html>`_.

Logstash Configuration
------------------------

We only ontroduced the instalaltion of Logstash in previous chapters without saying any word on its configuration, since it is the most complicated topic in ELK stack. Loosely speaking, Logstash provides two types of configuration:

- settings  : control the behavior of how Logstash executes;
- pipelines : define the flows how data get processed.

If Logstash is installed with a pacakge manager, such as rpm, its configuration files will be as below:

- /etc/logstash/logstash.yml  : the default setting file;
- /etc/logstash/pipelines.yml : the default pipeline config file.

logstash.yml
~~~~~~~~~~~~~~

There are few options need to be set (other options can use the default values):

- node.name               : specify a node name for the Logstash instance;
- config.reload.automatic : whether Logstash detects config changes and reload them automatically.

It is recommended to set config.reload.automatic as **true** since this will make it handy during pipeline tunings.

pipelines.yml
~~~~~~~~~~~~~~~

The default pipeline config file. It consists of a list of pipeline reference, each with:

- pipeline.id : a meaningful pipeline name specified by the end users;
- path.config : the detailed pipeline configuration file, refer to `Pipeline Configuration`_.

Below is a simple example, which defines 4 x pipelines:

::

  - pipeline.id: syslog.unity
    path.config: "/etc/logstash/conf.d/syslog_unity.conf"
  - pipeline.id: syslog.xio
    path.config: "/etc/logstash/conf.d/syslog_xio.conf"
  - pipeline.id: syslog.vsphere
    path.config: "/etc/logstash/conf.d/syslog_vsphere.conf"
  - pipeline.id: beats
    path.config: "/etc/logstash/conf.d/beats.conf"

This config file only specifies pipelines to use, but not define/configure pipelines. We will cover the details with `Pipeline Configuration`_.

Service Startup Script
~~~~~~~~~~~~~~~~~~~~~~~~

After Logstash installation (say it is installed through a package manager), a service startup script won't be created by default. In other words, it is not possible to control Logstash as a service with systemctl. The reason behind is that Logstash gives end users the ability to further tune how Logstash will act before making it as a serive.

The options can be tuned are defined in **/etc/logstash/startup.options**. Most of times, there is no need to tune it, hence we can install the service startup script directly as below:

::

  /usr/share/logstash/bin/system-install

After running the script, a service startup script will be installed as **/etc/systemd/system/logstash.service**. Now, one can control Logstash service with systemctl as other services.

Pipeline Configuration
------------------------

It is time to introduce how to configure a pipeline, which is the core of Logstash usage. It is really abstractive to understand pipelines without an example, so our introduction will use examples from now on.

Pipeline Skeleton
~~~~~~~~~~~~~~~~~~~~

Pipeline shares the same configuration skeleton (3 x sections: input, filter and output) as below:

::

  # This is a comment. You should use comments to describe
  # parts of your configuration.
  input {
    ...
  }

  filter {
    ...
  }

  output {
    ...
  }

The details of each section are defined through the usage of different plugins. Here are some examples:

- Define a file as the input source:

  ::

    input {
      file {
        path => "/var/log/apache/access.log"
      }
    }

- Multiple input soures can be specified:

  ::

    input {
      file {
        path => "/var/log/messages"
      }

      file {
        path => "/var/log/apache/access.log"
      }
    }

- Additional fields can be added as part of the data comming from the sources (these fields can be used for search once forwarded to destinations):

  ::

    input {
      file {
        path => "/var/log/messages"
        type => "syslog"
        tags => ["file", "local_syslog"]
      }

      file {
        path => "/var/log/apache/access.log"
        type => "apache"
        tags => ["file", "local_apache"]
      }
    }

- Different kinds of plugins can be used for each section:

  ::

    input {
      file {
        path => "/var/log/messages"
        type => "syslog"
        tags => ["file", "local_syslog"]
      }

      file {
        path => "/var/log/apache/access.log"
        type => "apache"
        tags => ["file", "local_apache"]
      }

      beats {
        type => "beats"
        port => 5044
        type => "beats"
        tags => ["beats", "filebeat"]
      }

      tcp {
        port => 5000
        type => "syslog"
        tags => ["syslog", "tcp"]
      }

      udp {
        port => 5000
        type => "syslog"
        tags => ["syslog", "udp"]
      }
    }

- An empty filter can be defined, which means no data modification will be made:

  ::

    filter {}

- Grok is the most powerful filter plugin, especially for logs:

  ::

    # Assume the log format of http.log is as below:
    # 55.3.244.1 GET /index.html 15824 0.043
    #
    # The grok filter will match the log record with a pattern as below:
    # %{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}
    #
    # After processing, the log will be parsed into a well formated JSON document with below fields:
    # client  : the client IP
    # method  : the request method
    # request : the request URL
    # bytes   : the size of request
    # duration: the time cost for the request
    # message : the original raw message
    input {
      file {
        path => "/var/log/http.log"
      }
    }
    filter {
      grok {
        match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
      }
    }

- Multiple plugins can be used within the filter section, and they will process data with the order as they are defined:

  ::

    filter {
        grok {
            match => { "message" => "%{COMBINEDAPACHELOG}"}
        }

        geoip {
            source => "clientip"
        }
    }

- Conditions are supported while define filters:

  ::

    filter {
      if [type] == "syslog" {
        grok {
          match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{DATA:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
        }
        date {
           match => [ "timestamp", "MMM dd HH:mm:ss", "MMM  d HH:mm:ss" ]
        }
      }
    }

- Multiple output destinations can be defined too:

  ::

    output {
      elasticsearch { hosts => ["localhost:9200"] }
      stdout { codec => rubydebug }
    }

By reading above examples, you should be ready to configure your own pipelines. We will introduce the filter plugin grok in more details since we need to use it frequently.o

The Grok Filter Plugin
~~~~~~~~~~~~~~~~~~~~~~~~

Reference
~~~~~~~~~~

- `Variables and Condtions <https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html>`_
- `Input Plugins <https://www.elastic.co/guide/en/logstash/current/input-plugins.html>`_
- `Filter Plugins <https://www.elastic.co/guide/en/logstash/current/filter-plugins.html>`_
- `Output Plugins <https://www.elastic.co/guide/en/logstash/current/output-plugins.html>`_
