# The Factors

With inspiration from [The Twelve-Factor App](http://12factor.net), I explore
how Radial helps to fulfil certain 12-factor principles using Docker features.

## Codebase

>One codebase tracked in revision control, many deploys

Docker images make this a breeze. Dockerfiles reproduce a build on any machine
and sharing them is straightforward across machines. Where Radial adds to this
principal, is that it specifies a container solely for the application itself,
separate from any configuration or environment information to make this more
modular. This is the spoke container.

## Configuration

>Store config in the environment

The typical interpretation of this factor is with regards to configuration that
varies in between deploys (staging, production, developer environments, etc).
This can be things such as credentials to external services, and per-deploy
values that are what make a staging deploy different from a production one for
example. 12-factor states these values should live in environment variables, not
in the code itself.

Radial wheels take this concept a bit further. As stated in "Codebase", since we
separate out ALL configuration from the application container, we have a little
more freedom for combining configuration and code because everything is designed
to be modular. Radial uses a hub container to store application configuration,
and since our fundamental "unit" for deploying apps is the wheel (multiple
containers) instead of the single container, we can now use the wheel repository
as a method to store this deploy specific information. Programs like [fig][fig]
allow us to think in terms of multiple containers coordinated together. It is in
a fig.yml, or some similar orchestration configuration, that we can specify our
remaining environment configuration. The spirit of this 12-factor principle is
still kept; our application source code is completely separate from any and all
instance and configuration settings.

[fig]:http://www.fig.sh

## Backing Services

>Treat backing services as attached resources

Because of spoke containers, use of an application can be ubiquitous across any
use case. Again, because application configuration in the hub container is
separate from the application in the spoke container, one can easily use the
same Docker image for any situation. No need to make an entirely separate image
just because you want to change configuration settings. This isn't specific to
this 12-factor principle, but this is especially useful when dealing with
database services, which usually a little more complex to setup.

## Build-Release-Run
 
>Strictly separate build and run stages

Docker really does force you to separate these stages. Build is represented by a
`docker build` command, release is a `docker create` command with environment
variables to input the configuration, and you run it immediately with `docker
run`.

The component of this principle that Radial helps with is the "release" stage;
the combining of application with configuration. Because configuration and
application code are represented by their own containers, all it takes is a
simple `--volumes-from` commands to link up a spoke container to it's
configuration in the hub. This combination can be very quick to implement
because we don't require a build step just to change configuration. Already have
a working configuration that you want to test with a new version of the binary?
Use the existing hub container image to test it out. This modularity makes the
various combinations easy to manage because it encourages smaller images that can
easily be pieced back together in case of rollback.

## Concurrency

While concurrency is more about designing a stateless app then it is about how
you organize your containers, I feel that Radial's modular design helps
encourage this. Because configuration is stored in the hub container, and
because volume sharing is done predictably (hub does `--volumes-from` axles,
spokes do `--volumes-from` the hub), spoke containers can be spun up and scaled
on an existing wheel and simply attached to it. This can give a simple wheel
more "features" such as service registration or application monitoring, or even
run multiple identical instances of already existing spoke containers.

## Disposability

>Maximize robustness with fast startup and graceful shutdown

Radial uses small and modular images for quick startup,
[Supervisord][supversior] for in container process management of one or more
running processes within the container itself, and carefully designed entrypoint
scripts to facilitate proper transfer of UNIX signals from the docker daemon and
enforce proper behavior of container start, restart, and shutdown procedures.

[supervisor]:http://supervisord.org

## Dev/prod parity

>Keep development, staging, and production as similar as possible

The easier it is to reuse the same core images for development and production,
the closer the development environment will be to production. Many more factors
go into supporting this principle of course, but as far as spreading an
application stack across Docker containers goes, if you can use the same web
server and database image on your development machine as your production
machines, only making small configuration changes in your hub container as
needed, we are one step closer to keeping this 12-factor principle.

## Logs

>Treat logs as event streams

As far as the application is concerned, writing to STDOUT or STDERR is mandatory
in order to work with [Supervisord][supervisor]. So this principle is preserved;
the application itself doesn't bother with log files. However, Supervisor itself
does in fact write these logs to files, and our end goal still is to make our
log events a _stream_, not just files. The Radial solution to this is to gather
all logs for the wheel in a single spot, the `/log` directory. Within it, unique
folders are created that allow for any number of spokes to write logs here
without name collisions. The idea is simple, if we can make all the logs
available in a predictable location, we can use whatever logging program we want
later to turn these log files into a stream. The log harvesting itself is a
feature for it's own spoke container, and Radial doesn't make that decision for
you by baking in the log management into it's containers.

## Admin Processes

>Run admin/management tasks as one-off processes

Radial spoke containers allow for a standard daemon mode as well as a custom
entrypoint-like command mode that allows one to specify an alternate behavior
for that spoke container. This feature allows one to use the same spoke
container to run a database (say postgresql) and manage it directly using
it's management program such as `psql`. This keeps the environment between the
admin process and the app identical. 

To further simplify this task, Radial organizes all Supervisord unix sockets as
well as any other application sockets in the hub container in a shared `/run`
directory for easy inter-spoke communication. This has the added benefit of also
allowing access to application management via the unix sockets without using
built-in SSH or opening any holes within the spoke container itself. One-off
admin/management tasks are added onto a wheel just like any other spoke
container would. Everything it needs will be available via the hub.
