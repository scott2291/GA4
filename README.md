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

The following Sbatch options were added to the top of `trimgalore.sh`:

```bash

#!/bin/bash
#SBATCH --account=PAS2880
#SBATCH --cpus-per-task=8
#SBATCH --time 30
#SBATCH --mail-type=FAIL
#SBATCH --output=slurm-trimgalore-%j.out
set -euo pipefail

```


1. Add Sbatch options to the top of the TrimGalore shell script to specify:
    -     The account/project you want to use
    -     The number of cores you want to reserve: use 8
    -     The amount of time you want to reserve: use 30 minutes
    -     The desired file name of Slurm log file
    -     That Slurm should email you upon job failure
    -     Optional: you can try other Sbatch options you’d like to test


    The following Sbatch options were added to the top of `trimgalore.sh`:

        ```bash

        #!/bin/bash
        #SBATCH --account=PAS2880
        #SBATCH --cpus-per-task=8
        #SBATCH --time 30
        #SBATCH --mail-type=FAIL
        #SBATCH --output=slurm-trimgalore-%j.out
        set -euo pipefail

        ```

1. By printing and scanning through the TrimGalore help info once again (see last week’s exercises), find the TrimGalore option that specifies how many cores it can use – add the relevant line(s) from the TrimGalore help info to your `README.md`. In the script, change the `trim_galore` command accordingly to use the available number of cores.

    The following information is from the TrimGalore --help command and specifies the number of cores it can use:

    ```bash
        -j/--cores INT          Number of cores to be used for trimming [default: 1]. For Cutadapt to work with multiple cores, it
                        requires Python 3 as well as parallel gzip (pigz) installed on the system. Trim Galore attempts to detect
                        the version of Python used by calling Cutadapt. If Python 2 is detected, --cores is set to 1. If the Python
                        version cannot be detected, Python 3 is assumed and we let Cutadapt handle potential issues itself.
                        
                        If pigz cannot be detected on your system, Trim Galore reverts to using gzip compression. Please note
                        that gzip compression will slow down multi-core processes so much that it is hardly worthwhile, please 
                        see: https://github.com/FelixKrueger/TrimGalore/issues/16#issuecomment-458557103 for more info).

                        Actual core usage: It should be mentioned that the actual number of cores used is a little convoluted.
                        Assuming that Python 3 is used and pigz is installed, --cores 2 would use 2 cores to read the input
                        (probably not at a high usage though), 2 cores to write to the output (at moderately high usage), and 
                        2 cores for Cutadapt itself + 2 additional cores for Cutadapt (not sure what they are used for) + 1 core
                        for Trim Galore itself. So this can be up to 9 cores, even though most of them won't be used at 100% for
                        most of the time. Paired-end processing uses twice as many cores for the validation (= writing out) step.
                        --cores 4 would then be: 4 (read) + 4 (write) + 4 (Cutadapt) + 2 (extra Cutadapt) + 1 (Trim Galore) = 15.

                        It seems that --cores 4 could be a sweet spot, anything above has diminishing returns.

    ```
    I am confused about this question, due to the cores math they do for Python3 in the above section of the `trimgalore --help` command.  I specified 8 cores using the `--cores` option. The following line was adjusted:

    ```bash 
        # Run TrimGalore
        apptainer exec "$TRIMGALORE_CONTAINER" \
            trim_galore \
            --cores 8 \
            --paired \
            --fastqc \
            --output_dir "$outdir" \
            "$R1" \
            "$R2"
    ```

1. To test the script and batch job submission, submit the script as a batch job only for sample `ERR10802863`.

    I used the following command:
     
   ```bash
   sbatch scripts/trimgalore.sh data/fastq/ERR10802863_R1.fastq.gz data/fastq/ERR10802863_R2.fastq.gz results/
   ```
     
   The output was:
     
   ```bash
    Submitted batch job 37831741
    ```

