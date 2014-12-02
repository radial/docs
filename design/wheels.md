## The Wheel

The wheel is the standard model for packaging your entire stack. You can see an
example repository [here](https://github.com/radial/template-wheel) to get a
more complete sense of how such a thing would work. While some components of the
Wheel repository are works-in-progress and subject to change, it's still worth
explaining here to get a sense of what I hoped to achieve with it.

### File structure

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

### Configuration Branch

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
