---
title: HW4 - Distributed Matrix Multiplication on WSE-2
layout: main
---

# Overview

This assignment is an introduction to programming on Cerebras's Wafer-Scale Engine 2 (WSE-2) using the Cerebras Software Language (CSL).

In this assignment, you will implement distributed matrix multiplication $C = A \cdot B$ on a 2D grid of processing elements, where $A \in \mathbb{R}^{M \times H}$, $B \in \mathbb{R}^{H \times N}$, $C \in \mathbb{R}^{M \times N}$. All elements are fp32. The PE grid has `kernel_x_dim` columns and `kernel_y_dim` rows.

# Algorithm

The algorithm proceeds in three phases:

## Phase 1 -- Column Broadcast

Each part of $B$ (a vector) is broadcast from a single PE in a column to all other PEs in that column. The broadcast is split into a sending-up route and a common broadcast-down route. Because two directions cannot share the same color, the wavelets must change color at the head PE. PEs take turns sending: the top PE (y=0) sends first, then y=1, and so on, controlled by a fabric switch sentinel that advances after each PE finishes transmitting.

```
    Column x                     Routing

    PE(x,0)  <-- color swap -->  broadcast_rx_color sent NORTH is
       |         at head PE      reflected back SOUTH as broadcast_tx_color
       v
    PE(x,1)  -- forward SOUTH
       |
       v
    PE(x,2)  -- terminal (no further forwarding)

  After PE(x,0) finishes, a SWITCH_ADV control wavelet
  shifts the source to PE(x,1), then PE(x,2), etc.
```

Each wavelet carries one `f32` element. All `kernel_y_dim` PEs in the column transmit sequentially, so every PE receives the B-column data from all other PEs in the column.

## Phase 2 -- Local SAXPY

Upon receiving each broadcast wavelet $b_h$ (a scalar from $B$), the PE executes:

$$r \mathrel{+}= A_j \cdot b_h$$

where $r$ is the local accumulation buffer `red_in`, using the hardware `fmac` (fused multiply-accumulate) operation. After the PE has accumulated enough data, the partial sum is reduced along the row.

## Phase 3 -- Row Reduction (Ring)

The partial vectors `red_in` of length $dM$ are reduced (summed) across the X-dimension using a ring. The ring physically sends data EAST and returns WEST via flyover, forming a cycle through all PEs.

```
  Ring across X-dimension (kernel_x_dim = 4 example):

  State rotation for successive reduce() calls:

  Call #    PE0      PE1      PE2      PE3      Chain (ISUM -> ... -> FSUM)
  -----   ------   ------   ------   ------    ---------------------------
    0      FSUM     ISUM     PSUM     PSUM     PE1 -> PE2 -> PE3 -> PE0
    1      PSUM     FSUM     ISUM     PSUM     PE2 -> PE3 -> PE0 -> PE1
    2      PSUM     PSUM     FSUM     ISUM     PE3 -> PE0 -> PE1 -> PE2
    3      ISUM     PSUM     PSUM     FSUM     PE0 -> PE1 -> PE2 -> PE3

  ISUM: Initiator -- send local red_in to fabric (no receive)
  PSUM: Partial   -- receive from fabric, add local red_in, forward
  FSUM: Final     -- receive from fabric, add local red_in, write to C
```

Each PE maintains a state machine that cycles through ISUM, PSUM, and FSUM states. After `kernel_x_dim` reductions, each PE has written one column to its local $C$ block via the rotating FSUM assignment.

## Due Date: Thursday March 27th 2026 at 11:59 PM
# Instructions
## Teams

Note that you will work individually for this assignment.

## Setting Up the SDK

This assignment uses the Cerebras SDK. We have placed a version of the SDK on Perlmutter, available at `/global/cfs/cdirs/m4341/cs6230/sdk-1.4.0`. To setup and install the SDK, first run:
```
student@perlmutter: ~/hw4/global/cfs/cdirs/m4341/cs6230/sdk-1.4.0/install.sh
```
then add the SDK to your path:
```
student@perlmutter: export PATH=$PATH:/global/cfs/cdirs/m4341/cs6230/sdk-1.4.0
```
verify this has worked by running:
```
student@perlmutter: cs_python
```
which should open a Python REPL if everything is working correctly. 

