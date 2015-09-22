# Multi-container applications for Watcher

This `docker-compose.yml` template file allows you to run a **complete and autonomous** runtime environment for the Watcher solution. It includes, by default, a Keystone service identity and a MariaDB database.


You can simply run this command, in this directory, to build your runtime environment: 

	$ docker-compose up -d

To check that watcher runs well, you can do this:

	$ docker exec -it watcher_console_1 bash

		root@console:/# cd /root
		root@console:/# source creds
		root@console:/# watcher audit-list
		+------+------+---------------------+-------+
		| UUID | Type | Audit Template Name | State |
		+------+------+---------------------+-------+
		+------+------+---------------------+-------+
		root@console:/# 


Note: if you are add `commands.bash` additional commands into your `$HOME/.bashrc` file, you can use theses commands also:

	$ cd
	$ ti watcher up -d
	$ ti-bash watcher console



You can also run the Watcher solution within your own OpenStack IAAS platform or your devstack runtime environment, by setting environment variables into the `docker-compose.yml` file. You can customize
`console`, `api`, `decision-engine` and `applier` services with theses variables:

- `KEYSTONE_NODE`: Keystone server address (IP or FQDN, default is *keystone.service.indeed.b-com.com*)
- `KEYSTONE_ADMIN_TOKEN`: Keystone admin token (default is *ADMIN_TOKEN*)
- `KEYSTONE_SERVICE_TENANT_NAME`: Keystone service project name (default is *service*)
- `MARIADB_NODE`: MySQL server address (IP or FQDN, default is *mariadb.service.indeed.b-com.com*)
- `RABBITMQ_NODE`: RabbitMQ server address (IP or FQDN, default is *rabbitmq.service.indeed.b-com.com*)
- `INFLUXDB_NODE`: InfluxDb server address (IP or FQDN, default is *influxdb.service.indeed.b-com.com*)
- `DEBUG_MODE`: Debug mode (true|false, default is *false*)
- `VERBOSE_MODE`: Verbose mode (true|false, default is *false*)
- `OS_IDENTITY_API_VERSION`: Keystone Identity API Version for admin user's credentials (2.0|3, default is *3*) 
- `OS_PROJECT_DOMAIN_ID`: Project Domain ID for admin user's credentials (default is *default*)
- `OS_USER_DOMAIN_ID`:  User Domain ID for admin user's credentials (default is *default*)
- `OS_USERNAME`:  User name for admin user's credentials (default is *admin*)
- `OS_PASSWORD`:  User password for admin user's credentials (default is *1234*)
- `OS_PROJECT_NAME`:  Project name for admin user's credentials (default is *admin*)

Update this `docker-compose.yml` file and run again your environment (from this directory) :

	$ docker-compose stop
	$ docker-compose rm
	$ docker-compose up -d

or
	$ cd 
	$ ti-rebuild watcher
