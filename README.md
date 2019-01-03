# GPU path profiling

This repository provides code  for a simple efficient path profiling on GPUs. It is inspired by Ball and Larus' paper: [Efficient
Path Profiling] (https://dl.acm.org/citation.cfm?id=243857). Because GPU applications are not as complex as CPU
applications (fewer branches), we do not implement the step 2 in DAG profiling (see section 3 in the paper). This means that every edge
in the control flow graph of a kernel is instrumented.

This code uses [SASSI] (https://github.com/NVlabs/sassi) for instrumentation, and is therefore limited to work only with
applications/OS/CUDA versions that are compatible with SASSI. 

The directory "sassi_handlers" contains the SASSI instrumentation code that needs to be copied to a *SASSI* libs directory. The "scripts" directory contains the script that parses outputs.

Assuming there is a SASSI installed on a system, these are the steps involved in using this code:
## 1. Generate static Control Flow Graph (CFG) for the application
This could also be done using the `cfg_profiler.cu` SASSI handler which will generate a `simple-cfg.dot`.
## 2. Process the CFG and generate the edge increment values
The script `process_cfg.py` accomplishes this, and expects to receive CFG file in the same format as
`simple-cfg.dot`. Each increment value for each edge between two Basic Blocks will ensure that there are no two
paths (from a basic block to another) that will have the same path sum. Redirect the output to `cfgs.txt`
``` python process_cfg.py simple-cfg.dot > cfgs.txt ```
## 3. Run application for profiling
Use SASSI handler `path_profiler.cu` that will parse the `cfgs.txt` and generate path sums for each path in each
kernel and kernel invocation. This handler dumps 2 files:
  a. `full_paths.txt`: This file has the number of executions for each path in each kernel and invocation *per warp*.
  b. `path_profile.txt`: This file has the total number of executions for each path in each kernel and invocation. Every execution is one warp.
## 4. Parse the path profile
Use the script print_paths.py to dump the full path strings. Instead of just path sum number, you will have the basic block
string that makes up each path. This also uses a DB in order to accomplish this. Working on releasing the script
that generates the DB.
