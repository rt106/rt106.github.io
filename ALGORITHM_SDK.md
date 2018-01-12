# Adding algorithms to Rt 106
Rt 106 makes it easy to add your own algorithms.  The procedure for adding an algorithm would be expected to take 2-3 hours, depending on the complexity of your algorithm.

(If your algorithm follows a vastly different paradigm or workflow than has previous been integrated into the Rt 106 platform, requiring changes to the platform itself, the process may take longer.)

Rt 106 is [Docker](https://docker.com/)-based.  Algorithms run from a Docker container.

An algorithm container contains the algorithm itself and an **adaptor** that interfaces the algorithm to the rest of Rt 106.  The adaptor has a generic part and an application-specific part.  Therefore, an algorithm container includes three components:

* generic adaptor
* application-specific adaptor
* algorithm

Here is a brief description of each of these components:

**generic adaptor** - This code receives requests to run the algorithm from the Rt 106 internal messaging system and returns status and response data to the messaging system.  It also downloads data from and uploads data to Rt 106's datastore.  

The generic adaptor has been designed to include as much of the adaptor functionality as possible, and *does not need to be changed* to integrate your algorithm.  You do not need change the generic adaptor in any way to perform your integration.

**application-specific adaptor** -- This is the minimal adaptor code and metadata that is specific for your algorithm.  The documentation below will guide you through the steps to build your own application-specific adaptor.

**algorithm** -- The code for your algorithm to integrate with Rt 106.

The steps required to integrate your algorithm are:

0. Configure required Rt 106 files.
1. Create an algorithm container for your algorithm.
2. Define your algorithm's metadata.
3. Write a short wrapper script for your algorithm.
4. Add your algorithm container to the Docker Compose network.

These steps are described below.  Rt 106 includes two complete examples:
* [simple region growing](https://github.com/rt106/rt106-simple-region-growing) - an example for medical imaging / radiology
* [wavelet-based nuclear segmentation](https://github.com/rt106/rt106-wavelet-nuclei-segmentation) - an example for pathology / microscopy

## What types of algorithms can be added to Rt 106?
Rt 106 is intended for algorithms that execute in batch mode.  In other words, the algorithms are expected to receive inputs and parameters and to produce outputs, but should not have interactive / user interface code intermixed with the algorithm.

Rt 106 can be used to create interactive *applications* by building the interactive elements into a custom user interface.  The algorithms themselves should still be structured as batch algorithms having no user interfaces of their own.

Rt 106 places no constraints on the implementation language for an algorithm.  Algorithms can be implemented in Python, C++, Java, ... Any algorithm that can run in a Docker container can be integrated.


## 1.  Gather required Rt 106 files
A subset of files from the [rt106-algorithm-sdk](https://github.com/rt106/rt106-algorithm-sdk) are needed.  Clone the rt106-algorithm-sdk repository and copy the following files into the directory that contains your algorithm:
* rt106SpecificAdaptorCode.py [required]
* rt106SpecificAdaptorDefinitions.json [required]
* entrypoint.sh [optional, if no changes are needed from SDK version]

## 2. Create an algorithm container
This guide describes the specific details of creating your algorithm's Docker container for Rt 106.
Docker is a pervasive open-source containerization technology.  General documentation for Docker is available from [Docker's website](https://docs.docker.com/).

Create a Dockerfile to construct your Rt 106 Docker container. See the [Docker](https://docs.docker.com/engine/reference/builder/) documentation for general details on Dockerfiles. An example Dockerfile from the [simple region growing](https://github.com/rt106/rt106-simple-region-growing) example algorithm is below:

```dockerfile
# Copyright (c) General Electric Company, 2017.  All rights reserved.

FROM rt106/rt106-algorithm-sdk

# add the artifacts emitted from the dev container
ADD rt106-simple-region-growing.tar.gz /rt106/bin

# add the adaptor code specialized for this algorithm
ADD rt106SpecificAdaptorCode.py rt106SpecificAdaptorDefinitions.json entrypoint.sh /rt106/

# set permissions
USER root
RUN chown -R rt106:rt106 /rt106

# set the working directory
WORKDIR /rt106

# establish user (created in the base image)
USER rt106:rt106

# configure the default port for an analytic, can be overridden in entrypoint
EXPOSE 7106

# entry point
CMD ["/rt106/entrypoint.sh"]
```
Notes:
* The base image for all Rt 106 algorithm containers is rt106/rt106-algorithm-sdk.  This base image contains the generic adaptor described above, and also a Python environment.
* rt106SpecificAdaptorCode.py and rt106SpecificAdaptorDefinitions.json are the files from the SDK, which you will modify below.
* The WORKDIR should be /rt106 as shown.
* entrypoint.sh needs to be called as shown.

Beyond these details, the Dockerfile must build your algorithm.  (The base image is based on Debian Linux.  Please contact the Rt 106 developers if this will not work for your algorithm.)

For algorithms implemented as Python scripts, you may be able to simply copy your source files into the container using the ADD command.  For more complex algorithms implemented using compiled code and class libraries, you may want to refer to the implementation of the simple region growing algorithm example, which uses a "dev" Docker image to create the executable which is then simply added to the "ops" Docker image as shown in the Dockerfile above using this line:
* ADD simpleRegionGrowing.tar.gz /rt106/bin

## 3. Define algorithm metadata
You need to specify a bit of metadata for your algorithm.  Much of this metadata describes the parameters, inputs, and outputs for your algorithm. This metadata is also used to automatically generate a default user interface for your algorithm. You also have the flexibility to create a custom, specialized user interface if that is appropriate for your application.  Doing so does require a custom front end for your application.

The metadata approach borrows heavily from the approach used in [3D Slicer](https://www.slicer.org/), although Rt 106 uses more modern web technologies (e.g. JSON rather than XML).

Metadata for an algorithm includes:
* name
* version
* queue name
* classification (so that an app can filter for algorithm types)
* inputs and parameters
* results
* display

This metadata is documented using a JSON format in rt106SpecificAdaptorDefinitions.json.  Several examples are provided in _rt106-algorithm-sdk_, _simple-region-growing_, and _wavelet-nuclear-segmentation_.  Please refer to these as a starting point for your own algorithm's metadata. A [JSON schema](https://raw.githubusercontent.com/rt106/rt106-frontend/master/rt106-server/algorithmDescriptionSchema.json) is under development.

Please refer to [REFERENCE.md]() for a detailed description of all the JSON metadata fields for algorithm integration.

## 4. Write a short wrapper script

Modify rt106SpecificAdaptorCode.py (copied above to your algorithm's directory), to add details to the Python function run_algorithm() which is called by the Rt 106 platform when there is a request to run your algorithm.  The function needs to:
* Download data from the datastore, checking for errors such as missing data.
* Call your algorithm, checking for errors that may be returned.
* Store generated result data back to the datastore.
* Return status and data back to the user interface.

Please refer to the supplied examples in _rt106-algorithm-sdk_, _simple-region-growing_, and _wavelet-nuclear-segmentation_.  You must implement the Python function

```python
def run_algorithm(datastore, context):
```

__datastore__ is an object that provides a number of functions for accessing the Rt 106 datastore.  Please refer to REFERENCE.md for its API.

__context__ is an object that contains all the _parameters_ that you defined in your JSON metadata file.  For example, if your JSON file includes a parameter called _inputSeries_, then within _run_algorithm_ you should have a variable _context['inputSeries']_ available to you.

_run_algorithm_ should return a Python object (JSON structure) that includes the result data and execution status.  The last line of the function will likely look something like this:

```python
return { 'result' : result_context, 'status' : status }
```

__result_context__ should be an object structure that includes fields for each element of the _results_ section of your JSON file.

__status__ may simply be a string such as "EXECUTION_FINISHED_SUCCESS" or an appropriate error message.

Your algorithm and adaptor will run within the Docker container created in step 2 above.  This container will have a Python environment available.  If you require any special libraries or packages for your algorithm or managing its data, you should add the setup for these to your Dockerfile.

### entrypoint.sh

Another file that may be needed for your _application-specific adaptor_ is entrypoint.sh.  This is the shell script that is called when the algorithm container is launched.  This script sets up a few things including the location of the datastore and the type of queuing system used.  Please refer to the _entrypoint.sh_ file in the _simple region growing_ algorithm example for the form of _entrypoint.sh_.  entrypoint.sh is not needed (and does not need to be listed in the Dockerfile) if the version supplied in [rt106-algorithm-sdk](https://github.com/rt106/rt106-algorithm-sdk) is sufficient.

## 5. Add algorithm container to the Docker Compose network.

Docker Compose can be used to launch the entire Rt 106 environment, including your own algorithm containers. (Docker Compose is not required for runnning Rt 106.  But it is an quick way to get an environment running.)

Add a service to the Docker Compose file to add your algorithm, e.g.

```yaml
  myalgorithm:
    image: rt206/my-algorithm:latest
    ports:
    - 7106
    environment:
      MSG_SYSTEM: amqp
      SERVICE_NAME: myalgorithm--v1_0_0
      SERVICE_TAGS: analytic
```
There are a multiple options for organizing your Docker Compose file(s).  First, your application could have a single docker-compose.yml file which simplifies launching your Rt 106 environment
```bash
$ docker-compose up
```
Another option is to use multiple Docker Compose files, for example, one that configures the core Rt 106 environment and a second that configures the algorithms you wish to use
```bash
$ docker-compose -f docker-compose.yml -f docker-compose.myapp.yml up
```
Complete documentation for Docker Compose can be found [here](https://docs.docker.com/compose/)

-----------------------------------------------------------------------------------------


For questions and help about these instructions, please contact [Jim Miller](mailto:millerjv@ge.com) and / or [Brion Sarachan](mailto:sarachan@ge.com).
