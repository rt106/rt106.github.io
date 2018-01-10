# How to Create Your Own Custom Application using Rt 106

Rt 106 enables you to create your own custom application in three different ways, listed here in increasing level of customization and effort required:
* __Add your own algorithms to Rad-Seed or Path-Seed.__  Rt 106 provides bare-bones applications for radiology and for pathology.  You can load your own data and integrate your own algorithms and work with these seed applications as they are.  This may be useful at least for initial testing and experiments.  
* __Build your own AngularJS app using the provided rt106-app AngularJS services.__  You can use the very powerful [AngularJS](https://angularjs.org/) framework for building a custom app.  Much of the functionality of Rt 106 is available through rt106-app.  There is also a detailed REST API available through rt106-server for still more control of the Rt 106 platform.
* __Build your own custom app using any tech stack you choose, using the provided rt106-server REST API.__  This is the most challenging approach but gives complete flexibility to the kind of application you would like to build.

The Rt 106 Reference Guide provides a complete listing of the REST APIs available from rt106-server and the AngularJS services available from rt106-app.

The examples provided with the source release (e.g. rt106-rad-seed, rt106-path-seed) follow the second approach, leveraging AngularJS.  This document provides a guide to building your own custom application using this approach.


## Creating a custom application in AngularJS leveraging rt106-app

The easiest approach is to start with a copy of rt106-rad-seed or rt106-path-seed, depending on whether you are developing an application for radiology or pathology.  These two seed projects are structurally similar.

### Development
Within either of the seed applications, there are just a small number of source files for you to modify.

#### rt106-app/index.html

This file loads the appropriate style sheets, Rt 106 components, and third-party libraries, as well as setting up the AngularJS environment.

Bootstrap v4 is used to structure the web-based user interface.  index.html should be straightforward to understand given familiarity with AngularJS and Bootstrap.

The structure of the rt106-rad-seed index.html file is:
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

JavaScript functions called from rt106-app/index.html are defined in the controller script, described in the next section.

#### rt106-app/controllers/rt106*Controller.js

This file is the AngularJS controller and includes functions that are called by the HTML page above.  The code in this file calls the Rt 106 AngularJS services and REST API which are documented in REFERENCE.md.

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

This file contains CSS styles that are specific for your application to suppliement other CSS styles defined by the Rt 106 platform.

### Building your application

#### Template files
The included seed application files may be correct as they are, or may need to be modified as noted below.

|File|Need to Modify?|
|----|---------------|
|.bowerrc|Probably OK as-is.|
|.dockerignore|Probably OK as-is, but you may want to consider this if you add more files to the working directory.|
|.gitignore|Probably OK as-is, but you may want to consider this if you add more files to the working directory.|
|bower.json|Probably OK as-is, but needs to be extended if you include additional web packages for your user interface|
|docker-compose.yml|You will probably need to modify this to use the correct name for your application and to include the algorithms that your application requires.|
|Dockerfile|This may be OK as it is, but will need to be modified if you include require additional libraries for your front-end application.|
|entrypoint.sh|Probably OK as-is.|
|package.json|Probably OK as-is.|
|README.md|It is best practice for you to update this documentation for your own application, using markdown format.|
|webrebuild.sh|This is provided for convenience.  If you would like to use this script, you should modify the paths for your own environment.|

#### Building your Docker image

The Docker image for your custom front end needs to be built using a line similar to to the __docker build__ command shown in webrebuild.sh.

#### Running the System

Assuming that you have access to the Rt 106 system, such as through Dockerhub, you should be able to run your system by simply executing:
```
docker-compose up
```
And then pointing a web browser (Chrome recommended) to [localhost:81](http://localhost:81).
