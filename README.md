# UNFI POC - Commands to execute

git clone https://github.com/hari-mutyala/HK-JmeterDocker.git
cd HK-JmeterDocker/
docker build -t <<reponame>>/hk-jmeter:v1 .
docker push <<reponame>>/hk-jmeter:v1
docker pull <<reponame>>/hk-jmeter:v1
docker run hmutyala/hk-jmeter:v1 -n -t bin/examples/API_PERF_library-management.xyz.jmx -l bin/reports/report2.log -e -o bin/reports
docker container cp dacc3306aaac:/opt/apache-jmeter-5.5/bin/reports .
cd reports/
zip -r API_PERF_library-management-results.zip .



# docker-jmeter
## Image on Docker Hub

Docker image for [Apache JMeter](http://jmeter.apache.org).
This Docker image can be run as the ``jmeter`` command.
Find Images of this repo on [Docker Hub](https://hub.docker.com/r/hmutyala/hk-jmeter).
Starting version 5.4 Docker builds/pushes
are [executed via GitHub Workflows](.github/workflows/docker.yml).

## Security Patches
As you may have seen in the news, a new zero-day exploit has been reported against the
popular Log4J2 library which can allow an attacker to remotely execute code.
The vulnerability has been reported with [CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228)
against the log4j-core jar and has been fixed in Log4J v2.16.0.

JMeter, at least in versions 5 and later uses the vulnerable Log4J versions.
The good news though is that the vulnerability applies only to remotely accessible Java web-services.
JMeter is a commandline/GUI tool one runs internally. Still it is good practice to
patch this problem.

**JMeter has been updated to 5.4.2 for security CVE-2021-45046 & CVE-2021-45046**.

https://jmeter.apache.org/changes.html#Non-functional%20changes

The update to 5.4.2 includes the updated Apache log4j2 to 2.16.0 (from 2.13.3), thanks for PR #51!

## Building

With the script [build.sh](build.sh) the Docker image can be build
from the [Dockerfile](Dockerfile) but this is not really necessary as
you may use your own ``docker build`` commandline. Or better: use one
of the pre-built Images from [Docker Hub](https://hub.docker.com/r/hmutyala/hk-jmeter).

### Build Options

Build arguments (see [build.sh](build.sh)) with default values if not passed to build:

- **JMETER_VERSION** - JMeter version, default ``5.4``. Use as env variable to build with another version: `export JMETER_VERSION=5.4`
- **IMAGE_TIMEZONE** - timezone of Docker image, default ``"Europe/Amsterdam"``. Use as env variable to build with another timezone: `export IMAGE_TIMEZONE="Europe/Berlin"`

## Running

The Docker image will accept the same parameters as ``jmeter`` itself, assuming
you run JMeter non-GUI with ``-n``.

There is a shorthand [run.sh](run.sh) command.
See [test.sh](test.sh) for an example of how to call [run.sh](run.sh).

## User Defined Variables

This is a standard facility of JMeter: settings in a JMX test script
may be defined symbolically and substituted at runtime via the commandline.
These are called JMeter User Defined Variables or UDVs.

See [test.sh](test.sh) and the [trivial test plan](tests/trivial/test-plan.jmx) for an example of UDVs passed to the Docker
image via [run.sh](run.sh).

See also: https://www.novatec-gmbh.de/en/blog/how-to-pass-command-line-properties-to-a-jmeter-testplan/

## Adjust Java Memory Options

By default, JMeter reads out the available memory from the host machine and uses a fixed value of 80% of it as a maximum. If this causes Issues, there is the option to use environment variables to adjust the JVM memory Parameters:

```JVM_XMN``` to adjust maximum nursery size

```JVM_XMS``` to adjust initial heap size

```JVM_XMX``` to adjust maximum heap size

All three use values in Megabyte range.

## Installing JMeter plugins

To run the container with custom JMeter plugins installed you need to mount a volume /plugins with the .jar files. For example:
```sh
sudo docker run --name ${NAME} -i -v ${LOCAL_PLUGINS_FOLDER}:/plugins -v ${LOCAL_JMX_WORK_DIR}:${CONTAINER_JMX_WORK_DIR} -w ${PWD} ${IMAGE} $@
```

The ${LOCAL_PLUGINS_FOLDER} must have only .jar files. Folders and another file extensions will not be considered.

### Configuring the custom JMeter plugins folder location

It is also possible to define an alternate location to the custom JMeter plugins folder. Simply define a environment variable called `JMETER_CUSTOM_PLUGINS_FOLDER` with the desired folder path like in the example bellow:

```sh
sudo docker run --name ${NAME} -i -e JMETER_CUSTOM_PLUGINS_FOLDER=/jmeter/plugins -v ${LOCAL_PLUGINS_FOLDER}:/jmeter/plugins -v ${LOCAL_JMX_WORK_DIR}:${CONTAINER_JMX_WORK_DIR} -w ${PWD} ${IMAGE} $@
```



## Specifics

The Docker image built from the
[Dockerfile](Dockerfile) inherits from the [Alpine Linux](https://www.alpinelinux.org) distribution:

> "Alpine Linux is built around musl libc and busybox. This makes it smaller
> and more resource efficient than traditional GNU/Linux distributions.
> A container requires no more than 8 MB and a minimal installation to disk
> requires around 130 MB of storage.
> Not only do you get a fully-fledged Linux environment but a large selection of packages from the repository."

See https://hub.docker.com/_/alpine/ for Alpine Docker images.

The Docker image will install (via Alpine ``apk``) several required packages most specificly
the ``OpenJDK Java JRE``.  JMeter is installed by simply downloading/unpacking a ``.tgz`` archive
from http://mirror.serversupportforum.de/apache/jmeter/binaries within the Docker image.

A generic [entrypoint.sh](entrypoint.sh) is copied into the Docker image and
will be the script that is run when the Docker container is run. The
[entrypoint.sh](entrypoint.sh) simply calls ``jmeter`` passing all argumets provided
to the Docker container, see [run.sh](run.sh) script:

```
sudo docker run --name ${NAME} -i -v ${WORK_DIR}:${WORK_DIR} -w ${WORK_DIR} ${IMAGE} $@
```

## Credits

Thanks to 
https://github.com/justb4/docker-jmeter/