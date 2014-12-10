# Radial Images

Here I'd like to dive more technically into the helper images available and how
they work in the topology.

## Busyboxplus

I've created a suite of lightweight [busybox images][busyboxplus] specifically
for their role as Hub and Axle containers: a super small base image, and a git
and cURL flavor.

The base Busyboxplus base image, available via `docker pull
radial/busyboxplus:base` is Internet enabled and minimal in size (1.27mb). It is
the default image for Axle containers since they are completely inert and don't
need to do anything.  Internet access was free in these containers size-wise so
I figured I'd include it.

The git flavor is the default flavor for Hub containers. Weighing in at 12.86mb,
it has a minimal version of git installed for basic
extraction/merging/committing. It's not meant for interactive git usage on the
development side, but is more for cloning and merging configuration on the
deployment side. Git already depends on cURL so this image also comes with a
full cURL installation as well. You can access this image via `docker pull
radial/busyboxplus:git`.

The cURL flavor, which is slightly smaller (4.23mb) is exactly as it sounds: the
base image plus cURL compiled from scratch for busybox without git. It is
available via `docker pull radial/busyboxplus:curl`. It's not used in any of the
Axle, Hub, or Spoke containers by default but is included as an alternate to the
git flavor because of cURL's incredible utility in so many ways even without
git.

If you are familiar with buildroot, you can see how these images were compiled
[here](https://github.com/radial/core-busyboxplus)

[busyboxplus]: https://index.docker.io/u/radial/busyboxplus/

## Core Distro

The [core-distro images](https://index.docker.io/u/radial/distro/) are the fully
featured operating system for our actual applications. Because I'm most familiar
with Ubuntu, this is what I've chosen as the main operating system. It consists
of the official Docker Ubuntu image with a couple modifications:

* The `/etc/apt/sources.list` mirrors changed from "archive" to each of the
  Amazon EC2 data centers.
    * Downloads from the default "archive" location can be very slow at times
      and it's hard to guess the speed or distance from other institutions and
      mirrors. So using Amazon EC2 from the start is a pretty safe way to go.
* Each Dockerfile also sets the appropriate timezone data for each data center
  region selection as well as the 'en_US.UTF-8' locale and American English
  language for all images regardless of region.
    * The idea is to optimize the mirrors and timezones for all the regions, but
      unify them all by language. This is because you speak just one language to
      your computer, but you'll deploy your apps all over the world and will
      want the timezones to work properly and the mirrors to be fast.

Source Dockerfiles can be found [here](https://github.com/radial/core-distro)

## Axle-Base

The [Axle-Base image](https://index.docker.io/u/radial/axle-base/) is just a
wrapper for the base Busyboxplus image but renamed to keep the naming scheme
consistent. It's sole purpose is for housing shared volumes. There should never
be a need to enter or modify this container directly once run, so it pretty much
does nothing.

Dockerfile source available [here](https://github.com/radial/imagebase-axle)

## Hub-Base

The [hub-base image](https://index.docker.io/u/radial/hub-base/) uses the git
flavor of Busyboxplus as it's OS core. This is meant to allow it to combine both
a [skeleton configuration](https://github.com/radial/config-supervisor) of
[Supervisor](/radial/supervisor) as well as any additional configuration you
need for your application. Both these configurations can be acquired statically
and captured in Dockers version control by building images using this image as a
base, or dynamically at runtime inside an exposed volume.

The method for achieving this is by defining environment variables inside a
special "build-env" file that is sourced at build time statically, or by
defining these environment variables at run time dynamically. These variables
are git repositories and branches that contain the configuration in which to
clone and combine using git.

The hub-base image is also responsible for modifying the file ownership and
permissions of these configuration files one combined with git. This is also
done using environment variables.

Further details on how to use this image can be found in [it's
repository](https://github.com/radial/imagebase-hub).

## Spoke-Base

Finally, we have the [spoke-base image][spoke]. It is based on the [core-distro
images](https://github.com/radial/core-distro) and adds Supervisor to it for use
in your final Spoke container running your application. Much of it's automation
is abstracted away in the default Supervisor configuration and it the Spoke's
standard entrypoint script. This allows your Dockerfiles to remain very simple
when you use this image as a base. Check out [it's repository][spoke] for more
details.

[spoke]:https://github.com/radial/imagebase-spoke