1. Monitor the job, and when it’s done, check that everything went well (if it didn’t, redo until you get it right). In your `README.md`, explain your monitoring and checking process. Then, remove all outputs (Slurm log files and TrimGalore output files) produced by this test-run.

    I used the following command:

    ```bash 
    squeue -u scott2291
    ```
     
   The output was:
     
   ```bash
     JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          37831562       cpu ondemand scott229  R    1:14:58      1 p0223
          37831741   cpu-exp trimgalo scott229  R       0:11      1 p0507
    ```
  
    I think that my first run ran as expected. I did not recieve any email error, my file was properly renamed, the output of the script went to the proper directory, and the FastQC HTML file contained the expected output

    I will now remove all of the outputs from this run with the following commands:

    ```bash
    ls
    rm slurm-trimgalore-37831741.out
    rm results/ERR10802863_R*
    ls
    ls results/
    ```

    The output was:

    ```bash
    data  README.md  results  scripts  slurm-trimgalore-37831741.out

    data  README.md  results  scripts
    ```

1. Illumina sequencing uses colors to distinguish between nucleotides as they are being added during the sequencing-by-synthesis process. However, newer Illumina machines (Nextseq and Novaseq) use a different color chemistry than older ones, and this newer chemistry suffers from an artefact that can erroneously produce strings of `G`s (“poly-G”) with high quality scores. Those `G`s should really be `N`s instead, and occur especially at the end of reverse (R2) reads.

    In the FastQC outputs for the R2 file that you just produced with TrimGalore (recall that it runs FastQC after trimmming!), do you see any evidence for this problem? Explain.

    > When exploring the `ERR10802863_R2_val_2_fastqc.html` file, There is a warning in the Overrepresented sequences tab that flagged a repeated sequence of `G`s. This indicates to me that we are using the newer Illumina color chemistry, and that we should find a way to filter out the repeated `G` sequence.

1. We’ll assume that the data was indeed produced with the newer Illumina color chemistry. In the TrimGalore help info, find the relevant TrimGalore option to deal with the poly-G probelm, and again add the relevant line(s) from the help info to your `README.md`. Then, use the TrimGalore option you found, but don’t change the quality score threshold from the default.

    The following lines are produced by the `trimgalore --help` command and are related to the repeated `G`s sequence issue:

    ```bash
    --2colour/--nextseq INT This enables the option '--nextseq-trim=3'CUTOFF within Cutadapt, which will set a quality
                        cutoff (that is normally given with -q instead), but qualities of G bases are ignored.
                        This trimming is in common for the NextSeq- and NovaSeq-platforms, where basecalls without
                        any signal are called as high-quality G bases. This is mutually exlusive with '-q INT'.

    ```

    I adjusted the following code in `trimgalore.sh` to trim repeated `G`s from the sequence data.

    ```bash
    # Run TrimGalore
    apptainer exec "$TRIMGALORE_CONTAINER" \
        trim_galore \
        --cores 8 \
        --2colour \
        --paired \
        --fastqc \
        --output_dir "$outdir" \
        "$R1" \
        "$R2"
    ```

1. Rerun TrimGalore with the added color-chemistry option. Check all outputs and confirm that usage of this option made a difference. Then, remove all outputs produced by this test-run again.
  
   I used the following command:
   ```bash
    sbatch scripts/trimgalore.sh data/fastq/ERR10802863_R1.fastq.gz data/fastq/ERR10802863_R2.fastq.gz results
    ```

    The output was:

    ```bash
    Submitted batch job 37831843
    ```

    When checking the `ERR10802863_R2_val_2_fastqc.html` file again, there is no warning under the Overrepresented sequences tab, which indicates that the repeated `G`s have been removed. 

    The command needed a specified number, and I used the 3 that I noticed from the `trimgalore --help` command.  I am not sure if this is too short of a specification, and I would be curious as to what you would reccommend to set this to.


### Bonus: Modify the script to rename the output FASTQ files

