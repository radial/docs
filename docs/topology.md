# The Axle-Hub-Spoke Topology

It's easy to think of a server cluster of Docker containers as a web of various
connections and workers, but this actually makes our job more difficult. Trying
to remember which containers were `--volumes-from` which other container and
which containers have public ports and which are merely exposed locally presents
some interesting challenges for scalability if one is trying to manage a cluster
manually.

Another such challenge I frequently came across was related to inconsistency in
the Docker Index regarding how each repository's config/data/log logic was
carried out in each of the containers. The stated goal of the Docker Index is
portability and sharing, and while the sharing part is built-in, the portability
part was lacking to me. There were very few images I could literally `docker
pull` and feel confident that it was setup the way I wanted and the image would
behave in a predictable manner. I found myself reconstructing the image almost
every single time I wanted to use someone else's image due to my specific
configuration needs. Unless individuals used strategies similar to those used
in the Radial Topology, the entire "image sharing" part of the Docker index
seemed unfulfilled to me.

Thus, for Radial, I've used the imagery of a wheel to help conceptually keep
things simple. Just like a real wheel, the main parts of a Radial "Wheel" are
specialized containers each labeled an Axle, a Hub, and a Spoke container that
each serves a specific purposes in our topology. Typifying containers this way
allows us to manage our configuration files, our data/code, and our logs in an
easy to remember way. With this axle-hub-spoke strategy, we have a predictable
method for the ways that:

* application code is retrieved and run while remaining portable
* configuration is loaded/stored, edited, and implemented
* everything is isolated, tested, run, and disposed of
* volumes get created, shared and persisted across containers
* logs are stored and collected
* admin processes are carried out and how those changes are kept

## The Axle Container

All wheels rotate on an axle, and if our cluster is made up of various "Wheels"
(application components, or stacks) each with their respective hubs and spokes,
then there is a fundamental starting point, the Axle container, that must be
created first to allow for some persistent containers and services.
Specifically, Axle containers:

* are used for persisting volumes that multiple Wheels will use for support or
  other persistent storage, but that have nothing to do with any one particular
  Wheel's configuration. One common example is log storage. Another is a data
  set that could be created by one container, collated by other, and
  displayed/hosted by a third on a website.
* can include bind-mount volumes unique to specific hosts. An example of this
  would be a music/video library that needs to be available normally to to some
  desktop workstation as well as served/worked managed through some process in
  the cluster.

## The Hub Container

Hub containers are a type of volume container with three purposes:

* Managing, storing, and versioning configuration and static files
* Creating/storing/persisting all new volumes needed for it's respective Spoke
  container (application or running process) in a typical 
  [volume container](http://crosbymichael.com/advanced-docker-volumes.html)
  fashion.
* Combining exposed volumes from all other relevant Axle containers that might
  be needed for this particular Wheel. Two examples are a log container or a
  specific media library.

The idea is that the Hub is the gathering point for all the complexity of your
app as it relates to your cluster: the volumes, the local bind mounts, the
configuration etc., so that your Spoke Container is free just to worry about the
code and the running process. This completely separates the configuration from
the application itself allowing further modularity with managing everything.

## The Spoke Container

Spoke containers are your applications and workers. They should, by design,
always assume that:

* the configuration normally stored in `/etc` is stored under `/config`
* that the data and other "stuff" that applications tend to generate or need to
  access (cache, helper folders, things normally found in a users home
  directory) is in `/data`
* and that the location of the logs will be in `/log`

All a spoke container needs to do is be run with `--volumes-from` it's
respective Hub container and all of the above will be accessible to it. A Spoke
containers purpose is to:

* Manage, store, version, and run the app/code/process
* Be completely configuration agnostic (mandatory)
    * what this means is that as a rule, configuration cannot be bundled with
      the Spoke container. It must be managed separately and stored in `/config`
      as mentioned previously in the Hub container for this Wheel.
* Be build-context agnostic (ideally).
    * Being build-context agnostic is simply not having to rely on Docker's
      `ADD` mechanism in the Dockerfile during the build process. The most
      portable strategy is one where all source code and configuration are
      stored in some type of version-controled repository that is accessible to
      the Spoke container from any build location. Spoke containers ideally
      should just need to know that the internet and it's respective Hub
      container exists for all it's needs. However, in small or DIY server
      clusters this is sometimes a feature that comes later, so the use of `ADD`
      is still possible in the overall strategy.
* Also know about any other containers that it needs to act upon if that is it's
  goal (ie. knowing about a load balancer or database if this is a web server).
  But as far as the logistics of running our Spoke container, every Spoke
  container should blissfully assume the same locations for configuration, for
  data, and for logs because of it's `--volumes-from` the Hub container.

## The Wheel

With the above responsibilities delegated amongst the three types of containers,
our application stack is now fully modularized. All-together, an instance of an
Axle, a Hub, and a Spoke container as they relate to a single application stack
can be referred to as a "Wheel" and [packaged all-together](/docs/wheels) as
such.
