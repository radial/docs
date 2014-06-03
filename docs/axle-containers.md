##Axle Containers

As mentioned before, Axle containers are for hosting regular or
bind-mounted volumes whose data need to be acted upon by multiple containers.

One example given before was for log files, another could be music or video
libraries. In the case for logs, we need to be free to spin up apps and assure
that the logs have a persistent and predictable place to go. Having a service
like [Logstash](http://logstash.net/) running in your cluster is made easier by
having all your logs in one place; a single "logs" Axle container. This specific
method is of course great for smaller scale operations where your cluster
doesn't have automatic service discovery built-in.

The second example was for bind-mounting files stored locally on the host. I
have an already existing, and very large audio/video library that I manage,
modify, and use directly (nfs mounts on my workstations) and indirectly through
applications in containers (web interfaces to access/stream the library). For
fully public websites, the Docker best-practice of course is to have nothing
bind-mounted so as to remain completely portable and machine-independent; and
that giant media library, according to Twelve-Factor, would need to live in a
database as an attached resource. Because the world isn't perfect however, this
option is still available in Radial for the sake of completeness; we have a
place in our topology for bind-mounts to happen.

Let's create our above example and make a "logs" Axle container. If we choose to
make a permanent image that we can reuse, the Dockerfile would possibly contain:

```
FROM    radial/axle-base
VOLUME  ["/log"]
CMD     ["/bin/true"]
```
It could be built via:

`docker build -t logs .`

and run via:

`docker run --name logs logs`

Or we can do it all in our run step with:

`docker run --name logs --volume /log radial/axle-base /bin/true`

Both methods are identical because the only data being written is to an exposed
volume at `/log`, so docker doesn't keep any data in it's version control in
either case. Use whichever fits your workflow.