## What You Must Implement

You have three files to modify:

### 1. `config.py` — Data Layout Booleans

Set six boolean flags that control how matrices are distributed across PEs and stored in memory. The host-side infrastructure (`prepare_h2d` / `reconstruct_d2h`) in `run.py` imports these flags and uses them to chunk and reassemble the matrices automatically.

- `*_GLOBAL_TRANSPOSE`: controls whether a matrix's row axis maps to Y-PEs (False) or X-PEs (True).
- `*_MEMORY_TRANSPOSE`: controls whether each PE stores its local block transposed in memory.

Think carefully about which matrix dimension must align with the reduction axis (X), and which memory layout gives contiguous access for column-vector operations.

### 2. `layout.csl` — Fabric Routing

Inside the `layout { }` block, implement:

- **Tile code assignment**: Use `@set_tile_code` to assign `pe.csl` to each PE with the appropriate parameters.
- **Reduction ring routing**: Configure `@set_color_config` for each PE so that partial sums flow from the last X-PE toward the first, with flyover colors for middle PEs.
- **Broadcast routing**: Configure `@set_color_config` so that each PE in a column can broadcast its B data to all other PEs in the same column.

### 3. `pe.csl` — On-PE Computation

Implement the per-PE logic:

- **Broadcast receive**: A data task that fires on each incoming wavelet, accumulates partial products via SAXPY (`fmac`), and triggers reduction when a full B-column has been received.
- **Reduction state machine**: A ring reduction across the X-dimension. PEs rotate through three roles (initiator, partial-sum, final-accumulator) across successive columns.
- **Broadcast send**: Send the local B data to the fabric, then send a control wavelet (`SWITCH_ADV`) to hand off to the next PE.
- **Completion**: Call `sys_mod.unblock_cmd_stream()` when all reductions finish.

Please do not modify any files besides `config.py`, `layout.csl`, and `pe.csl`.

## Running the Code

```bash
# Single configuration
./run.sh <sizeX> <sizeY> <M> <H> <N>

# All test configurations
./test_configs.sh

# Debug mode (fabric traces)
./run.sh 2 2 4 4 4 --debug

# Check performance
./test_perf.sh
```

## Constraints

- `kernel_x_dim >= 2`, `kernel_y_dim >= 2`
- $M$ mod `kernel_y_dim` $= 0$
- $H$ mod `kernel_x_dim` $= 0$
- $N$ mod `kernel_x_dim` $= 0$
- $N$ mod `kernel_y_dim` $= 0$

## Execution Flow

```
  Host                          Device (all PEs)
  ----                          ----------------
  memcpy A, B to PEs
          |
          v
  launch broadcast_pe()  --->  initialize_reduce_states based on tile position
                                broadcast B
                                  |
                                  +-- recv wavelet --> SAXPY
                                  +-- after enough wavelets --> reduce()
                                  +-- after all N columns --> terminate_kernel()
          |
          v
  memcpy C from PEs
  verify C == A * B
```

Total reductions per PE: `kernel_y_dim` $\times$ `dN_y` $= N$, one per global column of $C$.

# Grading

### IMPORTANT: There is a planned Perlmutter monthly maintenance between March 18th and March 19th. Please factor this into your time management for this assignment. 

Your score out of 100 points will be broken down into the following categories

- 20/100 points: Writeup

- 10/100 points: Checkpoint

- 20/100 points: Leaderboard Submission

- 50/100 points: Runtime Performance

Your score on the "Runtime Performance" part is determined by the `Runtime` value emitted by the `test_perf.sh` script provided in the repo. Similar to HW3, this part will be graded on a curve determined by the performance of everyone in the class. 

As with HW3, there will be a leaderboard for this homework that you can use to determine where you might land on the curve. 

**Right now, the leaderboard is under construction. Once it is complete, there will be a link to it placed here.**

Additionally, if your implementation attains a runtime below a certain threshold, we will guarantee at least a B+ on the 50 Runtime Performance points. **The threshold for a B+ is 428000 cycles as reported by `test_perf.sh`**.


## Checkpoint

The checkpoint for this assignment requires you to pass 5/20 of the correctness tests in `test_configs.sh`. You can pick any 5 you'd like. 

### The checkpoint is due on March 20th, 2026 at 11:59 PM ET.

