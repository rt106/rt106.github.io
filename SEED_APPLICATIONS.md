# Seed applications

Two examples applications are provided to illustrate one of the mechanisms
for building a custom application that using a Rt 106 backend to manage data,
algorithms, and algorithm executions.

* [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed) - a radiology style application based on processing DICOM images
* [rt106-path-seed](https://github.com/rt106/rt106-path-seed) - pathology research style application based on processing multi-channel microscopy images

# Running a seed application

A seed application can be run using one of the Docker Compose files

* [rt106-rad-seed](https://raw.githubusercontent.com/rt106/rt106-rad-seed/master/docker-compose.yml)
* [rt106-path-seed](https://raw.githubusercontent.com/rt106/rt106-path-seed/master/docker-compose.yml)

As when running the built in demonstration application in [Getting Started](GETTING_STARTED.md), create an ```.env``` file containing the path to bulk data
```
LOCAL_DATA_DIR=path_to_where_bulk_data_can_be_stored
```
Note: in this ```.env``` file, we are not setting ```Rt106_SERVE_APP``` since the Rt 106 Node.js does not need to serve an application itself.  A separate Node.js server is serving the seed application.

To start the seed application
```bash
$ docker-compose -f docker-compose.yml up
```

To stop the seed application
```bash
$ docker-compose -f docker-compose.yml down
```


# Building an application

The seed applications can be used as a template for building new applications. The seed applications run in their own Docker container, providing a Node.js server for the application. The Node.js server for the application communicates with the Rt 106 backend server.

See the [Custom Application SDK](CUSTOM_APPLICATION_SDK.md) for more information on building an application that uses a Rt 106 backend.
