---
title: HW3 - Distributed Memory Particle Simulation
layout: main
---

# Overview

This assignment is an introduction to parallel programming using a distributed memory model. Most of this page will be similar to the HW2 page.

In this assignment, we will be parallelizing a toy particle simulation [(similar simulations are used in mechanics, biology, and astronomy)](http://www.google.com/url?q=http%3A%2F%2Fwww.thp.uni-duisburg.de%2F~kai%2Findex_1.html&sa=D&sntz=1&usg=AOvVaw1_W7EMmAz6PnARXZNyR_dA). In our simulation, particles interact by repelling one another. 

# Asymptotic Complexity
## Serial Solution Time Complexity

If we were to naively compute the forces on the particles by iterating through every pair of particles, then we would expect the asymptotic complexity of our simulation to be O(n^2).

However, in our simulation, we have chosen a density of particles sufficiently low so that with n particles, we expect only O(n) interactions. An efficient implementation can reach this time complexity. 

## Parallel Speedup

Suppose we have a code that runs in time T = O(n) on a single processor. Then we'd hope to run close to time T/p when using p processors. After implementing an efficient serial O(n) solution, you will attempt to reach this speedup using MPI.

Important: You CANNOT use OpenMP for this assignment. All of your parallel speedup must come from distributed memory parallelism (MPI).


## Due Date: Thursday March 12th 2026 at 11:59 PM
# Instructions
## Teams

Note that you will work individually for this assignment. 

## Getting Set Up


The [starter code is available in the course Github repo](https://github.com/cs5220-26sp/hw3/tree/master) and should work out of the box. To get started, we recommend you log in to Perlmutter and download the first part of the assignment. This will look something like the following:
```
student@local:~> ssh perlmutter-p1.nersc.gov

student@login04:~> git clone git@github.com:your-repo.git

student@login04:~> cd hw3

student@login04:~/hw3> ls

CMakeLists.txt common.h  job-mpi  main.cpp  mpi.cpp job-leaderboard submit.py
```
There are five files in the base repository. Their purposes are as follows:

- `CMakeLists.txt`: 

The build system that manages compiling your code.

- `main.cpp`: 

     A driver program that runs your code.

- `common.h`: 

A header file with shared declarations

- `job-mpi`: 

A sample job script to run the MPI executable

- `mpi.cpp` --- You may modify this file:  

A skeleton file where you will implement your mpi simulation algorithm. It is your job to write an algorithm within the simulate_one_step and gather_for_save functions.

- `job-leaderboard`:

Job script that will benchmark your implementation comprehensively and produce output appropriate for submission to the leaderboard.

- `submit.py`:

Script that will submit the output of `job-leaderboard` to the online leaderboard for this assignment.

- `verf/`:

A directory containing correct serial and parallel outputs that is used by `job-leaderboard` to verify correctness.

- `check-correctness.py`:

Script that can be used to check correctness the same way as was done in HW2. 

Please do not modify any of the files besides mpi.cpp.

## Building our Code

First, we need to make sure that the CMake module is loaded.
```
student@login04:~/hw3> module load cmake
```
You should put the above command in your ~/.bash_profile file to avoid typing them every time you log in.

Next, let's build the code. CMake prefers out of tree builds, so we start by creating a build directory.
```
student@login04:~/hw3> mkdir build

student@login04:~/hw3> cd build

student@login04:~/hw3/build>
```
Next, we have to configure our build. We can either build our code in Debug mode or Release mode. In debug mode, optimizations are disabled and debug symbols are embedded in the binary for easier debugging with GDB. In release mode, optimizations are enabled, and debug symbols are omitted. For example:
```
student@login04:~/hw3/build> cmake -DCMAKE_BUILD_TYPE=Release ..

-- The C compiler identification is GNU 14.3.0

...

-- Configuring done

-- Generating done

-- Build files have been written to: /global/homes/s/student/hw3/build
```
Once our build is configured, we may actually execute the build:
```
student@login04:~/hw3/build> make

Scanning dependencies of target mpi
[ 33%] Building CXX object CMakeFiles/mpi.dir/main.cpp.o
[ 66%] Building CXX object CMakeFiles/mpi.dir/mpi.cpp.o
[100%] Linking CXX executable mpi
[100%] Built target mpi

student@login04:~/hw3/build> ls

CMakeCache.txt CMakeFiles  cmake_install.cmake  Makefile  mpi job-mpi submit.py job-leaderboard
```
We now have a binary (mpi) and a job script (job-mpi).

## Running the Program

You will need to test on at most two nodes for this assignment. To allocate two interactive Permutter CPU nodes instead of just one (as we did in previous assignments), the syntax is simple:
```
student@login04:~/hw3> salloc --nodes 2 --qos interactive --time 01:00:00 --constraint cpu --account=m4341

salloc: Granted job allocation 53324632

salloc: Waiting for resource configuration

salloc: Nodes nid02346 are ready for job

student@login04:~/hw3> cd build

student@login04:~/hw3/build> 
```


You now have a shell into one of the two allocated nodes. We recommend that you allocate only a single node and test on multiple MPI ranks with that node until you are ready to conduct a full scaling benchmark.

Unlike earlier assignments, you cannot directly run the executable from the command prompt! You must use srun or sbatch with the sample jobscript that we provide you. You can modify the jobscript to benchmark different runtime configurations.

If you choose to run the binary using srun within an interactive session, you should set any environment variables in your interactive session to match the environment variables set by the jobscript. After you have done so, here's how to run the simulation for two nodes, 1.5 million particles, and 32 MPI ranks per node, for a total of 64 total MPI ranks:
```
student@login04:~/hw3/build> srun -N 2 --ntasks-per-node=64 ./mpi -n 1500000 -s 1

Simulation Time = 1.92538 seconds for 1500000 particles.
```
To test on only a single node with 32 total MPI ranks, you can run:
```
student@login04:~/hw3/build> .srun -N 1 --ntasks-per-node=32 ./mpi -n 1500000 -s 1

Simulation Time = 11.3875 seconds for 1500000 particles.
```
Before you try writing any parallel code, you should make sure that you have a correct serial implementation. To benchmark the program in a serial configuration, run on a single node with --ntasks-per-node=1. 

# Grading

Your score out of 100 points will be broken down into the following categories

- 20/100 points: Writeup

- 10/100 points: Checkpoint

- 5/100 points: Leaderboard Submission

- 65/100 points: Final Performance

Your score on the "final performance" part is broken up into a serial portion and a parallel portion. The following metrics will be used

1. RS: serial runtime at 1e5 particles. Note that 'serial runtime' in this case is the runtime of your implementation in `mpi.cpp` on a single process, not the runtime of a separate serial implementation as in HW2.

2. PE1: Parallel efficiency on 2e6 particles when going from 64 processes on 1 node to 128 processes across 2 nodes. 

3. PE2: Parallel efficiency on 2e6 particles when going from 128 processes acrosss 2 nodes to 256 processes across 2 nodes. 

If `RS <= 9s`, i.e. your serial runtime is less than or equal to 9 seconds, then you will get 10/65 points. 

Then, your overall parallel performance metric is computed as `ParPerf = PE1 * 0.5 + PE2 * 0.5`

This metric will determine your score on the remaining 55 points of this assignment. Unlike previous assignments, we will not be using predetermined grade bins in order to give you a letter grade for these 55 points. The reason for this is that we usually set grade bins based on historical performance data for this class. However, with the rise and spread of GenAI, we have found historical data to no longer be informative for setting bins. Additionally, updates to Perlmutter over time make historical performance data less informative. Therefore, we will be determining your grade on these 55 points based on a curve that will be shaped based on the performance of everyone in the class. 

To add additional clarity to this grading process, we are introducing a **live leaderboard** that students can submit their performance metrics to over the course of the assignment. See below for details on submission, but this leaderboard will give you a sense of how people in the class are doing on average, and on where you might stand on the curve. 

A few more points to emphasize:

1. We will not curve your grade to be below a B-

2. 45/100 points are not determined by a curve

3. It is possible for everyone to get an A if all submissions perform similarly well.

## Checkpoint

The checkpoint for this assignment requires your implementation to achieve a runtime of at least 9 seconds on 1e5 particles when run on a single process. Note that this is identical to what you must do in order to get the 10 points based on the `RS` metric from above. 

### The checkpoint is due on March 5th, 2026 at 11:59 PM ET.

## Leaderboard Submission Details

The leaderboard can be found [here](https://leaderboard-zm07.onrender.com/) To submit to the leaderboard, follow these steps.

1. Submit `job-leaderboard` with `sbatch job-leaderboard`. This will perform a suite of correctness checks, and it will run a set of benchmarks that will determine `RS`, `PE1`, `PE2`, and `ParPerf`.

2. Wait for the job to run -- when done, it will produce an output file called `leaderboard-output.out`.

3. Run `python3 submit.py leaderboard-output.out` -- this will submit your results to the leaderboard. It will also generate a unique username that will be tied to your submissions and used to identify them on the leaderboard, for example `pink-unicorn-448`.

**We ask everyone to submit to the leaderboard at least once between March 6th and March 11th. 5/100 points will be associated with submitting to the leaderboard once.** Once you have submitted to the leaderboard, upload the `leaderboard-output.out` file you submitted to the HW3 (Leaderboard) assignment on Canvas. 

**Please do not submit fraudulent output files to the leaderboard. Only submit files that are produced by an un-modified version of `job-leaderboard`**. Ultimately, we will determine both your grade and the final curve based on the actual performance of everyone's code when run with the autograder, **not** based on what the leaderboard says. We will treat fake leaderboard submissions as an AI violation. We trust everyone to be good HPC citizens and submit genuine output files. 



It is also possible that there may be bugs in the leaderboard. If you find anything you suspect is a bug, please make an Ed post about it.

## Submission Details (Similar to HW1)

1. Make sure you have our most updated source code on Perlmutter.

2. Make sure you have only modified the file mpi.cpp, and it compiles and runs as desired.

3. Ensure that your write-up pdf is located in your source directory, next to mpi.cpp. It should be named CS5220<GROUPID>_hw3.pdf. 

4. From your build directory, run:
```
student@perlmutter:~/hw3/build> cmake -DGROUP_NO=004..
student@perlmutter:~/hw3/build> make package
```
This second command will fail if the PDF is not present.

6. Confirm that it worked using the following command. You should see output like:
```
student@perlmutter:~/hw3/build> tar tfz CS5220Group004_hw3.tar.gz
CS5220abc123_hw3/CS5220Group004_hw3.pdf
CS5220abc123_hw3/mpi.cpp
```
7. Download and submit your .tar.gz through canvas. 

## Writeup Details

Your write-up should contain:

- your name, cornell id (NetID), and perlmutter username,

- A strong scaling plot in log-log scale that shows the performance of your code on 1e6 and 2e6 particles using 64 processes on 1 nodes, 128 processes across 2 nodes with 64 processes per node, and 256 processes across 2 nodes with 128 processes per node. Include a line for ideal scaling as well.

- A short caption describing the plot

- GenAI and collaboration disclosure, similar to the other Homeworks.

## Notes:

- You must use GCC 14.3.0 (loaded by default on Perlmutter) for this assignment. If your code does not compile and run with GCC, it will not be graded.

- If your code produces incorrect results, it will not be graded.

**For info on running the rendering output and checking output correctness, please refer to the HW2 page.**

## Resources

- Programming in shared and distributed memory models are introduced in Lectures.

- Shared memory implementations may require using locks that are available as [omp_lock_t in OpenMP](http://www.google.com/url?q=http%3A%2F%2Fmsdn.microsoft.com%2Fen-us%2Flibrary%2F7d2zxc0s%28VS.80%29.aspx&sa=D&sntz=1&usg=AOvVaw0FzdPSC5-yQ4862oG6S1Vz) (requires omp.h)

- You may consider using atomic operations such as [__sync_lock_test_and_set](http://www.google.com/url?q=http%3A%2F%2Fgcc.gnu.org%2Fonlinedocs%2Fgcc-4.1.2%2Fgcc%2FAtomic-Builtins.html%23Atomic-Builtins&sa=D&sntz=1&usg=AOvVaw01sijj3wHUZmEMbKYsbmcJ) with the GNU compiler.

- Other useful resources: [pthreads tutorial](https://www.google.com/url?q=https%3A%2F%2Fcomputing.llnl.gov%2Ftutorials%2Fpthreads%2F&sa=D&sntz=1&usg=AOvVaw3QoLimlO4FTaFHEg8b59dc), [OpenMP tutorial](http://www.google.com/url?q=http%3A%2F%2Fopenmp.org%2Fwp%2F2008%2F11%2Fsc08-openmp-hands-on-tutorial-available%2F&sa=D&sntz=1&usg=AOvVaw1JtwWUv9otfdAKLwx5eGNo), [OpenMP specifications](http://www.google.com/url?q=http%3A%2F%2Fopenmp.org%2Fwp%2Fopenmp-specifications%2F&sa=D&sntz=1&usg=AOvVaw00kqf5nBOOzEvsHFaS50sk) and [MPI specifications](http://www.google.com/url?q=http%3A%2F%2Fwww.mpi-forum.org%2Fdocs%2Fdocs.html&sa=D&sntz=1&usg=AOvVaw305_KDOE9jscT6h7pRhmVQ).

- It can be very useful to use a performance measuring tool in this homework. Parallel profiling is a complicated business but there are a couple of tools that can help.

- [TAU (Tuning and Analysis Utilities)](http://www.google.com/url?q=http%3A%2F%2Fwww.cs.uoregon.edu%2FResearch%2Ftau%2Fhome.php&sa=D&sntz=1&usg=AOvVaw0o6qUyHzZxCk_yju-cvxtc) is a source code instrumentation system to gather profiling information. You need to "module load tau" to access these capabilities. This system can profile MPI, OpenMP and PThread code, and mixtures, but it has a learning curve.

- [HPCToolkit](http://www.google.com/url?q=http%3A%2F%2Fhpctoolkit.org%2F&sa=D&sntz=1&usg=AOvVaw1tkBg8bxr4VlRie20d9yFL) Is a sampling profiler for parallel programs. You need to "module load hpctoolkit". You can install the hpcviewer on your own computer for offline analysis, or use the one on NERSC by using the [NX client](https://www.google.com/url?q=https%3A%2F%2Fwww.nersc.gov%2Fusers%2Fnetwork-connections%2Fnersc-nx-service-x-windows-acceleration-at-nersc%2F&sa=D&sntz=1&usg=AOvVaw1oU9inA67MNeqsdZ5v8ksn) to get X windows displayed back to your own machine.

- If you are using TAU or HPCToolkit you should run in your $SCRATCH directory which has faster disk access to the compute nodes (profilers can generate big profile files).
