
Docker containers provide a consistent, compact and flexible means of
packaging application builds. Delivering applications with Docker on Mesos
promises a truly elastic, efficient and consistent platform for delivering a
range of applications on premises or in the cloud.

## Service Launch and Discovery

Let's launch a service in a Docker container and use Marathon's discovery
features to find and connect to it from another Docker container. In this
example, we'll launch Redis within a Docker container and then connect to it
using the Redis CLI.

Launching Docker containers with Marathon is a matter of assigning the
"executor" parameter to point to a Docker executor and making the container
name the first word of the command.

    :;  http POST http://localhost:8080/v1/apps/start \
                  id=multidis instances=2 mem=512 cpus=1 \
                  executor=/var/lib/mesos/executors/docker \
                  cmd='johncosta/redis'

    >>

    HTTP/1.1 204 No Content
    Content-Type: application/json
    Server: Jetty(8.y.z-SNAPSHOT)

Once we have two Redis instances running, we can query Marathon to find them
and connect to them.

    :;  http GET http://localhost:8080/v1/endpoints

    >>

    HTTP/1.1 200 OK
    Content-Type: text/plain
    Server: Jetty(8.y.z-SNAPSHOT)
    Transfer-Encoding: chunked

    multidis 10445 ip-10-184-7-73.ec2.internal:31001 ip-10-184-7-73.ec2.internal:31000

Using one of the `ip:port` pairs above, we can connect to one of our Redis
instances, using the same Docker image, to get access to a working Redis CLI:

    :;  sudo docker run -i -t johncosta/redis \
             redis-cli -h ip-10-184-7-73.ec2.internal -p 31001

    redis ip-10-184-7-73.ec2.internal:31001> SET x 7
    OK
    redis ip-10-184-7-73.ec2.internal:31001> GET x
    "7"

There's no need to install Redis or its supporting libraries on your Mesos
hosts.

## Reproducing This Example

Installation of the many components used here is straightforward. Mesos and
the Python bindings for Mesos are available as packages:

* [Ubuntu 13.04 Package](http://downloads.mesosphere.io/master/ubuntu/13.04/mesos_0.14.0_amd64.deb)
* [Python Mesos Bindings](http://downloads.mesosphere.io/master/ubuntu/13.04/mesos-0.14.0-py2.7-linux-x86_64.egg)

Marathon, a Scala program, can be run from a standalone Jar. We've made an
Upstart script for it, too.

* [Marathon Executable Jar](http://downloads.mesosphere.io/marathon/marathon-0.0.6-SNAPSHOT-jar-with-dependencies.jar)
* [Marathon Upstart Script](http://downloads.mesosphere.io/marathon/marathon.conf)

With Mesos and Marathon, you have everything you need to start running
reliable services across your cluster.

The Docker project provides [Ubuntu 13.04 installation instructions](http://docs.docker.io/en/latest/installation/ubuntulinux/#ubuntu-raring).

To follow along with the examples above, install the Docker executor as
`/var/lib/mesos/executors/docker`.

  [Docker Executor for Mesos](https://github.com/mesosphere/mesos-docker/blob/master/bin/mesos-docker)

For a standalone installation, we've put together a script that installs all
the needed components and configures them on Ubuntu 13.04. Just pipe it to a
shell running under `sudo` and you're all set:

    curl -fL https://raw.github.com/mesosphere/mesos-docker/master/bin/mesos-docker-setup | sudo bash