## Leaderboard Submission

Before the deadline on March 27th, you must submit once to the leaderboard. Before submitting to the leaderboard, you must pass all the correctness tests in `test_configs.sh`. The leaderboard will check to make sure you pass all tests before allowing your submission. 

**Details on leaderboard submission will be added here once the leaderboard is finished**

## Submission Details

1. Make sure you have our most updated source code on Perlmutter.

2. Make sure you have only modified the files `layout.csl`, `pe.csl`, and `config.py`.

3. Ensure that your write-up pdf is located in your source directory. It should be named CS5220Group<GROUPID>_hw4.pdf. 

4. From your build directory, run:

```
student@perlmutter:~/hw4> ./package.sh <GROUPID>
```

6. Download and submit your .tar.gz through canvas. 

## Writeup Details

Your write-up should contain:

- your name, cornell id (NetID), and perlmutter username,

- A description of your implementation approach

- A table containing the statstics emitted by `test_perf.sh` evaluating the efficiency of your implementation

- GenAI and collaboration disclosure, similar to the other homeworks.

**Unlike other homeworks**, this writeup should be 1 page, double columns, using the [ACM conference latex template](https://www.overleaf.com/latex/templates/acm-conference-proceedings-primary-article-template/wbvnghjbzwpc).

# Tips

## Debugging

### Simprint (`<simprint>`)

The `<simprint>` library lets you print values directly to the simulator log (`sim.log`) during execution. A helper function `simprint_pe_coords()` is provided in `pe.csl`. Call it before any print to identify the PE:

```csl
simprint_pe_coords();
prt.print_string("hello from this PE\n");

simprint_pe_coords();
prt.fmt_no_nl("current_row={d}\n", .{@as(u16, current_row)});

simprint_pe_coords();
prt.fmt_no_nl("reduce_count={d}, C[0]={f}\n", .{reduce_count, C[0]});
```

**Important:** Output is flushed to `sim.log` only when `"\n"` is encountered. Always end your format strings with `\n`.

### Reading `sim.log`

Each line in `sim.log` is prefixed with the cycle number and the absolute fabric coordinates of the PE:

```
@968  P4.1: Loc(0, 0) P4.1: sender beginning main_fn
@1156 P5.1: Loc(1, 0) P5.1: recv_task: in_data = 0, global = 0
@1888 P6.2: Loc(2, 1) P6.2: recv_task: in_data = 4, global = 20
```

The absolute coordinates differ from the logical ones by the fabric offsets (typically `off_x=4, off_y=1`), so `Loc(0,0)` corresponds to `P4.1`.

### Extracting Per-PE Logs

Use `extract_logs.sh` to split `sim.log` into per-PE files:

```bash
./extract_logs.sh <sizeX> <sizeY>
```

### Debug Mode (Fabric Traces)

Run with `--debug` to enable fabric landing/router traces in `sim.log`:

```bash
./run.sh 2 2 4 4 4 --debug
```

## File Structure

| File | You Edit? | Description |
|------|-----------|-------------|
| `pe.csl` | **Yes** | Per-PE kernel written in CSL. Core computation logic. |
| `layout.csl` | **Yes** | Fabric topology and routing configuration. |
| `config.py` | **Yes** | Six boolean flags controlling matrix distribution and memory layout. |
| `run.py` | No | Host-side Python driver. Generates matrices, distributes them, launches kernel, verifies correctness. **Reset by the autograder.** |
| `run.sh` | No | Compile-and-run wrapper. |
| `test_configs.sh` | No | Automated test sweep over ~30 configurations. |
| `test_perf.sh` | No | Performance benchmark. |
| `clean.sh` | No | Removes all build artifacts and logs. |
| `extract_logs.sh` | No | Splits `sim.log` into per-PE log files. |
| `package.sh` | No | Creates `tar.gz` for submission to canvas |

## Recommended Tutorials

Study these examples before starting:

- `topic-06-switches` — fabric switches and SWITCH_ADV
- `topic-14-color-swap` — color swap routing
- `gemv-06-routes-1` through `gemv-08-routes-3` — fabric routing patterns
- `topic-11-collectives` — broadcast and reduce patterns
- `sdklayout-02-routing` — `@set_color_config` API
