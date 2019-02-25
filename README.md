dockered QGIS
=========

Running QGIS from a docker container is a nice way on Linux to have several QGIS versions available.

The docker file provided here can be adapted to run just about any version of QGIS.

In this case, QGIS is not compiled but simply installed from the [ubuntugis](http://qgis.org/ubuntugis) packages.

The docker file also includes installation of gdal-bin and python-gdal.
The chromium browser is installed as well for accessing help.

OpenGL is configured as well to get rid of anoying warnings. I could not see any differences in graphics performance, but I think it won't hurt to keep it in place.


Building
-----------

build the container with tag qgis3:latest eg:

	docker build -t qgis3:latest path_to_docker_file_goes_here
	
#### Adapting the time zone ####

The `Dockerfile` contains the following line:

	ENV TZ Europe/Amsterdam
	
Adapt this line to your liking before building the container. You can refer to the [list of time zone names](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) to pick the right name.

#### Updating the QGIS version ####

If a new QGIS version is available in the [ubuntugis](http://qgis.org/ubuntugis) packages you can rebuild your container with:

	docker build --no-cache -t qgis3:latest path_to_docker_file_goes_here
	
This method is fail-safe, but takes a long time. Adapting the following line in the docker file will install a new version, but re-uses uncanged layers:

	RUN    echo "Update the number at the end of this line to install new version and retain cached layers: 1" >> /home/cache_defeat.txt
	
After that, build with:

	docker build -t qgis3:latest path_to_docker_file_goes_here

Running
-----------

To run QGIS from a container create a shell script similar to below, perhaps called `docker-qgis`, but you can call it anything you want. Actually, you don't have to create this file as it is included in this repository under the name `docker-qgis`.


	#!/bin/sh

	# Should be platform neutral - at least working on Linux and Windows
	USER_NAME=`basename $HOME`

	# HHHOME is used to pass the HOME directory of the user running qgis
	# and is used in "start.sh" to create the same user within the container.

	# Users home is mounted as home
	# --rm will remove the container as soon as it ends
	docker run --rm \
	    -i -t \
	    -v ${HOME}:/home/${USER_NAME} \
	    -v /tmp/.X11-unix:/tmp/.X11-unix \
	    -e DISPLAY=unix$DISPLAY \
	    -e HHHOME=${HOME} \
	    --cap-add SYS_ADMIN \
	    --network host \
	    --device=/dev/dri:/dev/dri \
	    qgis3:latest

Be sure to make the `docker-qgis` script (or whatever you called your script) an executable.

	chmod a+x docker-qgis

Copy this file into a directory listed in your PATH environment variable to run it from any place you like, eg:

	sudo cp docker-qgis /usr/local/bin

If you want to supply any [startup command line parameters](https://docs.qgis.org/testing/en/docs/user_manual/introduction/getting_started.html#advanced-command-line-options) to QGIS you can put them in the script above, after `qgis3:latest`. eg.

	... \
	qgis3:latest --no-logo

#### Mounting your home directory ####

The `-v ${HOME}:/home/${USER_NAME}` option in this script mounts your home directory in the container.

This means that your countainer does NOT run fully isolated.

This has the (dis)advantage that you can share plugins and the like between several different QGIS containers (or even a locally installed).

If you use the Chromium browser from the container it might corrupt the profile of your Chromium browser running in the host.

Troubleshooting
---------------------

If QGIS crashes or hangs it might leave an orphan docker process running. If you see the process with 

	docker ps

Then run 

	docker stop <process id or container name>

Else run 

	docker ps -a

then

	docker rm <process id or container name>