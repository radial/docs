# Putting it all together

## Axle-Base

The Axle-Base container is just a wrapper for a 2.5mb, full-chain, internet
enabled [busyboxplus
container](https://index.docker.io/u/radial/busyboxplus/) but renamed to
keep the naming scheme consistent. It's sole purpose is for housing shared
volumes. There should never be a need to enter or modify this container directly
once run. 

As mentioned before, Axle containers are for hosting regular or
bind-mounted volumes whose data need to be acted upon by multiple containers.

One example given before was for log files, another could be music or video
libraries. In the case for logs, we need to be free to spin up apps and assure
that the logs have a persistent and predictable place to go. Having a service
like [Logstash](http://logstash.net/) running in your cluster is made easier by
having all your logs in one place; a single "logs" Axle container.

The second example was for bind-mounting files stored locally on the host. I
have a very large audio/video library that I manage, modify, and use
directly (nfs mounts on my workstations) and indirectly through applications in
containers (web interfaces to access/stream the library). For fully public
websites, the Docker best-practice is to have nothing bind-mounted so as to
remain completely portable and machine-independent. This option is still
available of course using the same strategy, but just for the sake of
completeness, we have a place in our topology for bind-mounts to happen.

Let's create our above example and make a "logs" Axle container. If we choose to
make a permanent image that we can reuse, the Dockerfile would possibly contain:

```
FROM    radial/axle-base
VOLUME  ["/log"]
CMD     ["/bin/true"]
```
It could be build via:

`docker build -t logs .`

and run via:

`docker run --name logs logs`

Or we can do it all in our run step with:

`docker run --name logs --volume /log radial/axle-base /bin/true`

Both methods are identical because the only data being written is to an exposed
volume at `/log`, so docker doesn't keep any data in it's version control in
either case. Use whichever fits your workflow.

So why keep logs in the root path of `\log` instead of something that we're used
to such as `\var\log` anyway? The reason is that it will make our lives easier
in the long run if we free ourselves from the [Filesystem Hierarchy
Standard](http://www.pathname.com/fhs/) when it comes to our containers and just
stick everything in the root path. We'll see more examples to prove this point
later but the core of the argument is that between different programs, operating
systems, and different methods of inserting code into our containers via `ADD`
or using `VOLUME`, we introduce a lot of uncertainty with what directories we
are overwriting and where certain files can be found. So it's best to just use
our own paths and filesystem logic from the start and be consistent with it
throughout our cluster.

## Hub-Base

While the Axle containers are more fundamental in nature, as their name
suggests, the Hub containers are where everything comes together. Each of our
Wheels will have their own Hub container for housing and managing configuration
files for the application and/or services being run in our Spoke container. They
also house any other specialty volumes as well such as our Axle
container named "logs" which is built and ready to go. So we use it with
`--volumes-from logs` to include it in our Hub container.

### Dynamic Configuration

What does the [Hub-Base](https://github.com/radial/imagebase-hub/blob/master/Dockerfile) Dockerfile look like? To start:

```
FROM            radial/busyboxplus-curl
MAINTAINER      Brian Clements <brian@brianclements.net>
```

I've created a suite of lightweight busybox images specifically for their role
as Hub containers. The base
[busyboxplus][busyboxplus] image is
Internet enabled and minimal in size (~2.5mb).

[busyboxplus]: https://index.docker.io/u/radial/busyboxplus/

Here we have a placeholder location for our repository of "none" that we can
alter at run time.

`ENV CONFIG_REPO none`

The next bit shows our filesystem logic in it's full glory.

```
VOLUME          ["/config", "/data", "/log"]
```

We don't use any standard or default paths for anything! There are a couple
similar reasons for this, let's go through them.

* Why `/config`?
    * Letting our app install a default configuration and then needing to either `ssh`
      or `docker run -it myapp-hub /bin/sh` to edit it is a nightmare to
      standardize; avoid it like the plague. We want to `ADD` an already primed
      configuration file or retrieve it from some other server or repository.

    * Using `ADD` however overwrites files, which could work in the case of
      using a default location of `/etc/myservice/config.conf`... or is it
      `/etc/myservice.d/partone.conf`, or is it `/etc/default/config.conf`... or
      is it... well, you get the idea.  `/etc` is pretty standard, but not
      standard enough to be able to 100% of the time pull an image from the
      Docker Index and plop a configuration file in and know that your app or
      service will know where to look. This is why we don't rely on default
      locations for configuration files but explicitly state our configuration
      path when we run our app. It's pretty standard for programs support a
      custom configuration path at runtime so we will include this as one of our
      topology rules for running apps in Spoke containers.

    * Another bad idea is to try and mount volumes to system paths such as
      `/etc` or something similar and have other containers "insert" the config.
      We might unknowingly break all kinds of other services by deleting other
      configuration files with our volume mount.

    * If you're thinking why not just go interactive with `docker -it` and then
      disconect or something similar? Well you could but it's not exactly
      something you can standardize. Also, remember that it's tricky to setup a
      container and "re-attatch" to it and have a shell running there waiting
      for you if you need to use the `CMD` line to actually do things other then
      just run a shell. But more importantly, do you really want to modify
      configuration files inside a container using whatever stock editor is
      supplied by the operating system? Probably not.

* Why `/data`?
    * Similar to point 2 above; with many different services and programs using
      different default paths for their data directories, and many of those same
      services using different locations even between operating system
      installations, we want to know that there is a 100% guaranteed match and
      consistency between our persistent volume storage Hub container and
      our application Spoke container. We want piece of mind that as long as we
      specify either at run time or in our configuration that the data directory
      resides at `/data` we are good to go with no questions asked.

* Why `/log`?
    * We've already created an Axle container for our logs at `/log`. So we've
      used `--volumes-from` our already-running "logs" container to make it
      available now to this Hub container and by association to our application
      Spoke container (by later running the Spoke container with
      `--volumes-from` this Hub container). Keeping with the now established
      pattern, we can also specify the path (and non-conflicting name) for our
      log file at run time for our app.

So here is the key topology principle:

**Never rely on default locations for
our configuration files, our data directories, and our logs. Declare custom paths
at runtime and use the same paths for everything; ideally `/config`, `/data`,
and `/log`.**

```
ENV     CONFIG_REPO     none
CMD     wget --no-check-certificate -P /config/ $CONFIG_REPO >/dev/null 2>&1 &&\
        chmod 644 -R /config /data /log >/dev/null 2>&1
```

So why is this busybox image internet enabled? Well busybox comes with a simple
version of `wget` that we can use to retrieve configuration files. The example
below and in the actual image is for proof of concept, but shows how one can, at
runtime, use the `CONFIG_REPO` variable to grab a configuration file.  With the
[cURL flavor](https://index.docker.io/u/radial/busyboxplus-curl/) of the
[busyboxplus][busyboxplus] image and some other additional Dockerfile logic, you
could potentially grab your configuration from an
[etcd](https://github.com/coreos/etcd) service in your cluster for example.

The last half of the `CMD` step is to unify our file permissions so that they
can be accessed by other system users (deamon, nobody, www, etc.) specific to
the application in our Spoke container, but only editable by root. This must be
done in the `CMD` step because we've already exposed the `/config` directory as
a volume, so it is no longer editable by `RUN` steps in our Dockerfile.

This is the first half of the Hub-Base Dockerfile. And you might have noticed
that there is no `ADD` yet. This is because this part of the Dockerfile is for
use "on the go." Having a template image sitting on your host via `docker pull
radial/hub-base` allows us to not even need to bother with a Dockerfile. If we
are retrieving our configuration from some external source with wget or cURL,
and not adding a local file in a Dockerfile, we can just launch the Hub
container in one go:

```
docker run --name myapp-hub --volumes-from logs \
--env CONFIG_REPO="https://github.com/radial/docs/raw/master/README.md" \
radial/hub-base
```

Which should launch the container, retrieve our configuration, and place it in
our `/config` folder. If we ever update our configuration, it's almost trivial
to spin up new Hub containers with different configuration and to destroy old
ones.

Keep in mind however that the configuration using this method is not kept in
Dockers version control. You cannot "commit" it to an image because the
configuration file is loaded into a shared volume and that data can't be
committed to image. This is almost a non-issue however because in order for it
to work in the first place, our configuration is already stored in some other
location, which ideally is already version controlled. 

### Static Configuration

So what if we did want to incorporate our configuration into Docker's version
control mechanism? Luckily this is an option for us as well using the very same
Dockerfile. The second half of the Hub-Base Dockerfile utilizes a relatively new
function called `ONBUILD` which gives us the ability to give our Dockerfile two
lives. `ONBUILD` is used when you declare `FROM radial/hub-base` in a new, empty
Dockerfile. When this one-lined new Dockerfile is built, it will execute the
`ONBUILD` statements defined in the radial/hub-base image, which are the `ADD`,
`VOLUME`, and `CMD` steps needed to persist our configuration. To clarify, the
steps below are in the original Hub-Base Dockerfile, but don't get used until
you create your own, empty, Dockerfile with the single declaration of `FROM
radial/hub-base`.

Since we've already established our own filesystem, and it's quite simple to
remember, all we need to do is create the folders `/config`, `/data` (if for
some reason you need to), and `/log` (again, you won't typically need
to) in the same path as your new Dockerfile and the following step:

`ONBUILD ADD     . /`

will automatically populate your container with the contents of those folders
when you `docker build -t myapp-hub:v1.0 .`. It's important to note that you
will pretty much only need to make and use the `/config` folder as it will be
the only folder we actually add files to. The other two are usually taken care
of by running the Hub container itself and/or our already existing `logs` Axle
container.

**Note:** As of Docker 0.10, you can now use the same name for the container as
you used for your image. If you have an older version of docker, you need to
prefix the images you build with something to prevent name collisions with the
containers you run so that you can use the most logical name for the container
and not the image when you run the container itself.

What if our configuration is accessible via some other server or service as
before? Well since `docker build` doesn't let us set environment variables like
`docker run` does, we need to put these environment variables into a file called
`build-env` and source it. This line achieves that.

`ONBUILD RUN     test -f /build-env && source /build-env || true`

This file is located in the same path as your new, one line Dockerfile. 
`docker build` fails if any of the individual steps fail, so that's why we need
the testing logic in there to make sure we are "true" whether the file exists or
it doesn't.

The contents of our `/build-env` file could be:

`CONFIG_REPO="https://github.com/radial/docs/raw/master/README.md"`

Makes sense. Now that our `CONFIG_REPO` is set we grab the configuration at that
location:

`ONBUILD RUN     wget --no-check-certificate -P /config/ $CONFIG_REPO >/dev/null 2>&1 || true`

Everything is in order now. Any manually added configuration is in our new
container and/or we've grabbed our remote configuration from our version
control. Since these files are already added via `RUN` and `ADD` processes,
declaring `VOLUME` AFTER we've obtained them makes them persist in Docker's
version control PLUS also shares the volumes themselves with other containers.

```
ONBUILD VOLUME  ["/config", "/data", "/log"]
```

Again, we unify our file permissions as our last step.

`ONBUILD CMD     chmod 644 -R /config /data /log >/dev/null 2>&1`

To recap, what actions do you actually do? If I wanted to pull my configuration from an
external source and build my Hub container, I would only need to create
a Dockerfile containing:

`FROM radial/hub-base`

and a build-env file containing:

`CONFIG_REPO="https://github.com/radial/docs/raw/master/README.md"`

Then `docker build -t myapp-hub:v1.0 .` puts it all together. We have an
persistent image now, with our configuration in `/config`, with folders `/config`,
`/data`, and `/log` shared and ready for use by our application Spoke container,
AND a tag with this version of our configuration, "v1.0".

We've succeeded at persisting our configuration in a permanent image PLUS
sharing the volumes without having to enter the container at all. Nice! Run it
with `docker run --name myapp-hub myapp-hub:v1.0`.
