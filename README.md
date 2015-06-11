# Radial Documentation

## **PENDING UPDATES**

Radial was originally conceived very early in the Docker story and this has been
an alpha quality experiment to try and solve some of the gaps in the Docker
ecosystem. Many important features were not implemented yet and the ecosystem
was exploding with many different ways to use containers and best practices were
being stumbled upon slowly over time. Many things have happened since Radial's
inception that have made the Docker ecosystem a little more mature:

* Kubernetes has come on the scene and dominated the way orchestration is
  handled at scale
* The Docker daemon has matured considerably and allows for much more control of
  the processes running in the containers
* The orchestration and networking picture for Docker has a more clear
  trajectory with the introduction of machine, compose, and swarm
* Tooling around Docker has also matured and there is less and less reinventing
  of common tasks that must be done

So with all this considered, I've revisited what Radial is, and what it should
be in relation to all these changes and I now have a better idea of what Radial
can contribute to the bigger picture. In general, this is a focus on simple
implementation of container best practices that are orchestration agnostic and
can scale both large and small for the average user that does not want to make
custom Dockerfiles in order to use a program. Some of the upcoming changes
include:

* Pulling supervisor from the containers.
    * Radial will strictly uphold the one process per container principal as it
      is much easier now to send signals, enter container processes, and link
      containers to each other. There is less and less evidence in my experience
      to warrent having multiple processes in a container ever.
* Standardizing setup and entrypoint scripts.
    * Drop-in setup and running of final binaries
* Simpler log management
    * Make use of the logging driver in Docker 1.6
* Simpler hub scripts
    * using new env-file
* Spec
    * define a spec to promote standardization and interoperability

And many more. Check out the beta branches of each repo to check out the latest
and greatest. Comments always welcome, Thanks!

## What is Radial?

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

## What Can I Do With It?

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

## What's The Strategy?

In short, I use the analogy of an "axle", a "hub", and a "spoke" to help
conceptually link together Docker containers into different classes based on
what we need the entire stack (the "wheel") to be doing. Axle containers are
volume containers, hub containers manage application configuration, and spoke
containers are our application containers. I've created specialty Docker images
streamlined for each of these classes and are available on the [Docker
Index](https://index.docker.io/u/radial). 

These images use tools such as busybox to keep container size down, git to clone
and combine configuration, and Supervisor to manage our all our processes and
glue it all together.

Radial makes configuration a first-class citizen by stripping it out of the
application container and putting it in it's own container (named the "hub")
which is later mounted into the application container (the "spoke"). This allows
interchangeable and modular application containers for many use cases.

## The Documentation

### Concepts

* [The Topology](/concepts/topology.md)
* [The Factors](/concepts/factors.md)
* [Terms](/concepts/terms.md)

### Design

* [File System Structure](/design/filesystem.md)
* [Wheels](/design/wheels.md)
* [Radial Images](/design/images.md)
* [Supervisor](/design/supervisor.md)

### Instructions

* [Axle Containers](/instructions/axle-containers.md)
* Hub Containers
* Spoke Containers

## Help

These documents are works-in-progress as I put their suggestions into practice.
Updates are to come as suggestions and testing come in.  General help and all
other discussions are welcome over at the
[forum](https://groups.google.com/forum/?pli=1#!forum/radial-docker-topology).
For any bugs, please use the GitHub bug tracker for the appropriate code
repository.
