##What is Radial?

Radial is a [Docker](http://docker.io) container topology strategy for managing
app stacks on a per-app basis. It was created to help me understand, record, and
put into practice the intersection between [Twelve-Factor
App](http://12factor.net) best-practices and Docker features. The results are my
own topology strategies for manually implementing Docker containers in a
predictable, intuitive, and flexible way. I am documenting it and organizing it
here for all to critique, use, and make suggestions to. It should be a good
primer for enthusiasts like myself and professionals alike to start to
understand how the different features of Docker meet the needs of hosting
processes in a server cluster.

##What Can I Do With It?

Radial is a strategy guide that is is outlined in this documentation and is
implemented in the suite of accompanying Docker
[images](https://index.docker.io/u/radial/) and Dockerfiles that this project
supplies. Since Radial deals entirely with how you spread your application
across containers, it is intended to be compatible with all the wonderful
Docker-based PaaS tools that are popping up all over, but should not require any
of them or (hopefully) conflict with their use so that one can still manually
implement these application stacks manually if need be.

##What's The Strategy?

In short, I use the analogy of an "Axle", a "Hub", and a "Spoke" to help
conceptually link together Docker containers into different classes based on
what we need the entire stack (the "Wheel") to be doing. I've created specialty
Docker images streamlined for each of these classes that are available on the
[Docker Index](https://index.docker.io/u/radial). 

These images use tools such as busybox to keep container size down, git to clone
and combine configuration, and Supervisor to manage our all our processes and
glue it all together.

##The Documentation

###Concepts
* [The Factors](/docs/factors)
* [The Topology](/docs/topology)
* [Wheels](/docs/wheels)
* [Radial Images](/docs/images)
* [Supervisor](/docs/supervisor)

###Instructions

TODO

* Axle Containers
* Hub Containers
* Spoke Containers

###Examples

TODO

###Next

These documents are works-in-progress as I put their suggestions into practice.
Updates are to come as suggestions and testing come in. Feel free to
pull-request or head over to the
[forum](https://groups.google.com/forum/?pli=1#!forum/radial-docker-topology)
to discuss your ideas. Do you use similar strategies for your container
topology?

Some implications and future goals for the Radial Topology:

* The ability to create and `docker push` images in twos; one for the
  application itself (the Spoke container) and one for the configuration (the
  Hub container). As long as one person creates their individual Hub container
  for a given app using the Radial guidelines, one can feel confident that their
  Spoke container will "just work". Also, while one can publish their respective
  Spoke containers on the Index without any concern of sharing sensitive data,
  one can publish their Hub containers to a private index for versioning (or
  just build them on the fly of course like these tutorials suggest). This can
  really speed up the testing, sharing, reproduction, and updating of
  application stacks.
* Create official images of common apps using the Radial topology and publish in
  the docker index accordingly: radial/mysql, radial/sshd etc.
* Create suite of worker/admin/helper/debug containers for one-off admin processes.
  This can include log viewing/management, git repository, private docker index.
  etc.
* Many more examples in documentation
* flowcharts and images
* Testing, examples, and instructions for use with various PaaS tools. Does this
  mean plug-ins? cli-tool? Wheels need to be built/destroyed/rebuilt in specific
  ways, might need a basic wrapper fuction around Docker for this.

##Help

General help and all other discussions are welcome over at the
[forum](https://groups.google.com/forum/?pli=1#!forum/radial-docker-topology).
For any bugs, please use the GitHub bug tracker for the appropriate code
repository.
