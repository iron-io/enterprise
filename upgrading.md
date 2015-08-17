# Upgrading

We loosely use [SemVer](http://semver.org/), with the modification being that
MINOR versions are for breaking changes, potentially within the API or in the
format of a disk or network protocol. MAJOR versions are left for drastic API
changes.

Most upgrades will not require much, but some changes break compatibility. This
leaves 2 recommended 'upgrade' paths, hinging on whether there are breaking
changes or not.

## IronMQ

#### Backwards Compatible changes (PATCH version changes)

The recommended way to run IronMQ is to use systemd to run the `iron/mq` docker
image. There are other ways to run IronMQ, but they are not officially supported
and your system administrator should make a document similar to this with their
way of doing things.

All that is needed to update IronMQ is to change the docker version being ran.
In the `ironmq.service` file, any line with `iron/mq:$version` needs to be
modified to the new version. Alternatively, we offer convenient release tracks
that track MINOR version. For example, using `iron/mq:3.2` will not require
modifying the `ironmq.service` file, updates will automatically happen when a
process restarts to get any PATCH releases. Otherwise, releases can be found
on [dockerhub](https://hub.docker.com/r/iron/mq/tags/).

After modifying the `ironmq.service` file, if necessary, simply `$ sudo systemctl
restart ironmq` on each node. In order to ensure there is no downtime, watch the
IronMQ logs to see a line that contains `msg=membership` and has the full set of
nodes as a list before restarting the next node. You should also wait until any
lines about `starting replication...` or `finished replication...` stop for a
minute or two. This ensures the cluster stays in a stable state through the
upgrade.

TL;DR `$ sudo systemctl restart ironmq` and be patient

#### Backwards Incompatible changes (MINOR or MAJOR version changes)

You will need a few things prepared before being able to upgrade breaking
changes in the supported way:

* `$ docker pull iron/mig`
* IronMQ cluster running the newer version

The goal being to create a new cluster with the newer version to delete the old
cluster, without losing data and without downtime (if necessary).

After preparing these two items, you are ready to get started.
If you need to ensure that there is no downtime, that will be discussed.

First, we're going to run the migrator to migrate any metadata before switching
traffic over. This is to ensure that new messages that go to push queues don't
get put on pull queues initially. Please take care to use the `--skip-messages`
flag at this point.

`$ docker run -it --rm iron/mig ./move-queues all -auth $auth_host -token
$admin_token --from $old_ironmq_host --to $new_ironmq_host_or_load_balancer
--skip-messages`

Now, we can move traffic from the old IronMQ nodes to the new IronMQ nodes. The
officially supported way to do this is with a load balancer, we recommend using
HAProxy, however, there are other ways to direct traffic to IronMQ. The HAProxy
config file can be found at `/etc/haproxy/haproxy.cfg` and in the correct
backend should be the IronMQ nodes to replace. DO NOT shut down the IronMQ
nodes, in the next steps we will be recovering the messages from them.

After removing all of the old nodes from the load balancer and inserting the new
nodes -- and waiting for traffic to start flowing -- we are ready to migrate messages
over.

`$ docker run -it --rm iron/mig ./move-queues all -auth $auth_host -token
$admin_token --from $old_ironmq_host --to $new_ironmq_host_or_load_balancer`

Where `$old_ironmq_host` can be any host from the old cluster, IronMQ will be
able to find every queue from any host, e.g. `10.1.2.3:8080`.
`$new_ironmq_host_or_load_balancer` can also be any host in the new IronMQ
cluster or the load balancer, routing will be taken care of by IronMQ.

The only caveat is that if there are any queues that have messages that were
posted with delays, the migrator tool will need to be run until the queues are
flushed after these messages become available.

At this point, the old cluster can be shutdown. It is possible to do this
upgrade without using different servers, simply starting the newer IronMQ on a
different port on the same host and having the load balancer direct traffic to
the new nodes.

We realize making backwards incompatible changes is a pain, and strive to only
make such changes when absolutely necessary. We're also happy to help via SSH or
the normal support channels.

## IronAuth

#### Backwards Compatible changes (PATCH version changes)

As noted above, the recommended way to install IronAuth is also via systemd and
Docker. For details, read the IronMQ section above. This should simply be a matter of
restarting each process with the upgraded version.

`$ sudo systemctl restart ironauth`

#### Backwards Incompatible changes (MINOR or MAJOR version changes)

Since IronAuth typically has a low write in contrast to queues, we offer an
application level JSON backup system for easy backups / restores.

###### Get all auth data (save this to a file):

`GET /1/dump?oauth=$ADMIN_TOKEN`

with curl:

`$ curl -s $auth_host/1/dump\?oauth\=$ADMIN_TOKEN > dump.json`

###### Restore auth data, from dump:

__WARNING__: this will wipe the database before restoring. Please make sure that the
dump file looks valid and is up to date before using this.

`POST /1/dump?oauth=$ADMIN_TOKEN`

with curl:

`$ curl -s -d "@dump.json" $auth_host/1/dump\?oauth\=$ADMIN_TOKEN`

###### How to (in place update)

* GET a dump (see above), save it to a file
* stop IronAuth: `$ sudo systemctl stop ironauth` on each machine
* remove each data directory, by default: `rm -rf /mnt/data/ironauth/data` \*\*\*
* start IronAuth with updated version: `$ sudo systemctl start ironauth`
* restore data with POST dump from above -- response is '{"restored"}'

\*\*\* we recommend making a new ironauth cluster so that this step is not
necessary, just to be safe.

If there are any issues, please post logs to support room. We use this facility
ourself for upgrading auth, so expect that we identify issues in the upgrade
process early on if there are any.
