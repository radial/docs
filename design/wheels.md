# The Wheel

The Wheel is the standard model for packaging your entire stack. You can see an
example repository [here](https://github.com/radial/template-wheel) to get a
more complete sense of how such a thing would work. While some components of the
Wheel repository are works-in-progress and subject to change, it's still worth
explaining here to get a sense of what I hoped to achieve with it.

Note that what is described here is a Wheel repository. It is a single
repository that can be used in conjunction with container orchestration to
produce the "running" Wheel, which is a collection of interchangeable containers.

## File structure

```
.
├── axle
├── hub
│   ├── Dockerfile
│   ├── build-env
│   ├── config
│   │   ├── someprogram.conf
│   │   └── supervisor
│   │       └── conf.d
│   │           └── someprogram.ini
│   ├── data
│   │   ├── media
│   │   │   └── dataset
│   │   └── src
│   │       └── someprogram.sh
│   └── log
│       └── logfile
└── spoke
    └── Dockerfile
```

While the structure of the 'axle' and 'spoke' folders is technically not
mandatory, it is suggested to keep it like this for compatibility with future
planned features. 

The folder structure of the 'hub' folder however IS mandatory. Deviating will
break the 'hub-base' and 'spoke-base' images. Application configuration sits in
`hub/config` and the accompanying Supervisor '.ini' file goes in
`hub/config/supervisor/conf.d`.

## Configuration Branch

Your application has a repository for it's code. Your deployment code and
configuration however, as it pertains to using Radial, is stored in the
Wheel repository. The 'hub-base' image has the ability to `ADD` your
configuration (whatever files you put in your `hub/config` and
`hub/config/supervisor/conf.d` folders) as well as pull it from a repository. If
pulling from a repository is your method of choice, it is suggested that your
make a separate branch in your Wheel repository with just the contents of your
`hub/config` folder and all it's subfolders and files. The 'hub-base' image
contains logic to pull the 'config' branch of your Wheel repository and merge it
with the [skeleton configuration][config-supervisor] used by Supervisor.

The folder structure of your 'config' branch would be as follows:

```
.
├── someprogram.conf
└── supervisor
    └── conf.d
        └── someprogram.ini
```
The two files 'someprogram.conf' and 'someprogram.ini' demonstrate the needs of
a typical application, but this folder structure can easily support more
complicated situations.

[config-supervisor]: https://github.com/radial/config-supervisor

## Fig

Since Radial makes liberal use of containers and "separates concerns" to an
extreme degree, a basic orchestration tool is needed to help manage the
building, linking, and deploying of all the containers. Other tools can surely
be used for this, but for the sake of simplicity, Radial uses [Fig][fig] for now
for demonstration and testing.

[fig]: http://www.fig.sh

## Dynamic vs. Static Modes

Packaged with each wheel, are example `fig.yml` and `fig-dynamic.yml` files.
They demonstrate the two ways that you can produce images. 

### Axles

Axle containers are volume containers. So they're use can vary depending on
need. Typically, volume containers are dynamic, meant to store the data of some
other container running a database perhaps. They also could potentially be used
statically and house some type of static data set, source code, or media.

### Hubs

Hub containers allow for any combination of `docker build`, `docker run`
(without building), and configuration branches stored in `$WHEEL_REPO`
environment variables. In order of most "static" to most "ephemeral", you could:

  1. Manually create and store the configuration files in a wheel repository,
     then use `docker build` to capture the files in an image. This image can be
     transported and stored with the configuration files always maintained
     within.
  2. Use `docker build` to create a static hub container, the same as point 1, but
     if you don't have access to the actual configuration files, and they happen
     to be stored already in a wheel repository "config" branch, you can specify
     these `$WHEEL_REPO` repositories in a `/hub/build-env` file that gets
     sourced on `docker build` therby cloning the configuration into the hub
     container and storing it statically the same.
  3. For a quicker and more dynamic mode, you can simply run the
     `radial/hub-base` image without build, but specify `$WHEEL_REPO`
     environment variables at run time and the hub will clone them. **This mode
     is ephemeral** and as soon as you remove this running container, the
     configuration will be gone as well. This should not be a problem however
     since you already have your configuration stored safely in version control.
     The hub simply acts as a volume container for your configuration in this
     situation.

### Spokes

After a spoke container is initially built, it is supposed to always be
dynamically run after that. A spoke container captures a single build of a
binary and will eventually run it; and does nothing else. The configuration for
this spoke is stored in the hub container thereby never requiring any
customization of the spoke container.
