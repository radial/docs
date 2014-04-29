# The Axle-Hub-Spoke Topology

Combing through GitHub issues and the Docker-user forums yields many answers to
each one of the issues raised before, but very little in the way of implementing
them all together in a coherent way. It's easy to think of a server cluster of
Docker containers as a web of various connections and workers, but this actually
makes our job more difficult. Trying to remember which containers were
`--volumes-from` which other container and which containers have public ports
and which are merely exposed locally presents some interesting challenges for
scalability if one is trying to manage a cluster manually.

Another such issue was related to inconsistency in the Docker Index regarding
how each of the config/data/log logic was carried out in each of the containers.
The stated goal of the Docker Index is portability and sharing, and while the
sharing part happens automatically, the portability part was lacking to me.
There were very few images I could literally `docker pull` and feel confident
that it was setup the way I wanted and the image would behave in a predictable
manner. I found myself reconstructing the image almost every single time I
wanted to use someone else's image due to my specific configuration needs.
Unless individuals used strategies similar to those outlined here in this Radial
Topology, the entire "image sharing" part of the Docker index seemed unfulfilled
to me.

Thus, I've put together a strategy based on the imagery of a wheel. The main
parts of the wheel are the axle, the hub, and the spoke. Each of these
components make up different types of containers that serve specific purposes in
our topology. Typifying containers this way allows us to manage our
configuration files, our data/code, and our logs in an easy to remember way.
With this axle-hub-spoke strategy, we have a predictable method for the ways
that:

* code is retrieved and run while remaining portable
* configuration is loaded/stored, edited, and implemented
* everything isolated, tested, and run, and disposed
* volumes get created, shared and persisted across containers
* logs are stored and collected
* admin processes are carried out and how those changes are kept

Three container types make up the design, and each type has clearly stated
roles and respective Docker command options that govern each.

## The Axle Container

All wheels rotate on an axle, and if our cluster is made up of various "Wheels"
(application components, or stacks) each with their respective hubs and spokes,
then there is a fundamental starting point, the Axle container, that must be
created first to allow for some persistent containers and services.
Specifically, Axle containers:

* are used for persisting volumes that multiple Wheels will use for
  support or other persistent storage, but that have nothing to do with and one
  particular Wheel's configuration. One common example is log storage. Another
  is a data set that could be created by one container, collated by other, and
  displayed/hosted by a third on a website.
* can include bind-mount volumes unique to specific hosts. An example of
  this would be a music/video library that needs to be available normally to to
  some desktop workstation as well as served/worked managed through some process
  in the cluster.

## The Hub Container

Hub containers are a type of volume container with three
purposes:

* Managing, storing, and versioning configuration and static files
* Creating/storing/persisting all new volumes needed for it's respective Spoke
  container (application or running process) in a typical "volume container"
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
always assume that the configuration is stored under `/config`, the data
generated (or accessed) is in `/data` and the location of the logs will be in
`/log`. All they need to do is be run with `--volumes-from` their respective Hub
container with an appropriate `CMD` option (to specify the above three locations
via the command line) and they should be good to go. Their purpose is to:
    
* Manage, store, version, and run the app/code/process
* Be completely configuration agnostic (mandatory) and build-context agnostic
    (ideally). Spoke containers just need to know that the internet exists, and that it's
    respective Hub container exists for all it's needs.
* Of course, it will also know about any other containers that it needs to act
  upon if that is it's goal. But as far as the logistics of running our Spoke
  container, every Spoke container should blissfully assume the same locations
  for configuration, for data, and for logs.

## The Wheel

With the above responsibilities delegated amongst the three types of containers,
our application stack is now fully modularized. All-together, an instance of an
Axle, a Hub, and a Spoke container as they relate to a single application stack
can be referred to as a "Wheel".
