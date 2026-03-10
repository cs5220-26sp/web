---
title: Final Project - Project Proposal
layout: main
---

## Overview

The project proposal counts for **5% of the course grade** (5 out of 30 points allocated to the final project). It is your opportunity to define the scope of your work and receive early feedback before the project period begins.

- **Format:** PDF only — no other file types will be accepted.
- **Length:** Minimum 1 page, maximum 2 pages. [Double column ACM conference paper format](https://www.overleaf.com/latex/templates/acm-conference-proceedings-primary-article-template/wbvnghjbzwpc) (same template as the Cerebras assignment).
- **Due:** March 27, 2026 (11:59 am EST).

Critically, **no late penalty or slip days apply to this assignment**. In order to give you useful feedback by April 3, I need proposals submitted by March 27 at the latest. I will need to receive a proposal from everyone to avoid downstream impact on the project grade, but late submission will result in no credit for the project proposal.

---

## Project Requirements

Every project must meaningfully incorporate all three of the following elements. These will directly inform how your final report is evaluated.

**Parallel programming** — Your project must involve hands-on parallel programming. This includes, but is not limited to: OpenMP, MPI, CUDA, or Cerebras CSL. The parallelization strategy should be central to your project, not an afterthought.

**Performance optimization** — You are expected to actively optimize your implementation. Examples include SIMD vectorization, cache-friendly data layouts, memory access pattern tuning, and/or arithmetic intensity improvements.

**Performance analysis** — You must rigorously measure and analyze performance. Examples include strong and weak scaling plots, runtime breakdowns by kernel or phase, roofline analysis, and/or communication modeling (e.g., alpha-beta model analysis).

If you are already working on a research project that involves parallel computing, you are encouraged to build on it. This is a great opportunity to deepen that work. If you are unsure whether your topic is a good fit, post on Ed Discussion or ask during OHs to Prof. Guidi.

---

## Proposal Feedback

The instructor will review the proposals over spring break and send feedback or requests for changes by April 3. If you have not heard from the instructor by then, your proposal is approved and you are good to go.

---

## Group Setup

- Groups must have **2–3 students**. Groups of 4 or more are not allowed — no exceptions.
- Solo projects (groups of 1) require instructor approval and a valid reason. By default, you should find a teammate — working in a group is strongly encouraged. If you are looking for teammates, post your project idea(s) on Ed Discussion to connect with others.
- There are no restrictions on major or graduate/undergraduate composition.
- Go to **People → Project Groups** on Canvas and add your group to an empty slot before submitting.

The expected scope and depth of the project scales with group size. A 3-person group is held to a higher bar than a 2-person group. Keep this in mind when scoping your proposal.

---

## Required Sections

Your proposal must address all five of the following questions. Use them as section headers in your document.

1. **Hypothesis** — What is the question you are seeking to explore, understand, and to some extent answer? What do you expect the results of your investigation to be?
2. **Context** — Will you be collaborating with people outside the course? Is this a subproject within a larger research effort? Will related work be used in other classes?
3. **Key Prior Work** — Identify approximately three pieces of prior work that motivate, inform, or contrast with your proposed project.
4. **Empirical Methodology** — What will you do to investigate your hypothesis and establish results? What computational resources will you need?
5. **Challenges and Obstacles** — What do you anticipate might prevent you from pursuing this investigation as fully as you would like?

---

## Full Project Timeline

| Date     | Milestone |
|----------|-----------|
| Mar 13 | Proposal assignment opens on Canvas |
| Mar 27 | Proposal due (no late submissions) |
| Spring break | Instructor reviews proposals |
| Apr 3 | Feedback or change requests sent; no news means your project is approved and you can start after Spring break |
| Apr 6 | Project work begins |
| Apr 29 | Checkpoint report and poster due (all groups) |
| Apr 30 & May 5 | Poster sessions (attendance required at both) |
| Mid-May | Final report due (date TBD by university registrar) |

---

## Grading (30% of Course Grade)

| Component | Weight | Notes |
|-----------|----------|---------|
| Proposal | 5% | This assignment |
| Poster & Poster session participation | 5% | Checkpoint submission by Apr 29 **AND** attend both Apr 30 **AND** May 5 lectures (poster session); note the **and** condition missing one of these is going to result in a 0 in this portion. |
| Final report | 20% | Please see final report assignment for rubric (it will be published by March 13) |

Posters (PDF) will be made publicly available on the course website.

---

## What Makes a Good Final Report

The final report will be evaluated on the following four components, in order of importance. Not all projects will have all four, but components 1 and 2 are expected in every project. The majority of your report should focus on parallelism, not on the problem description itself.

1. **Practical content and creativity**; implementation and tuning effort
2. **Experimental data**: scaling/performance analysis, interesting inputs or outputs
3. **Theoretical content and creativity**: design and analysis of algorithms
4. **Impact**: difficulty and timeliness of the contribution

More information on the final report will be posted by March 13, 2026.

---

## Submission Details

Please submit a single PDF on [Canvas](https://canvas.cornell.edu/courses/85162/assignments/912279) using the double-column ACM conference paper format (no other file types or formats will be accepted).

---

## Project Ideas

**Parallel BFS / SSSP on Large Graphs with MPI** *(2–3 people)*
Implement parallel Breadth-First Search or (and if 3 people) Single-Source Shortest Paths on distributed memory using MPI, targeting large sparse graphs (e.g., social networks or road networks from the SNAP dataset). The students should explore 1D vs 2D partitioning, frontier communication strategies, and analyze scaling behavior and communication volume with an alpha-beta model.

**Parallel Genome Sequence Alignment with OpenMP** *(2 people)*
Parallelize pairwise sequence alignment (e.g., Smith-Waterman or seed-and-extend) using OpenMP and SIMD intrinsics, targeting a batch of reads against a reference. The students should analyze thread scaling, vectorization efficiency, and memory bandwidth utilization, and experiment with cache-friendly data layouts for the scoring matrix.

**Parallel Neural Network Training with MPI** *(2 people)*
Implement data-parallel stochastic gradient descent (SGD) training of a simple neural network using MPI, focusing on gradient aggregation strategies (e.g., ring all-reduce vs. tree reduction). The students should analyze communication overhead versus computation time, produce strong scaling results across different batch sizes, and model communication cost with alpha-beta analysis.

**GPU-Accelerated Sparse Triangular Solve (SpTRSV)** *(3 people)*
Implement and optimize SpTRSV in CUDA, exploring level-scheduling and wavefront parallelism to expose concurrency in the triangular solve. The students should analyze the impact of matrix structure on parallelism, experiment with different sparse formats, and benchmark across matrices of varying size and sparsity pattern from SuiteSparse.
