---
title: Hello, Perlmutter
layout: main
---

Due 2026-01-29.

The goal of this homework is to get you up and running on Perlmutter.

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

## Programming models

The course introduces MPI and OpenMP, the [recommended programming model on Perlmutter][mpi-openmp], starting with MPI. The [default MPI][default-mpi] on Perlmutter is the Cray MPI implementation.

[mpi-openmp]: https://docs.nersc.gov/development/programming-models/
[default-mpi]: https://docs.nersc.gov/development/programming-models/mpi/

## Running jobs

NERSC uses a [job queue system based on Slurm][jobs]. Users submit
jobs to a queue (using `sbatch`) with some set of parameters
describing resources that will be used.  The script that is submitted
includes additional parameters, shell commands to set up the
appropriate environment, and shell commands to run whatever is to be
run. There is an [example submission script](hello-submit) in the
class [demos][demos] (to be released on 2026-01-20) repository.

It sometimes takes a bit for things to run, and it is not so easy to
tell whether a code is taking a while because it has not been
scheduled or because something has gone wrong.  You can use the `sqs`
and `squeue` [monitoring tools][monitor] to check on the status of a
running job.

It is considered **bad form** to run anything computationally intensive on
the **login nodes**. Please do not to that.

[jobs]: https://docs.nersc.gov/jobs/
[monitor]: https://docs.nersc.gov/jobs/monitoring/
[hello-submit]: https://github.com/cs5220-26sp/demos/blob/master/hello-mpi/hello-submit.sh
[demos]: https://github.com/cs5220-26sp/demos

## Your tasks (due 2026-01-29):

- Create a NERSC account. It usually takes 2 to 5 business days to be vetted and approved, so do this right away.
- Run the MPI ping-pong example from the [demos][demos] (to be released on 2026-01-20) subdirectory and submit
  your timings from on Perlmutter on [Canvas](https://canvas.cornell.edu/courses/85162). 

[demos]: https://github.com/cs5220-26sp/demos
