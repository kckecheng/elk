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
   c. Forward data to the Elasticsearch cluster;

5. Elasticsearch store and index data;
6. Kibana showcase data.

Logstash Pipeline
------------------

Based on the "ELK Data Flow", we can see Logstash sits at the middle of the data process and is responsible for data **gathering (input), filtering/aggregating/etc. (filter), and forwarding (output)**. The process of event processing (**input -> filter -> output**) works as a pipe, hence is called pipeline.

Pipeline is the core of Logstash and is the most important concept we need to understand during the use of ELK stack. Each component of a pipeline (input/filter/output) actually is implemented by using a plugin. The most **frequently used plugins** are as below:

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

Pipeline Configuration
------------------------

It is really abstractive to understand pipeline without an example, so let's continue our introduction with examples.
