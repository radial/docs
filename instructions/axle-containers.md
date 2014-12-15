# Axle Containers

As mentioned before, Axle containers are for hosting regular or
bind-mounted volumes. This can be host-specific persistent data, or any other
type of [volume container](http://crosbymichael.com/advanced-docker-volumes.html).

By default, Radial uses such a container to store the Supervisor log output of
each spoke container. Rather then make a decision on which log management one
should use, they are instead simply collated into a single location. It's the
job of some other spoke container to harvest/serve/analyze these logs later if
one so chooses.

Let's create our above example and make a "logs" Axle container. If we choose to
make a permanent image that we can reuse, the Dockerfile would possibly contain:

```
FROM    radial/axle-base
VOLUME  ["/log"]
CMD     ["IDLE"]
```

Note that the program "IDLE" is the equivalent of `tail -f /dev/null` meant
solely to keep the container from exiting. This makes container management a bit
more intuitive regarding volume containers.

It could be built via:

`docker build -t logs .`

and run via:

`docker run --name logs logs`

Building this container however doesn't have any benefits, so usually it can
just be run directly from command line with:

`docker run --name logs --volume /log radial/axle-base IDLE`
