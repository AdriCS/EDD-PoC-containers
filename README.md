# EDD-PoC-containers

During this workshop we will cover the main basic steps for deploying a real application framework: [GrimoireLab](http://grimoirelab.github.io/)

The framework itself is open-source, and stands for doing the following:
>Automatic and incremental data gathering from almost any tool (data source) related with contributing to Open Source development (source code management, issue tracking systems, forums, etc.)

>Automatic gathered data enrichment, merging duplicated identities, adding additional information about contributors affiliation, calculation delays, geographical data, etc.

>Data visualization, allowing filtering by time range, project, repository, contributor, etc.

## Requirements

* docker-ce =>18.04.0-ce
* docker-compose => 1.21.2
* docker-machine => 0.5.0
* virtualbox => 5.2.10-dfsg-6

## Installation

* docker-ce

```
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
$ sudo add-apt-repository \\n   "deb [arch=amd64] https://download.docker.com/linux/debian \\n   $(lsb_release -cs) \\n   stable"
$ sudo apt-get install docker-ce
```

* docker-compose

```
$ sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```

* docker-machine

```
$ base=https://github.com/docker/machine/releases/download/v0.14.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```


* virtualbox

```
$ sudo apt-get install virtualbox
```

>**WARNING:** For running Elasticsearch properly, you must modify the vm.max_map_count value as follows:

```
sudo sysctl -w vm.max_map_count=262144
```

After setting the needed tools, we will cover the following sections:

1. [Running the whole framework in a single container](https://github.com/albertinisg/EDD-PoC-containers/tree/develop/section1)
2. [Running the whole framework in different isolated containers](https://github.com/albertinisg/EDD-PoC-containers/tree/develop/section2)
3. [Running the whole framework in different isolated services. The cluster](https://github.com/albertinisg/EDD-PoC-containers/tree/develop/section3)
4. [Running a testing application in a clusterized environment](https://github.com/albertinisg/EDD-PoC-containers/tree/develop/section4)
4. Running the whole framework in a production environment. The cloud

## License

Licensed under GNU General Public License (GPL), version 3 or later.