if you choose not to do this Bonus part, you can simply move on to Part C.

The TrimGalore output FASTQ files are oddly named, ending in `_R1_val_1.fq.gz` and `_R2_val_2.fq.gz` – check the output files from your initial run to see this. This is not necessarily a problem, but could trip you up in a next step with these files.

Therefore, modify your TrimGalore script to rename the output files after running TrimGalore, giving them the same names as the input files. Then, rerun the script to check that your changes were successful.

## Part C: TrimGalore batch jobs in a loop

After your test-run, it is time to run TrimGalore on all samples by submitting batch jobs using a loop.

1. Write a `for` loop in your `README.md` to submit a TrimGalore batch job for each pair of FASTQ files that you have in your `data` dir.

    I first rewrote the `trimgalore.sh` to reassign the `R2` to a search and replace from the `R1` input. I did this to ensure that each pair is ran together. I changed `R2=$2` to `R2="${R1/_R1/_R2}"`. Then I ran the following loop to confirm I am looping over all the `R1` files in the `data` dir:

    ```bash

    for fastq_file in data/fastq/*_R1.fastq.gz; do
        echo "# Running an analysis on $fastq_file"
    done

    ```

    The output was 

    ```bash
    # Running an analysis on data/fastq/ERR10802863_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802864_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802865_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802866_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802867_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802868_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802869_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802870_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802871_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802874_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802875_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802876_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802877_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802878_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802879_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802880_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802881_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802882_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802883_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802884_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802885_R1.fastq.gz
    # Running an analysis on data/fastq/ERR10802886_R1.fastq.gz
    ```

    Now I will run this loop to run a batch job of each file pair:

    ```bash
    for fastq_file in data/fastq/*_R1.fastq.gz; do
        #echo "# Running an analysis on $fastq_file"
        sbatch scripts/trimgalore.sh $fastq_file results
    done
    ```

    The output of this loop was:

    ```bash
    Submitted batch job 37831966
    Submitted batch job 37831967
    Submitted batch job 37831968
    Submitted batch job 37831969
    Submitted batch job 37831970
    Submitted batch job 37831971
    Submitted batch job 37831972
    Submitted batch job 37831973
    Submitted batch job 37831974
    Submitted batch job 37831975
    Submitted batch job 37831976
    Submitted batch job 37831977
    Submitted batch job 37831978
    Submitted batch job 37831979
    Submitted batch job 37831980
    Submitted batch job 37831981
    Submitted batch job 37831982
    Submitted batch job 37831983
    Submitted batch job 37831984
    Submitted batch job 37831985
    Submitted batch job 37831986
    Submitted batch job 37831987
    ```

