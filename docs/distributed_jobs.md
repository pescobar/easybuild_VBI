# Distributed jobs with EasyBuild


## Introduction

EasyBuild has ability to use a separate backend for executing individual build tasks. Consequently a given backend can be used to distribute jobs to compute nodes on a cluster.

Currently the recommended backend is GC3Pie. GC3Pie itself supports building locally, or submitting jobs to a resource manager such as Torque or LSF. 

Detailed documentation on this functionality is available in [Submitting jobs](http://easybuild.readthedocs.org/en/latest/Submitting_jobs.html) section of official EasyBuild documentation.

## Caveats and issues

Currently using various `--try*` options with EasyBuild will result in a build failure, when uses with GC3Pie and remote nodes. This is being addressed via [Issue #141](https://github.com/hpcugent/easybuild/issues/141).

# Running distributed jobs

## How it works

To run jobs on our cluster we have to provide two sets of required options:

* A set of `--job*` options to EasyBuild
* GC3Pie configuration with details on what queue should be used and what the limits are.

We have provided various GC3Pie configurations under `/apps/easybuild/config/gc3pie`, and their filenames indicate their function:

* `/apps/easybuild/config/gc3pie/<QUEUE_NAME>-<ROLE>.conf`
* `QUEUE_NAME` can be one of the following: `sfx_q`, `sfxsmp_q`, `sandybridge_q`
* `ROLE` is either `setup` for regular users, or `hpcadmin` for system administrators

Typical options used with EasyBuild are:

* `--job` - enable submitting jobs to a backend
* `--job-backend=` - specify what type of backend should be used. We're likely to ever use `GC3Pie`
* `--job-backend-config=` - path to a file containing configuration for the backend. VBI provides working examples for major queues on shadowfax
* `--job-cores=N` - number of cores that should be requested for each individual job. It __cannot__ exceed number of cores available on a single node in a given queue. Recommended values for the following queues are: sfx_q: `12`, sfxsmp_q: `40`, sandybridge_q: `16`

	
## Examples

All of the examples below assume that your environment is ready to use EasyBuild, including loading appropriate modules and/or setting effective group and umask.

### Individual users

* Build `sed-4.2.2-goolf-1.4.10.eb` using `sfx_q` queue:

```
eb sed-4.2.2-goolf-1.4.10.eb --robot --job --job-cores=12 --job-backend=GC3Pie --job-backend-config=/apps/easybuild/config/gc3pie/sfx_q-setup.sh
```



### HPC System Administrators

* Build `OpenMPI-1.8.4-GCC-4.9.2.eb` using `sandybridge_q` queue:

```
eb OpenMPI-1.8.4-GCC-4.9.2.eb --robot --job --job-cores=16 --job-backend=GC3Pie --job-backend-config=/apps/easybuild/config/gc3pie/sfx_q-hpcadmin.sh
```


## GC3Pie configuration details

In the configuration files for GC3Pie we have to specify __prologue__ option, which points to a file that will be prepended to the generated qsub script. We use it primarily for loading all the necessary modules to run EasyBuild and GC3Pie.

### Sample configurations

* running jobs via sfx_q queue:

```
# torque/PBS options
[resource/pbs]
enabled = yes
type = pbs
queue = sfx_q

# use specific group to access the queue.
# note: all resulting files will be owned by that group
qsub = qsub -W group_list=sfx

# prologue script, used to load modules for easybuild and GC3Pie
prologue = /apps/easybuild/config/gc3pie/prologue/shadowfax-setup.sh

# use settings below when running GC3Pie on a cluster login node
frontend = localhost
transport = local
auth = none

# set limits
max_walltime = 2 days
# maximum number of jobs is roughly max_cores / max_cores_per_job
max_cores_per_job = 12
max_cores = 120
max_memory_per_core = 4 GiB
architecture = x86_64
```
* corresponding prologue script

```
# to run easybuild jobs via gc3pie and torque, we need to load appropriate modules

. /etc/profile.d/modules.sh

module load site/shadowfax/easybuild/setup
module load EasyBuild
module load GC3Pie
```

# Sample runs

* Installing a metric ton of things:

```
~> eb hwloc-1.6.2-GCC-4.7.2.eb FFTW-3.3.3-gompi-1.4.10.eb  GCC-4.6.0.eb GCC-4.9.2.eb sed-4.2.2-goolf-1.4.10.eb libtool-2.4.2-GCC-4.9.2.eb M4-1.4.17-GCC-4.9.2.eb Autoconf-2.69-GCC-4.9.2.eb Automake-1.15-GCC-4.9.2.eb numactl-2.0.10-GCC-4.9.2.eb hwloc-1.10.0-GCC-4.9.2.eb OpenMPI-1.8.4-GCC-4.9.2.eb --robot --job  --job-backend=GC3Pie --job-backend-config=/apps/easybuild/config/gc3pie/sfx_q-setup.conf --job-cores=12 --job-polling-interval=60

== temporary log file in case of crash /tmp/eb-Jytyku/easybuild-_5d4XM.log
== resolving dependencies ...
== GC3Pie job overview: 5 submitted (total: 12)
== GC3Pie job overview: 2 terminated, 2 failed, 3 running (total: 12)
== GC3Pie job overview: 2 terminated, 2 failed, 3 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 3 terminated, 1 ok, 2 failed, 2 running (total: 12)
== GC3Pie job overview: 4 terminated, 2 ok, 2 failed, 1 running (total: 12)
== GC3Pie job overview: 4 terminated, 2 ok, 2 failed, 1 running (total: 12)
== GC3Pie job overview: 4 terminated, 2 ok, 2 failed, 1 running (total: 12)
== GC3Pie job overview: 4 terminated, 2 ok, 2 failed, 1 running (total: 12)
== GC3Pie job overview: 4 terminated, 2 ok, 2 failed, 1 running (total: 12)
== GC3Pie job overview: 4 terminated, 2 ok, 2 failed, 1 running (total: 12)
== GC3Pie job overview: 5 terminated, 3 ok, 2 failed (total: 12)
== GC3Pie job overview: 5 terminated, 3 ok, 2 submitted, 2 failed (total: 12)
== GC3Pie job overview: 7 terminated, 4 ok, 3 failed (total: 12)
== GC3Pie job overview: 7 terminated, 4 ok, 1 submitted, 3 failed (total: 12)
== GC3Pie job overview: 8 terminated, 5 ok, 3 failed (total: 12)
== GC3Pie job overview: 8 terminated, 5 ok, 1 submitted, 3 failed (total: 12)
== GC3Pie job overview: 9 terminated, 6 ok, 3 failed (total: 12)
== GC3Pie job overview: 9 terminated, 6 ok, 1 submitted, 3 failed (total: 12)
== GC3Pie job overview: 10 terminated, 7 ok, 3 failed (total: 12)
== GC3Pie job overview: 10 terminated, 7 ok, 1 submitted, 3 failed (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 1 submitted, 4 failed (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed, 1 running (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed, 1 running (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed, 1 running (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed, 1 running (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed, 1 running (total: 12)
== GC3Pie job overview: 11 terminated, 7 ok, 4 failed, 1 running (total: 12)
== GC3Pie job overview: 12 terminated, 8 ok, 4 failed (total: 12)
== GC3Pie job overview: 12 terminated, 8 ok, 4 failed (total: 12)
== Done processing jobs
== GC3Pie job overview: 12 terminated, 8 ok, 4 failed (total: 12)
== Submitted parallel build jobs, exiting now
== temporary log file(s) /tmp/eb-Jytyku/easybuild-_5d4XM.log* have been removed.
== temporary directory /tmp/eb-Jytyku has been removed. 
```
