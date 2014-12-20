# Terms

Here I'll clarify some terms that have been used loosely thus far in the context
of the Radial container topology:

**Stack|Application Stack**: _One or more specific pieces of software that work
in tandem to deliver a coherent result._

In Radial, the Wheel is most analogous to an application stack in its scope, but
it is not limited to a stack alone. A Wheel can allow for common configuration
profiles, sharing of unix sockets, logs, cache etc., all while maintaining
separation of each application in the stack due to their being in their own
application containers (spokes). Typically, this implies that a Wheel must
reside on a single host but this isn't necessarily always true.
    
**Service**: _An abstract software functionality that can be made up of one or
more application stacks. A stack can be sufficient by itself to deliver a
service, or a service may be made up of many stacks._

A single database on a single host can be used for multiple purposes; it by
itself can be considered a service. The components of a LAMP stack on a single
host by themselves are transparent, yet altogether, provide a web service.
Variations of core/monitoring/api/master/worker stacks across all nodes in a
server cluster could provide a distributed file system or computation engine.
That is a single service made up of many different heterogeneous stacks on
multiple hosts.

**Wheel**: _An instance of one or more optional axle container(s), a mandatory
hub container, and at least one spoke container._

A Wheel is a flexible entity. Since it is a modular collection of cooperating
containers, a Wheel can be your atomic unit for both stacks and services in any
combination you can think up. When designed right, spoke containers are
interchangeable because any and all configuration is stored in the hub
container, separate from the spoke. So the `radial/postgresql` container used
by itself, is the same used in the LAMP stack, is the same used as some part of
a distributed database service across many hosts etc. The goal of Radial is to
eliminate Dockerfile explosion all because of slightly different use-cases of
similar services, all while not forfeiting Docker or web best practices.


