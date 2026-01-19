---
title: Hello, Perlmutter
layout: main
---

Due 2026-01-29.

The goal of this homework is to get you up and running on Perlmutter.

# Perlmutter Basics

## Create a NERSC account

Please create an account on [NERSC Perlmutter][nersc] on 2026-01-21 (you cannot do this before that date). It usually takes **2 to 5 business days** to be vetted and approved, so apply as soon as possible, as extensions will not be granted if you delay your application. 

Please see [Ed][ed] for the information needed to create an account. For privacy reasons, this data will not be posted on the public website; apologies for the extra step.

<!-- PI: Giulia Guidi
Repository: m4341
Reason or justification: CS 5220 -->

["Hello, Perlmutter"][hw0] is quick but requires Perlmutter access. Please see the [syllabus](https://www.cs.cornell.edu/courses/cs5220/2026sp/syllabus.html) for the late enrollment policy.

[nersc]: https://iris.nersc.gov/add-user
[ed]: https://edstem.org/us/courses/92417/discussion
[hw0]: https://www.cs.cornell.edu/courses/cs5220/2026sp/hw/hw0.html

## Logging into Perlmutter

The NERSC documentation has information about [how to
connect][connect] to Perlmutter.  The short version is "use ssh",
but I strongly recommend using ssh with the [sshproxy][sshproxy]
setup to avoid typing your password (and one-time password)
repeatedly.  With the proxy set up, you should only need to enter
your password (including one-time password) once every 24 hours.

[connect]: https://docs.nersc.gov/connect/
[sshproxy]: https://docs.nersc.gov/connect/mfa/#sshproxy

## Loading modules

When you log into Perlmutter, you start off with a fairly minimal set
of tools available in the standard path.  A variety of compilers and
other tools are all available, but you will need to learn a little
about [environment modules][lmod] to use them.

On Perlmutter there are standard [compiler wrappers][wrappers] for
building parallel codes.  These compiler wrappers include all the
requisite compiler and linker directives to make sure that things are
set up.  Make sure that you use the compiler wrappers (typically
called `cc`, `CC`, and `ftn` for C, C++, and Fortran) when you compile
in order to get this convenience.  The compilers are also set up so
that they target the appropriate behavior for the compute nodes.

[lmod]: https://docs.nersc.gov/environment/lmod/
[wrappers]: https://docs.nersc.gov/development/compilers/wrappers/


## Running jobs

NERSC uses a [job queue system based on Slurm][jobs]. Users submit
jobs to a queue (using `sbatch`) with some set of parameters
describing resources that will be used.  The script that is submitted
includes additional parameters, shell commands to set up the
appropriate environment, and shell commands to run whatever is to be
run. There is an [example submission script][hello-submit] in the
class [demos][demos] (to be released on 2026-01-20) repository.

It sometimes takes a bit for things to run, and it is not so easy to
tell whether a code is taking a while because it has not been
scheduled or because something has gone wrong.  You can use the `sqs`
and `squeue` [monitoring tools][monitor] to check on the status of a
running job.

For running smaller workloads, it is also possible to request a so-called [interactive job](https://docs.nersc.gov/jobs/interactive/) that uses between 1-4 compute nodes.
This is done using the `salloc` command.
An interactive job gives you live, uninterrupted access to the requested compute nodes, meaning you can run commands on them from your terminal the same way you would on your laptop.
This is very useful for debugging programs at smaller scale, since you do not have to wait for your job to make its way through the queue. 

It is considered **bad form** to run anything computationally intensive on
the **login nodes**. Please do not do that.

One final note -- when compiling code, it is generally recommended to use the login nodes, since code will compile much faster than on the compute nodes.

[jobs]: https://docs.nersc.gov/jobs/
[monitor]: https://docs.nersc.gov/jobs/monitoring/
[hello-submit]: https://github.com/cs5220-26sp/demos/blob/master/hello-mpi/hello-submit.sh
[demos]: https://github.com/cs5220-26sp/demos

## Your tasks (due 2026-01-29):

- Create a NERSC account. It usually takes 2 to 5 business days to be vetted and approved, so do this right away.
- Run the MPI ping-pong example from the [demos][demos] repo (to be released on 2026-01-20) subdirectory and submit
  your timings from on Perlmutter on [Canvas](https://canvas.cornell.edu/courses/85162). 

### Ping-Pong Example

Here is some more information on the ping-pong part of the assignment:

The ping-pong program is a small microbenchmark that sends buffers of various sizes back and forth between two nodes of Perlmutter using MPI.
For each buffer size, the time needed to perform the exchange is noted.
Once all of the buffer sizes have been tested, the timing data is used to fit the [Alpha-beta Model](https://downey.io/notes/omscs/cse6220/distributed-memory-model-mpi-collectives/) to Perlmutter's network architecture. 
The Alpha-beta model is a simple two-term model used to reason about how long it takes to communicate data between nodes on an HPC cluster. 
You do not need to understand the details of the Alpha-beta model to complete this assignment, but if you are curious, you can read more about it at the link supplied above.

The `ping-pong` directory in the [demos][demos] repo contains everything you'll need for this assignment. In particular, it contains the following files:

- `Makefile` -- used for building the ping pong executable 
- `ping-plotter.py` -- used to plot the raw timing data emitted by the ping pong program and fit the Alpha-beta model to the data. Instructions on how to run this script can be found below.
- `ping-submit.sh` -- jobscript used to submit a batch job to SLURM that will run the ping pong program and write the timing results to a file called `perlmutter.txt`
- `ping.c` -- contains the actual ping pong program

To compile the ping pong program, run `make` in the `ping-pong` directory.

To submit a job that will run the program, run `sbatch ping-submit.sh`. To check on the status of the job, type `sqs`.

Once your job has run, you can plot the results by first loading the `python` module via `module load python`, then running `python3 ping-plotter.py perlmutter`.
This will create a plot named `perlmutter.svg`.

To complete this assignment, you should submit a `.gz` file on [Canvas](https://canvas.cornell.edu/courses/85162) containing the following

- `perlmutter.txt` -- produced by `ping-submit.sh`
- `perlmutter.svg` -- produced by `ping-plotter.py`



[demos]: https://github.com/cs5220-26sp/demos
