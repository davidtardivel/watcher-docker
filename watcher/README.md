Install the Watcher module
==========================


-   WATCHER containers are:
     -   mariadb,
     -   rabbitmq,
     -   keystone,
     -   console,
     -   watcher-api,
     -   watcher-decision-engine,
     -   watcher-applier,
     -   horizon (including Watcher plugin)


The file *watcher-ti/docker-compose.xml* is a global orchestration template file. You can customize it in order to deploy and configure all containers or a part of them, according to your needs.

Use DevStack Identity Service Keystone
--------------------------------------

1. Edit the template file *docker-compose.yml*:

       $ cd watcher
       $ vi docker-compose.xml

   and remove/comment the *keystone* service.

2. in this file, set the next environment variables (you have to set them in several services): 
   -   DevStack modules settings:
       -   **KEYSTONE\_NODE**: DevStack Keystone IP address
       -   **KEYSTONE\_ADMIN\_TOKEN**: DevStack Keystone admin token (default is *ADMIN\_TOKEN*)
       -   **KEYSTONE\_SERVICE\_TENANT\_NAME**: DevStack service project name (default is *service*)

   -   *admin* user’s credentials :

       -   **OS\_IDENTITY\_API\_VERSION**: DevStack Keystone Identity API Version (2.0|3, default is *3*)
       -   **OS\_PROJECT\_DOMAIN\_ID**: DevStack Project Domain ID (default is *default*)
       -   **OS\_USER\_DOMAIN\_ID**: DevStack User Domain ID (default is *default*)
       -   **OS\_USERNAME**: DevStack User name (default is *admin*)
       -   **OS\_PASSWORD**: DevStack User password (default is *1234*)
       -   **OS\_PROJECT\_NAME**: DevStack Project name (default is *admin*)

3. Into DevStack Identity Service, create the Watcher service user (eg *watcher*): 

        $ keystone user-create --name=watcher --pass=YOUR_WATCHER_PASSWORD 
        $ keystone user-role-add --user=watcher --tenant=service --role=admin

      or

        $ openstack user create --password YOUR_WATCHER_PASSWORD --enable watcher
        $ openstack role add --project SERVICE_PROJECT_NAME --user watcher admin
     
   with:
  
   - YOUR_WATCHER_PASSWORD: the user password
   - SERVICE_PROJECT_NAME: the keystone project name for services

4. You must register the Watcher Service with the Identity Service so that other OpenStack services can locate it. To register the service:

        $ keystone service-create --name=watcher --type=infra-optim --description="Infrastructure Optimization service"

      or 

        $ openstack service create --name watcher infra-optim 

5. Create the endpoints:

        $ keystone endpoint-create \
          --service-id=the_service_id_above \
          --publicurl=http://YOUR_WATCHER_API_IP:9322 \
          --internalurl=http://YOUR_WATCHER_API_IP:9322 \
          --adminurl=http://YOUR_WATCHER_API_IP:9322
      or

        $ openstack endpoint create --region YOUR_REGION watcher public http://YOUR_WATCHER_API_IP:9322 enal
        $ openstack endpoint create --region YOUR_REGION watcher admin http://YOUR_WATCHER_API_IP:9322
        $ openstack endpoint create --region YOUR_REGION watcher internal http://YOUR_WATCHER_API_IP:9322
   with:
   - YOUR_REGION: the OpenStack Region in which declare the watcher service
   - YOUR_WATCHER_API_IP: the Watcher service IP address (or FQDN)

Use DevStack SQL MariaDb database
---------------------------------

1. Edit the template file *docker-compose.yml*:

       $ cd watcher
       $ vi docker-compose.xml

   and remove/comment the *mariadb* service.


2. In this file, set the next environment variable (you have to set it in several services): 
      -   **MARIADB\_NODE**: DevStack MySQL server IP address.

3. On DevStack, create the Watcher database, with these commands

        $ mysql -u<mysql_user> -p<mysql_pwd> -e 'DROP DATABASE IF EXISTS watcher;'
        $ mysql -u<mysql_user> -p<mysql_pwd> -e 'CREATE DATABASE watcher;'
        $ mysql -u<mysql_user> -p<mysql_pwd> -e 'GRANT ALL PRIVILEGES ON watcher.* TO "watcher"@"localhost" IDENTIFIED BY "watcher";'
        $ mysql -u<mysql_user> -p<mysql pwd> -e 'GRANT ALL PRIVILEGES ON watcher.* TO "watcher"@"%" IDENTIFIED BY "watcher";'



Use DevStack RabbitMQ server
----------------------------

1. Edit the template file *docker-compose.yml*:

       $ cd watcher
       $ vi docker-compose.xml

   and remove/comment the *rabbitmq* service.


2. In this file, set the next environment variable (you have to set it in several services): 
      -   **RABBITMQ\_NODE**: DevStack RabbitMQ server IP address.




Debugging
---------

You can enable/disable debug and verbose mode, by setting these parameters:

1. Edit the template file *docker-compose.yml*:

       $ cd watcher
       $ vi docker-compose.xml

2. In this file, set the next environment variable (you can set them in several services): 
      -   **DEBUG\_MODE**: Debug mode (true|false, default is *false*)
      -   **VERBOSE\_MODE**: Verbose mode (true|false, default is *false*)



Run Watcher
===========
To start Watcher services, you can simply run this command: 

  $ cd watcher
  $ docker-compose up -d


Use Watcher
===========

You can use the *console* container to play with Watcher.

1. Connect to the Watcher console container:

        $ docker ps

2. Get the *console* container ID and attach to it :

        $ docker exec -it < watcher_console_container_ID > bash
   and use watcher CLI:

        root@console:/# cd /root

        root@console:/# source creds

        root@console:~# watcher audit-list
        +------+------+---------------------+-------+
        | UUID | Type | Audit Template Name | State |
        +------+------+---------------------+-------+
        +------+------+---------------------+-------+

Important notes
---------------

Please check your Compute’s hypervisor configuration to handle correctly
instance migration:

> [OpenStack - Configure Migrations]

  [OpenStack - Configure Migrations]: http://docs.openstack.org/admin-guide-cloud/content/section_configuring-compute-migrations.html

