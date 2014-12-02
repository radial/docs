#Config, Data, Logs, Run

There is an interesting filesystem strategy that I've found to be quite
refreshing when dealing with exactly where to store information in my
containers. We all know and love the many...MANY options the [Filesystem
Hierarchy Standard][fhs] gives us for storing various types of data needed for
our application. The flexibility is great, but it's also debilitating when we,
or someone else, needs to enter into a container and try to sort out where
things are. 

So for Radial, I've proposed a clear and consistent standard for all data that
enters a container.
* All configuration, and this includes internal application configuration as
  well as any other instance/deploy configuration that later gets inserted at
  build time, that would normally be in `/etc` somewhere is stored in `/config`.
* All static data that **should** be persisted in typical 
  [volume container](http://crosbymichael.com/advanced-docker-volumes.html)
  fashion such as a data-set that is important to the application, as well as a
  location catch-all for caching and other data that could typically be thrown
  somewhere in a users `/home` directory should go in `/data`.
* All logs are writing to `/log` in a sub-folder that is named after the
  container ID. Since `/log` is an Axle container in the typical Radial
  topology, avoiding name collisions is mandatory. What better method then to
  use the containers own unique ID.
* `/run` is used just like one would expect on typical systems for the location
  of socket files and other runtime data.

So why these paths instead of `/etc` or `/var/log` anyway? The reason is that it
will make our lives easier in the long run if we free ourselves from the
[Filesystem Hierarchy Standard][fhs] when it comes to our containers and just
stick everything in the root path in unique folders that will never change.
We'll see more examples to prove this point later but the core of the argument
is that between different programs, operating systems, and different methods of
inserting code into our containers via `ADD` or using `VOLUME`, we introduce a
lot of uncertainty with what directories we are overwriting and mounting over
and where certain files can be found. Plus, it is just much easier to have 3
distinct locations that are completely free from the typical filesystem stuff
for someone that is entering the container looking for things. So it's best to
just use our own paths and filesystem logic from the start and be consistent
with it throughout our cluster.

In more detail:

##Why `/config`?
* Letting our app install a default configuration and then needing to either
  `ssh` or `docker run -it myapp-hub /bin/sh` to edit it is a nightmare to
  standardize; avoid it like the plague. We want to `ADD` an already primed
  configuration file or retrieve it from some other server or repository.
* Using `ADD` however overwrites files, which could work in the case of using a
  default location of `/etc/myservice/config.conf`... or is it
  `/etc/myservice.d/partone.conf`, or is it `/etc/default/config.conf`... or is
  it... well, you get the idea.  `/etc` is pretty standard, but not standard
  enough to be able to 100% of the time pull an image from the Docker Index and
  plop a configuration file in and know that your app or service will know where
  to look. This is why we don't rely on default locations for configuration
  files but explicitly state our configuration path when we run our app. It's
  pretty standard for programs support a custom configuration path at runtime so
  we will include this as one of our topology rules for running apps in Spoke
  containers.
* Another bad idea is to try and mount volumes to system paths such as `/etc` or
  something similar and have other containers "insert" the config.  We might
  unknowingly break all kinds of other services by deleting other configuration
  files with our volume mount.
* If you're thinking why not just go interactive with `docker -it` and then
  disconnect or something similar? Well you could but it's not exactly something
  you can standardize. Also, remember that it's tricky to setup a container and
  "re-attach" to it and have a shell running there waiting for you if you need
  to use the `CMD` line to actually do things other then just run a shell. But
  more importantly, do you really want to modify configuration files inside a
  container using whatever stock editor is supplied by the operating system?
  Probably not.

## Why `/data`?
* Similar to point 2 above; with many different services and programs using
  different default paths for their data directories, and many of those same
  services using different locations even between operating system
  installations, we want to know that there is a 100% guaranteed match and
  consistency between our persistent volume storage Hub container and our
  application Spoke container. We want piece of mind that as long as we specify
  either at run time or in our configuration that the data directory resides at
  `/data` we are good to go with no questions asked.

## Why `/log`?
* If we've already created an Axle container for our logs at `/log`, we'll use
  `--volumes-from` our already-running "logs" container to make it available now
  to this Hub container and by association to our application Spoke container
  (by later running the Spoke container with `--volumes-from` the Hub
  container). Keeping with the now established pattern, we can also specify the
  path (and non-conflicting name) for our log file at run time for our app and
  have our entire clusters logs (for that host) now available.

## Why `/run`?
Since containers don't run anything by default, this space is largely unused in
containers. Radial will use this folder however to share unix sockets between
various Spokes. This is also the location of each spokes Supervisor socketfile.
This allows one to control all inner processes of a wheel from one place.

## Take-aways
So here is the key topology principle:

**Never rely on default locations for our configuration files, our data
directories, and our logs. Declare custom paths at runtime and use the same
paths for everything; ideally `/config`, `/data`, and `/log`.**

[fhs]: http://www.pathname.com/fhs/
