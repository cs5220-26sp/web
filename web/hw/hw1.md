---
title: HW1 - Optimizing Matrix Multiplication
layout: main
---

## Problem Statement

Your task in this assignment is to write an optimized matrix multiplication function for NERSC's Perlmutter supercomputer. We will give you a generic matrix multiplication code (also called matmul or dgemm), and it will be your job to tune our code to run efficiently on the processors. We are asking you to write an optimized single-threaded matrix multiply kernel. This will run on only one core.

We consider a special case of matmul:

$$C = C + A*B$$

where $A$, $B$, and $C$ are n x n matrices. This can be performed using 2n^3 floating point operations (n^3 adds, n^3 multiplies), as in the following pseudocode:

```
for i = 1 to n
    for j = 1 to n
        for k = 1 to n
            C(i,j) = C(i,j) + A(i,k) * B(k,j)
        end
    end
end
```




## Due Date

Checkpoint: February 5th, 11:59 PM

Final submission: February 12th, 11:59 PM


## Getting Started with Perlmutter

Please read through the Perlmutter tutorial, available on the [demos repo](https://github.com/cs5220-26sp/demos).

## Getting Set Up

The starter code is available [on Github](https://github.com/cs5220-26sp/hw1/tree/main) and should work out of the box.

To use SSH to connect to GitHub, [follow these instructions](https://help.github.com/en/github/using-git/which-remote-url-should-i-use#cloning-with-ssh-urls).

Every student must create their own forked, PRIVATE GitHub repo to track the changes. One can find [simple instructions here](https://junyonglee.me/github/How-to-make-forked-private-repository/) on how to make a private repo that can pull new features from the course public repo.

The staff members must be given access to every student repository. The staff usernames are: giuliaguidi, Hooninator, anishbh, and CtrlAltGiri
. When you create your private repo, in the repo settings, find category "Collaborators" and add both staff members.

To get started, we recommend you log in to Perlmutter and download the first part of the assignment. This will look something like the following:

```
student@local:~> ssh perlmutter-p1.nersc.gov

student@login04:~> git clone git@github.com:your-repo.git

student@login04:~> cd hw1

student@login04:~/hw1> ls

CMakeLists.txt README.md benchmark.c dgemm-blas.c dgemm-blocked.c dgemm-naive.c job.in
```


There are seven files in the base repository. Their purposes are as follows:

- `CMakeLists.txt`: The build system that manages to compile your code.

- `README.md`: README file explaining the build system in more detail.

- `benchmark.c`: A driver program that runs your code.

- `dgemm-blas.c`: A wrapper that calls the vendor's optimized BLAS implementation of matrix multiply (here, MKL).

- `dgemm-blocked.c`: You may only modify this file. A simple blocked implementation of matrix multiply. It is your job to optimize the square_dgemm() function in this file.

- `dgemm-naive.c`: illustrative purposes, a naive implementation of matrix multiply using three nested loops.

- `job.in`: Template job script that is filled in by the build system for each of blas, blocked, and naive. Please do not modify any of the files besides dgemm-blocked.c.

## Building our Code

First, we need to make sure that the CMake module is loaded.

```
student@login04:~/hw1> module load cmake
```

You should put these commands in your `~/.bash_profile` file to avoid typing them every time you log in.

Next, let's build the code. CMake prefers out-of-tree builds, so we start by creating a build directory.

```
student@login04:~/hw1> mkdir build

student@login04:~/hw1> cd build

student@login04:~/hw1/build>
```


Next, we have to configure our build. We can either build our code in Debug mode or Release mode. In debug mode, optimizations are disabled and debug symbols are embedded in the binary for easier debugging with GDB. In release mode, optimizations are enabled, and debug symbols are omitted. For example:

```
student@login04:~/hw1/build> cmake -DCMAKE_BUILD_TYPE=Release ..

-- The C compiler identification is GNU 13.2.1

...

-- Configuring done

-- Generating done

-- Build files have been written to: /global/homes/s/student/hw1/build
```


Once our build is configured, we may actually execute the build:

```
student@perlmutter:~/hw1/build> make

Scanning dependencies of target benchmark

[ 14%] Building C object CMakeFiles/benchmark.dir/benchmark.c.o

[ 14%] Built target benchmark

...

[ 85%] Building C object CMakeFiles/benchmark-naive.dir/dgemm-naive.c.o

[100%] Linking C executable benchmark-naive

[100%] Built target benchmark-naive


student@login04:~/hw1/build> ls

benchmark-blas benchmark-naive CMakeFiles job-blas job-naive

benchmark-blocked CMakeCache.txt cmake_install.cmake job-blocked Makefile
```


We now have three binaries (benchmark-blas, benchmark-blocked, and benchmark-naive) and three corresponding job scripts (job-blas, job-blocked, and job-naive). Feel free to create your own job scripts by copying one of these to the above source directory.

## Other build considerations

You might find that your code works in Debug mode, but not Release mode. To add debug symbols to a release build, run:

```
student@perlmutter:~/hw1/build> cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_C_FLAGS="-g3" ..
```


You can add arbitrary extra compiler flags in this way, just remember to re-run make after you do this.

## Running our Code

The easiest way to run the code is to submit a batch job. We've already provided batch files which will launch jobs for each matrix multiply version.

```
student@perlmutter:~/hw1/build> sbatch job-blas

Submitted batch job 27505251

student@perlmutter:~/hw1/build> sbatch job-blocked

Submitted batch job 27505253

student@perlmutter:~/hw1/build> sbatch job-naive

Submitted batch job 27505255
```




Our jobs are now submitted to Perlmutter's job queue. We can now check on the status of our submitted jobs using a few different commands.

```
student@perlmutter:~/hw1/build> squeue -u username

JOBID PARTITION NAME USER ST TIME NODES NODELIST(REASON)

27505255 debug_perlmutter job-naiv student PD 0:00 1 (QOSMaxJobsPerUserLimit)

27505253 debug_perlmutter job-bloc student R 0:32 1 nid00545

27505251 debug_perlmutter job-blas student R 0:39 1 nid12790


student@perlmutter:~/hw1/build> sqs

JOBID ST USER NAME NODES REQUESTED USED SUBMIT QOS ESTIMATED_START FEATURES REASON

27505255 R student job-naive 1 1:00 0:36 2020-01-19T10:56:43 debug_perlmutter 2020-01-19T10:57:25 None

27505253 R student job-blocked 1 1:00 1:19 2020-01-19T10:56:39 debug_perlmutter 2020-01-19T10:56:42 None
```

When our job is finished, we'll find new files in our directory containing the output of our program. For example, we'll find the files `job-blas.o27505253` and `job-blas.e27505253`. The first file contains the standard output of our program, and the second file contains the standard error.

## Interactive Session

You may find it useful to launch an interactive session when developing your code. This lets you compile and run code interactively on a compute node that you've reserved. In addition, running interactively lets you use the special interactive queue, which means you'll receive your allocation quicker. To launch an interactive Perlmutter node for 1 hour and run a benchmark (after you've built your code), try this:

```
student@perlmutter:~/hw1> salloc --nodes 1 --qos interactive --time 01:00:00 --constraint cpu --account=mxxxx

salloc: Granted job allocation 53324632

salloc: Waiting for resource configuration

salloc: Nodes nid02346 are ready for job

student@nid02346:~/hw1> cd build

student@nid02346:~/hw1/build> ./benchmark-blocked

```

For a quick way to request an interactive job with 1 CPU node, add the following to your `.bash_profile`

```
function cpu() {
    salloc --nodes $1 --qos interactive --time 01:00:00 --constraint cpu --account=m4341
}
```
then run

```
student@perlmutter:~/hw1> cpu 1
```
and you'll be dropped into an interactive job with one CPU node.


Interactive nodes may not always be available, depending on system demand. Be aware that compiling your code is much slower on an interactive node compared to a login node.

## Editing the Code

One of the easiest ways to implement your homework is to directly change the code on Perlmutter. For this you need to use a command line editor like nano or vim.

```
student@perlmutter:~/hw1> vim dgemm-blocked.c
```


Use Esc and :q to exit. (:q! if you want to discard changes). 

Alternatively, VScode allows remote development using the [ssh extension](https://code.visualstudio.com/docs/remote/ssh), and other popular IDEs also support remote development through similar extensions.


## Our Harness

The `benchmark.c` file generates matrices of a number of different sizes and benchmarks the performance. It outputs the performance in FLOPS and in a percentage of theoretical peak attained. Your job is to get your matrix-multiply's performance as close to the theoretical peak as possible.

On Perlmutter, theoretical peak is computed as 3.5 GHz * 4 vector width * 2 vector pipelines * 2 flops for FMA = 56 GF/s.

You may find a more detailed description of Perlmutter's architecture at https://docs.nersc.gov/systems/perlmutter/architecture/.

## Optimizing


Now, it's time to optimize! A few optimizations you might consider adding:

- Perform blocking. The dgemm-blocked.c already gets you started with this, although you'll need to tune block sizes.
- Write a register-blocked kernel, either by writing an inner-level fixed-size matrix multiply and hoping (and maybe [checking](https://www.google.com/url?q=https%3A%2F%2Fgodbolt.org%2F&sa=D&sntz=1&usg=AOvVaw20IJKua-JEkYL-xXtBiMD9).) that the compiler inlines it, writing [AVX](https://www.google.com/url?q=https%3A%2F%2Fsoftware.intel.com%2Fsites%2Flandingpage%2FIntrinsicsGuide%2F&sa=D&sntz=1&usg=AOvVaw1WiGo2NuYSKanuXSaqLOUq) intrinsics, or even writing inline assembly instructions.
- Add manual prefetching.

You may, of course, proceed however you wish. We recommend you look through the lecture notes as reference material to guide your optimization process. 
There are also some papers in the next section of this write-up that provide detailed information on optimizing matrix multiply. 

The development environment on Perlmutter supplies several different compilers, including Cray. However, we want you to use the GNU C compiler for this assignment.

Still, you might want to try your code with different compilers to see if one outperforms the other. If the difference is significant, consider using the [Compiler Explorer](https://www.google.com/url?q=https%3A%2F%2Fgcc.godbolt.org%2Fz%2Fv2fTDJ&sa=D&sntz=1&usg=AOvVaw372oypTwxKaKj1tbrn_Hke) to figure out why GCC isn't optimizing your code as well. For more information on the compilers available, see the [NERSC docs](https://docs.nersc.gov/systems/perlmutter/software/).


## References

- **Goto, K., and van de Geijn, R. A. 2008. Anatomy of High-Performance Matrix Multiplication, ACM Transactions on Mathematical Software 34, 3, Article 12**: This is a very detailed description of several different blocking strategies, some of which are less effective than others (the paper explains which are likely to be the best and why).
This paper also gives a thorough description of the intuition behind why blocking works, and it gives some advice on how to tune block sizes. 
There are also a few advanced optimizations described here, such as packing and introducing multiple levels of blocking.
Not much is said here on how to write the microkernel.

- **Low, T. M., Igual, F. D., Smith, T. M., & Quintana-Orti, E. S. (2016). Analytical modeling is enough for high-performance BLIS. ACM Transactions on Mathematical Software (TOMS), 43(2), 1-18.**: This gives a more detailed analytical model that can be used to tune block sizes and register block sizes for the microkernel.

- **Lim, R., Lee, Y., Kim, R., & Choi, J. (2018). An implementation of matrix–matrix multiplication on the Intel KNL processor with AVX-512. Cluster Computing, 21(4), 1785-1795**: Digestible shorter paper about optimizing GEMM. Particularly nice for its description of how to implement the microkernel using vector intrinsics. Note that it describes an implementation of the microkernel using AVX512 instructions, which are different from the AVX2 instructions used on Perlmutter's CPUs. However, much of what is said here will be helpful.


- **Chellappa, S., Franchetti, F., and Puschel, M. 2008. How To Write Fast Numerical Code: A Small Introduction, Lecture Notes in Computer Science 5235, 196-259**.

- **Bilmes, et al. The PHiPAC (Portable High Performance ANSI C) Page for BLAS3 Compatible Fast Matrix Matrix Multiply**.

- **Lam, M. S., Rothberg, E. E, and Wolf, M. E. 1991. The Cache Performance and Optimization of Blocked Algorithms, ASPLOS'91, 63-74**.




## Available Compilers

The development environment on Perlmutter supplies several different compilers, including Cray. However, we want you to use the GNU C compiler for this assignment.

Still, you might want to try your code with different compilers to see if one outperforms the other. If the difference is significant, consider using the Compiler Explorer to figure out why GCC isn't optimizing your code as well. For more information on the compilers available, see the NERSC docs.

## Grading

We will grade your assignment by reviewing your report and benchmarking your code's performance. To benchmark your code, we will compile it with the exact process detailed above, with the GNU compiler. Note that code that does not return correct results will receive significant penalties.

The benchmark performance of your code will be worth 70/100 points and your report will receive a grade out of 20/100 points. The other 10/100 points will come from the checkpoint assignment (see below).


### Checkpoint

In order to encourage you to start working on the assignment early and not leave things to the last minute, there is a **checkpoint** for this homework assignment that is due on February 5th at 11:59 PM EST.
For the checkpoint, you need to demonstrate some performance improvement over the starter code.

**For the checkpoint, your code must run at >= 10% of peak**.

For reference, the starter code in `dgemm-blocked.c` will get around 6% of peak, and the naive approach in `dgemm-naive.c` will get around 4% of peak.

The checkpoint is worth 10/100 total points, and you will get all 10 points if you hit the required performance threshold. 


### HW1 target average performance of your code across multiple runs

These are the average percentages of peak you must get in order to attain different letter grades. Note that you should compile your code with the `-DALL_SIZES` option set to `ON` in CMake in order to get an accurate sense as to what grade bin you fall into. 
These letter grades will be applied only to the 70/100 points associated with the performance of your code. 

- \>= 50.0% of peak performance: A+
- \>= 40.0% of peak performance: A
- \>= 35.0% of peak performance: A-
- \>= 25.0% of peak performance: B+
- \>= 15.0% of peak performance: B
- \>=   10.0% of peak performance: B-
- <    10.0% of peak performance: C+

The letter grades will scale the 70/100 points associated with performance in the following way:

- A+: 1.00
- A: 0.98
- A-: 0.90
- B+: 0.85
- B: 0.82
- B-: 0.80
- C+: 0.70

## Submission Details [IMPORTANT]

1. Make sure you have our most updated source code on your Permultter machine. We have updated the CMake file for this submission.
2. Make sure you have only modified the file dgemm-blocked.c, and it compiles and runs as desired.
3. On Cavas, under the "People" section, there is a hw1 tab. Click on the tab and you'll see canvas has assigned a group id to each of you individually. Use the search bar to enter your name and find your group id. Treat it as a three digit number. (If you are group 4, your group id is "004").
4. Ensure that your write-up pdf is located in your source directory, next to dgemm-blocked.c. It should be named CS5220Group004_hw1.pdf.
5. Without the pdf in your source directory, the following command will fail. From your build directory, run:
```
student@perlmutter:~/hw1/build> cmake -DGROUP_NO=004 ..
student@perlmutter:~/hw1/build> make package
```
6. Confirm that it worked using the following command. You should see output like:
```
student@perlmutter:~/hw1/build> tar tfz CS5220Group004_hw1.tar.gz
CS5220Group004_hw1/CS5220Group004_hw1.pdf
CS5220Group004_hw1/dgemm-blocked.c
```
7. Download and submit your .tar.gz through canvas.

**This process should be followed for both the checkpoint and final submissions**. For the checkpoint assignment, you can just create an empty pdf file for your report. 

## Report Write-up Direction

Your write-up report should contain:

- Your name, cornell id, and perlmutter username,
- A line plot showing the GFLOPS/s attained by your dgemm kernel on each matrix size in the complete set of matrix sizes (make sure to run CMake with -DALL_SIZES=ON to run the full suite of matrix sizes). You should also have a horizontal line on this plot that denotes theoretical peak, and this line should be easily distinguishable from the line that denotes the performance of your implementation. You can also optionally include a line corresponding to the performance achieved by `dgemm-blas.c`.
- A brief (<1000 character) explanation of what the plot shows. This can be a caption or it can be in the body of the document.
- Disclosure of any collaborations or use of generative AI. 

## Notes

Your grade will mostly depend on three factors:

- whether or not it is correct (ie. finishes running without exiting early)
- performance sustained by your codes on the Perlmutter supercomputer
- quality of plot and explanation in the report

There are other formulations of matmul (e.g., [Strassen](http://www.google.com/url?q=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FStrassen_algorithm&sa=D&sntz=1&usg=AOvVaw2Pf7JDXXL7Es4SQ6Vt-g3G)) that are mathematically equivalent, but perform asymptotically fewer computations - we will not grade submissions that do fewer computations than the 2 n^3 algorithm. 

You must use the GNU C Compiler 13.2.1 for this assignment. If your code does not compile and run with GCC 13.2.1, it will not be graded. 

Besides compiler intrinsic functions and built-ins, your code (dgemm-blocked.c) must only call into the C standard library.

GNU C provides [many](http://www.google.com/url?q=http%3A%2F%2Fgcc.gnu.org%2Fonlinedocs%2Fgcc%2FC-Extensions.html&sa=D&sntz=1&usg=AOvVaw3sz1i1xHTqEt8jHmLrNIf8) extensions, which include intrinsics for vector (SIMD) instructions and data alignment. 

To manually vectorize, you should prefer to add compiler intrinsics to your code; avoid using inline assembly, at least at first.

The [Compiler Explorer project (aka godbolt)](https://godbolt.org/) will be useful for exploring the relationship between your C code and its corresponding assembly. Release mode builds compile with -O3.

You may assume that A and B do not alias C; however, A and B may alias each other. It is semantically correct to qualify C (the last argument to square_dgemm) with the C99 restrict keyword. There is a lot online about restrict and pointer-aliasing - [this](http://www.google.com/url?q=http%3A%2F%2Fcellperformance.beyond3d.com%2Farticles%2F2006%2F05%2Fdemystifying-the-restrict-keyword.html&sa=D&sntz=1&usg=AOvVaw3kWQYySvzgBsjxXPQdeumD) is a good article to start with, along with the [Wikipedia article](https://www.google.com/url?q=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRestrict&sa=D&sntz=1&usg=AOvVaw08belFyY7mihifQ87Gq0vJ) on the restrict keyword.

The matrices are all stored in [column-major order](http://www.google.com/url?q=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FRow-major_order&sa=D&sntz=1&usg=AOvVaw2T7_9V3fulEXaKRdPke-RD), i.e. `C[i,j] == C[i,j] == C[i+j*n]`, for i=1:n, where n is the number of rows in C. Note that we use 1-based indexing when using mathematical symbols and MATLAB index notation (parentheses), and 0-based indexing when using C index notation (square brackets).

We will check correctness by the following component-wise error bound:

|square_dgemm(n,A,B,0) - AB| < epsn*|A|*|B|


where eps := 2^-52 = 2.2 * 10^-16 is the [machine epsilon](http://www.google.com/url?q=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FMachine_epsilon&sa=D&sntz=1&usg=AOvVaw17QoaBLnDBEKSu4ksDUmMu).

## Documentation

- [GCC documentation](https://gcc.gnu.org/onlinedocs/gcc-11.2.0/gcc/) - Perlmutter's version currently is GCC 13.2.1
- [GCC's vector extensions](https://gcc.gnu.org/onlinedocs/gcc-8.3.0/gcc/Vector-Extensions.html#Vector-Extensions)
- [GCC's built-ins](https://www.google.com/url?q=https%3A%2F%2Fgcc.gnu.org%2Fonlinedocs%2Fgcc-8.3.0%2Fgcc%2FOther-Builtins.html%23Other-Builtins&sa=D&sntz=1&usg=AOvVaw3p4zP_OjIeTSggK6zrssIf)
- [GCC's variable attributes](https://www.google.com/url?q=https%3A%2F%2Fgcc.gnu.org%2Fonlinedocs%2Fgcc-8.3.0%2Fgcc%2FCommon-Variable-Attributes.html%23Common-Variable-Attributes&sa=D&sntz=1&usg=AOvVaw1JaXRf4EawtUhnkManVmiZ)
- [GCC's function attributes](https://www.google.com/url?q=https%3A%2F%2Fgcc.gnu.org%2Fonlinedocs%2Fgcc-8.3.0%2Fgcc%2FCommon-Function-Attributes.html%23Common-Function-Attributes&sa=D&sntz=1&usg=AOvVaw0PVbmQ-2mlORlz8Bo5zElE)

You are also welcome to learn from the source code of state-of-art BLAS implementations such as [OpenBLAS](https://github.com/OpenMathLib/OpenBLAS). However, you should not reuse those codes in your submission.

*We emphasize these are example scripts because for these as well as all other assignment scripts we provide, you may need to adjust the number of requested nodes and cores and amount of time according to your needs (your allocation and the total class allocation is limited). To understand how you are charged, [READ THIS](http://www.google.com/url?q=http%3A%2F%2Fwww.nersc.gov%2Fusers%2Faccounts%2Fuser-accounts%2Fhow-usage-is-charged%2F&sa=D&sntz=1&usg=AOvVaw25aePnroPVI3vO90toSiC6) alongside the given scripts. For testing (1) try running and debugging on your laptop first, (2) try running with the minimum resources you need for testing and debugging, (3) once your code is fully debugged, use the amount of resources you need to collect the final results for the assignment. This will become more important for later assignments, but it is good to get in the habit now.*