1. Monitor the batch jobs and when they are done, check that everything went well (if it didn’t, redo until you get it right). In your `README.md`, explain your monitoring and checking process. In this case, it is appropriate to keep the Slurm log files: move them into a dir `logs` within the TrimGalore output dir.

    To monitor my batch job, I first checked if all of my jobs were in the queue by using the following command:

    ```bash
    squeue -u scott2291
    ```

    The output was:

    ```bash
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          37831966       cpu trimgalo scott229  R       0:11      1 p0032
          37831967       cpu trimgalo scott229  R       0:11      1 p0056
          37831968       cpu trimgalo scott229  R       0:11      1 p0063
          37831969       cpu trimgalo scott229  R       0:11      1 p0083
          37831970       cpu trimgalo scott229  R       0:11      1 p0124
          37831971       cpu trimgalo scott229  R       0:11      1 p0187
          37831972       cpu trimgalo scott229  R       0:11      1 p0193
          37831973       cpu trimgalo scott229  R       0:11      1 p0002
          37831974       cpu trimgalo scott229  R       0:11      1 p0002
          37831975       cpu trimgalo scott229  R       0:11      1 p0019
          37831976       cpu trimgalo scott229  R       0:11      1 p0020
          37831977       cpu trimgalo scott229  R       0:11      1 p0027
          37831978       cpu trimgalo scott229  R       0:11      1 p0027
          37831979       cpu trimgalo scott229  R       0:11      1 p0030
          37831980       cpu trimgalo scott229  R       0:11      1 p0030
          37831981       cpu trimgalo scott229  R       0:11      1 p0042
          37831982       cpu trimgalo scott229  R       0:11      1 p0042
          37831983       cpu trimgalo scott229  R       0:11      1 p0068
          37831984       cpu trimgalo scott229  R       0:11      1 p0069
          37831985       cpu trimgalo scott229  R       0:11      1 p0008
          37831986       cpu trimgalo scott229  R       0:11      1 p0008
          37831987       cpu trimgalo scott229  R       0:11      1 p0008
          37831856   cpu-exp ondemand scott229  R      30:25      1 p0831
    ```

    This output confirmed that all of my jobs were running simutaneously. I then used the `ls` command to confirm that separate `slurm` files were created for each job. The following code block contains the output of the `ls` command:

    ```bash
    data                           slurm-trimgalore-37831969.out  slurm-trimgalore-37831976.out  slurm-trimgalore-37831983.out
    README.md                      slurm-trimgalore-37831970.out  slurm-trimgalore-37831977.out  slurm-trimgalore-37831984.out
    results                        slurm-trimgalore-37831971.out  slurm-trimgalore-37831978.out  slurm-trimgalore-37831985.out
    scripts                        slurm-trimgalore-37831972.out  slurm-trimgalore-37831979.out  slurm-trimgalore-37831986.out
    slurm-trimgalore-37831966.out  slurm-trimgalore-37831973.out  slurm-trimgalore-37831980.out  slurm-trimgalore-37831987.out
    slurm-trimgalore-37831967.out  slurm-trimgalore-37831974.out  slurm-trimgalore-37831981.out
    slurm-trimgalore-37831968.out  slurm-trimgalore-37831975.out  slurm-trimgalore-37831982.out

    ```

    Finally, I checked that every R1 and R2 fastq file produced the proper files in the `results` directory using the following command:

    ```bash
    ls results/
    ```

    The output was:

    ```bash
    ERR10802863_R1.fastq.gz_trimming_report.txt  ERR10802870_R1_val_1.fq.gz                   ERR10802879_R2_val_2_fastqc.zip
    ERR10802863_R1_val_1_fastqc.html             ERR10802870_R2.fastq.gz_trimming_report.txt  ERR10802879_R2_val_2.fq.gz
    ERR10802863_R1_val_1_fastqc.zip              ERR10802870_R2_val_2_fastqc.html             ERR10802880_R1.fastq.gz_trimming_report.txt
    ERR10802863_R1_val_1.fq.gz                   ERR10802870_R2_val_2_fastqc.zip              ERR10802880_R1_val_1_fastqc.html
    ERR10802863_R2.fastq.gz_trimming_report.txt  ERR10802870_R2_val_2.fq.gz                   ERR10802880_R1_val_1_fastqc.zip
    ERR10802863_R2_val_2_fastqc.html             ERR10802871_R1.fastq.gz_trimming_report.txt  ERR10802880_R1_val_1.fq.gz
    ERR10802863_R2_val_2_fastqc.zip              ERR10802871_R1_val_1_fastqc.html             ERR10802880_R2.fastq.gz_trimming_report.txt
    ERR10802863_R2_val_2.fq.gz                   ERR10802871_R1_val_1_fastqc.zip              ERR10802880_R2_val_2_fastqc.html
    ERR10802864_R1.fastq.gz_trimming_report.txt  ERR10802871_R1_val_1.fq.gz                   ERR10802880_R2_val_2_fastqc.zip
    ERR10802864_R1_val_1_fastqc.html             ERR10802871_R2.fastq.gz_trimming_report.txt  ERR10802880_R2_val_2.fq.gz
    ERR10802864_R1_val_1_fastqc.zip              ERR10802871_R2_val_2_fastqc.html             ERR10802881_R1.fastq.gz_trimming_report.txt
    ERR10802864_R1_val_1.fq.gz                   ERR10802871_R2_val_2_fastqc.zip              ERR10802881_R1_val_1_fastqc.html
    ERR10802864_R2.fastq.gz_trimming_report.txt  ERR10802871_R2_val_2.fq.gz                   ERR10802881_R1_val_1_fastqc.zip
    ERR10802864_R2_val_2_fastqc.html             ERR10802874_R1.fastq.gz_trimming_report.txt  ERR10802881_R1_val_1.fq.gz
    ERR10802864_R2_val_2_fastqc.zip              ERR10802874_R1_val_1_fastqc.html             ERR10802881_R2.fastq.gz_trimming_report.txt
    ERR10802864_R2_val_2.fq.gz                   ERR10802874_R1_val_1_fastqc.zip              ERR10802881_R2_val_2_fastqc.html
    ERR10802865_R1.fastq.gz_trimming_report.txt  ERR10802874_R1_val_1.fq.gz                   ERR10802881_R2_val_2_fastqc.zip
    ERR10802865_R1_val_1_fastqc.html             ERR10802874_R2.fastq.gz_trimming_report.txt  ERR10802881_R2_val_2.fq.gz
    ERR10802865_R1_val_1_fastqc.zip              ERR10802874_R2_val_2_fastqc.html             ERR10802882_R1.fastq.gz_trimming_report.txt
    ERR10802865_R1_val_1.fq.gz                   ERR10802874_R2_val_2_fastqc.zip              ERR10802882_R1_val_1_fastqc.html
    ERR10802865_R2.fastq.gz_trimming_report.txt  ERR10802874_R2_val_2.fq.gz                   ERR10802882_R1_val_1_fastqc.zip
    ERR10802865_R2_val_2_fastqc.html             ERR10802875_R1.fastq.gz_trimming_report.txt  ERR10802882_R1_val_1.fq.gz
    ERR10802865_R2_val_2_fastqc.zip              ERR10802875_R1_val_1_fastqc.html             ERR10802882_R2.fastq.gz_trimming_report.txt
    ERR10802865_R2_val_2.fq.gz                   ERR10802875_R1_val_1_fastqc.zip              ERR10802882_R2_val_2_fastqc.html
    ERR10802866_R1.fastq.gz_trimming_report.txt  ERR10802875_R1_val_1.fq.gz                   ERR10802882_R2_val_2_fastqc.zip
    ERR10802866_R1_val_1_fastqc.html             ERR10802875_R2.fastq.gz_trimming_report.txt  ERR10802882_R2_val_2.fq.gz
    ERR10802866_R1_val_1_fastqc.zip              ERR10802875_R2_val_2_fastqc.html             ERR10802883_R1.fastq.gz_trimming_report.txt
    ERR10802866_R1_val_1.fq.gz                   ERR10802875_R2_val_2_fastqc.zip              ERR10802883_R1_val_1_fastqc.html
    ERR10802866_R2.fastq.gz_trimming_report.txt  ERR10802875_R2_val_2.fq.gz                   ERR10802883_R1_val_1_fastqc.zip
    ERR10802866_R2_val_2_fastqc.html             ERR10802876_R1.fastq.gz_trimming_report.txt  ERR10802883_R1_val_1.fq.gz
    ERR10802866_R2_val_2_fastqc.zip              ERR10802876_R1_val_1_fastqc.html             ERR10802883_R2.fastq.gz_trimming_report.txt
    ERR10802866_R2_val_2.fq.gz                   ERR10802876_R1_val_1_fastqc.zip              ERR10802883_R2_val_2_fastqc.html
    ERR10802867_R1.fastq.gz_trimming_report.txt  ERR10802876_R1_val_1.fq.gz                   ERR10802883_R2_val_2_fastqc.zip
    ERR10802867_R1_val_1_fastqc.html             ERR10802876_R2.fastq.gz_trimming_report.txt  ERR10802883_R2_val_2.fq.gz
    ERR10802867_R1_val_1_fastqc.zip              ERR10802876_R2_val_2_fastqc.html             ERR10802884_R1.fastq.gz_trimming_report.txt
    ERR10802867_R1_val_1.fq.gz                   ERR10802876_R2_val_2_fastqc.zip              ERR10802884_R1_val_1_fastqc.html
    ERR10802867_R2.fastq.gz_trimming_report.txt  ERR10802876_R2_val_2.fq.gz                   ERR10802884_R1_val_1_fastqc.zip
    ERR10802867_R2_val_2_fastqc.html             ERR10802877_R1.fastq.gz_trimming_report.txt  ERR10802884_R1_val_1.fq.gz
    ERR10802867_R2_val_2_fastqc.zip              ERR10802877_R1_val_1_fastqc.html             ERR10802884_R2.fastq.gz_trimming_report.txt
    ERR10802867_R2_val_2.fq.gz                   ERR10802877_R1_val_1_fastqc.zip              ERR10802884_R2_val_2_fastqc.html
    ERR10802868_R1.fastq.gz_trimming_report.txt  ERR10802877_R1_val_1.fq.gz                   ERR10802884_R2_val_2_fastqc.zip
    ERR10802868_R1_val_1_fastqc.html             ERR10802877_R2.fastq.gz_trimming_report.txt  ERR10802884_R2_val_2.fq.gz
    ERR10802868_R1_val_1_fastqc.zip              ERR10802877_R2_val_2_fastqc.html             ERR10802885_R1.fastq.gz_trimming_report.txt
    ERR10802868_R1_val_1.fq.gz                   ERR10802877_R2_val_2_fastqc.zip              ERR10802885_R1_val_1_fastqc.html
    ERR10802868_R2.fastq.gz_trimming_report.txt  ERR10802877_R2_val_2.fq.gz                   ERR10802885_R1_val_1_fastqc.zip
    ERR10802868_R2_val_2_fastqc.html             ERR10802878_R1.fastq.gz_trimming_report.txt  ERR10802885_R1_val_1.fq.gz
    ERR10802868_R2_val_2_fastqc.zip              ERR10802878_R1_val_1_fastqc.html             ERR10802885_R2.fastq.gz_trimming_report.txt
    ERR10802868_R2_val_2.fq.gz                   ERR10802878_R1_val_1_fastqc.zip              ERR10802885_R2_val_2_fastqc.html
    ERR10802869_R1.fastq.gz_trimming_report.txt  ERR10802878_R1_val_1.fq.gz                   ERR10802885_R2_val_2_fastqc.zip
    ERR10802869_R1_val_1_fastqc.html             ERR10802878_R2.fastq.gz_trimming_report.txt  ERR10802885_R2_val_2.fq.gz
    ERR10802869_R1_val_1_fastqc.zip              ERR10802878_R2_val_2_fastqc.html             ERR10802886_R1.fastq.gz_trimming_report.txt
    ERR10802869_R1_val_1.fq.gz                   ERR10802878_R2_val_2_fastqc.zip              ERR10802886_R1_val_1_fastqc.html
    ERR10802869_R2.fastq.gz_trimming_report.txt  ERR10802878_R2_val_2.fq.gz                   ERR10802886_R1_val_1_fastqc.zip
    ERR10802869_R2_val_2_fastqc.html             ERR10802879_R1.fastq.gz_trimming_report.txt  ERR10802886_R1_val_1.fq.gz
    ERR10802869_R2_val_2_fastqc.zip              ERR10802879_R1_val_1_fastqc.html             ERR10802886_R2.fastq.gz_trimming_report.txt
    ERR10802869_R2_val_2.fq.gz                   ERR10802879_R1_val_1_fastqc.zip              ERR10802886_R2_val_2_fastqc.html
    ERR10802870_R1.fastq.gz_trimming_report.txt  ERR10802879_R1_val_1.fq.gz                   ERR10802886_R2_val_2_fastqc.zip
    ERR10802870_R1_val_1_fastqc.html             ERR10802879_R2.fastq.gz_trimming_report.txt  ERR10802886_R2_val_2.fq.gz
    ERR10802870_R1_val_1_fastqc.zip              ERR10802879_R2_val_2_fastqc.html
    ```

    Now that the jobs have been completed and the proper `slurm` files have been created, I will move all the `slurm` files to a new directory named `logs`. To acheive this goal, I used the following commands:

    ```bash
    ls

    mkdir logs
    mv slurm-trimgalore-378319* logs
    ls

    ls logs

    ```

    The output was:

    ```bash
    data                           slurm-trimgalore-37831969.out  slurm-trimgalore-37831976.out  slurm-trimgalore-37831983.out
    README.md                      slurm-trimgalore-37831970.out  slurm-trimgalore-37831977.out  slurm-trimgalore-37831984.out
    results                        slurm-trimgalore-37831971.out  slurm-trimgalore-37831978.out  slurm-trimgalore-37831985.out
    scripts                        slurm-trimgalore-37831972.out  slurm-trimgalore-37831979.out  slurm-trimgalore-37831986.out
    slurm-trimgalore-37831966.out  slurm-trimgalore-37831973.out  slurm-trimgalore-37831980.out  slurm-trimgalore-37831987.out
    slurm-trimgalore-37831967.out  slurm-trimgalore-37831974.out  slurm-trimgalore-37831981.out
    slurm-trimgalore-37831968.out  slurm-trimgalore-37831975.out  slurm-trimgalore-37831982.out

    data  logs  README.md  results  scripts

    slurm-trimgalore-37831966.out  slurm-trimgalore-37831972.out  slurm-trimgalore-37831978.out  slurm-trimgalore-37831984.out
    slurm-trimgalore-37831967.out  slurm-trimgalore-37831973.out  slurm-trimgalore-37831979.out  slurm-trimgalore-37831985.out
    slurm-trimgalore-37831968.out  slurm-trimgalore-37831974.out  slurm-trimgalore-37831980.out  slurm-trimgalore-37831986.out
    slurm-trimgalore-37831969.out  slurm-trimgalore-37831975.out  slurm-trimgalore-37831981.out  slurm-trimgalore-37831987.out
    slurm-trimgalore-37831970.out  slurm-trimgalore-37831976.out  slurm-trimgalore-37831982.out
    slurm-trimgalore-37831971.out  slurm-trimgalore-37831977.out  slurm-trimgalore-37831983.out
    ```

    I will also add the `logs` and `results` directories to my `.gitignore`, so they will not be tracked. To accomplish this task, I used the following commands:

    ```bash
    git status
    
    echo "results/" >> .gitignore
    echo "logs/" >> .gitignore

    git status
    ```

    The output was:

    ```bash
    On branch main
    Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
            modified:   README.md
            modified:   scripts/trimgalore.sh

    Untracked files:
    (use "git add <file>..." to include in what will be committed)
            logs/
            results/

    no changes added to commit (use "git add" and/or "git commit -a")

    On branch main
    Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
            modified:   .gitignore
            modified:   README.md
            modified:   scripts/trimgalore.sh

    no changes added to commit (use "git add" and/or "git commit -a")
    ```

## Part D: Publish your repo on Github

You’ll publish your Git repo on GitHub and “hand in” your assignment by creating an Issue.

1. Create a repository on GitHub, connect it to your local repo, and push your local repo to GitHub.

1. Create a new issue and tag GitHub users `menukabh` and `jelmerp`, asking us to take a look at your assignment.

