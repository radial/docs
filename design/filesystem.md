# Config, Data, Logs, Run

The [Filesystem Hierarchy Standard][fhs] provides a great deal of flexibility
regarding where and how to organize our file system. On top of that, many of our
individual preferences are heavily influenced by the distributions we use day to
day. This complicates more than it helps however when it comes to the type of
distributed and multi-layered filesystem that emerges when we start to combine
Docker containers using volumes. Some helpful usage patterns have emerged in the
community, and for Radial, I incorporate some of those practices along with some
additional items.

* All configuration that would normally be in `/etc` is stored in `/config`.
  This includes internal application configuration as well as any other
  instance/deploy configuration (Supervisor) that could later get inserted at
  build time.
* All static data that is persisted in typical [volume
  container](http://crosbymichael.com/advanced-docker-volumes.html) fashion
  should find it's way to a sub-directory of `/data`. This directory is also a
  catch-all for caching and other runtime data that could typically be thrown
  somewhere in a users `/home` directory. Effectively, this directory is a
  combination of `/mnt`, `/opt`, `/home`, and `/var`.
* All logs are written to `/log` in a sub-folder that is named after the
  container ID. Since `/log` is typically it's own axle container, this
  directory could potentially collect the logs of many wheels across a single
  host, or even across an entire small cluster of hosts. Thus, the naming of
  each folder is the same as the spoke containers short ID to prevent
  collisions.
* `/run` is used just like one would expect on typical systems for the location
  of socket files and other runtime data.

In more detail:

## Why `/config`?
* We want to avoid editing configuration once our container is running like the
  plague. It's not easily reproducible and promotes run-away container
  modification, which we don't want.
* Relying on the standard location for application configuration doesn't work
  well when we want modular containers. Things can get lost when doing complex
  volume sharing and mounting in multiple containers when the configuration
  paths are not standardized from the onset.
* It's pretty standard for programs to support a custom configuration path at
  runtime so we leverage this common option to our advantage and always point
  our binaries configuration to it's own location in `/config`.

## Why `/data`?
Similar to point 2 above; with many different services and programs using
different default paths for their data directories, and many of those same
services using different locations even between operating system installations,
we want to know that there is a 100% guaranteed match and consistency between
our persistent volume storage hub container and our application spoke container.
We want piece of mind that as long as we specify either at run time or in our
configuration that the data directory resides at `/data` we are good to go with
no questions asked.

## Why `/log`?
If we've already created an axle container for our logs at `/log`, we'll use
`--volumes-from` our already-running "logs" container to make it available now
to our hub container and by association to our application spoke container
(itself, running with `--volumes-from` the hub container). Keeping with the now
established pattern, we can also specify the path (and non-conflicting name) for
our log file at run time for our app and have our logs now available in one
location for processing using whatever method we choose.

## Why `/run`?
Since containers don't run anything by default, this space is largely unused in
containers. Radial will use this folder however to share unix sockets between
various spokes. This is also the location of each spokes Supervisor socketfile.
This allows one to control all inner processes of a wheel from one place.

[fhs]: http://www.pathname.com/fhs/
