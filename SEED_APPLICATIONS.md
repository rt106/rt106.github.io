# Seed applications

Two example applications are provided to illustrate one of the mechanisms
for building a custom application that uses a Rt 106 backend to manage data,
algorithms, and algorithm executions:

* [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed) - a radiology style application based on processing DICOM images,
* [rt106-path-seed](https://github.com/rt106/rt106-path-seed) - pathology research style application based on processing multi-channel microscopy images.

## Running a seed application

A seed application can be run using one of the Docker Compose files

* [rt106-rad-seed](https://raw.githubusercontent.com/rt106/rt106-rad-seed/master/docker-compose.yml)
* [rt106-path-seed](https://raw.githubusercontent.com/rt106/rt106-path-seed/master/docker-compose.yml)

As when running the built-in demonstration application described in [Getting Started](GETTING_STARTED.md), create an ```.env``` file containing the path to bulk data
```
LOCAL_DATA_DIR=path_to_where_bulk_data_can_be_stored
```
Note: in this ```.env``` file, we are not setting ```Rt106_SERVE_APP``` since the Rt 106 Node.js server does not need to serve an application itself. It just needs to serve the REST endpoints for the Rt 106 platform.  In this example, a separate Node.js server is serving the seed application.

To start the seed application
```bash
$ docker-compose -f docker-compose.yml up
```

Connect a web browser to [http://localhost:81](http://localhost:81) to connect to the seed application.

To stop the seed application
```bash
$ docker-compose -f docker-compose.yml down
```

## Deep dive on ```docker-compose.yml```

Below is the ```docker-compose.yml``` for the rt106-rad-seed

```yaml
# Copyright (c) General Electric Company, 2017.  All rights reserved.
version: '3'
services:
  # one instance of consul in server mode, can use multiple on different VMs for redundancy
  consul:
    command: -server -bootstrap-expect 1
    image: progrium/consul:latest
    ports:
    - 8300:8300
    - 8400:8400
    - 8500:8500
    - 8600:53/udp
  # one instance of registrator for each VM running containers for the platform or for analytics
  registrator:
    command: -internal consul://consul:8500
    image: gliderlabs/registrator:latest
    links:
    - consul
    volumes:
    - /var/run/docker.sock:/tmp/docker.sock
  # one instance of rabbitmq for the platform
  rabbitmq:
    image: rt106/rt106-rabbitmq:latest
    ports:
    - 5672:5672
    - 15672:15672
  # datastore can scale if placed behind a load balancer
  datastore:
    image: rt106/rt106-datastore-local:latest
    ports:
    - 5106:5106
    environment:
      SERVICE_NAME: datastore
      LOAD_DEMO_DATA: "on"
    volumes:
    - "${LOCAL_DATA_DIR}:/rt106/data"
  # one instance of mysql for the platform
  mysql:
    image: rt106/rt106-mysql:latest
    ports:
    - 3306:3306
    volumes:
    - rt106-mysql-volume:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: rt106mysql
  # web server main rt106 system can scale if placed behind a load balancer
  web:
    image: rt106/rt106-frontend:latest
    ports:
    - 80:8106
    environment:
      SERVICE_NAME: web
    depends_on:
    - "rabbitmq"
  # web server for seed app can scale if placed behind a load balancer
  seed:
    image: rt106/rt106-rad-seed:latest
    ports:
    - 81:8306
    environment:
      SERVICE_NAME: seed
  # analytics can scale (but we cannot expose the port to the host, but still to list a port for Registrator to recognize)
  #   the DNS in Consul will provide a unique port number to route through
  algorithmtemplate:
    image: rt106/rt106-algorithm-sdk:latest
    ports:
    - 7106
    environment:
     MSG_SYSTEM: amqp
     SERVICE_NAME: algorithm-template--v1_0_0
     SERVICE_TAGS: analytic

volumes:
  rt106-mysql-volume:
```

The ```docker-compose.yml``` file has several sections, each is detailed below.

### Service Discovery

Rt 106 uses [Consul](https://www.consul.io/) and [Registrator](http://gliderlabs.github.io/registrator/latest/) for service discovery.  Service discovery allows for
applications to detect when algorithms are added and removed from the Rt 106 environment. Consul maintains a catalog of services and their addresses. Registrator detects when Docker containers start and stop and communicates this activity with Consul.

```yaml
# one instance of consul in server mode, can use multiple on different VMs for redundancy
consul:
  command: -server -bootstrap-expect 1
  image: progrium/consul:latest
  ports:
  - 8300:8300
  - 8400:8400
  - 8500:8500
  - 8600:53/udp
# one instance of registrator for each VM running containers for the platform or for analytics
registrator:
  command: -internal consul://consul:8500
  image: gliderlabs/registrator:latest
  links:
  - consul
  volumes:
  - /var/run/docker.sock:/tmp/docker.sock
```

### Rt 106 Services

The base Rt 106 services include

* RabbitMQ for managing work queues, with two queues per algorithm (request and response)
* MySQL for tracking provenance of algorithm executions
* Datastore for managing bulk data
* Web server providing platform REST endpoints to applications

See the [Rt 106 Architecture](ARCHITECTURE.md) for more information.

```yaml
# one instance of rabbitmq for the platform
rabbitmq:
  image: rt106/rt106-rabbitmq:latest
  ports:
  - 5672:5672
  - 15672:15672
# datastore can scale if placed behind a load balancer
datastore:
  image: rt106/rt106-datastore-local:latest
  ports:
  - 5106:5106
  environment:
    SERVICE_NAME: datastore
    LOAD_DEMO_DATA: "on"
  volumes:
  - "${LOCAL_DATA_DIR}:/rt106/data"
# one instance of mysql for the platform
mysql:
  image: rt106/rt106-mysql:latest
  ports:
  - 3306:3306
  volumes:
  - rt106-mysql-volume:/var/lib/mysql
  environment:
    MYSQL_ROOT_PASSWORD: rt106mysql
# web server main rt106 system can scale if placed behind a load balancer
web:
  image: rt106/rt106-frontend:latest
  ports:
  - 80:8106
  environment:
    SERVICE_NAME: web
  depends_on:
  - "rabbitmq"
```

### Rt 106 Volumes

To ease setup and manage persistence beyond the Docker container lifecycle, Rt 106
uses a Docker volume for managing its internal database.

```yaml
volumes:
  rt106-mysql-volume:
```

### Seed application

The seed applications provide there own web server to serve the application. This web server
serves the HTML and Javascript content for the application while using the standard Rt 106 web server to access
the Rt 106 platform REST endpoints.

```yaml
# web server for seed app can scale if placed behind a load balancer
seed:
  image: rt106/rt106-rad-seed:latest
  ports:
  - 81:8306
  environment:
    SERVICE_NAME: seed
```

### Algorithms

To add a algorithm to the seed application, copy and modify the example section below.  The environment
variable ```SERVICE_TAGS: analytic``` identifies the container as providing an algorithm to Rt 106. The
```SERVICE_NAME``` needs to match the algorithm container configuration. See the [Algorithm SDK](ALGORITM_SDK.md) for more information.

```yaml
# analytics can scale (but we cannot expose the port to the host, but still to list a port for Registrator to recognize)
#   the DNS in Consul will provide a unique port number to route through
algorithmtemplate:
  image: rt106/rt106-algorithm-sdk:latest
  ports:
  - 7106
  environment:
   MSG_SYSTEM: amqp
   SERVICE_NAME: algorithm-template--v1_0_0
   SERVICE_TAGS: analytic
```


## Building an application

The seed applications can be used as a template for building new applications. The seed applications run in their own Docker container, providing a Node.js server for the application. The Node.js server for the application communicates with the Rt 106 backend server.

See the [Custom Application SDK](CUSTOM_APPLICATION_SDK.md) for more information on building an application that uses a Rt 106 backend.
