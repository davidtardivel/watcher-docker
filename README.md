Watcher Installation
====================

To test Watcher, you can deploy it next to DevStack and run Watcher in a Docker environment. Watcher image containers integrate automatic provisioning scripts to simplify installation and configuration. You can install DevStack and Docker tools on the same machine.

You have to be familiar with Docker and its *docker-compose* orchestration tool. 


Install devstack
----------------

If you need to install DevStack, you can refer to this page : 
`DevStack - an OpenStack Community Production <http://docs.openstack.org/developer/devstack/>`_
:: 
    $ git clone https://git.openstack.org/openstack-dev/devstack

    $ cd devstack; ./stack.sh

Install docker & docker compose
-------------------------------

To install Docker, you can refer to this page : `Supported installation <https://docs.docker.com/installation/>`_ and choose your distribution.

1. On Ubuntu simply do::

    $ sudo apt-get install curl

    $ curl -sSL https://get.docker.com | sh

2. To test your installation::

    $ sudo docker run hello-world


To install Docker compose, you can refer to this page: `Install Docker Compose <https://docs.docker.com/compose/install/>`_
::
    $ curl -L https://github.com/docker/compose/releases/download/1.4.0/docker-compose-)\`uname -s\`-\`uname -m\` \> /usr/local/bin/docker-compose

    $ chmod +x /usr/local/bin/docker-compose


Watcher containers
------------------

Containers orchestration templating is composed of 3 groups of containers, each group are independent:

- SERVICE_DISCOVERY : containers providing an automatic service discovery tool, based on 
    - `consul <https://github.com/hashicorp/consul>`_
    - `registrator <https://github.com/gliderlabs/registrator>`_

- WATCHER : containers providing a basic OpenStack identity Service and the Watcher modules (including Horizon plugin)
    - mariadb, 
    - rabbitmq, 
    - keystone, 
    - console, 
    - watcher-api, 
    - watcher-decision-engine, 
    - watcher-applier, 
    - horizon

- METERING: containers providing a complete metering chain for Watcher.
    - riemann, 
    - influxdb, 
    - nanoconfig-server, 
    - watcher-metering-publisher, 
    - watcher-metering-agent, 
    - grafana


Install the Service Discovery tool
----------------------------------

1.  Upload the Watcher orchestration template files::

    $ git clone ssh://gitolite@forge.b-com.com/indeed-ccl/TESTING/DOCKER/watcher-ti.git


2.  Deploy the containers::

    $ cd watcher-ti/service-discovery

    $ docker-compose up -d

Install the Watcher module
--------------------------

1. Customize the template file *docker-compose.yml*:
    - comment the *keystone* service (you will use the DevStack one)
    - comment the *mariadb* service if you want to use the DevStack db
    - comment the *rabbitmq* service if you want to use the DevStack RabbitMQ service
    - comment the *horizon* service if you don't want to use Horizon with the Watcher UI plugin

2. For all non commented services, don't forget to set environment variables in the template file *docker-compose.yml*:

  -  DevStack modules settings:

    - **KEYSTONE_NODE**: DevStack Keystone IP address
    - **KEYSTONE_ADMIN_TOKEN**: DevStack Keystone admin token (default is *ADMIN_TOKEN*)
    - **KEYSTONE_SERVICE_TENANT_NAME**: DevStack service project name (default is *service*)
    - **MARIADB_NODE**: DevStack MySQL server IP address, if used.
    - **RABBITMQ_NODE**: DevStack RabbitMQ server IP address, if used

  -  Watcher modules debug mode:

    - **DEBUG_MODE**: Debug mode (true|false, default is *false*)
    - **VERBOSE_MODE**: Verbose mode (true|false, default is *false*)

  -  *admin* user's credentials :

    - **OS_IDENTITY_API_VERSION**: DevStack Keystone Identity API Version (2.0|3, default is *3*) 
    - **OS_PROJECT_DOMAIN_ID**: DevStack Project Domain ID  (default is *default*)
    - **OS_USER_DOMAIN_ID**:  DevStack User Domain ID (default is *default*)
    - **OS_USERNAME**:  DevStack User name (default is *admin*)
    - **OS_PASSWORD**:  DevStack User password (default is *1234*)
    - **OS_PROJECT_NAME**:  DevStack Project name (default is *admin*)


3.  Deploy the containers::

    $ cd watcher-ti/watcher

    $ docker-compose up -d

At startup the *console* container will automatically declare the watcher service into the DevStack Keystone Identity Service.


Run Watcher commands
--------------------

You can use the *console* container to play with Watcher.

1.  Connect to the Watcher console container::

    $ docker ps

    Get the *console* container ID and attach to it
    :: 
 
    $ docker exec -it <watcher_console_container_ID> bash

    root@console:/# cd /root

    root@console:/# source creds

    root@console:~# watcher audit-list
    +------+------+---------------------+-------+
    | UUID | Type | Audit Template Name | State |
    +------+------+---------------------+-------+
    +------+------+---------------------+-------+


Important notes
===============

Please check  your Compute's hypervisor configuration to handle correctly instance migration:

    `OpenStack - Configure Migrations <http://docs.openstack.org/admin-guide-cloud/content/section_configuring-compute-migrations.html>`_