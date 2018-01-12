# Create a custom application using Rt 106

You can create a custom application using Rt 106 in three different ways, listed here in increasing level of customization and effort required:
* __Add algorithms to [Rad-Seed](https://github.com/rt106/rt106-rad-seed) or [Path-Seed](https://github.com/rt106/rt106-path-seed).__  Rt 106 provides bare-bones applications for radiology and for pathology.  You can load your own data and integrate your own algorithms and work with these seed applications as they are.  This may be useful at least for initial testing and experiments.  
* __Build an AngularJS app using the provided rt106-app AngularJS services.__  You can use the very powerful [AngularJS](https://angularjs.org/) framework for building a custom app.  Much of the functionality of Rt 106 is available through rt106-app.  There is also a detailed REST API available through rt106-server for still more control of the Rt 106 platform.
* __Build a custom app using any tech stack, using the provided [rt106-server REST API](REFERENCE.md).__  This is the most challenging approach but gives complete flexibility to the kind of application you would like to build.

The [Rt 106 API Reference Guide](REFERENCE.md) provides a complete listing of the REST APIs available from rt106-server and the AngularJS services available from rt106-app.

The examples provided with the source release (e.g. [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed), [rt106-path-seed](https://github.com/rt106/rt106-path-seed)) follow the second approach, leveraging AngularJS.  This document provides a guide to building your own custom application using this approach.


## Creating a custom application in AngularJS leveraging rt106-app

The easiest approach is to start with a copy of [rt106-rad-seed](https://github.com/rt106/rt106-rad-seed) or [rt106-path-seed](https://github.com/rt106/rt106-path-seed), depending on whether you are developing an application for radiology or pathology.  These two seed projects are structurally similar.

### Development
Within either of the seed applications, there are just a small number of source files for you to modify.

#### rt106-app/index.html

index.html loads the appropriate style sheets, Rt 106 components, and third-party libraries, as well as setting up the AngularJS environment.

Bootstrap v4 is used to structure the web-based user interface.  index.html should be straightforward to understand given familiarity with AngularJS and Bootstrap.

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

From this starting point, you can of course use standard HTML / CSS / AngularJS code to evolve the user interface any way you would like.

JavaScript functions called from [rt106-app/index.html](https://github.com/rt106/rt106-rad-seed/blob/master/rt106-app/index.html)  are defined in the controller script, described in the next section.

#### rt106-app/controllers/rt106*Controller.js

This is the AngularJS controller and includes functions that are called by index.html.  The controller code calls the Rt 106 AngularJS services and REST API which are documented in the [Rt 106 API Reference](REFERENCE.md).

As a staring point, the structure of this file is as follows.  It can be modified as required to suit the needs of your application.
* Some required header lines.
* Define variables.
* Initialize the list of patients.
* Initialize the list of algorithms and set up a periodic scan to refresh the list.
* Set up the algorithm execution environment.
* Start periodic polling of the algorithm execution list.
* Define a few functions for interactive user interface manipulations.
* Define function callbacks for interacting with the user interface, e.g. clicking on patients, running algorithms, etc.  These make appropriate calls to the rt106-app AngularJS services.

#### rt106-app/css/rt106*Styles.css

These are the CSS styles specific for your application to supplement the CSS styles defined by the Rt 106 platform.

### Building your application

#### Template files
The seed application files may need to be modified as noted below.

|File|Need to Modify?|
|----|---------------|
|.bowerrc|Used to tell bower where to install packages.|
|.dockerignore|Add any files that you do not want included in a Docker image.|
|.gitignore|Modify as needed.|
|bower.json|Modify as needed to update applicatio  metadata and dependencies (Javascript libraries).|
|docker-compose.yml|Modify to set the name for your application and to specify the algorithms to use.|
|Dockerfile|Modify if the application needs additional libraries to be installed.|
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

Assuming that you have access to the Rt 106 system, such as through Dockerhub, you should be able to run your system by simply executing:
```bash
$ docker-compose up
```
And then pointing a web browser (Chrome recommended) to [localhost:81](http://localhost:81).
