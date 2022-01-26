ELK Installation
==================

As we know, ELK is mainly consit of Elasticsearch, Logstash and Kibana, hence the instalaltion consists of 3 corresponding sections. ELK stack can be installed on bare-metal/VM, and can also be run by using docker and kunernetes, it even can be serviced through public clouds like AWS and GCP.

For non-public cloud based IT solutions, it is recommended to maintain a **local deployment**. At the same time, installing Elasticsearch on **bare-metal/VM** is recommended since Elasticsearch needs to store and index data frequently although docker/kubernetes also support data persistence (unless there is already a working kubernetes setup including well defined CSI support, it is not cost effective to maintain a kubernetes cluster just because of ELK setup).

Elasticsearch
---------------

Installation
~~~~~~~~~~~~~~

Elasticsearch can be installed through using tarball on Linux, but the prefered way is to use a package manager, such as **rpm/yum/dnf on RHEL/CentOS**, **apt on Ubuntu**.

The detailed installation step won't be covered in this document since it is well documented by its `official guide <https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html>`_.

**Notes:** An Elasticsearch cluster, which consists of several nodes, should be used to provide scalability and resilience for most use cases, therefore the pacakge should be installed by following the same steps on all involved cluster nodes.

Configuration
~~~~~~~~~~~~~~~

After installing Elasticsearch on all cluster nodes, we need to configure them to form a working cluster. Fortunatelly, Elasticsearch ships with good defaults and requires very little configuration.

- **Config file location** : **/etc/elasticsearch/elasticsearch.yml** is the primary config file for Elasticsearch if it is installed through a pacakge manager, such as rpm;
- **Config file format** : the config file is in YAML format, with a simple `Syntax <https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html>`_;

The **detailed configuration** is straightforward, let's explain them one by one:

1. **cluster.name** : the name of the Elasticsearch cluster. All nodes should be configured with an identical value;
2. **node.name** : the name for node. It is recommended to use a resolvable (through /etc/hosts or DNS) hostname;
3. **path.data** : where the data will be stored. If a different path than the default is specified, remember to change the permission accordingly;
4. **path.logs** : where the Elasticsearch log will be stored. If a different path than the default is specified, remember to change the permission accordingly;
5. **network.host** : the IP/FQDN each node will listen on. It is recommended to use **0.0.0.0** to bind all available IP address;
6. **discovery.seed_hosts** : the list of nodes which will form the cluster;
7. **cluster.initial_master_nodes** : the list of nodes which may act as the master of the cluster;

Here is a sample configuration file:

::

  cluster.name: elab-elasticsearch
  node.name: e2e-l4-0680-240

  path.data: /home/elasticsearch/data
  path.logs: /home/elasticsearch/log

  network.host: 0.0.0.0

  discovery.seed_hosts: ["e2e-l4-0680-240", "e2e-l4-0680-241", "e2e-l4-0680-242"]
  cluster.initial_master_nodes: ["e2e-l4-0680-240", "e2e-l4-0680-241", "e2e-l4-0680-242"]

Startup
~~~~~~~~

After configuration, the cluster can be booted. Before that, please make sure the port **9200**, which is the default port Elasticsearch listens at, has been opened on firewall. Of course, one can specify a different port and configure the firewall accordingly.

The cluster can be booted easily by starting the service on each node with systemctl if Elasticsearch is installed through a package manger. Below are the sample commands:

::

  # Run below commands on all nodes
  # Disable the firewall directly - not recommended for production setup
  systemctl stop firewalld
  systemctl disable firewalld
  # Start elasticsearch
  systemctl enable elasticsearch
  systemctl start elasticsearch
  systemctl status elasticsearch

If everything is fine, your cluster should be up and running within a minute. It is easy to verify if the cluster is working as expected by checking its status:

::

  curl -XGET 'http://<any node IP/FQDN>:9200/_cluster/state?pretty'

Kibana
-------

Kibana is the front end GUI for Elasticsearch showcase. Since it does not store or index data, it is suitable to run as a docker container or a kubernetes service. However, since we have already privisioned bare-metal/VM for the Elasticsearch cluster setup, installing kibana on any/all nodes of the cluster is also a good choice.

The detailed installation step is straightforward, please refer to the `official installation document <https://www.elastic.co/guide/en/kibana/current/install.html>`_
The **configuration for kibana** is also quite easy:

- **server.host** : specify the IP address Kibana will bind to. **0.0.0.0** is recommended;
- **server.name** : specify a meaningful name for the Kibana instance. A resolvable hostname is recommended;
- **elasticsearch.hosts** : specify a list of elasticsearch cluster Kibana will connect to.

Below is a sample config file:

::

  server.host: "0.0.0.0"
  server.name: "e2e-l4-0680-242"

  elasticsearch.hosts: ["http://e2e-l4-0680-240:9200", "http://e2e-l4-0680-241:9200", "http://e2e-l4-0680-242:9200"]

After configuring Kibana, it can be started with systemctl:

::

  systemctl disable firewalld
  systemctl stop firewalld
  systemctl enable kibana
  systemctl start kibana

If everyghing goes fine, Kibana can be accessed through **http://<IP or FQDN>:5601/**

Logstash
---------

The instalaltion of Logstash is also pretty easy and straightforward. We won't waste any words here for it, please refer to the `official installation guide <https://www.elastic.co/guide/en/logstash/current/installing-logstash.html>`_.

Please **keep in mind** : although Logstash can be installed together on the same server(s) as elasticsearch and Kibana, it is not wise to do so. It is highly recommended to install Logstash near to the soures where logs/metrics are generated.

In the meanwhile, since Logstash is the central place to foward logs/metrics to Elasticsearch cluster, its capability and resilience is important for a smoothly working setup. Generally speacking, this can be achived by particioning and load balancing (we won't provide the guide within this document):

- **Particioning** : leverage different Logstash deployment for differnet solutions/applications. Let's say there are web servers and databases within a production environment, then deploying different Logstash instances for them is a good choice - the capactiy of Logstash is extended, and each solution won't impact each other if its assocaiated Logstash fails;
- **Load balancing** : for each solution/application, it is recommended to deploy several Logstash instances and expose them with a load balancer (such as **HAProxy**) for high availability.

Regarding the configuration of Logstash, we will cover it in the introdcution section of Logstash pipelines.
