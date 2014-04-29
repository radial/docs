# The Factors

The crux of all the following issues is this design principle in docker:

*Shared volumes are flexible, but dynamic with data, and the build process is
static, but very inflexible if you want to remain portable.*

With inspiration from [The Twelve-Factor App](http://12factor.net), I explore
some of the relevant factors as they pertain to Dockers design.

## Codebase

>One codebase tracked in revision control, many deploys

Docker gives us two methods: `ADD` in a Dockerfile during the build process or
some other method of checking out/downloading code from an external code
repository using operating system tools from your container.  Depending on when
this is done (in the Dockerfile build-process via `ADD`, or using the `CMD` or
`ENTRYPOINT` of the resulting container, or if the code is injected into the
container some other way) we get varying combinations of portability vs
management. Hard-code your repository location into your Dockerfile or insert
the code in the build via `ADD` and you loose portability of the Dockerfile, and
if you're not careful how you add the code, you might never be able to actually
secure it in a Docker "image" because it will be forever linked to a running
container via a shared `VOLUME` which is not captured via `docker commit`.

**Issue 1: What is a viable way to get app code into docker containers?**

Note: for those that don't care about sharing their Dockerfile on the index,
just having a Dockerfile sit in their code base and use `ADD` is a great way to
go. I'm just trying to present some methods for those that want to keep that
aspect of portability and eat their cake too.

## Configuration

>Store config in the environment

Configuration and docker; it's quite a task. We need to be able to modify the
default configuration of our servers and processes and our own apps, but still
also want portability of our Dockerfile AND the ability to tinker with the
configuration at a later date in an intuitive way that doesn't break our
deployment or require rebuilding everything from scratch in a Dockerfile.

**Issue 2: What is a viable way to get configuration into docker containers?**


## Build-Release-Run

>Strictly separate build and run stages

Docker forces you to separate these stages. So what's the issue? It's in actual
implementation of these stages that is unclear. Putting everything into one
Dockerfile is very slow to build, and not very portable. Plus, one would ideally
not want to bother with anything other then deploying *only* the change they
made to the configuration/codebase/environment etc., so that they can make use of
Docker's caching system and be able to quickly deploy/revoke updates.

The more modular and separate different aspects of the deployment are, the
easier it is to modify and piece them together again; modify
configuration but keep the app, compile the app with a new version of the
language used to make it, test the entire thing on a new operating system etc.
Docker makes these things trivially easy to do technically, but it's not quite
clear how best to manage this process yet.

**Issue 3: What is a viable way to deploy my changes easily, clearly, and
sequentially?**

## Disposability

>Maximize robustness with fast startup and graceful shutdown

This again requires a little bit of Dockerfile/image strategy in order to get
that lean and quick startup as well as graceful exit/restart of processes when
parts of your deployment change.

**Issue 4: What is a viable way to design the topology of my images and running
containers to get modularity and easy startup/shutdown/restart of my deployment
components?**

## Dev/prod parity

>Keep development, staging, and production as similar as possible

Docker on your workstation, Docker on your server cluster; done! Yes, but
sometimes with caveats. You ideally want everything under version control
(Dockerfiles, app code, the deployment itself) and accessible via repositories
so that YOU put together implementations on your workstation the same way a
production server does in your cluster

**Issue 5: What is a viable strategy to match development and production
practices and methods?**

## Logs

>Treat logs as event streams

Logs are handled uniquely in Docker it seems because the first inclination is
for one to run a process in the foreground and let all log/debug information
just stream to `stdout` just to keep their container from quitting on them. But
as Twelve-Factor states, we need a process agnostic way to handle them outside
of our app and across containers. `docker logs` is handy for quick status
updates of your container, but as a long term way to handle log output, it is
just not feasible. Try running a very verbose server for a couple days and then
running `docker logs` and see what happens. EVERYTHING from the very beginning
streams down your monitor...and streams...and streams...Plus you can't save and
analyze it later without host dependent hacks.

**Issue 6: What is a viable strategy for handling logs accross Docker
containers in a cluster environment?**

## Admin Processes

>Run admin/management tasks as one-off processes

The mutability and dynamic nature of Docker volumes conflict at times with our
desire to want to `ssh` into the container to do some of these admin tasks. One
must leverage the layered nature of Docker images to do what needs to be done
and incorporate the changes into our method of versioned deployment.

**Issue 7: What is a viable strategy for running admin processes on already
running containers?**
