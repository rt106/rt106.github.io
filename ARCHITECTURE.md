# Rt 106 Architecture

Rt 106 has a [Docker](http://www.docker.com)-based service oriented architecture. Rt 106 is designed to scale
across a variety of deployments, ranging from cloud deployments support many users
and algorithms down to laptop deployments for simple use cases and demos.

## Server

The hub of a Rt 106 installation is the ```rt106-server```. The ```rt106-server``` is a [Node.js](https://nodejs.org/) application that provides REST endpoints for all Rt 106 services.  The ```rt106-server``` is the main component
of the Rt 106 architecture to which applications will interface. The ```rt106-server``` provides access to data, algorithms, and algorithm execution history and status.

The ```rt106-server``` can be scaled to support user demand by placing several instances of the ```rt106-server``` behind a load balancer.

## Datastore

The Rt 106 Datastore provides REST endpoints for listing, reading, and writing bulk data. Rt 106 provides a Datastore that
can manage data on a [local filesystem](https://github.com/rt106/rt106-datastore-local). Future Datastores will also be able to manage data on AWS S3. Two data models are currently supported in Rt 106.  The first is clinically oriented to manage patients, images, waveforms, and clinical records. See [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed) for an example application using the clinical data model. The second is pathology research oriented to manage slides, regions, and channels. See [rt106-path-seed](https://github.com/rt106/rt106-path-seed) for an example application using the pathology research data model.

The ```rt106-datastore``` can be scaled to support user demand by placing several instances of a Datastore behind a load balancer.

## Algorithms

Algorithms are managed in Rt 106 as separate Docker containers. Each algorithm provides a small REST interface which allows Rt 106 to query the algorithm for a description of its inputs, parameters, outputs, and display. Rt 106 provides an [Algorithm SDK](ALGORITHM_SDK.md) for adding algorithms to Rt 106.  The SDK provides a base Docker image with the necessary tooling for connecting to Rt 106, access an algorithm job queue, and download data and uploading results.  Algorithms can be authored in any language. Adding an algorithm to Rt 106 requires creating a JSON description of the algorithm, creating a script to run the algorithm, and creating a Docker image.

Algorithms can be scaled to support user demand by instantiating multiple instances of an algorithm.  No load balancer is needed to scale algorithms as algorithms will pull work from a job queue.  An algorithm instance will only execute one algorithm request at a time.

## Applications

Applications interface to the ```rt106-server``` to access data and request algorithm executions.  Rt 106 provides a Javascript library ```rt106-app``` to make it simple to create Rt 106 applications.  ```rt106-app``` is based on [AngjularJS](https://angularjs.org/). ```rt106-app``` can be pulled into an application's codebase using [Bower](https://bower.io/). (Future versions of Rt 106 will move to using Yarn or Webpack.)

Multiple applications can access a single ```rt106-server``` and ```rt106-datastore```.  Or, each application could have its own ```rt106-server``` and ```rt106-datastore```.

## Queues

Each algorithm in Rt 106 has a job queue (currently based on [RabbitMQ](https://www.rabbitmq.com/)). Multiple instances of the same algorithm share a single job queue.

## Service Discovery

Rt 106 uses Service Discovery to manage a dynamic catalog of algorithms.  Service Discovery allows for algorithms to be added, removed, and versioned in a Rt 106 deployment without system administration of ```rt106-server```.  Rt 106 uses [Consul](https://www.consul.io/) and [Registrator](http://gliderlabs.github.io/registrator/latest/) to support Service Discovery.
