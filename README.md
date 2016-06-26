## README outline
- [Project status](#project-status)
- [NEMEA System](#nemea-system)
   - [Parts of the system](#parts-of-the-system)
   - [Repositories](#repositories)
- [Dependencies](#dependencies)
- [Installation](#installation)
   - [Binary packages](#binary-packages)
   - [Source codes](#source-codes)
   - [Vagrant](#vagrant)
   - Packer
- [Quick start and how to](#quick-start-and-how-to)
   - [Deploy NEMEA](#deploy-nemea)
   - [Create your own module](#create-your-own-module)
- [NEMEA Related publications](#nemea-related-publications)



## Project status
Travis CI build: [![Build Status](https://travis-ci.org/CESNET/Nemea.svg?branch=master)](https://travis-ci.org/CESNET/Nemea)



# NEMEA System

## Parts of the system

![NEMEA parts](doc/NEMEA-parts.png)

## Repositories

The NEMEA system consists of:
* [NEMEA framework](https://github.com/CESNET/Nemea-Framework): The heart of the system that provides interconnection of modules, data format (and its handling) and common functions, algorithms and data structures.
* [NEMEA modules](https://github.com/CESNET/Nemea-Modules): Base modules of the system for export&storage of flow data, replay of stored flow data, filtering, merging, and others. It also contains a basic flow exporter capable of extracting L7 information. 
* [NEMEA detectors](https://github.com/CESNET/Nemea-Detectors): Detection modules that can detect and report various types of malicious traffic such as DoS, DDoS, scanning, bruteforce attacks, etc.
* [NEMEA Supervisor](https://github.com/CESNET/Nemea-Supervisor): Central management and monitoring tool of the NEMEA system. It takes care of running modules according to a specified configuration.

This file describes the installation and basic usage of the NEMEA system.
To see more general information, please have a look at
https://www.liberouter.org/nemea.



## Dependencies

### Building environment

* autoconf
* automake
* gcc
* gcc-c++
* libtool
* libxml2-devel
* libxml2-utils (contains xmllint on Debian)
* make
* pkg-config

### Optional dependencies of modules and detectors

* rpm-build (build of RPM packages)
* libpcap (flow_meter)
* libnfdump (http://sourceforge.net/projects/libnfdump/) or libnf (https://github.com/VUTBR/nf-tools/tree/master/libnf/c) (nfreader)
* libidn (blacklistfilter)
* bison and flex (unirecfilter)

### How to install dependencies:

```
dnf install -y autoconf automake gcc gcc-c++ libtool libxml2-devel make pkg-config libpcap libidn bison flex
```

On older systems use `yum` instead of `dnf`.

On Debian-based systems:

```
apt-get update;
apt-get install TODO
```



# Installation

There are three different ways of installation of the NEMEA system covered
in this document: **vagrant**, **binary packages** and **source codes**.


## Binary packages

The NEMEA system can be installed from binary RPM packages.
To add CESNET's repository containing the packages, run (as root/sudo):
```
rpm -ivh https://homeproj.cesnet.cz/rpm/liberouter/devel/x86_64/liberouter-devel-1.0.0-1.noarch.rpm
```

After that, NEMEA can be installed as any other package (run as root/sudo):
```
yum install nemea
```

Note: `yum` was replaced by `dnf` in some distributions.

For development purposes, there is `nemea-framework-devel` package that installs
all needed development files and docs.


## Source codes

The whole system is based on GNU/Autotools build system that makes dependency checking and
building process much more easier.

To clone the read-only repositories, use:

```
git clone --recursive https://github.com/CESNET/nemea
```

After successful clone, use:
```
./bootstrap.sh
```
that will create `configure` scripts and other needed files.

The `configure` script supplies various possibilities of
configuration and it uses some environmental variables that influence the build
and compilation process. For more information see:
```
./configure --help
```

We recommend to set paths according to the used operating system, e.g.:
```
./configure --prefix=/usr --bindir=/usr/bin/nemea --sysconfdir=/etc/nemea --libdir=/usr/lib64
```

After finishing `./configure`, build process can be started by:

```
make
```

The make(1) tool has various parameters, to build the NEMEA package faster on
multicore systems, we recommend to use parameter -j with the number of jobs
that should be run in parallel.

When the compilation process ends without any error, the package can be installed
into paths that were set by `configure`. It is recommended NOT to change
target paths by passing variables directly to make(1).
The installation can be done by (usually it requires root / sudo):

```
make install
```

Congratulations, the whole NEMEA system should be installed right now... :-)


## Vagrant

To try the system "out-of-box", you can use [Vagrant](https://www.vagrantup.com/).
For more information see [./vagrant/](./vagrant/).




# Quick start and how to
## Deploy NEMEA
## Create your own module











Quick Start Guide
=================

The heart of the NEMEA system is a NEMEA module. NEMEA modules are building blocks - independent system processes
that can be connected with each other. Information about every module can be found in its help:
````
./module -h
```

Every NEMEA module can have one or more communication interfaces (IFC) implemented in
[libtrap](https://github.com/CESNET/Nemea-Framework/tree/master/libtrap). There are two types of IFCs: **input** and **output**. Numbers of module's IFCs
can be found in its help.

At the beginning, let's try the `logreplay` module ([modules/logreplay](https://github.com/CESNET/Nemea-Modules/tree/master/logreplay)).
The help output shows that `logreplay` has one output IFC:
```
Name: LogReplay
Inputs: 0
Outputs: 1
Description:
  This module converts CSV from logger and sends it in UniRec. The first row
  of CSV file has to be data format of fields.
```

The complement module is `logger` ([modules/logger](https://github.com/CESNET/Nemea-Modules/tree/master/logger)), help output:
```
Name: Logger
Inputs: variable
Outputs: 0
Description:
  This module logs all incoming UniRec records to standard output or into a
  specified file. Each record is written as one line containing values of its
  fields in human-readable format separated by chosen delimiters (CSV format).
  If you use more than one input interface you have to specify output format
  by parameter "-o".
```

Two modules can be interconnected using one input IFC and one output IFC.

The [./use-cases](./use-cases) directory contains example scripts that demonstrate usage and functionality of
NEMEA modules. `logreplay` and `logger` can be found in [./use-cases/logger-repeater.sh](./use-cases/logger-repeater.sh).
Start the script to see how flow records are replayed from CSV file by `logreplay` and received by `logger`:
```
cd use-cases
./logger-repeater.sh generate
```

To get usage of scripts from `use-cases`, execute a script without parameter. The `generate` parameter of
`logger-repeater.sh` can be used to generate CSV file automatically. For more information, see source codes of
scripts.

`logreplay` is one of possible ways of getting data into the NEMEA system.
There is a [nfreader](https://github.com/CESNET/Nemea-modules/tree/master/nfreader) module that is able to read and replay `nfdump` files.
Last but not least, there is an [ipfixcol](https://github.com/CESNET/ipfixcol/) with [ipfixcol2unirec](https://github.com/CESNET/ipfixcol/tree/master/plugins/storage/unirec)
that is capable of exporting flow data in UniRec format and sending it via libtrap IFC.

Manage NEMEA modules efficiently
================================

The Nemea system can be managed and monitored by a special module called
[Supervisor](https://github.com/CESNET/Nemea-Supervisor).

Some modules that are contained in Nemea-Modules and Nemea-Detectors provide their default
configuration in [nemea-supervisor/configs/](https://github.com/CESNET/Nemea-Supervisor/tree/master/configs/).
To use prepared configuration, run `make` in `nemea-supervisor/configs` and start:
```
nemea-supervisor/supervisor -f nemea-supervisor/configs/supervisor_config.xml
```
To start `supervisor` in an interactive mode, use `-d`

For more information about Supervisor see its [README](https://github.com/CESNET/Nemea-Supervisor/blob/master/README.md).

Note: It is totally up to user whether to use `nemea-supervisor/configs` or not.  It is just
an example of a working configuration.


NEMEA Related Publications
==========================

* Tomáš Čejka, Radoslav Bodó, Hana Kubátová: Nemea: Searching for Botnet Footprints. In: Proceedings of the 3rd Prague Embedded Systems Workshop (PESW), Prague, CZ, 2015.

* Tomáš Čejka, Václav Bartoš, Lukáš Truxa, Hana Kubátová: [Using Application-Aware Flow Monitoring for SIP Fraud Detection](http://link.springer.com/chapter/10.1007/978-3-319-20034-7_10). In: Proc. of 9th International Conference on Autonomous Infrastructure, Management and Security (AIMS15), 2015.

* Tomáš Čejka, Zdeněk Rosa and Hana Kubátová: [Stream-wise Detection of Surreptitious Traffic over DNS](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?reload=true&arnumber=7033254). In: Proc. of 19th IEEE International Workshop on Computer Aided Modeling and Design of Communication Links and Networks (CAMAD 2014). Athens, 2014.

* Václav Bartoš, Martin Žádník, Tomáš Čejka: [Nemea: Framework for stream-wise analysis of network traffic](http://www.cesnet.cz/wp-content/uploads/2014/02/trapnemea.pdf), CESNET technical report 6/2013.

