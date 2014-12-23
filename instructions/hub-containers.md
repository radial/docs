# Hub Containers

Hub containers are the big workhorses of the Radial topology. They are
the gathering point for all the complexity of your wheel. 

## Configuration

The big idea with 12-factor and configuration is that it is:
  1. stored separate from application code
  2. in version control
  3. only to be mixed with application code at deploy time
  4. and transported via environment variables

The Radial topology addresses each point accordingly:
  1. Configuration has it's own container (the hub).
  2. We use git to manage various version of configuration and can access it
     remotely or directly using a wheel repository.
  3. Wheels are modular components and various version of configuration can be
     easily combined with spoke containers depending on need.
  4. Environment variables are used for everything.

## Static Mode

*TODO*

* building
    * config
        * build-env file
        * local files

## Dynamic Mode

*TODO*
* running
    * config
        * env vars

### Tunables

Tunable environment variables; modify at runtime. Italics are defaults.

- **$SUPERVISOR_REPO**: [_https://github.com/radial/config-supervisor.git_]
Repository location for default Supervisor daemon configuration.
- **$SUPERVISOR_BRANCH**: [_master_] Repository default branch.
- **[$WHEEL_REPO[_APP1]...]**: Additional repositories to download and merge
with the default SUPERVISOR_REPO repository.
- **[$WHEEL_BRANCH[_APP1]...]**: Branches to pull for given WHEEL_REPO
repositories.
- **$UPDATES**: [_False_|True] Update configuration from the selected
WHEEL_REPO repositories (if any) on container restart.
- **$PERMISSIONS_DEFAULT_DIR**: [_"755"_] Default (recursive) directory
permissions for /config, /data, and /log.
- **$PERMISSIONS_DEFAULT_FILE**: [_"644"_] Default (recursive) file permissions
for files contained in /config, /data, and /log.
- **$PERMISSIONS_EXCEPTIONS**: [_empty_] A single string, separated by spaces,
containing a list of files/directories to chmod/chown.
- The format for a single entry:
{\<path to dir or file\>}{:\<octal mode\>}[:\<user\>][:\<group\>]
- These values, separated by ':' are passed directly into `chown` and
    `chmod` so things like `/config/*` work for directory contents and numeric
    user and group ids work as well.
- Some examples:
    - `/config/supervisor/conf.d/*:700`
    - `/config/supervisor/supervisord.conf:700:root:root`
    - `/config/supervisor/myprogram.conf:777:myprogramuser`
