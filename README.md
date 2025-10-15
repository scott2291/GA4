# GA 4

## Part A: Setting up & Git

1. Start a VS Code session at OSC in the folder `/fs/ess/PAS2880/users/$USER`. Create a new dir for this assignment, `/fs/ess/PAS2880/users/$USER/GA4`, and switch to that folder in VS Code using the “Open Folder” option. This should be your working dir for the entire assignment.

    > done!

1. Initialize a Git repository. Commit to the repo throughout the assignment as you see fit, but at least once for each “Part” of this assignment. Use and commit a `.gitignore` file as appropriate. (Tip: Many of you would do well to recall the feedback you got on your Git repo from the previous assignment!)

    > done!

1. Inside dir `GA4`, create a `README.md` file and open it in the VS Code editor. Use this file throughout the assigment to add your answers where appropriate.

    > done!

1. Inside dir `GA4`, also create dirs `scripts` and `results`. We’ll make your dir self-contained again (like a typical situation for a research project).

    > done

1. As a starting script, store the code below in a shell script `scripts/trimgalore.sh`. This is a script to run TrimGalore, which should be very similar to your final script from last week’s exercises.

    > done!

## Part B: A TrimGalore batch job

You’ll start with the TrimGalore script you just created, which should be much like that from last week’s exercises. But this time, instead of running the TrimGalore script “directly” with `bash`, you will submit it as a batch job. Then, in the next section, you will submit many batch jobs at the same time: one for each sample.


1. Add Sbatch options to the top of the TrimGalore shell script to specify:
    -     The account/project you want to use
    -     The number of cores you want to reserve: use 8
    -     The amount of time you want to reserve: use 30 minutes
    -     The desired file name of Slurm log file
    -     That Slurm should email you upon job failure
    -     Optional: you can try other Sbatch options you’d like to test

1. By printing and scanning through the TrimGalore help info once again (see last week’s exercises), find the TrimGalore option that specifies how many cores it can use – add the relevant line(s) from the TrimGalore help info to your `README.md`. In the script, change the `trim_galore` command accordingly to use the available number of cores.

1. To test the script and batch job submission, submit the script as a batch job only for sample `ERR10802863`.

1. Monitor the job, and when it’s done, check that everything went well (if it didn’t, redo until you get it right). In your `README.md`, explain your monitoring and checking process. Then, remove all outputs (Slurm log files and TrimGalore output files) produced by this test-run.

1. Illumina sequencing uses colors to distinguish between nucleotides as they are being added during the sequencing-by-synthesis process. However, newer Illumina machines (Nextseq and Novaseq) use a different color chemistry than older ones, and this newer chemistry suffers from an artefact that can erroneously produce strings of `G`s (“poly-G”) with high quality scores. Those `G`s should really be `N`s instead, and occur especially at the end of reverse (R2) reads.

In the FastQC outputs for the R2 file that you just produced with TrimGalore (recall that it runs FastQC after trimmming!), do you see any evidence for this problem? Explain.

1. We’ll assume that the data was indeed produced with the newer Illumina color chemistry. In the TrimGalore help info, find the relevant TrimGalore option to deal with the poly-G probelm, and again add the relevant line(s) from the help info to your `README.md`. Then, use the TrimGalore option you found, but don’t change the quality score threshold from the default.

1. Rerun TrimGalore with the added color-chemistry option. Check all outputs and confirm that usage of this option made a difference. Then, remove all outputs produced by this test-run again.

### Bonus: Modify the script to rename the output FASTQ files

f you choose not to do this Bonus part, you can simply move on to Part C.

The TrimGalore output FASTQ files are oddly named, ending in `_R1_val_1.fq.gz` and `_R2_val_2.fq.gz` – check the output files from your initial run to see this. This is not necessarily a problem, but could trip you up in a next step with these files.

Therefore, modify your TrimGalore script to rename the output files after running TrimGalore, giving them the same names as the input files. Then, rerun the script to check that your changes were successful.

## Part C: TrimGalore batch jobs in a loop

After your test-run, it is time to run TrimGalore on all samples by submitting batch jobs using a loop.

1. Write a `for` loop in your `README.md` to submit a TrimGalore batch job for each pair of FASTQ files that you have in your `data` dir.

1. Monitor the batch jobs and when they are done, check that everything went well (if it didn’t, redo until you get it right). In your `README.md`, explain your monitoring and checking process. In this case, it is appropriate to keep the Slurm log files: move them into a dir `logs` within the TrimGalore output dir.

## Part D: Publish your repo on Github

You’ll publish your Git repo on GitHub and “hand in” your assignment by creating an Issue.

1. Create a repository on GitHub, connect it to your local repo, and push your local repo to GitHub.

1. Create a new issue and tag GitHub users `menukabh` and `jelmerp`, asking us to take a look at your assignment.

