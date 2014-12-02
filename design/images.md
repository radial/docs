##Radial Images

Here I'd like to dive more technically into the images available and how they
work in the topology.

###Busyboxplus

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

The cURL flavor, which is slightly smaller (4.23mb) is exactly as it sounds:
the base image plus cURL compiled from scratch for busybox without git. It is
available via `docker pull radial/busyboxplus:curl`. It's not used in any of the
Axle, Hub, or Spoke containers by default but is included as an alternate to the
git flavor because of cURL's incredible utility in so many ways even without
git.

If you are familiar with buildroot, you can see how these images were compiled
[here](https://github.com/radial/core-busyboxplus)

[busyboxplus]: https://index.docker.io/u/radial/busyboxplus/

###Core Distro

The [core-distro images](https://index.docker.io/u/radial/distro/) are the fully
featured operating system for our actual applications. Because I'm most familiar
with Ubuntu, this is what I've chosen as the main operating system. It consists
of the official Docker Ubuntu image with a couple modifications:

* The `/etc/apt/sources.list` mirrors changed from "archive" to each of the Amazon
EC2 data centers.
    * Downloads from the default "archive" location can be very slow at times and it's
    hard to guess the speed or distance from other institutions and mirrors. So
    using Amazon EC2 from the start is a pretty safe way to go.
* Each Dockerfile also sets the appropriate timezone data for each data center
region selection as well as the 'en_US.UTF-8' locale and American English
language for all images regardless of region.
    * The idea is to optimize the mirrors and timezones for all the regions, but unify
    them all by language. This is because you speak just one language to your
    computer, but you'll deploy your apps all over the world and will want the
    timezones to work properly and the mirrors to be fast.

Source Dockerfiles can be found [here](https://github.com/radial/core-distro)

###Axle-Base

The [Axle-Base image](https://index.docker.io/u/radial/axle-base/) is just a
wrapper for the base Busyboxplus image but renamed to keep the naming scheme
consistent. It's sole purpose is for housing shared volumes. There should never
be a need to enter or modify this container directly once run, so it pretty much
does nothing.

Dockerfile source available [here](https://github.com/radial/imagebase-axle)

###Hub-Base

The [hub-base image](https://index.docker.io/u/radial/axle-base/) uses the git
flavor of Busyboxplus as it's OS core. This is meant to allow it to combine both
a [skeleton configuration](https://github.com/radial/config-supervisor) of
[Supervisor](/radial/supervisor) as well as any additional configuration you
need for your application. Both these configurations can be acquired statically
and captured in Dockers version control or dynamically inside an exposed volume.

The method for achieving this is by defining environment variables inside a file
that is sourced at build time (static), or by defining these environment
variables at run time (dynamic) which will clone them all the same using git,
but by this point their destination folders will already be shared volumes,
which persist data only as long as the container exists. This allows us both a
durable version-controlled and a quick command-line only method for extracting
our configuration and getting a Hub container up and running.


###Spoke-Base

Finally, we have the [spoke-base
image](https://github.com/radial/imagebase-spoke). It is based on the
[core-distro images](https://github.com/radial/core-distro) and adds SSH and
Supervisor to it for use in your final Spoke container running your application.
Specifically, it will:

* Install SSH server and supervisor from Ubuntu repository
* Generate a fresh set of host-keys every time you build using `FROM` this image
* If you have a GitHub account, you can set '$GH_USER' with your username
  somewhere in your Spoke Dockerfile and the spoke-base image will automatically
  append them to your container's `/root/.ssh/authorized-keys` file for
  ready-to-access SSH login. Not setting the variable results in nothing being
  downloaded. \*
* `chown` all configuration files to whatever you set '$CHOWN_USER' and
  '$CHOWN_GROUP' to be in your Spoke Dockerfile
* `chown root:root` your supervisor configuration for security
* Automatically create a folder in `/log` (which is typically an Axle container
  shared by all Wheels running on the host) based on your container ID where
  supervisor will automatically dump all of your containers logs into
* Automatically load whatever programs are set in `/config/supervisor/conf.d`,
  which includes the SSH server and your app.

\* NOTE: Acquiring public keys in this way is 'secure' in that use of public keys
are always secure, but it is not the wisest strategy to use the same key-pair
for multiple venues (server cluster, public website, etc.). More robust key
management is a feature for a future date. For now, this suffices for prototyping
and development on small clusters.
