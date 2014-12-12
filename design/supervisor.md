# Process Management

[Supervisor](http://supervisord.org) is a common choice for process management
inside Docker containers. Many already have experience with it and it is quite
simple to setup and use, thus it was included here as well for the Radial
Topology.

## Configuration

The [Supervisor skeleton][super_skel] is a default configuration for how
Supervisor will be run in our Spoke container. It should suffice for most of
your needs, but it is also configurable in case you would like to make
modifications. This is done by creating your own `supervisord.conf` and making
it available in a git repository with the same folder structure as the template.
Depending on if you build your hub dynamically or statically, you select this
repository by setting the environment variables 'SUPERVISOR_REPO' and
'SUPERVISOR_BRANCH' in either your hub Dockerfile, your `docker run` arguments,
or by setting it in your `hub/build-env` file in your Wheel repository for
sourcing during build. In either case, git will clone and pull the designated
branch at the specified repository location. To find out more about which
features of git are supported by the Hub container, check out the repository for
the [hub-base image](https://github.com/radial/imagebase-hub).

In case you would like to use your own Supervisor configuration, there are some
options you must be mindful of not to change to make sure that the rest of the
Wheel will continue to work properly.

## Folder Structure

```
supervisor
├── conf.d
│   └── sshd.ini
└── supervisord.conf
```
The 'supervisor' folder needs to be in the root of your repository and the
folder naming and structure can't deviate from the above tree. Note that the
supervisor folder sits directly in your `/config` directory in the root of your
Hub container.

## Subprocess init Files

It is a Docker best-practice is to keep the number of processes in a container
to a minimum. However, multiple processes are easily supported. By default, each
spoke starts up a group identified by the `$SPOKE_NAME` variable in the spoke
Dockerfile. You can have any number of subprocesses configurations split amongst
any number of files. They all get put in the `/conf/supervisor/conf.d` folder
when building your hub container. As long as you properly group them, the entire
group will get started. See
[this](https://github.com/radial/wheel-log.io/blob/master/hub/config/supervisor/conf.d/logio.ini)
wheel's configuration for an example.

**Note**: use of the "autostart" feature is discouraged when you have multiple
spokes attached to your hub container. All the Supervisor subprocess
configuration files will be detected by all the other Supervisor daemons in
every other spoke. So to keep from starting every process automatically in every
spoke, we instead resort to having each spoke manually launch it's own
respective subprocess. Again, this is done via a properly configured subprocess
group and use of `$SPOKE_NAME`.

## Logging

By default, the Spoke container, when run, will create a folder in `/log` named
as the containers ID. It gets it from the '$HOSTNAME' variable that is set in
the docker container by default. Luckily, Supervisor has access to this variable
as well, so the [Supervisor skeleton][super_skel] by default will dump all the
logs into each spokes unique folder. This is what allows us to use the same axle
container to store the logs from all our spoke containers without the worry of
name collisions. We then have only one place to look for viewing and backup our
logs.

[super_skel]: https://github.com/radial/config-supervisor  

## Supervisor Sockets

The Supervisor daemon can be controlled with the `supervisorctl` command via a
unix socket located in `/run/supervisor/$HOSTNAME`. When built, all Supervisor
daemon sockets are found in the `/run/supervisor` directory within their
respective `$HOSTNAME` folder. The `/run` directory is being shared by the hub
container. Future plans are in the works to expand the role of these sockets,
the `/run/supervisor` directory, and other parts of Supervisor in general for
automatic monitoring, resource reporting, and even service discovery. They will
be add-on spoke services just like any other.
