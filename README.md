# LOCLA: Local Optimization for Chromosome-Level Assembly

#### The docker image of LOCLA is freely accessible on [**our dockerhub website**](https://hub.docker.com/repository/docker/lsbnb/locla/general)

## Introduction:

LOCLA is a novel genome assembly optimization tool, LOCLA, that iteratively enhances the quality of an assembly by locating sequencing reads on partially assembled scaffolds and thus enable gap filling and further scaffolding. LOCLA utilizes reads of diverse sequencing techniques, e.g.,10x Genomics (10xG) Linked-Reads, [**PacBio HiFi Reads](https://www.pacb.com/technology/hifi-sequencing) and [**Oxford Nanopore Technologies**](https://nanoporetech.com/). 
**In our own experiments, we assembled 10x Genomics (10xG) Linked-Reads via [**Supernova assembler**](https://support.10xgenomics.com/de-novo-assembly/software/overview/latest/welcome) and TGS reads via [**Canu**](https://github.com/marbl/canu). Additional tools incorporated in our pipeline include *Hybrid Scaffold* developed by [**Bionano Genomics**]( https://bionanogenomics.com/ ) and [*RagTag*](https://github.com/malonge/RagTag) **

## Usage:

The following are the minimum required commands to run each main module separately:

#### 1. LCB Gap Filling:
```
  python /opt/10x_program/runStep1to3.py -f raw_fastq_dir/ -g draft.fa --id PROJECTID -a bwa_mem -o 10x_preprocess/
  /opt/LCB_GapFilling/ProduceBXList.sh -f draft_bwa_mem_C70M60.sam -a draft.fa -o LCB_GapFilling/
  /opt/LCB_GapFilling/Assemble.sh -r /10x_preprocess/nonDupFq/split -o LCB_GapFilling/
  /opt/LCB_GapFilling/Fill.sh -a draft.fa -o LCB_GapFilling/
```
#### 2. GCB Gap Filling:
It can be a stand alone module without any 10x Genomics resource.
```
  /opt/GCB_GapFilling/Fill.sh -a draft.fa -g g-contigs.fa -o GCB_GapFilling/

```
#### 3. LCB Scaffolding:
```
  #rename draft.fa by length first to fit our format
  /opt/Rename_byLength.sh -a draft.fa -o LCB_Scaffolding/
  python /opt/10x_program/runStep1to3.py -f raw_fastq_dir/ -g draft_rename.fa --id PROJECTID -a bwa_mem -o 10x_preprocess/
  /opt/LCB_Scaffolding/CandidatePair.sh -f draft_bwa_mem_C70M60_ScafA_ScafB_BXCnt.tsv -p draft_bwa_mem_C70M60_ScafHeadTail_BX_pairSum.tsv -o LCB_Scaffolding/
  /opt/LCB_Scaffolding/Assemble.sh -f draft_bwa_mem_C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv -r 10x_preprocess/nonDupFq/split -o LCB_Scaffolding/
  /opt/LCB_Scaffolding/Scaffolding.sh -f draft_bwa_mem_C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv -a draft_rename.fa -o LCB_Scaffolding/
```

#### 4. GCB Scaffolding:
```
  #rename draft.fa by length first to fit our format
  /opt/Rename_byLength.sh -a draft.fa -o GCB_Scaffolding/
  python /opt/10x_program/runStep1to3.py -f raw_fastq_dir/ -g draft_rename.fa --id PROJECTID -a bwa_mem -o 10x_preprocess/ 
  /opt/GCB_Scaffolding/CandidatePair.sh -f draft_rename_bwa_mem_C70M60_ScafA_ScafB_BXCnt.tsv -p draft_rename_bwa_mem_C70M60_ScafHeadTail_BX_pairSum.tsv -o GCB_Scaffolding/
  /opt/GCB_Scaffolding/Scaffolding.sh -f draft_gcbgf_lbgf_labs_bwa_mem_C70M60_ScafA_ScafB_BXCnt_rmMultiEnd.tsv -a draft_rename.fa -g g-contigs.fa -o GCB_Scaffolding/
```

The four main modules can be used in any order and iterated several times.The source codes are accessible to all.

For advanced usage see [**TECHNOTES.md**](TECHNOTES.md).
