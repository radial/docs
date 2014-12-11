# Process Management

[Supervisor](http://supervisord.org) is a common choice for process management
inside Docker containers. Many already have experience with it and it is quite
simple to setup and use, thus it was included here as well for the Radial
Topology.

## Configuration

The [Supervisor skeleton][super_skel] is a
default configuration for how Supervisor will be run in our Spoke container. It
should suffice for most of your needs, but it is also configurable in case you
would like to make modifications. This is done by setting the environment
variables 'SUPERVISOR_REPO' and 'SUPERVISOR_BRANCH' somewhere in your Hub Dockerfile or by
setting it in your `hub/build-env` file in your Wheel repository for sourcing
during build. In either case, git will clone and pull the designated branch at
the specified repository location. To find out more about which features of git
are supported by the Hub container, check out the repository for the [hub-base
image](https://github.com/radial/imagebase-hub).

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

## Per-Process init Files

In addition to 'sshd.ini', which is enabled by default, any other processes you
will need for your container (and remember, a Docker best-practice is to keep
these to a minimum), can have their own '.ini' file put in the
`/conf/supervisor/conf.d` folder when building your Hub container.

## Logging

By default, the Spoke container, when run, will create a folder in `/log` that
is exactly the containers ID. It gets it from the '$HOSTNAME' variable that is
set in the docker container by default. Luckily, Supervisor has access to this
variable as well, so the [Supervisor skeleton][super_skel] by default will dump
all the logs into the resulting folder. This is what allows us to use the same
Logs Axle container to store the logs from all our Spoke containers without
worry of name collisions. We then have only one place to look for
viewing/data-mining/backing-up our logs.

[super_skel]: https://github.com/radial/config-supervisor  
