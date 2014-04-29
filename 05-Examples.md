# Examples

## Simple Configuration

Let's say we have some data-mining process we are going to use. We have some
configuration files and some sensor data to input into our Hub container, how do
we plan this out?

First, our Dockerfile would contain:

`FROM radial/hub-base`

Now, let's think about what it is we're capturing here: both a snapshot of our
configuration and of the dataset, whatever that happens to be. You need to
decide what it is you're going to configure and what you need out of these Hub
containers and design them accordingly. Perhaps we will need to run many of
these data-mining processes from this same data set, but try out different
versions of our configuration? In that case, it might be wise to actually stick
our data set in an Axle container by itself and use `--volumes-from` when
running our Hub container. In doing that, we can then be free to build many
versions of our configuration and tag them on build knowing that our sensor data
is safe.

In the converse situation where the configuration is static, but our data is
dynamic, then our initial dataset will be captured in our image, but our new
data run by the process will only be dynamically available in our running Hub
container. This case is simple because we can't version our dynamic data, so we
would just build our Hub container with `docker build -t dataminer-hub .`
and be done with it. Remember, as soon as a different service needs to access
this data set, we need to move it to it's own Axle container. 

Let's say though that for simplicities sake, we have both static data that is
only accessed by this Wheel and a static configuration. We would put the
configuration in `/config` and our data set in `/data`, build it with `docker
build -t dataminer-hub .`, run the Hub container with `docker run --name
dataminer-hub dataminer-hub`, and run the Spoke container with `docker run
--name dataminer --volumes-from dataminer-hub radial/dataminer`.
