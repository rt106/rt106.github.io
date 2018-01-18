# Create an application using Rt 106

You can create a custom application using Rt 106 in four different ways, listed here in increasing level of customization and effort required:
* __Add algorithms to [Rad-Seed](https://github.com/rt106/rt106-rad-seed) or [Path-Seed](https://github.com/rt106/rt106-path-seed).__  Rt 106 provides bare-bones applications for radiology and for pathology.  You can load your own data and integrate your own algorithms and work with these seed applications as they are.  This is useful for initial testing and learning about Rt 106.  
* __Build an AngularJS app using the provided rt106-app AngularJS services.__  You can use the  [AngularJS](https://angularjs.org/) framework for building a custom application.  Much of the functionality of Rt 106 for building applications is available through rt106-app.  There is also a detailed REST API available through rt106-server for deeper control of the Rt 106 platform. There are two variants to this approach
  - __Use the Rt 106 web server to serve the application.__ This requires setting two variables in your ```.env``` file (see [Getting Started](GETTING_STARTED.md)),
    ```
    Rt106_SERVE_APP=my-app
    Rt106_SERVE_DIR=path_on_host_to_app_code
    ```
  and setting a volume mapping in your ```docker-compose.dev.yml``` file
      ```yaml
      web:
        volumes:
        - "${Rt106_SERVE_DIR}:/rt106/my-app"
        environment:
          Rt106_SERVE_APP: ${Rt106_SERVE_APP}
      ```
  This configuration maps a directory on your host into the container and tells the Rt 106 web server to serve the application from that mapped directory.
  - __Use a second web server to serve your application content__ but defers to the Rt 106 web server for Rt 106 platform REST endpoints.
* __Build a custom app using any tech stack, using the provided [rt106-server REST API](REFERENCE.md).__  This is the most challenging approach but gives complete flexibility to the kind of application you would like to build.

The [Rt 106 API Reference Guide](REFERENCE.md) provides a complete listing of the REST APIs available from rt106-server and the AngularJS services available from rt106-app.

The examples provided with the source release (e.g. [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed), [rt106-path-seed](https://github.com/rt106/rt106-path-seed)) follow the second variant of the second approach, serving an AngularJS application from a second web server. This document provides a guide to building your own custom application using this approach.


## Creating an AngularJS application with rt106-app

The easiest approach is to start with a copy of [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed) or [rt106-path-seed](https://github.com/rt106/rt106-path-seed), depending on whether you are developing an application for radiology or pathology.  These two seed projects are structurally similar.

### Development
Within either of the seed applications, creating a custom application requires modifying a small number of files.

#### rt106-app/index.html

index.html loads style sheets, Rt 106 components, and third-party Javascript libraries, as well as setting up the AngularJS environment.

[Bootstrap v4](https://getbootstrap.com/) is used to structure the web-based user interface.  index.html should be straightforward to understand given familiarity with AngularJS and Bootstrap.

The structure of the [rt106-rad-seed/rt106-app/index.html](https://github.com/rt106/rt106-rad-seed/blob/master/rt106-app/index.html) file is:
* Load CSS style sheets.
* Declare AngularJS controller.
* Display navigation bar.
* Display top row.
  * Display patients list / patient details tabs.
  * Display image tools.
  * Display image viewer.
* Display second row.
  * Display algorithm list.
  * Display execution list.
* Load third-party scripts.
* Load Rt 106 scripts.

From this starting point, you can evolve the HTML / CSS / AngularJS for you application.

JavaScript functions called from [rt106-app/index.html](https://github.com/rt106/rt106-rad-seed/blob/master/rt106-app/index.html)  are defined in the controller script, described in the next section.

#### rt106-app/controllers/rt106*Controller.js

rt106*Controller.js is the AngularJS controller and provides functions that are called by index.html.  The controller code calls the Rt 106 AngularJS services and REST API which are documented in the [Rt 106 API Reference](REFERENCE.md).

The structure of this file is as follows
* Required header lines.
* Define variables.
* Initialize the list of patients.
* Initialize the list of algorithms and set up a periodic scan to refresh the list.
* Set up the algorithm execution environment.
* Start periodic polling of the algorithm execution list.
* Define functions for interactive user interface manipulations.
* Define function callbacks for interacting with the user interface, e.g. clicking on patients, running algorithms, etc.  These call into the rt106-app AngularJS services.
Most of the changes to the application logic will be made to this file.

#### rt106-app/css/rt106*Styles.css

rt106*Styles.css are the CSS styles specific for your application to supplement the CSS styles defined by the Rt 106 platform.

### Building your application

#### Template files
Additional files that may need to be modified for your applciation:

|File||
|----|---------------|
|.bowerrc|Used to tell bower where to install packages. No changes usually needed.|
|.dockerignore|Modify as needed, add any files that should never be included in the Docker image.|
|.gitignore|Modify as needed.|
|bower.json|Modify as needed to update application metadata and dependencies (Javascript libraries).|
|docker-compose.yml|Modify to set the name of the application and to specify the algorithms to use.|
|Dockerfile|Modify as needed - install additional libraries or to copy files to different locations.|
|entrypoint.sh|No changes usually needed.|
|package.json|Modify as needed to update application metadata and dependencies.|
|README.md|Update to document the application.|
|webrebuild.sh|Optional convenience script to manage the Docker image lifecycle.|

#### Building the application Docker image

To build the docker container for the rad seed:
```bash
$ docker build -t rt106/my-rt106-app:latest .
```
If you use HTTP proxies in your environment, you may need to build using:
```bash
$ docker build -t rt106/my-rt106-app:latest  --build-arg http_proxy=$http_proxy --build-arg https_proxy=$https_proxy  --build-arg no_proxy=$no_proxy .
```

#### Running the System

To run your application:
```bash
$ docker-compose up
```
and point a web browser (Chrome recommended) to [localhost:81](http://localhost:81). Docker Compose will download the rest of the Rt 106 system from [Docker Hub](https://cloud.docker.com/swarm/rt106/repository/list).
