---
title: Keystone Setup
project: riakcs
version: 1.4.0+
document: api
toc: true
index: true
audience: advanced
keywords: [authentication, openstack]
---

Keystone is a python application and requires a number of dependencies
to be installed. This document covers how to use python, pip, and
virtualenv to set up an isolated test environment for running and
testing Keystone.

## Prerequisites

1. Make sure python is installed. Hopefully this is obvious if you read the paragraph above.
1. Install pip, the latest, greatest python package tool. Installation
   instructions are
   [here](http://guide.python-distribute.org/installation.html#installing-pip). `pip`
   is also available to install via `homebrew` for Mac OSX users.
1. Install virtualenv and virtualenvwrapper: `pip install virtualenv virtualenvwrapper`
1. Setup `virtualenvwrapper`. Add the following lines to `.bash_profile` to
   get the `virtualenvwrapper` scripts in the path.

   ```
   export WORKON_HOME=$HOME/.virtualenvs
   source /usr/local/bin/virtualenvwrapper.sh
   ```
1. Clone the Keystone repo: `git clone https://github.com/openstack/keystone.git`
1. Descend into the Keystone repo and checkout the proper tag: `git checkout grizzly-2`

## Create a virtual envrionment for Keystone

```
mkvirtualenv keystone-test
```

The `keystone-test` virtual environment is now created, activated, and
ready to use.

## Install the Keystone dependences

The dependencies for running Keystone can be found in
`tools/pip-requires`.  At the time of this writing, `grizzly-2` is the
latest tag of Keystone and the dependencies are based on versions that
work with that tag. Use `pip` to install the depencies as follow:

```
pip install -r tools/pip-requires
```

## Configure Keystone

The next step is to select the appropriate options in the
`keystone.conf` configuration file. A sample configuration that is
useful for local testing with Riak CS can be found
[[here|Keystone Configuration Sample]]. This configuration file sets up logging
to `./log/keystone/keystone.log` and uses the templated catalog
backend to set up the Riak CS object store service. This catalog
backend uses a local file to populate the service catalog. The default
file in the previously referenced sample `keystone.conf` file uses
`etc/default_catalog.templates`, but this can be changed. Set the
contents of `etc/default_catalog.templates` to be the following:

```
# config for TemplatedCatalog, using camelCase because I don't want to do
# translations for keystone compat
catalog.RegionOne.identity.publicURL = http://localhost:$(public_port)s/v2.0
catalog.RegionOne.identity.adminURL = http://localhost:$(admin_port)s/v2.0
catalog.RegionOne.identity.internalURL = http://localhost:$(public_port)s/v2.0
catalog.RegionOne.identity.name = Identity Service

catalog.RegionOne.object-store.publicURL = http://localhost:8080/v1/AUTH_$(tenant_id)s
catalog.RegionOne.object-store.adminURL = http://localhost:8080/
catalog.RegionOne.object-store.internalURL = http://localhost:8080/v1/AUTH_$(tenant_id)s
catalog.RegionOne.object-store.name = 'Object Store Service'
```

## Optional configuration

For testing it can be easier to configure keystone to use UUID as the token format. To do this edit `keystone.conf` and set the following:

```
token_format = UUID
```

## Prepare the database

```
./bin/keystone-manage db_sync
```

## Run Keystone

```
./bin/keystone-all --config-file /<absolute-path-to-keystone-repo>/keystone/etc/keystone.conf -d --debug
```

The following script can be used to set a number of useful environment
variables to make using the keystone client less cumbersome.

```
#!/bin/bash

export OS_SERVICE_TOKEN=ADMIN
export OS_SERVICE_ENDPOINT=http://localhost:35357/v2.0
export OS_IDENTITY_API_VERSION=2.0
export OS_AUTH_URL=http://localhost:5000/v2.0
export OS_USERNAME=test
export OS_PASSWORD=test
export OS_TENANT_NAME=test
```