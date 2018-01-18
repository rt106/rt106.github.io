# Getting Started

Rt 106 is a [Docker](https://www.docker.com/)-based system with a micro-service architecture.  The simplest
way to get up and running is with a single host setup using Docker Compose. This
type of setup supports Linux, Mac OS X, and Windows 10. We recommend using [Docker for Mac](https://www.docker.com/docker-mac)
and [Docker for Windows](https://www.docker.com/docker-windows) on the Mac and Windows platforms. The older Docker Toolbox is not supported.

1. Install Docker and Docker Compose on the host computer.
2. Install the example [Rt 106 Docker-compose script](https://raw.githubusercontent.com/rt106/rt106/master/docker-compose.yml). This script can be installed
using Git.
```bash
$ git clone https://github.com/rt106/rt106.git
```
3. Create a file ```.env``` next to the ```docker-compose.yml``` in the ```rt106``` directory. The contents of the ```.env``` file should be
```
Rt106_SERVE_APP=public
LOCAL_DATA_DIR=path_to_where_bulk_data_can_be_stored
```
```Rt106_SERVE_APP``` is used configure the Rt 106 web server to serve a small demonstration application. ```LOCAL_DATA_DIR``` is used to configure a ```rt106-datastore``` to use the host filesystem to store images and other bulk data.
4. Create a file ```docker-compose.dev.yml``` next to the ```docker-compose.yml``` that contains
    ```yaml
    version: '3'
    services:
      # configure the Rt 106 web serve to serve a small demo application
      web:
        environment:
          Rt106_SERVE_APP: ${Rt106_SERVE_APP}
      # tell the datastore to download a set of demonstration data
      datastore:
        environment:
          DOWNLOAD_RAD_DEMO_DATA: "on"
    ```
This configures Rt 106 to serve a small demonstration application.  It also tells the ```rt106-datastore``` to download a set of demonstration data from the [Visible Human Project](https://www.nlm.nih.gov/research/visible/visible_human.html).
5. Run Rt 106
```bash
$ docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```
6. Point a browser at [http://localhost:80](http://localhost:80)
7. To stop the Rt 106 system
```bash
$ docker-compose -f docker-compose.yml -f docker-compose.dev.yml down
```

It will take a few minutes for the demonstration data to download. Unfortunately, the status of the download is not yet displayed on the user interface.  Refresh to see if the demonstration data has completed downloading. Once downloaded, there will be two patients listed - Adam and Eve.  Once the data is downloaded, you can edit the ```docker-compose.dev.yml``` file and set ```DOWNLOAD_DEMO_DATA``` to ```off``` to skip this step in the future.

If your host computer sits behind a proxy server, you may need to modify the ```docker-compose.dev.yml``` file as below to
configure the ```rt106-datastore``` to route through the proxy server in order to download the demonstration data.
```yml
# tell the datastore to download a set of demonstration data
datastore:
  environment:
    DOWNLOAD_RAD_DEMO_DATA: "on"
    http_proxy: url_for_proxy_server
    https_proxy: url_for_secure_proxy_server
    no_proxy: list_of_hosts_to_skip_proxy_server
```


## For more information

* [Example applications](SEED_APPLICATIONS.md) - Example applications for radiology and pathology research that use a Rt 106 backend
* [Architecture](ARCHITECTURE.md) - Key Rt 106 concepts and designs
* [Algorithm SDK](ALGORITHM_SDK.md) - How to build a container around an algorithm that interfaces with Rt 106
* [Custom Application SDK](CUSTOM_APPLICATION_SDK.md) - How to build a custom user experience in front of Rt 106
* [API Reference](REFERENCE.md) - Full API reference
* [Developer Setups](DEVELOPER.md) - Recommended configurations for developing algorithms and applications
