---
status: done
---

<style type="text/css"> pre em { font-style: normal; background-color: yellow; } pre strong { font-style: normal; font-weight: bold; color: \#008; } </style>

Friday Exercise 1.2: Plan Joe's Workflow
========================================

This exercise outlines our goal (creating a production workflow) and the steps needed to achieve it. Before starting this section, make sure to first read [Exercise 1.1](/materials/day5/part1-ex1-science-intro), which has important background information about Joe's intended work and how he has submitted jobs so far.

Your Mission
------------

Your goal is to plan out Joe’s workflow based upon small-scale test jobs and, later, to write a full-scale DAG to actually run his workflow in production. In particular, you will need to do the following:

- **Optimize number of permutation jobs:** Joe wants to take advantage of high-throughput parallelization.  This means, for one trait, determining how many jobs to submit for the **permutation** step (where each job is calculating a portion of the new total of 100,000 permutations per trait).  For each trait, is it better to run 10 jobs that each generate 10,000 permutations?  Or 1000 jobs that generate 10 permutations?  You'll have to find out.  

- **Optimize memory and cpu requests**: For both the **permutation** and **QTL mapping** steps, what values should you use for `request_memory` and `request_cpus`?

- **Put all the pieces together in a single DAG file**: We want to create a DAG that runs the **permutation** and **QTL mapping** jobs for each of the three traits, with the necessary `tar` scripts running at the appropriate times on the submit server.  

We'll be working on the first two steps during this session, with final testing and the last step (DAG creation) after the break in [Exercise 2.1](/materials/day5/part2-ex1-execute-workflow).

!!! note 
	In what follows, **the only files you will need to modify are the submit files** (and as specifically instructed). You will not need to modify any of the input files, output files, or scripts/executables/programs. It is advisable that you split up some of the work within your pair or group in order to be time-efficient.

Draw a Diagram
--------------

If you haven't already -- draw a digram of Joe's workflow, based on what you [learned from Joe](/materials/day5/part1-ex1-science-intro).  Keep in mind that there are 3 traits for which the *permutation* and *QTL mapping* steps need to be completed, but these trait analyses are each completely independent (each of the steps described needs to be run for trait 1, trait 2, and trait 3, but they don't overlap at all).  (If you started drawing a diagram while reading the previous page, just extend it here.)  

Think about what Joe's intended workflow means for the shape of the DAG, including PARENT-CHILD dependencies for JOBs in the DAG and the fact that the *permutation* step could be broken up into multiple processes of fewer total permutations, each.  The tar steps will probably need to be PRE or POST scripts (you decide which is best).
We'll come back to this diagram in the [next exercise](/materials/day5/part2-ex1-execute-workflow) after the break, when it's time to construct the full DAG.

Optimize Job Components
-----------------------

Before we assemble the full DAG, we want to optimize each component of the workflow. This will include 3 optimization steps.  

!!! note
	Eventually we'll want to apply our job optimizations to the submit files for all three traits, but for testing purposes, it's okay to just focus on one trait.  In your group, pick whether you want to use trait 1, 2, or 3 for testing and use the submit files for that trait for all of your tests.  

### 1. Optimize length of *permutation* jobs, test resource use

To get good HTC scaling, we want the jobs that run for the permutation step to each take somewhere between 10 minutes and 1 hour (ideally ~30 minutes).  So our first optimization task is to determine the number of permutations that can be generated by a single job in this amount of time.  (Eventually - as part of the final workflow - we'll submit batches of these optimized permutation jobs so that we generate 100,000 total permutations for each trait.)  

To determine the right number of permutations per job, you will need to run some test jobs, to see how long it takes a single job to create, say, 10, 100, or 1000 permutations.  

To run these test jobs:

1.  Add `request_cpus = 1` according to Joe's indication
1.  Add reasonable first guesses for `request_memory` and `request_disk` (say, 1 GB?).
1.  Make a few copies of this submit file so that you can change the last argument (the number of permutations) from "10000" to "10", "100", or "1000", e.g. from: 

		:::file
		arguments = arguments = 1_$(Process) run_perm.R 1 $(Process) 10000

	to something like: 
	
		:::file
		arguments = arguments = 1_$(Process) run_perm.R 1 $(Process) 10

	You'll want to run multiple tests here (`10` as given above, and probably at least `100` and `1000`).  You can split this testing with a partner.  

Getting ready for the next step:

1.  After each set of *permutation* tests finishes, you’ll need to use `tarit.sh` (with the correct argument) before running the test jobs for the QTL step.

### 2. Test the *QTL mapping* job

Once you have results from your previous *permutation* step testing, you can start testing the *QTL mapping* job.  

1. Add lines for `request_memory`, `request_disk`, and `request_cpus` to one of the *QTL* submit files.  The resource needs (RAM and disk space) and execution time of each *QTL mapping* job will likely increase with the total number of permutations from the previous *permutation* step, though the execution time will likely still be short (according to Joe). 
1. Submit the modified *QTL* submit file.
1. When the QTL job completes (hopefully quickly!) look at the log file.  

If you have time, try running the QTL job again with results from a different permutation test (that may be a different sized input).  Does the disk and memory usage change?  

### 3. Optimization planning

In order to optimize Joe's overall workflow, we can choose the following values, based on our test jobs from steps 1 and 2:

1. How many permutations should each permutation job create to run in about 30 minutes?  10? 100? 1000?  Something in-between?  Based on your testing, choose an appropriate value. 

	!!! note
		You can use the "condor_history" feature (similar to condor_q, but for completed jobs) to easily view and compare the "RUNTIME" for jobs in a "cluster" (using the cluster value as an argument to `condor_history`).

1. For each trait, we want to generate 100,000 permutations.  How many jobs do you need to submit (for each trait) to generate this amount, based on the number of permutations created per job (what you chose in the previous point)?  Essentially, you want *number of jobs* X *permutations per job* to equal 100,000 total permutations...and do that for each of the three phenotype traits. 

1. Make sure to examine the log files of both your *permutation* and *QTL* test jobs, so that you can extrapolate how much memory and disk should be requested in the submit files for the full-scale DAG.

**When you're done with 3, move on to [Exercise 2.1](/materials/day5/part2-ex1-execute-workflow)**

