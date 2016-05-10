docker-influxdb
=====================

Forked from tutum/influxdb

InfluxDB image 


Usage
-----

To create the image `containersol/influxdb`, execute the following command on containersol-docker-influxdb folder:

    docker build -t containersol/influxdb .

You can now push new image to the registry:

    docker push containersol/influxdb


  

Running your InfluxDB image
---------------------------

Start your image binding the external ports `8083` and `8086` in all interfaces to your container. Ports `8090` and `8099` are only used for clustering and should not be exposed to the internet:

    docker run -d -p 8083:8083 -p 8086:8086 containersol/influxdb

`Docker` containers are easy to delete. If you delete your container instance and your cluster goes offline, you'll lose the InfluxDB store and configuration. If you are serious about keeping InfluxDB data persistently, then consider adding a volume mapping to the containers `/data` folder:

    docker run -d --volume=/var/influxdb:/data -p 8083:8083 -p 8086:8086 containersol/influxdb


Configuring your InfluxDB
-------------------------
Open your browser to access `localhost:8083` to configure InfluxDB. Fill the port which maps to `8086`. *There is no default user anymore in version 0.9 but you can set `auth-enabled: true` in the config.toml.*

Alternatively, you can use RESTful API to talk to InfluxDB on port `8086`. For example, if you have problems with the initial database creation for version `0.9.x`, you can use the new `influx` cli tool to configure the database. While the container is running, you launch the tool with the following command:

  ```
  docker exec -ti influxdb-container-name /usr/bin/influx
  Visit https://enterprise.influxdata.com to register for updates, InfluxDB server management, and monitoring.
  Connected to http://localhost:8086 version 0.11.1
  InfluxDB shell 0.11.1
  >
  ```

Initially create Database
-------------------------
Use `-e PRE_CREATE_DB="db1;db2;db3"` to create database named "db1", "db2", and "db3" on the first time the container starts automatically. Each database name is separated by `;`. For example:

```docker run -d -p 8083:8083 -p 8086:8086 -e ADMIN_USER="root" -e INFLUXDB_INIT_PWD="somepassword" -e PRE_CREATE_DB="db1;db2;db3" containersol/influxdb:latest```

Alternatively, create a database and user with the InfluxDB 0.9 shell:

```
  > CREATE DATABASE db1
  > SHOW DATABASES
  name: databases
  ---------------
  name
  db1
  > USE db1
  > CREATE USER root WITH PASSWORD 'somepassword' WITH ALL PRIVILEGES
  > GRANT ALL PRIVILEGES ON db1 TO root
  > SHOW USERS
  user  admin
  root  true
```



Initially execute influxql script (Available in influxdb:0.9)
------------------------------------------------------------
Use `-v /tmp/init_script.influxql:init_script.influxql:ro` if you want that script to been executed on the first time the container starts automatically. Each influxql command on separated line. For example:

- Docker run command
```
docker run -d -p 8083:8083 -p 8086:8086 -e ADMIN_USER="root" -e INFLUXDB_INIT_PWD="somepassword" -v /tmp/init_script.influxql:init_script.influxql:ro containersol/influxdb:latest
```

- The influxdb script
```
CREATE DATABASE mydb
CREATE USER writer WITH PASSWORD 'writerpass'
CREATE USER reader WITH PASSWORD 'readerpass'
GRANT WRITE ON mydb TO writer
GRANT READ ON mydb TO reader
```



Graphite API support
----------------------------------------
InfluxDB has plugin to support the [Graphite Carbon API](http://graphite.readthedocs.org/en/1.0/feeding-carbon.html). This can be customized via the following variables:

- GRAPHITE_DB: name of the database the graphite plugin shall write the incoming metrics to
- GRAPHITE_BINDING: by default the graphite plugin listens on ':2003'. You can provide any `ipaddress:port`
- GRAPHITE_PROTOCOL: 'udp' or 'tcp' (default)
- GRAPHITE_TEMPLATE: By default the template is set to `instance.profile.measurement*` which will parse a metric and create tags from it

```docker run -d -p 8083:8083 -p 8086:8086 -p 2015:2015 -e ADMIN_USER="root" -e INFLUXDB_INIT_PWD="somepassword" -e PRE_CREATE_DB=my_db -e GRAPHITE_DB="my_db" -e GRAPHITE_BINDING=':2015' -e GRAPHITE_PROTOCOL="udp" -e GRAPHITE_template="tag1.tag2.tag3.measurement*" containersol/influxdb```

More details on the configuration of InfluxDB's graphite plugin can be found at: https://github.com/influxdb/influxdb/blob/master/services/graphite/README.md


Collectd support
----------------------------------------
InfluxDB has a plugin to support the [collectd network plugin](https://collectd.org/wiki/index.php/Plugin:Network). This can be customized via the following variables:

- COLLECTD_DB: name of the database the collectd plugin shall write the incoming metrics to
- COLLECTD_BINDING: by default the collectd plugin listens on ':25826'. You can provide any `ipaddress:port`
- COLLECTD_RETENTION_POLICY: custom retention policy
- types.db: default types.db from collectd version 5.5.0 is provided. For custom types consider adding a volume mapping for /usr/share/collectd/types.db

```docker run -d -p 8083:8083 -p 8086:8086 -p 25826:25826/udp -e ADMIN_USER="root" -e INFLUXDB_INIT_PWD="somepassword" -e PRE_CREATE_DB=my_db -e COLLECTD_DB="my_db" -e COLLECTD_BINDING=':25826' -e COLLECTD_RETENTION_POLICY="mypolicy" containersol/influxdb```

More details on the configuration of InfluxDB's collectd plugin can be found at: https://github.com/influxdb/influxdb/blob/master/services/collectd/README.md


UDP support
----------------------------------------
If you provide a `UDP_DB`, influx will open a UDP port (4444 or if provided `UDP_PORT`) for reception of events for the named database.

```docker run -d -p 8083:8083 -p 8086:8086 -p 4444:4444/udp --expose 8090 --expose 8099 --expose 4444 -e UDP_DB="my_db" containersol/influxdb```

Clustering (Available in influxdb:0.11.1)
----------------------------------------

```bash
# (make sure firewall allows ports 8088, 8089)
docker run -p 8088:8088 -e FORCE_HOSTNAME=192.168.0.1:8088 -e JOIN=192.168.0.1:8088,192.168.0.1:8089,192.168.0.1:8090-t containersol/influxdb
docker run -p 8089:8088 -e FORCE_HOSTNAME=192.168.0.1:8089 -e JOIN=192.168.0.1:8088,192.168.0.1:8089,192.168.0.1:8090 -t containersol/influxdb
docker run -p 8089:8088 -e FORCE_HOSTNAME=192.168.0.1:8090 -e JOIN=192.168.0.1:8088,192.168.0.1:8089,192.168.0.1:8090 -t containersol/influxdb
```



