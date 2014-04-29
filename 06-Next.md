# Next

These documents are works-in-progress as I put their suggestions into practice.
Updates are to come as suggestions and testing come in. Feel free to contribute!
Do you use similar strategies for your container topology?

Some implications and future goals for the Radial Topology:

* The ability to create and `docker push` images in twos; one for the
  application itself (the Spoke container) and one for the configuration (the
  Hub container). As long as one person creates their individual Hub container
  for a given app using the Radial guidelines, one can feel confident that their
  Spoke container will "just work". Also, while one can publish their respective
  Spoke containers on the Index without any concern of sharing sensitive data,
  one can publish their Hub containers to a private index for versioning (or
  just build them on the fly of course like these tuturials suggest). This can
  really speed up the testing, sharing, reproduction, and updating of
  application stacks.

* Create official images of common apps using the Radial topology and publish in
  the docker index accordingly: radial/mysql, radial/sshd etc.

* Create Spoke container base
    * Use [supervisor](http://supervisord.org/) to allow for universal "foreground" support of any
        process as well as log management and user control in a consistent and
        predictable way.
    * Must include task of chowning /config /data /log to whatever system
        user that spoke container needs first. The Hub containers manage
        permissions in a general sense, but it's the spokes job to declare and
        `chown` them for itself. This keeps it modular.
    * include logic for versioning and managing app source code similar to how a
      Hub container manages configuration.
    * figure out universal naming of log files for multiple spawns of the same
      process in the cluster

* Create suite of worker/admin/debug containers that one can switch out the
  default `FROM radial/distro` image (which is just Ubuntu Trusty) with `FROM
  radial/debug` to build their app on top of the standard debugging tools and
  packages allowing one to keep the production container as small as possible.
  Or even use `--env` switches on run to tell supervisor to apt-get the debug
  packages and run whatever it needs.

* flowcharts and images
