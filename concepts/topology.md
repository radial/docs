# The Axle-Hub-Spoke Topology

Before many of the Docker container orchestration services became popular and
mature, manual lifecycle tasks of a cluster of containers was a bit of a mess. I
wanted to streamline my understanding of application stack components and fit
them to Docker features intuitively and systematically. Even now with tools like
[Fig][fig], [Kubernetes][kube], and others, I'm finding these topology
principals very easy to transfer no matter the orchestration tool.

One challenge I frequently came across early on was related to inconsistency in
the Docker Index regarding how each repository's config/data/log logic was
carried out in each of the containers. The stated goal of the Docker Index is
portability and sharing, and while the sharing part is built-in, the portability
part was lacking to me. There were very few images I could literally `docker
pull` and feel confident that it was setup the way I wanted and the image would
behave in a predictable manner. I found myself reconstructing the image almost
every single time I wanted to use someone else's image due to my specific
configuration needs. Unless individuals used strategies similar to those used in
the Radial Topology, the entire "image sharing" part of the Docker index seemed
a bit unfulfilled to me.

Thus, for Radial, I've used the imagery of a wheel to help conceptually keep
things simple. Just like a real wheel, the main parts of a Radial "wheel" are
specialized containers each labeled an "axle", a "hub", and a "spoke" container
that each serves a specific purposes in our topology. Typifying containers this
way allows us to manage our configuration files, our data/code, and our logs in
an easy to remember way. With this axle-hub-spoke strategy, we have a
predictable method for the ways that:

* application code is retrieved and/or run while remaining portable
* configuration is loaded/stored, edited, and implemented
* everything is isolated, tested, run, and disposed of
* volumes get created, shared and persisted across containers
* logs are stored and collected
* admin processes are carried out and how those changes are kept

[kube]:http://googlecloudplatform.github.io/kubernetes
[fig]:http://www.fig.sh


## The Axle Container

All wheels rotate on an axle, and if our cluster is made up of various "wheels"
(application components, stacks, or what
[Kubernetes][kube] would call a
"pod"), each with their respective hubs and spokes, then there is a fundamental
starting point, the axle container, that must be created first to allow for some
persistent containers and services.  Specifically, axle containers:

* are used for persisting volumes that one or multiple wheels will use for
  support or other persistent storage, but that have nothing to do with any one
  particular wheel's configuration, or application binary. One common example is
  log storage. Another is a data set that could be created by one container,
  collated by other, and displayed/hosted by a third on a website.
* can include bind-mount volumes unique to specific hosts. An example of this
  would be a music/video library that needs to be available normally to to some
  desktop workstation as well as served/worked managed through some process in
  the cluster.

## The Hub Container

Hub containers are a type of volume container with three purposes:

* Managing, storing, and versioning configuration and static files
* Creating/storing/persisting all new volumes needed for it's respective spoke
  container (application or running process) in a typical 
  [volume container](http://crosbymichael.com/advanced-docker-volumes.html)
  fashion.
* Combining exposed volumes from all other relevant axle containers that might
  be needed for this particular wheel. Two examples are a log container or a
  specific media library.
* Be a standardized location for unix sockets and other inter-spoke
  communication.

The idea is that the hub is the gathering point for all the complexity of your
wheel as it relates to your stack: the volumes, the local bind mounts, the
configuration etc., so that your spoke containers are free just to worry about
the code and the running process. This completely separates the configuration
from the application itself allowing further modularity with managing
everything.

## The Spoke Container

Spoke containers are your applications and workers. They should, by design,
always assume that:

* the configuration normally stored in `/etc` is stored under `/config`
* that the data and other "stuff" that applications tend to generate or need to
  access (cache, helper folders, things normally found in a users home
  directory) is in `/data`
* and that the location of the logs will be in `/log`

All a spoke container needs to do is be run with `--volumes-from` it's
respective hub container and all of the above will be accessible to it. A spoke
containers purpose is to manage, store, version, and run the app/code/process.
It should:

* Be completely configuration agnostic. What this means is that as a rule,
  configuration cannot be bundled with the spoke container. It must be managed
  separately and stored in `/config` as mentioned previously in the hub
  container for this Wheel.
* Be modular. The spoke container image for a singular database service running
  3 containers (axle for database storage, a hub for configuration, and a spoke
  for the database binary) can be the same spoke container used as part of a 6
  container LAMP stack (axle for database storage, axle for log storage, hub for
  configuration and hosting sockets between the web server, database, and web
  app spoke containers)
* Should blissfully assume the standardized locations for configuration, for
  data, and for logs via it's use of `--volumes-from` the hub container.

Designing spoke containers this way encourages safer application containers
because it doesn't attempt to make any user-friendly/configuration/security
trade-offs.

## Wheels

With the above responsibilities delegated amongst the three types of containers,
our application stack can be fully modularized. All-together, an instance of any
combination of axles, a single hub, and any number of spoke containers can be
referred to as a "wheel" and [packaged all-together](/design/wheels.md) as such. At
the moment, wheels designed to be an atomic unit that is located on a single
host.
