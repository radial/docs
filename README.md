#Radial Documentation

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

Radial is a container topology strategy that is is outlined in this
documentation and is implemented in the suite of accompanying Docker
[images](https://index.docker.io/u/radial/) and Dockerfiles that this project
supplies. Since Radial deals entirely with how you spread your application stack
across containers, it is intended to be compatible with all orchestration, PAAS,
and cluster management tools without requiring or locking you into any single
one of them. If you like to use these images manually, they should be able to
scale up with you into complex cluster use. Hopefully, there are some good ideas
are in here for you to use or replicate. Better still, hopefully these images
themselves will serve you well in all your use.

##What's The Strategy?

In short, I use the analogy of an "axle", a "hub", and a "spoke" to help
conceptually link together Docker containers into different classes based on
what we need the entire stack (the "wheel") to be doing. I've created specialty
Docker images streamlined for each of these classes that are available on the
[Docker Index](https://index.docker.io/u/radial). 

These images use tools such as busybox to keep container size down, git to clone
and combine configuration, and Supervisor to manage our all our processes and
glue it all together.

Radial makes configuration a first-class citizen by stripping it out of the
application container and putting it in it's own container (named the "hub")
which is later mounted into the application container (the "spoke"). This allows
interchangeable and modular application containers for many use cases.

##The Documentation

###Concepts

* [The Topology](/concepts/topology)
* [The Factors](/concepts/factors)

###Design

* [File System Structure](/design/filesystem)
* [Wheels](/design/wheels)
* [Radial Images](/design/images)
* [Supervisor](/design/supervisor)

###Instructions

* [Axle Containers](/instructions/axle-containers)
* Hub Containers
* Spoke Containers

##Help

These documents are works-in-progress as I put their suggestions into practice.
Updates are to come as suggestions and testing come in.  General help and all
other discussions are welcome over at the
[forum](https://groups.google.com/forum/?pli=1#!forum/radial-docker-topology).
For any bugs, please use the GitHub bug tracker for the appropriate code
repository.
