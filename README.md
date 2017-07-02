# bzub's Bare-Metal Cluster Lab
These are my personal cluster provisioning configs/scripts based on Matchbox and
Bootkube.  I want to make them as generic as possible so they can be adapted to
your environment with minimal changes, this is a work in progress.

## Getting Started
**TODO**

## Features

### Cluster Environment Groups
I want to make it easy to group nodes into different clusters. In my case I
deploy a testing environment and a "production" environment. This is supported
by a few metadata variables in the `groups/` config for each node, and the
`profiles/` config for each environment.

#### Node Specific Configuration
The `groups/` configs hold node-specific information like node number, low memory
support, and master/worker role.

#### Environment Specific Configuration
The `profiles/` configs hold environment specific information like a suffix for
the hostname (i.e. "-test"), IP subnet, and CoreOS image version.

### OS In Memory

Currently the configs are focused on booting CoreOS via PXE and run almost
everything in memory. The exceptions are directories that need to have
their data persisted between reboots for stability of the cluster:
- /etc/kubernetes (for kubeconfig, CNI config, and Bootkube checkpointer files)
- /var/lib/rook (needed for Rook cluster storage device settings)

#### Image Stores
rkt and docker directories hold image data that I don't want to download on
every reboot, so the current solution uses a small script to save downloaded
images to a shared NFS directory, and another small script that loads the images
on system startup.  This should be replaced with a private image registry in the
future.

#### Low-Memory Node Support
There is a metadata variable called `low_mem` that should be set to `true` on
nodes that have less than ~12GB RAM.  With `low_mem` enabled, the following
directories are instead mounted from a block-device file that's served from an
NFS server.  These directories take up the most space on a Kubernetes node in my
experience, and would not fit in memory on smaller nodes.
- /var/lib/docker
- /var/lib/rkt
