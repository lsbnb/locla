## GABOLA Technical Notes
### § Preprocess Module:

#### *Step I. Produce non-duplicate split fastqs sorted by barcodes from raw linked reads*
If you choose to run the three steps of Preprocess module separately: **This step should be performed before running any GABOLA modules**, and it only needs to be run **once** in your entire pipeline.

**Input:**

- Path to the directory storing raw linked reads (see: https://github.com/10XGenomics/longranger for read file format)

**Output:**

- Non-duplicate, trimmed and filtered FASTQ files for Read 1 and Read 2 separately (NonDupR1.fq and NonDupR2.fq)
- Directories for every barcode containing their respective reads that are trimmed and filtered (OUTPUT_FOLDER/nonDupFq/split). 
> For example, Barcode name: AAAACCCCGTGTGTGT.
> Directory Format: OUTPUT_FOLDER/nonDupFq/split/AAAACCCCGTGTGTGT_1
```
usage: python /opt/10x_program/step1_preprocessFastq.py [-h] -f FASTQS --id PROJECTID -o OUTPUT_FOLDER [-q QUALITY] 
                                                 [-l LENGTH] [--trimr2 {False,True}] [-d NONDUP] [-t THREADS]
positional arguments:
  -f FASTQS, --fastqs FASTQS
                        Input the folder path of FASTQ files which were
                        produced from longranger mkfastq. Files in the folder
                        must be named like: [Sample Name]_S1_L00[Lane
                        Number]_[Read Type]_001.fastq.gz. (see:
                        https://github.com/10XGenomics/longranger) If your
                        fastq files are in a different folder, please move
                        them to the same folder. (input file must be in fastq
                        format, such as .fq or .fastq)
  --id PROJECTID        Input project name. It will be your output prefix.
  -o OUTPUT_FOLDER, --output_folder OUTPUT_FOLDER
                        Output folder path. Your output files will be in this
                        folder. ps. longranger will output your folder at pwd,
                        so you must cd to your output folder.
                        
optional arguments:
  -q QUALITY, --quality QUALITY
                        Input the minimum quality score for quality trimming
                        of FASTQ files. Same as the option for trim_galore -q
                        [default:20]
  -l LENGTH, --length LENGTH
                        Input the minimum length for quality trimming of FASTQ
                        files. Same as the option for trim_galore --length
                        [default:50]
  --trimr2 {False,True}
                        (Optional) R2 trimming. For better mapping quality,
                        choose to trim barcode (16 mer) and adapter (6+1 mer),
                        overall 23 mer, from R2 prefix. If you choose not to
                        trim R2, set this parameter to False or 0.
                        [default:True]
  -d NONDUP, --deduplicate NONDUP
                        Get non-duplicate read pairs for each barcode and keep
                        BX size larger than 3 reads. If you do not want to de-
                        duplicate, please set "-d 0". [default:3]
  -t THREADS, --threads THREADS
                        Number of threads. Input 0 will use all
                        threads.(default: 1)
  -h, --help            show this help message and exit

```
#### *Step II. Filter alignment by CIGAR match quality and MAPQ score*
If you choose to run the three steps of Preprocess module separately: this step should be performed before running LAB-Gap Filling, LCB-Scaffolding or GCB-Scaffolding.

**Input:**

- Draft assembly

**Output:**

- A SAM file of filtered read-to-scaffold alignment with CIGAR match quality >= 70 and Mapping Quality = 60 
> File name ends with *C70M60.sam*
```
usage: python /opt/10x_program/step2_alignment_andFilter.py [-h] -a {bwa_mem,kart} -g GENOME -f1 FASTQ_R1 -f2 FASTQ_R2 
                                                     -o OUTPUTPREFIX [-c CIGAR_MAP_QUALITY] [-m MAPQ] [-t THREADS]
positional arguments:
  -a {bwa_mem,kart}, --aligner {bwa_mem,kart}
                        Align reads to scaffolds. Choose one of the following
                        aligners: BWA MEM or Kart.
  -g GENOME, --genome GENOME
                        Path of genome fasta.
  -f1 FASTQ_R1, --fastqR1 FASTQ_R1
                        Path of qualified and reformated Fastq R1.
  -f2 FASTQ_R2, --fastqR2 FASTQ_R2
                        Path of qualified and reformated Fastq R2.
  -o OUTPUTPREFIX, --outputPrefix OUTPUTPREFIX
                        Prefix of output samfile. Do not use "." in this
                        prefix. ie, -o PROJ_NAME will get PROJ_NAME.sam &
                        PROJ_NAME_ProperPair.sam & PROJ_NAME_C70M60.sam
optional arguments:
  -h, --help            show this help message and exit
  -c CIGAR_MAP_QUALITY, --CIGAR CIGAR_MAP_QUALITY
                        CIGAR match quality: count([M=X])/len(seq)*100. From 1
                        to 100. [default: 70]
  -m MAPQ, --MAPQ MAPQ  MAPQ score. From 0 to 60. [default: 60]
  -t THREADS, --threads THREADS
                        Number of threads. Input 0 will use all
                        threads.[default: 1]
```
#### *Step III. Generate barcode information on each scaffold*
If you choose to run the three steps of Preprocess module separately: this step should be performed before running LCB-Scaffolding or GCB-Scaffolding.

**Input:**
- Read-to-assembly filtered SAM file from Step II

**Output:**
- A TSV file for possible scaffold pairs and shared barcode counts 
> File name ends with *C70M60_ScafA_ScafB_BXCnt.tsv*
- A TSV file for barcodes on every scaffold end
> File name ends with *C70M60_ScafHeadTail_BX_pairSum.tsv*

```
usage: python /opt/10x_program/step3_process_samfile.py [-h] -f FASTA [-r RANGE] -s SAM
                                                 [--max_number_of_scafendcnt SCAFENDCNT]
                                                 [--min_rp THRESHOLD_OF_BX_RP_PER_END] 
                                                 [--rpN ESTABLISH_COMBINATION_OF_READ_PAIR]
                                                 [--BXN ESTABLISH_NUMBER_OF_BX] [-g WITHGAP] [-t THREADS]

positional arguments:
  -f FASTA, --fasta FASTA
                        Reference genome. (fasta format, named as .fa or .fasta)
  -s SAM, --sam SAM     Path of the SAM file you want to process.

optional arguments:
  -h, --help            show this help message and exit
  -r RANGE, --range RANGE
                        Range length of each scaffold Head/Tail.
                        [default:20000]
  --max_number_of_scafendcnt SCAFENDCNT
                        Max number of Scaffold ends sharing the same BX.
                        [default:0]
  --min_rp THRESHOLD_OF_BX_RP_PER_END
                        Threshold of BX read pair/per end. The certified
                        barcode control value. If the value is set to 5 means
                        barcode mapping to genome must be supported by 6 read
                        pair. [default:0]
  --rpN ESTABLISH_COMBINATION_OF_READ_PAIR
                        Establish Number of barcode read pair at each scaffold
                        end. If value is set to 10 means this program will
                        generate combination from (1,1) to (10*,10*).
                        [default:10]
  --BXN ESTABLISH_NUMBER_OF_BX
                        Determines the number of barcode supporting pairs of
                        scaffold end. If the value is set to 20 means this
                        program will generate BX1 to BX20*. [default:20]
  -g WITHGAP, --withgap WITHGAP
                        Determines whether the Head/Tail range is with or
                        without gap. True means that the Head/Tail range will
                        be counted with gap. If you input [-r 20000], then the
                        Head/Tail range will be calculated from both ends
                        (Head/Tail) to 20000 base, and there may be some gaps
                        in said range(N base). False means that the Head/Tail
                        range will count without gap (N base). (default: True)
  -t THREADS, --threads THREADS
                        Number of threads you want to use. Input 0 will use
                        all threads.(default: 1)

```
#### *Run Step I to III altogether*
This program is the combination of the three parts of the preprocess module:

```
usage: python /opt/10x_program/runStep1to3.py [-h] -f FASTQS -g GENOME --id PROJECTID -o OUTPUT_FOLDER
                      [-t THREADS] [-a {bwa_mem,kart}] [-q QUALITY]
                      [-l LENGTH] [--trimr2 {False,True}] [-d NONDUP]
                      [-c CIGAR_MAP_QUALITY] [-m MAPQ] [-r RANGE]
                      [-gap WITHGAP] [--detail {False,True}]
                      [--min_rp THRESHOLD_OF_BX_RP_PER_END]
                      [--rpN ESTABLISH_COMBINATION_OF_READ_PAIR]
                      [--BXN ESTABLISH_NUMBER_OF_BX]
positional arguments:
  -f FASTQS, --fastqs FASTQS
                        Input the folder path of FASTQ files which were
                        produced from longranger mkfastq. Files in the folder
                        must be named like: [Sample Name]_S1_L00[Lane
                        Number]_[Read Type]_001.fastq.gz. (see:
                        https://github.com/10XGenomics/longranger). If your
                        fastq files are in a different folder, please move to
                        the same folder. (input file must be in fastq format,
                        such as .fq or .fastq)
  -g GENOME, --genome GENOME
                        Path of genome fasta.
  --id PROJECTID        Input project name. It will be your output prefix.
  -o OUTPUT_FOLDER, --output_folder OUTPUT_FOLDER
                        Output folder path. Your output files will be in this
                        folder.
                        
optional arguments:
  -h, --help            show this help message and exit
  -t THREADS, --threads THREADS
                        Number of threads. Input 0 will use all
                        threads.[default: 1]
  -a {bwa_mem,kart}, --aligner {bwa_mem,kart}
                        Choose one of the following aligners: BWA MEM or Kart,
                        to align reads. BWA MEM: https://github.com/lh3/bwa,
                        KART: https://github.com/hsinnan75/Kart.
                        [default:bwa_mem]
  -q QUALITY, --quality QUALITY
                        Input the minimum quality score for quality trimming
                        of FASTQ files. Same as the option for trim_galore -q
                        [default:20]
  -l LENGTH, --length LENGTH
                        Input the minimum length for quality trimming of FASTQ
                        files. Same as the option for trim_galore --length
                        [default:50]
  --trimr2 {False,True}
                        (Optional) R2 trimming. For better mapping quality,
                        choose to trim barcode (16 mer) and adapter (6+1 mer),
                        overall 23 mer, from R2 prefix. If you choose not to
                        trim R2, set this parameter to False or 0.
                        [default:True]
  -d NONDUP, --deduplicate NONDUP
                        (Optional) Get non-duplicate read pairs for each
                        barcode and keep BX size larger than 3 reads. If you
                        do not want to de-duplicate, please set "-d 0".
                        [default:3]
  -c CIGAR_MAP_QUALITY, --CIGAR CIGAR_MAP_QUALITY
                        CIGAR match quality: count([M=X])/len(seq)*100. From 1
                        to 100. [default: 70]
  -m MAPQ, --MAPQ MAPQ  MAPQ score. From 0 to 60. [default: 60]
  -r RANGE, --range RANGE
                        The range length of each scaffold Head/Tail.
                        [default:20000]
  -gap WITHGAP, --withgap WITHGAP
                        Determines whether the Head/Tail range is with or
                        without gap. True means the Head/Tail range will be
                        counted with gap. If you input [-r 20000], then the
                        Head/Tail range will be calculated from both ends
                        (Head/Tail) to 20000 base, and there may be some gaps
                        in that range (N base). False means the Head/Tail
                        range will be counted without gap (N base). [default:
                        True]
  --detail {False,True}
                        Print out the details of process log. [default: False]
  --min_rp THRESHOLD_OF_BX_RP_PER_END
                        Threshold of BX read pair/per end. The certified
                        barcode control value. If the value is set as 5, this
                        means the number of barcodes mapping to genome must be
                        supported by 6 read pairs. [default:0]
  --rpN ESTABLISH_COMBINATION_OF_READ_PAIR
                        Establish Number of barcode read pairs at each end of
                        the scaffolds. If the value is 10, this means this
                        program will generate a combination ranging from (1,1)
                        to (10*,10*). [default:10]
  --BXN ESTABLISH_NUMBER_OF_BX
                        Determine the number of barcodes supporting scaffold
                        ends. If the value is set as 20, this means this
                        program will generate BX1 to BX20*. [default:20]

```
### § Main Module 1 - LCB Gap Filling:
If you choose to run the Preprocess module separately, you have to at least run Preprocess Step II before LCB Gap Filling.

#### *Step I. Produce barcode list for each gap*

**Input:**

- Read-to-scaffold alignment SAM file from Preprocess step II
- Draft assembly FASTA file

**Output:**

- Directory for each scaffold containing:
  - Position of reads mapped onto this scaffold 
  > File name ends with *C70M60ReadPos.tsv*
  - Position of gaps on this scaffold
  > File name: *gap_pos_{scafname}.txt*
  - Barcode list of each gap
  > File name: *gapRange_BXs.txt*
-  ProduceBXList_Record.log

```
        Usage: /opt/LCB_GapFilling/ProduceBXList.sh -f SAMFILE -a FASTA -o OUTDIR [-n NUM ]
        [-t THREADS] [-rp MIN_READPAIR_onScaf] [-b BarcodeNonDupReadPairCnt.txt ] [-q  JobQueue]
        [-c MIN_BarcodeList ] [-s MIN_READPAIR_onGap]

        positional arguments:
                -f SAMFILE
                   High quality read-to-scaffold alignment produced from Preprocess Step II
                   (C70M60.sam)
                -a FASTA
                   Draft assembly with gaps to be filled
                -o OUTDIR
                   Output directory

       optional arguments:
                -n NUM
                   Only perform Gap Filling on top n scaffolds, [default=ALL]
                -t THREADS
                   Number of threads [default=16]
                -rp MIN_READPAIR_onScaf
                   Minimum number of read pairs for barcode on scaffold [default=3]
                -s MIN_READPAIR_onGap
                   Minimum number of read pair for barcode on gap’s flanking [default=2]
                -c MIN_BarcodeList
                   Minimum size of gap’s barcode list [default=10]
                -b BarcodeNonDupReadPairCnt.txt
                   BarcodeNonDupReadPairCnt.txt by preprocessing of reads;
                   only required if there are more than one job queue
                -q  JobQueue
                    Number of job queues for gap-filling [default=1]; required if you want to
                    run GABOLA in parallel on several machines.
                    For example, if you set the parameter as “-q 3”,
                    our program will produce three lists: JobList1, JobList2 and JobList3.
                    Each list containing the gaps to be filled with the format of <scafname><gapID>

```
#### *Step II. Assemble contigs for each gap from barcode list*

**Input:**

- Non-duplicate split reads FASTQ file produced by Preprocess Step I

**Output:**

- Directory for each scaffold containing a FASTA file of assembled contigs for each gap
> i.e. *scaffolds_gapID.fasta*
- Assemble_JobID_Record.log
```
        Usage: /opt/LCB_GapFilling/Assemble.sh -r FASTQ [-t THREADS] [-q JobQueue_NUM ] -o OUTDIR

              positional arguments:
                         -r FASTQ
                            Directory for non-duplicated split fastq from Preprocess step I
                         -o OUTDIR
                            Output directory (same as outdir of ProduceBXList.sh)

              optional arguments:
                         -t THREADS
                            Number of tasks/gaps per run (one task occupies 10 threads);
                            depending on how many available CPUs [default=8]
                         -q JobQueue_NUM
                            Index of Task queue [default=1]

```
#### *Step III. Fill gaps with best aligned contig*

**Input**
- Draft assembly FASTA file

**Output**

- LCB Gap-filled FASTA file
> *i.e. {draft_assembly_prefix}_LCBFilled.fa*
- LCBFill_Record.log

```
        usage: /opt/LCB_GapFilling/Fill.sh -a FASTA [-n NUM ] [-t THREADS ] [-l MIN_LEN]
        [-c MIN_COV] [-d MAX_DIS] [-M MIN_MAPLEN] [-I MIN_MAPIDENTITY] -o OUTDIR

        positional arguments:
                   -a FASTA
                      Draft assembly FASTA file to be gap-filled
                   -o OUTDIR
                      Output directory (same as outdir of ProduceBXList.sh)

        optional arguments:
                   -n NUM
                      Top n scaffolds [default=ALL]
                   -t THREADS
                      Number of threads [default=8]
                   -l MIN_LEN
                      Minimum length of assembled contigs [default=1000]
                   -c MIN_COV
                      Minimum coverage of assembled contigs, this property is produced by
                      SPAdes assembler [default=2]
                   -d MAX_DIS
                      Maximum distance between flanking and contigs [default=30000]
                   -M MIN_MAPLEN
                      Minimum mapped length of contigs [default=300]
                   -I MIN_MAPIDENTITY
                      Minimum mapping identity of contigs [default=80]
```
### § Main Module 2 – GCB Gap Filling:

#### *Fill gaps with contigs generated by other sequence assemblers*

**Input:**
- Draft assembly FASTA file
- G-contigs FASTA file (contigs generated by other sequence assemblers)

**Output:**
- For each scaffold:
  - contigRange.txt
  - gapFilling.log
- GCBFilled_Record.log
- Gap-filled FASTA file
> File name as *{draft_assembly_prefix}_GCBFilled.fasta*

```
        usage: /opt/GCB_GapFilling/Fill.sh -a FASTA -g G_CONTIGS [-t THREADS] -o OUTDIR
        [-M MIN_MAPLEN] [-I MIN_MAPIDENTITY]

        positional arguments:
                   -a FASTA
                      Draft assembly FASTA file to be filled
                   -g G_CONTIGS
                      Scaffolds/contigs from other assemblies or long reads for filling in gaps
                   -o OUTDIR
                      Output directory

        optional arguments:
                   -t THREADS
                      Number of threads [default=16]
                   -M MIN_MAPLEN
                      Minimum mapped length of contigs [default=500]
                   -I MIN_MAPIDENTITY
                      Minimum mapping identity of contigs [default=80]
```
### § Main Module 3 – LCB Scaffolding:
If you choose to run the Preprocess module separately, you have to at least run both Preprocess Step II & III before LCB Scaffolding.

#### *Step I. Collect barcodes for each candidate scaffold end pair*

**Input:**

- Scaffold pair TSV file from Preprocess module step III
- Barcode list on scaffold ends TSV file from Preprocess module step III

**Output:**
- New directory HeadTail_BXList/ containing:
  - Barcode list on head and tail for every scaffold
- Candidate scaffold pair TSV file
 > File name ends with *C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv*

```
        usage: /opt/LCB_Scaffolding/CandidatePair.sh
               -f SCAFFOLD_PAIR -p BXLIST [-v PAIR_NUM] -o OUTDIR

        positional arguments:
               -f SCAFFOLD_PAIR
                  Scaffold pairs and shared barcode counts from Preprocess module step III
                  (C70M60_ScafA_ScafB_BXCnt.tsv)
               -p BXLIST
                  Barcodes on every scaffold end from Preprocess module step III
                  (C70M60_ScafHeadTail_BX_pairSum.tsv)
               -o OUTDIR
                  Output directory

        optional arguments:
               -v PAIR_NUM
                  Remove multiple ends, keep top v pairs [default=2]
```
#### *Step II. Assemble contigs for each candidate scaffold end pair*

**Input:**

- Candidate scaffold pair TSV file from Step I.
- Non-duplicate split reads FASTQ files from Preprocess module Step I

**Output:**

- Directory for each scaffold end pair:
  - Barcode list
  - FASTA file of assembled contigs
- Assemble.log 
```
        usage: /opt/LCB_Scaffolding/Assemble.sh
               -f SCAFFOLD_PAIR -r FASTQ [-t THREADS] [-n NUM] -o OUTDIR

        positional arguments:
               -f SCAFFOLD_PAIR
                  Candidate scaffold pairs and shared barcode counts
                  (C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv)
               -r FASTQ
                  Directory for non-dup split fastq from Preprocess module step I
               -o OUTDIR
                  Output directory (same as Preprocessing.sh)

        optional arguments:
               -t THREADS
                  Number of tasks/scaffold pairs per run (one task occupies 10 threads);
                  depending on how many available CPUs [default=8]
               -n NUM
                  Process top n scaffold end pairs [default=2000]
```
#### *Step III. Connect scaffold end pairs*

**Input:**
- Candidate scaffold pair TSV file from Step I
- Draft Assembly FASTA file
> sequences in fasta have to be named in the format of *scaffold{num}|size{num}*
 
**Output:**

- LCB-scaffolded FASTA file
> i.e. *{draft_assembly_prefix}_LCBScaffold.fa*
- LCB-scaffolded FASTA file renamed by length
> i.e. *{draft_assembly_prefix}_LCBScaffold_rename.fa*
- NewScaf.log recording new scaffolds connected by LAB Scaffolding and their components
- LCBScaffold.log
```
        usage: /opt/LCB_Scaffolding/Scaffolding.sh
               -f SCAFFOLD_PAIR -a FASTA [-l MIN_MAPLEN] [-c MIN_MAPIDENTITY] -o OUTDIR

        positional arguments:
               -f SCAFFOLD_PAIR
                  Candidate scaffold pairs and shared barcode counts
                  (C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv)
               -a FASTA
                  Draft assembly for LAB scaffolding
               -o OUTDIR
                  Output directory (same as Preprocessing.sh)

        optional arguments:
               -l MIN_MAPLEN
                  Minimum mapped length for contig on each scaffold end [default=1000]
               -c MIN_MAPIDENTITY
                  Mapping identity for contigs on each scaffold end [default=70]
```
## § Main Module 4 – GCB Scaffolding:
If you choose to run the Preprocess module separately, you have to at least run both Preprocess Step II & III before GCB Scaffolding.

#### *Step I. Collect barcodes for each candidate scaffold end pair*

**Input:**

- Scaffold pair TSV file from Preprocess module step III
- Barcode list on scaffold ends TSV file from Preprocess module step III

**Output:**

- New directory HeadTail_BXList/:
  - Barcode list on head and tail for every scaffold *{scafname}_Head/Tail_BXList*
- Candidate scaffold pair TSV file *C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv*

```
        usage: /opt/GCB_Scaffolding/CandidatePair.sh
               -f SCAFFOLD_PAIR -p BXLIST [-v PAIR_NUM] -o OUTDIR

        positional arguments:
               -f SCAFFOLD_PAIR
                  Scaffold pairs and shared barcode counts from Preprocess module step III
                  (C70M60_ScafA_ScafB_BXCnt.tsv)
               -p BXLIST
                  Barcodes on every scaffold end from Preprocess module step III
                  (C70M60_ScafHeadTail_BX_pairSum.tsv)
               -o OUTDIR
                  Output directory

        optional arguments:
               -v PAIR_NUM
                  Remove multiple ends, keep top v pairs [default=2]

```

#### *Step II. Connect scaffold end pairs*

**Input:**

- Candidate scaffold pair TSV file from Step I
- Draft Assembly FASTA file
> sequences have to be named in the format of *scaffold{num}|size{num}*
- G-contigs FASTA file

**Output:**

- GCB-scaffolded FASTA file 
> *i.e. {draft_assembly_prefix}_GCBScaffold.fa*
- GCB-scaffolded FASTA file renamed by length
> *i.e. {draft_assembly_prefix}_GCBScaffold_rename.fa*
- NewScaf.log recording new scaffolds connected by GCB-Scaffolding and their components
- GCBScaffold.log

```
        usage: /opt/GCB_Scaffolding/Scaffolding.sh
               -f SCAFFOLD_PAIR -a FASTA -g G_CONTIGS [-l MIN_MAPLEN]
               [-c MIN_MAPIDENTITY] [-t THREADS] [-n NUM] -o OUTDIR

        positional arguments:
                -f SCAFFOLD_PAIR
                   Candidate scaffold pairs and shared barcode counts
                   (C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv)
                -a FASTA
                   Draft assembly for GCB scaffolding
                -x G_CONTIGS
                   Scaffolds/contigs from other assemblies or long reads for filling in gaps
                -o OUTDIR
                   Output directory (same as Preprocessing.sh)

        optional arguments:
                -l MIN_MAPLEN
                   Minimum mapped length for contig on each scaffold end [default=1000]
                -c MIN_MAPIDENTITY
                   Mapping identity for contigs on each scaffold end [default=70]
                -t THREADS
                   Number of tasks/scaffold pairs per run;
                   depending on how many available CPUs [default=40]
                -n NUM
                   Process top n scaffold end pairs [default=2000]

```
