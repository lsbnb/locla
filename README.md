# locla
LOCLA: Local Optimization for Chromosome-Level Assembly
#### The docker image of LOCLA is freely accessible on [**our dockerhub website**]([https://hub.docker.com/repository/docker/lsbnb/locla/general])

## Introduction:

LOCLA is a novel genome assembly optimization tool, LOCLA, that iteratively enhances the quality of an assembly by locating sequencing reads on partially assembled scaffolds and thus enable gap filling and further scaffolding. **In our own experiments, we used [**Supernova assembler**](https://support.10xgenomics.com/de-novo-assembly/software/overview/latest/welcome) first to generate an initial haplotype genome draft. Then, we incorporated *Hybrid Scaffold* developed by [**Bionano Genomics**]( https://bionanogenomics.com/ ) to expand scaffold lengths before performing GABOLA. Aside from linked reads, we also used long reads obtained from [**PacBio SMRT Sequencing**](https://www.pacb.com/smrt-science/smrt-sequencing/) and [**Oxford Nanopore Technologies**](https://nanoporetech.com/) for gap-filling. The assembly tool for long reads utilized here is [**Canu**](https://github.com/marbl/canu).**
