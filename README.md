# Hyrax docker

Current Version:
 
 **Hyrax-1.13.4** \[libdap-3.19.0 : 
 besd-3.18.0 : 
 olfs-1.16.3 : 
 ncWMS-2.2.2\]  

 
## Overview

The hyrax-docker project can build the following images:
* **hyrax**- A complete Hyrax server in a single container. It may be built with or without a bundled ncWMS.
* **besd** - The BackEndServer (BES) component of Hyrax.  
* **olfs** - The OLFS component of Hyrax, a Java Web Application deployed in Apache Tomcat.
* **ncwms** - The ncWMS application from our friends at the [Reading e-Science Centre](http://www.met.reading.ac.uk/resc/home/).

Each of these images can be run standalone or, combined via docker compose or ansible.

By default the service starts up providing access to the default 
data shipped with Hyrax, but can easily be started to serve data from 
the host machine.

> NOTE: This code is based on work started by Gareth Williams at CSIRO and contributed
to OPeNDAP. We are grateful for their support.

## Images

### hyrax
This image contains a complete Hyrax server. Currently based on **CentOS-7** and _yum_ installed **Tomcat-7**.
#### build arguments
* **USE_NCMWS** - Setting the value of the argument to "true" (_--build-arg USE_NCWMS=true_) will cause the ncWMS application to be included in the container. 
* **DEVELOPER_MODE** - Setting the value of the argument to "true"
 (--build-arg DEVELOPER_MODE=true) instructs the build to insert default 
 authentication credentials into the ncWMS admin interface so that it 
 maybe be accessed in the running container. Otherwise the ncWMS admin 
 page is unreachable and not required as its configuration is copied into the image during the build.
#### Environment Variables and Commandline arguments
* **SERVER_HELP_EMAIL (-e)** - The email address of the support person for the service. This will be returned in error and help pages.
* **FOLLOW_SYMLINKS (-s)** - Instructs the server to follow symbolic links in the file system.
* **NCWMS_BASE (-n)** - The system needs to know the public accessible service base for the ncWMS, this will be something like 
http://yourhost:8080 If all you want is to test it on your local system 
then the default value of http://localhost:8080 will suffice.

#### Commandline Examples:
Launch Hyrax using commandline switches to set the admin email to \(_-e support@erehwon.edu_\), enable symbolic link traversal \(_-s_\), and set the ncWMS service base to _\(-n http://foo.bar.com:8080\)_

```docker run --name besd -p 10022:10022 besd -e support@erehwon.edu -s -n http://foo.bar.com:8080```

Launch Hyrax using command line defined environment variables to set the  admin email to \(_-e SERVER_HELP_EMAIL=support@foo.com_\), enable symbolic link traversal \(_-s_\), and set the ncWMS service base to \(_-e NCWMS_BASE=http://foo.bar.com_\)

`docker run --name besd -p 10022:10022 -e FOLLOW_SYMLINKS=true -e SERVER_HELP_EMAIL=support@foo.com -e NCWMS_BASE=http://foo.bar.com besd
`

**NOTE:** _The environment variables are set to the left of the image name. The commandline switches occur AFTER the image name._

### besd
Manifests just the BES service part of the Hyrax server.
#### build arguments (_none_)
#### Environment Variables and Commandline arguments
* **SERVER_HELP_EMAIL (-e)** - The email address of the support person for the service. This will be returned in error and help pages.
* **FOLLOW_SYMLINKS (-s)** - Instructs the server to follow symbolic links in the file system.

#### Commandline Examples:
Launch besd using commandline switches to set the admin email to \(_-e support@erehwon.edu_\)and enabling symbolic link traversal \(_-s_\)

```docker run --name besd -p 10022:10022 besd -e support@erehwon.edu -s ```

Launch Hyrax using command line defined environment envariables to set the  admin email to \(_-e SERVER_HELP_EMAIL=support@foo.com_\) and enable symbolic link traversal \(_-s_\)

`docker run --name besd -p 10022:10022 -e FOLLOW_SYMLINKS=true -e SERVER_HELP_EMAIL=support@foo.com besd
`

**NOTE:** _The environment variables are set to the left of the image name. The commandline switches occur AFTER the image name._

### olfs
#### build arguments
* **USE_NCMWS** - Setting the value of the argument to "true"
 (_--build-arg USE_NCWMS=true_) will cause the OLFS to be configured to
  provide  ncWMS links, but will not include the ncWMS application in the image.
#### Environment Variables and Commandline arguments
* **NCWMS_BASE (-n)** - The system needs to know the public accessible service base for the ncWMS, this will be something like 
http://yourhost:8080 If all you want is to test it on your local system ### ncwms
then the default value of http://localhost:8080 will suffice.

#### Commandline Examples:
Launch the olfs using commandline switches to set the ncWMS service base to \(_-n http://foo.bar.com:8080_\)

```docker run --name besd -p 10022:10022 besd -n http://foo.bar.com:8080```

Launch the olfs using command line defined environment variables to set the ncWMS service base to \(_-e NCWMS_BASE=http://foo.bar.com_\)

`docker run --name besd -p 10022:10022 -e NCWMS_BASE=http://foo.bar.com besd
`

**NOTE:** _The environment variables are set to the left of the image name. The commandline switches occur AFTER the image name._

## Docker-Compose 
We provide several YAML files for docker-compose. All of the files are written to load the file `./local.env` in order to set the environment variables described above. A template for this file may be found in `./local.env.org`, copy it to `./local.env` and edit that to configure your Hyrax instance.

### hyrax.yml     
This builds and launches a composed Hyrax made up of a single **besd** and a single **olfs** container. Log directories for the OLFS, Tomcat, and the BES are mapped to the ./logs directory.

**Start:** `docker-compose -f hyrax.yml up`

**Stop:** `docker-compose -f hyrax.yml down --remove-orphans`

### hyrax_wms.yml
This builds and launches a composed Hyrax made up of a single **besd**, a single **olfs**, and a single **ncWMS* container. Log directories for the OLFS, Tomcat, and the BES are mapped the `./logs` directory.

**Start:** `docker-compose -f hyrax_wms.yml up`

**Stop:** `docker-compose -f hyrax_wms.yml down --remove-orphans`

### developer.yml 
This builds and launches a hyrax_wms, but in developer mode. Log directories for olfs, tomcat, and besd are mapped to the `./logs` directory.

**Start:** `docker-compose -f developer.yml up`

**Stop:** `docker-compose -f developer.yml down --remove-orphans`


## Ansible
It's possible that the exisiting **playbook.yml** file will work, but it has not been tested.


## Building and Running

To build the single container hyrax, clone this project
```
git clone https://github.com/opendap/hyrax-docker
git checkout trials
```
and use `docker build`
```
docker build -t hyrax_image hyrax
```    
to include ncWMS in the image, do this:
```
docker build -t hyrax_image --build-arg USE_NCWMS=true hyrax
```

To run the container:
```
docker run -h hyrax -p 8080:8080 --name=hyrax_container hyrax_image
```

However, you can use different port numbers so that several Hyrax
instances can run on one host (e.g., you can have a development server
running on port 8080 on the native OS and then use `-p 9090:8080` to
bind the Hyrax in the docker container to the localhost:9090.

To run the Dockerized Hyrax on port 80, use
```
sudo docker run -h hyrax -p 80:8080 --name=hyrax_container hyrax_image
```

To stop the container
```
docker stop hyrax_container
docker rm hyrax_container
```
where the argument to `docker stop` is the value passed in for the
`--name` parameter with `docker run`. `docker rm` is needed to start
the container again if you change any of the `docker run` parameters.

## Using Hyrax docker

To serve data other than the default data included with Hyrax, use the
'volume' option with `docker run` to map the path to data on your host
to `/usr/share/hyrax` in the Hyrax docker container.

```
docker run --hostname hyrax --port 8080:8080 --volume <your path>:/usr/share/hyrax --name=hyrax_container hyrax_image
```

# License

Copyright (c) 2017 OPeNDAP, Inc.

Authors: Nathan David Potter <ndp@opendap.org>,
Dan Holloway <dholloway@opendap.org>,
James Gallagher <jgallagher@opendap.org>

This library is free software; you can redistribute it and/or
modify it under the terms of the GNU Lesser General Public
License as published by the Free Software Foundation; either
version 2.1 of the License, or (at your option) any later version.

This library is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public
License along with this library; if not, write to the Free Software
Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA

You can contact OPeNDAP, Inc. at PO Box 112, Saunderstown, RI. 02874-0112.

# Acknowledgements

Based on https://bitbucket.csiro.au/projects/ASC/repos/hyrax-docker/,
Dec 19, 2016, by gareth.williams@csiro.au. That project was licensed
under a CSIRO variation of a MIT / BSD Open Source License. The
license text is in the file CSIRO_MIT_LICENSE

Ideas have been drawn from https://github.com/Unidata/thredds-docker and
various other contributions on dockerhub, including the official postgres
container's exemplar use of variables with an entrypoint.


ORIGINAL README FOLLOWS HERE
----------------------------

Build and run docker containers for the Hyrax bes and olfs.

Overview
--------

A generic Hyrax setup is provided with separate docker containers for the olfs
and besd components to run OPeNDAP services. See:
https://www.opendap.org/software

The bes(d) is built on a centos container using the default centos rpm setup,
with minimal alterations. Any data in /usr/share/hyrax will be served,
including via symlinks.

The olfs is built on a hardened tomcat container from unidata and has minor
tweaks to find the besd. A minimal OGC WMS setup is included by incorporating
https://github.com/Reading-eScience-Centre/ncwms (ncWMS2) with minimal
configuration.

The containers should run without further customisation to serve test data that
is shipped with the bes. Additional customisation can be made via entrypoint
scripts. The entrypoint scripts are used to alter the key conf files on startup
based on environment variables which will be described below. Further local
customisation (and adding your own data) can be acheived by mapping volumes
into the bes container at run time.

Using Hyrax docker
------------------

There is duplicate setup for making the containers build and  run. It can be
driven by docker-compose or ansible (with duplicate embedded docker-compose
syntax). The containers could be run separately but the olfs service would need
to be reconfigured to find a bes.

Environment variables
---------------------

NCWMS_BASE=localhost:8080

> The viewers.xml configuration for ncWMS2 requires specifying a client
  accessible host:port combination, The default setting will result in godiva 
  only working for clients on the localhost. 

SERVER_HELP_EMAIL=help@replaceme

FOLLOW_SYMLINKS=Yes # No is the alternative option

> Key tunable parameters from /etc/bes/bes.conf can be altered with environment
  variables.  This could readily be extended for other tunables like cache size.

To get environment variables to docker-compose (or the docker-compose settings
duplicated in the ansible playbook), an environment file named local.env must
be provided. There is a starting point in the repository at local.env.orig and
.gitignore is set to exclude local.env from the repository.
```
cp local.env.orig local.env # and edit ...
```

Running the containers
----------------------

To run ansible:
```
ansible-playbook -i "localhost," -c local playbook.yml
```

In the ansible case, the running containers (hyrax_olfs_1 and hyrax_besd_1)
should be stopped, removed and images cleaned up with separate docker commands:
```
docker stop hyrax_olfs_1 hyrax_besd_1
docker rm hyrax_olfs_1 hyrax_besd_1
docker rmi olfs besd # optional
```

or, To run docker-compose (containers will be hyraxdocker_besd_1, hyraxdocker_olfs_1):
```
docker-compose build
docker-compose up
# control-c to exit in this case or
docker-compose up -d
docker-compose down
```

Adding your own data
--------------------

Any directories mapped to /usr/share/hyrax will be served. If symbolic links
are enabled, this directory can contain links to other directories that are
mapped.

Adding ssl support
------------------

ssl support can be added to the tomcat config or as an additional proxy layer
(say with an nginx docker container).

ToDo
----

  * [ ] feedback to opendap.org (and Reading and TPAC?) - started
    * [ ] how to run beslistener in the foreground and are all those 
      options needed
    * [x] link for current rpm was/is broken...
    * [ ] contribute this back
  * [ ] avoid duplication of docker-compose config if possible
  * [ ] devise a versioning/tagging scheme to meet both upstream and docker 
    needs
  * [ ] establish a branch/merge strategy and guidance for contributors
  * [x] introduce gosu to enable entrypoints to be run as root and then drop to
    a regular user for the service. - done for olfs via gosu in underlying 
    unidata/tomcat-docker:8 container and custom entrypoint (overriding the 
    unidata one). Using a tomcat security manager might be useful later.
  * [ ] remove need for /etc/.java to be writeable by tomcat

License 
-------

The original author (Gareth Williams) considers the content in this project to
be recipes/data intended to be shared, so a permissive license is applied.
Embedded bash, sed and perl code for manipulation of config data is not
considered to be of particular novel value. Hyrax and ncWMS2 packages are
sourced by the Dockerfiles but not distributed as part of this project. This
may need to be revisited if a container is published, say via dockerhub, but as
the packages are installed from rpm and jar, the included permissive licenses
are expected to be sufficient. 

If you want to contribute to this project under compatible license terms,
simply do so and add your name to the list of contributors. If you wish to use
the work and license it differently, you can do so but are obliged to
acknowledge this work and any acknowledged works that require so. At time of
writing, a dated reference to
https://bitbucket.csiro.au/projects/ASC/repos/hyrax-docker/ would suffice.

hyrax-docker (c) by contributors:
  * gareth.williams@csiro.au

This work is licensed under a CSIRO variation of a MIT / BSD Open Source
License.  The license text is in the file CSIRO_MIT_LICENSE

Acknowledgements
----------------

Ideas have been drawn from https://github.com/Unidata/thredds-docker and
various other contributions on dockerhub, including the official postgres
container's exemplar use of variables with an entrypoint.
