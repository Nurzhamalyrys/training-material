---
layout: tutorial_hands_on
title: VGP assembly pipeline - short version
zenodo_link: 'https://zenodo.org/record/5887339'
enable: false
level: Intermediate
tags:
 - pacbio
 - eukaryote
 - VGP
questions:
- "what combination of tools can produce the highest quality assembly of vertebrate genomes?"
- "How can we evaluate how good it is?"
objectives:
- "Learn the tools necessary to perform a de novo assembly of a vertebrate genome"
- "Evaluate the quality of the assembly"
time_estimation: '1h'
key_points:
- "The VGP pipeline allows to generate error-free, near gapless reference-quality genome assemblies"
- "The assembly can be divided in four main stages: genome profile analysis, HiFi long read phased assembly with hifiasm, Bionano hybrid scaffolding and Hi-C hybrid scaffolding"
contributors:
- delphine-l
- astrovsky01
- gallardoalba
- pickettbd
---


# Introduction
{:.no_toc}

The Vertebrate Genome Project (VGP), emerged from the G10K Consortium, aims to generate high-quality, near error-free, gap-free, chromosome-level, haplotype-phased, annotated reference genome assemblies for every vertebrate species ({% cite Rhie2021 %}). VGP has developed a fully automated *de-novo* genome assembly pipeline, which uses a combination of three different technologies: Pacbio HiFi, Bionano optical maps and Hi-C chromatine interaction maps.

As a result of the collaboration with the VGP team, a training including a step-by-step detailed description was developed for the Galaxy Training Network ({% cite Lariviere2022 %}). However, due to its complexy, it can be too time consuming for those who are not interested in understanding each of the analysis stages in depth. For this reason, we decided to make available to the community a workflow-centered version of the training.

The Galaxy Workflow System (GWS) facilitates analysis repeatability, allowing to minimize the number of manual steps required to execute an analysis workflow and automatizing the process of input parameter and software tool version tracking. The objetive of this training is to explain how to run the VGP workflow, focusing on what are the required inputs and which outputs are generated and delegating how the steps re executed to the GWS.

> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Getting Started on Galaxy

This tutorial assumes you are comfortable getting data into Galaxy, running jobs, managing history, etc. If you are unfamiliar with Galaxy, we recommed you to visit the [Galaxy Training Network](https://training.galaxyproject.org). Consider starting with the following trainings:
- [Introduction to Galaxy](https://training.galaxyproject.org/training-material/topics/introduction/slides/introduction.html)
- [Galaxy 101](https://training.galaxyproject.org/training-material/topics/introduction/tutorials/galaxy-intro-101/tutorial.html)
- [Getting Data into Galaxy](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/get-data/slides.html)
- [Using Dataset Collections](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/collections/tutorial.html)
- [Introduction to Galaxy Analyses](https://training.galaxyproject.org/training-material/topics/introduction)
- [Understanding the Galaxy History System](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/history/tutorial.html)
- [Downloading and Deleting Data in Galaxy](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/download-delete-data/tutorial.html)


# VGP assembly workflow structure

The VGP assembly pipeline has a modular organization, consisting in five main subworkflows (fig. 1), each one integrated by a series of data manipulation steps. Firstly, it allows the evaluation of intermediate steps, which facilitates the modification of parameters if necessary, without the need to start from the initial stage. Secondly, it allows to adapt the workflow to the available data.

> ![Figure 1: VGP pipeline modules](../../images/vgp_assembly/VGP_workflow_modules.png "VGP assembly pipelie. The implemented version of the VGP workflow is modular, consisting in five main independent subworkflows. In addition, it includes some additional workflows (not shown in the figure), required for exporting the results to Genome Ark.")

The VGP pipeline integrates two workflows to generate scaffolds from the contig level assemblies generated from the HiFi reads. When Hi-C data and Bionano data are available, the default pipeline will run the Bionano workflow first, followed by the Hi-C workflow. However, it is possible that Bionano data may not be available, in which case the HiC workflow can be used directly on the initial assembly, taking advantage of the modular characteristics of the pipeline.

> ### {% icon comment %} Input option order
> This tutorial assumes the input datasets are high-quality. QC on raw read data should be performed before it is used. QC on raw read data is outside the scope of this tutorial.
{: .comment}

## Get data

The first step is to get the datasets from Zenodo. The VGP assembly pipeline uses data generated by a variety of technologies, including PacBio HiFi reads, Bionano optical maps, and Hi-C chromatin interaction maps.
    
> ### {% icon hands_on %} Hands-on: Data upload
>
> 1. Create a new history for this tutorial
> 2. Import the files from [Zenodo]({{ page.zenodo_link }})
>
>    - Open the file {% icon galaxy-upload %} __upload__ menu
>    - Click on **Rule-based** tab
>    - *"Upload data as"*: `Datasets`
>    - Copy the tabular data, paste it into the textbox and press <kbd>Build</kbd>
>
>       ```
>   Hi-C_dataset_F   https://zenodo.org/record/5550653/files/SRR7126301_1.fastq.gz?download=1   fastqsanger.gz    Hi-C
>   Hi-C_dataset_R   https://zenodo.org/record/5550653/files/SRR7126301_2.fastq.gz?download=1   fastqsanger.gz    Hi-C
>   Bionano_dataset    https://zenodo.org/record/5550653/files/bionano.cmap?download=1   cmap    Bionano
>       ```
>
>    - From **Rules** menu select `Add / Modify Column Definitions`
>       - Click `Add Definition` button and select `Name`: column `A`
>       - Click `Add Definition` button and select `URL`: column `B`
>       - Click `Add Definition` button and select `Type`: column `C`
>       - Click `Add Definition` button and select `Name Tag`: column `D`
>    - Click `Apply` and press <kbd>Upload</kbd>
>   
> 3. Import the remaining datasets from [Zenodo]({{ page.zenodo_link }})
>
>    - Open the file {% icon galaxy-upload %} __upload__ menu
>    - Click on **Rule-based** tab
>    - *"Upload data as"*: `Collections`
>    - Copy the tabular data, paste it into the textbox and press <kbd>Build</kbd>
>
>       ```
>   dataset_01    https://zenodo.org/record/6098306/files/HiFi_synthetic_50x_01.fasta?download=1  fasta    HiFi  HiFi_collection
>   dataset_02    https://zenodo.org/record/6098306/files/HiFi_synthetic_50x_02.fasta?download=1  fasta    HiFi  HiFi_collection
>   dataset_03    https://zenodo.org/record/6098306/files/HiFi_synthetic_50x_03.fasta?download=1  fasta    HiFi  HiFi_collection
>       ```
>
>    - From **Rules** menu select `Add / Modify Column Definitions`
>       - Click `Add Definition` button and select `List Identifier(s)`: column `A`
>       - Click `Add Definition` button and select `URL`: column `B`
>       - Click `Add Definition` button and select `Type`: column `C`
>       - Click `Add Definition` button and select `Group Tag`: column `D`
>       - Click `Add Definition` button and select `Collection Name`: column `E`
>    - Click `Apply` and press <kbd>Upload</kbd>
>
{: .hands_on}


> ### {% icon details %} Working with your own data
>
> If working on a genome other than the example yeast genome, you upload the VGP data from the [VGP/Genome Ark AWS S3 bucket](https://genomeark.s3.amazonaws.com/index.html) as following:
>
> > ### {% icon hands_on %} Hands-on: Import data from Genome Ark
> >
> > 1. Open the file {% icon galaxy-upload %} __upload__ menu
> > 2. Click on **Choose remote files** tab
> > 3. Click in the **Genome Ark** buttom and then click in **species**
> {: .hands_on}
>
> You can find the VGP data following this path: `/species/${Genus}_${species}/${speciman_code}/genomic_data`. Inside a given datatype directory (e.g. `pacbio`), select all the relevant files individually until all the desired files are highlighted and click the <kbd>Ok</kbd> buttom. Note that there may be multiple pages of files listed. Also note that you may not want every file listed.
>
> {% snippet faqs/galaxy/collections_build_list.md %}
>
{: .details}

## Import workflows from WorkflowHub
    
Once we have imported the datasets, the next step is to import the VGP workflows from the [Workflowhub server](https://workflowhub.eu/). WorkflowHub is a workflow management system which allows workflows to be FAIR, citable, have managed metadata profiles, and be openly available for review and analytics.

> ### {% icon hands_on %} Hands-on: Import a workflow
>
> 1. Click in the **Workflow** menu, located in the top bar.
>   ![Workflow menu](../../images/vgp_assembly/top_bar.png)
> 2. Click in the <kbd>Import</kbd> buttom, located in the right corner.
> 3. In the section **Import a Workflow from Configured GA4GH Tool Registry Servers (e.g. Dockstore)**, click in *Search form*.
> 4. In the **TRS Server: *workflowhub.eu*** menu you should type `name:vgp`
>    ![Figure 3: Workflow menu](../../images/vgp_assembly/workflow_list.png)
> 5. Click in the desired workflow, and finally select the last available version.
{: .hands_on}

After that, the imported workflows will appear in the main workflow menu. In order to initialize the workflow, we just need to click in the {% icon workflow-run %} **Run workflow** icon, marked with a red square in the figure 2.

![Figure 2: Workflow menu](../../images/vgp_assembly/imported_workflows.png  "Workflow main menu. The workflow menu lists all the workflows that have been imported. It provides useful information for organizing the workflows, such as last update and the tags. The worklows can be run by clicking in the play icon, marked in red in the image.")

Once we have imported the datasets and the workflows, we can start with the genome assembly.

> ### {% icon comment %} Workflow-centric Research Objects
>
> In WorkfloHub, workflows are packaged, registered, downloaded and exchanged as Research Objects using the RO-Crate specification, with test and example data, managed metadata profiles, citations and more.
>
{: .comment}

# Genome profile analsysis

[{% icon exchange %} Switch to long version]({% link topics/assembly/tutorials/vgp_genome_assembly/tutorial.md %}#genome-profile-analysis)

Now that our data and workflows are imported, we can run our first workflow. Before the assembly can be run, we need to collect metrics on the properties of the genome under consideration, such as the expected genome size. The present pipeline uses **Meryl** for generating the k-mer database and **Genomescope** for determing the genome size based on a k-mer analysis.

> ### {% icon hands_on %} Hands-on: VGP genome profile analysis workflow
>
> 1. Click in the **Workflow** menu, located in the top bar
> 2. Click in the {% icon workflow-run %} **Run workflow** buttom corresponding to `VGP genome profile analysis`
> 3. In the **Workflow: VGP genome profile analysis** menu:
>   - {% icon param-collection %} "*Collection of Pacbio Data*": `7: HiFi_collection`
>   - "*K-mer length*": `32`
>   - "*Ploidy*": `2`
> 4. Click in the <kbd>Run workflow</kbd> buttom
>
{: .hands_on}

Once the workflow have finished, we can evaluate the linear plot generated by **Genomescope** (fig. 3), which includes valuable information such as k-mer profiles, fitted models and estimated parameters. This file corresponds to the dataset `26`.

![Figure 3: Genomescope plot](../../images/vgp_assembly/genomescope_plot.png "GenomeScope2 21-mer profile. The first peak located at coverage 21x corresponds to the heterozygous peak. The second peak at coverage 50x, corresponds to the homozygous peak. Estimate of the heterozygous portion is 0.637%. The plot also includes informatin about the the inferred total genome length (len), genome unique length percent (uniq), overall heterozygosity rate (het), mean k-mer coverage for heterozygous bases (kcov), read error rate (err), average rate of read duplications (dup) and k-mer size (k).")

This distribution is the result of the Poisson process underlying the generation of sequencing reads. As we can see, the k-mer profile follows a bimodal distribution, indicative of a diploid genome. The distribution is consistent with the theoretical diploid model (model fit > 93%). Low frequency *k*-mers are the result of sequencing errors. GenomeScope2 estimated a haploid genome size is around 11.7 Mb, a value reasonably close to *Saccharomyces* genome size. Additionally, it revealed that the variation across the genomic sequences is 0.69%.


# HiFi phased assembly with hifiasm

[{% icon exchange %} Switch to long version]({% link topics/assembly/tutorials/vgp_genome_assembly/tutorial.md %}#hifi-phased-assembly-with-hifiasm)

After the genome profiling, the next step is to run the **VGP HiFi phased assembly with hifiasm and HiC data workflow**. This workflow uses **hifiasm** to generate initial primary and alternate pseudohaplotype assemblies. In addition, this workflow includes three tools for evaluating the assembly completeness: **QUAST**, **BUSCO** and **Merqury**.

> ### {% icon hands_on %} Hands-on: VGP HiFi phased assembly with hifiasm and HiC data workflow
> 1. Click in the **Workflow** menu, located in the top bar
> 2. Click in the {% icon workflow-run %} **Run workflow** buttom corresponding to `VGP HiFi phased assembly with hifiasm and HiC data`
> 3. In the **Workflow: VGP HiFi phased assembly with hifiasm and HiC data** menu:
>   - {% icon param-collection %} "*Pacbio Reads Collection*": `7. HiFi_collection`
>   - {% icon param-file %} "*Meryl database*": `12: Meryl on data 11, data 10, data 9: read-db.meryldb`
>   - {% icon param-file %} "*HiC forward reads*": `3: Hi-C_dataset_F`
>   - {% icon param-file %} "*HiC reverse reads*": `2: Hi-C_dataset_R`
>   - {% icon param-file %} "*Genomescope summary dataset*": `19: Genomescope on data 13 Summary`
>   - "*K-mer length*": `32`
>   - "*Ploidy*": `2`
>   - "*Is genome large (>100Mb)?*": `No`
> 4. Click in the <kbd>Run workflow</kbd> buttom
>
> > ### {% icon comment %} Input option order
> > Note that the order of the input may differ slightly.
> {: .comment}
>
{: .hands_on}

Let's have a look at the HTML report generated by **QUAST** (fig. 4), which corresponds with the dataset  `52`. It summarizes the main statistics related with the assembly completeness.

![Figure 5: QUAST initial plot](../../images/vgp_assembly/QUAST_initial.png "QUAST report. Statistics of the primary and alternate assembly (a). Cumulative length plot (b).")

According to the report, both assemblies are quite similar; the primary assembly includes 33 contigs, whose cumulative length is around 23.5Mbp. On the other hand, the second alternate assembly includes 35 contigs, whose total lenght is 25.5Mbp. As we can see in the figure 4a, the assemblies are much larger than the estimated genome sizes (dotted line), which means that both include duplicated sequences.

> ### {% icon question %} Questions
>
> 1. What is the longest contig in the primary assembly? And in the alternate one?
> 2. What is the N50 of the primary assembly?
> 3. Which percentage of reads mapped to each assembly?
>
> > ### {% icon solution %} Solution
> >
> > 1. The longest contig in the primary assembly is 1.532.843 bp, and 1.532.843 bp in the alternate assembly.
> > 2. The N50 of the primary assembly is 922.430 bp.
> > 3. According to the report, 100% of reads mapped to both the primary assembly and the alternate assembly.
> >
> {: .solution}
>
{: .question}

Next, we are going to evaluate the outputs generated by **BUSCO**. This tool provides quantitative assessment of the completeness of a genome assembly in terms of expected gene content. It relies on the analysis of genes that should be present only once in a complete assembly or gene set, while allowing for rare gene duplications or losses ({% cite Simo2015 %}).

![Figure 6 : BUSCO](../../images/vgp_assembly/BUSCO_full_table.png "BUSCO full table. It contains the complete results in a tabular format with scores and lengths of BUSCO matches, and coordinates.")

As we can see in the report, the results are simplified into four categories: *complete and single-copy*, *complete and duplicated*, *fragmented* and  *Missing BUSCOs*.

> ### {% icon question %} Questions
>
> 1. How many complete BUSCO genes have been identified in the primary assembly?
> 2. How many BUSCOs genes are absent?
>
> > ### {% icon solution %} Solution
> >
> > 1. According to the report, our assembly contains the complete sequence of 2080 complete BUSCO genes (97.3%).
> > 2. 19 BUSCO genes are missing.
> >
> {: .solution}
>
{: .question}

Despite **BUSCO** being robust for species that have been widely studied, it can be inaccurate when the newly assembled genome belongs to a taxonomic group that is not well represented in [OrthoDB](https://www.orthodb.org/). Merqury provides a complementary approach for assessing genome assembly quality metrics in a reference-free manner via *k*-mer copy number analysis.

> ### {% icon hands_on %} Hands-on: *k*-mer based evaluation with Merqury
>
> 1. {% tool [Merqury](toolshed.g2.bx.psu.edu/repos/iuc/merqury/merqury/1.3) %} with the following parameters:
>    - *"Evaluation mode"*: `Default mode`
>        - {% icon param-file %} *"k-mer counts database"*: `Merged meryldb`
>        - *"Number of assemblies"*: `Two assemblies
>            - {% icon param-file %} *"First genome assembly"*: `Primary contigs FASTA`
>            - {% icon param-file %} *"Second genome assembly"*: `Alternate contigs FASTA`    
>
{: .hands_on}


By default, **Merqury** generates three collections as output: stats, plots and QV stats. The "stats" collection contains the completeness statistics, while the "QV stats" collection contains the quality value statistics. Let's have a look at the compy number (CN) spectrum plot, known as the *spectra-cn* plot (fig. 6).

![Figure 7: Merqury plot](../../images/vgp_assembly/merqury_cn_plot.png "Merqury CN plot corresponding to the primary assembly. This plot tracks the multiplicity of each k-mer found in the Hi-Fi read set and colors it by the number of times it is found in a given assembly. Merqury connects the midpoint of each histogram bin with a line, giving the illusion of a smooth curve."){:width="80%"}

As suggested previously by the report generated by **QUAST** (fig. 5), the assemblies contain a large proportion of duplicated reads. The red area represents one-copy *k*-mers in the genome, while the blue area represents two-copy *k*-mers, originating from haplotype-specific duplications. From that figure we can state that the sequencing coverage is around 50x.

# Post-assembly processing

[{% icon exchange %} Switch to long version]({% link topics/assembly/tutorials/vgp_genome_assembly/tutorial.md %}#post-assembly-processing)

An ideal haploid representation would consist of one allelic copy of all heterozygous regions in the two haplomes, as well as all hemizygous regions from both haplomes ({% cite Guan2019 %}). However, in highly heterozygous genomes, assembly algorithms are frequently not able to identify the highly divergent allelic sequences as belonging to the same region, resulting in the assembly of those regions as separate contigs. In order to prevent potential issues in downstream analysis, we are going to run the **VGP purge assembly with purge_dups workflow**, which will allow to identify and reassign allelic contigs.

> ### {% icon hands_on %} Hands-on: VGP purge assembly with purge_dups pipeline  workflow
>
> 1. Click in the **Workflow** menu, located in the top bar
> 2. Click in the {% icon workflow-run %} **Run workflow** buttom corresponding to `VGP purge assembly with purge_dups pipeline`
> 3. In the **Workflow: VGP purge assembly with purge_dups pipeline** menu:
>   - {% icon param-file %} "*Hifiasm Primary assembly*": `39: Hifiasm HiC hap1`
>   - {% icon param-file %} "*Hifiasm Alternate assembly*": `40: Hifiasm HiC hap2`
>   - {% icon param-collection %} "*Pacbio Reads Collection - Trimmed*": `22: Cutadapt`
>   - {% icon param-file %} "*Genomescope model parameters*": `20: Genomescope on data 13 Model parameters`
> 4. Click in the <kbd>Run workflow</kbd> buttom
>
{: .hands_on}

This workflow generates a large number of outputs, among which we should highlight the datasets `74` and `91`, which correspond to the purged primary and alternative assemblies respectively.

# Hybrid scaffolding with Bionano optical maps

[{% icon exchange %} Switch to long version]({% link topics/assembly/tutorials/vgp_genome_assembly/tutorial.md %}#hybrid-scaffolding-using-bionano-data)

Once the assemblies generated by **hifiasm** has been purged, the next step is to run the **VGP hybrid scaffolding with Bionano optical maps workflow**. It will allow to integrate the information provided by optical maps with primary assembly sequences in order to detect and correct structural variants, such as chimeric joins and misoriented contigs. In addition, this workflow includes some additonal steps from evaluating the outputs. 

> ### {% icon hands_on %} Hands-on: VGP hybrid scaffolding with Bionano optical maps workflow
>
> 1. Click in the **Workflow** menu, located in the top bar
> 2. Click in the {% icon workflow-run %} **Run workflow** buttom corresponding to `VGP hybrid scaffolding with Bionano optical maps`
> 3. In the **Workflow: VGP hybrid scaffolding with Bionano optical maps** menu:
>   - {% icon param-file %} "*Bionano data*": `1: Bionano_dataset`
>   - {% icon param-file %} "*Hifiasm Purged Assembly*": `TO DO`
>   - {% icon param-file %} "*Estimated genome size - Parameter File*": `60: Estimated Genome size`
>   - "*Is genome large (>100Mb)?*": `No`
> 4. Click in the <kbd>Run workflow</kbd> buttom
{: .hands_on}

Once the workfow have finished, let's have a look at the assembly reports.

As we can observe in the cumulative plot of the file `119` (fig. 7a), the total length of the assembly (12.160.926 bp) is slightly larger than the expected genome size. With respect to the NG50 statistic (fig. 10b), the value is 922.430 bp, which is significantly higher than the value obtained during the first evaluation stage (813.039 bp).
        
![Figure 10: QUAST and BUSCO plots](../../images/vgp_assembly/QUAST_cummulative.png ". Cumulative length plot (a). NGx plot. The y-axis represents the NGx values in Mbp, and the x-axis is the percentage of the genome (b). Assembly evaluation after runnig Bionano. BUSCO genes are defined as "Complete (C) and single copy (S)" when are found once in the single-copy ortholog database, "Complete (C) and duplicated (D)" when single-copy ortholog genes which were found more than once, "Fragmented (F)" when genes are matching just partially to a single-copy ortholog DB, and "Missing (M)" when genes which are expected but were not detected (c).")
    
It is also recommended to examine **BUSCO** outputs. In the summary image (fig. 7c), which can be found in the daset `117`, we can appreciate that most of the universal single-copy orthologs are present in our assembly.

> ### {% icon question %} Questions
>
> 1. How many scaffolds are in the primary assembly after the hybrid scaffolding?
> 2. What is the size of the largest scaffold? Has improved with respect to the previous evaluation?
> 3. What is the percertage of completeness on the core set genes in BUSCO? Has increased the completeness?
>
> > ### {% icon solution %} Solution
> >
> > 1. The number of contigs is 17.
> > 2. The largest contig is 1.531.728 bp long. This value hasn't changed.
> > 3. The percentage of complete BUSCOs is 95.7%. Yes, it has increased, since in the previous evaluation the completeness percentage was 88.7%.
> >
> {: .solution}
>
{: .question}
    

# Hybrid scaffolding with Hi-C data

[{% icon exchange %} Switch to long version]({% link topics/assembly/tutorials/vgp_genome_assembly/tutorial.md %}#hybrid-scaffolding-based-on-hi-c-mapping-data)

 In this final stage, we will run the **VGP hybrid scaffolding with HiC data**, which exploit the fact that the contact frequency between a pair of loci strongly correlates with the one-dimensional distance between them with the objective of linking the Bionano scaffolds to a chromosome scale by using **SALSA2**.

> ### {% icon hands_on %} Hands-on: VGP hybrid scaffolding with HiC data
>
> 1. Click in the **Workflow** menu, located in the top bar
> 2. Click in the {% icon workflow-run %} **Run workflow** buttom corresponding to `VGP hybrid scaffolding with HiC data`
> 3. In the **Workflow: VGP hybrid scaffolding with HiC data** menu:
>   - {% icon param-file %} "*Scaffolded Assembly*": `114: Concatenate datasets on data 110 and data 109`
>   - {% icon param-file %} "*HiC Forward reads*": `3: Hi-C_dataset_F (as fastqsanger)`
>   - {% icon param-file %} "*HiC Reverse reads*": `2: Hi-C_dataset_R (as fastqsanger)`
>   - {% icon param-file %} "*Estimated genome size - Parameter File*": `50: Estimated Genome size`
>   - "*Is genome large (>100Mb)?*": `No`
> 4. Click in the <kbd>Run workflow</kbd> buttom
{: .hands_on}

In order to evaluate the Hi-C hybrid scaffolding, we are going to compare the contact maps before and after running the HiC hybrid scaffolding workflow (fig. 8), corresponding to the datasets `130` and `141` respectively.
  
![Figure 8: Pretext final contact map](../../images/vgp_assembly/hi-c_pretext_final.png "Hi-C map generated by Pretext after the hybrid scaffolding based on Hi-C data. The red circles indicate the  differences between the contact map generated after (a) and before (b) Hi-C hybrid scaffolding.")

Among the most notable differences that can be identified between the contact maps, it can be highlighted the regions marked with red circles, where inversion can be identified.
        

# Conclusion

To sum up, it is worthwhile to compare the final assembly with the [_S. cerevisiae_ S288C reference genome](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/146/045/GCF_000146045.2_R64/GCF_000146045.2_R64_assembly_stats.txt).

![Table 1: Final stats](../../images/vgp_assembly/stats_conclusion.png "Comparison between the final assembly generating in this training and the reference genome. Contiguity plot using the reference genome size (a). Assemby statistics (b).")

With respect to the total sequence length, we can conclude that the size of our genome assembly is almost identical to the reference genome (fig.16a,b). It is conspicuous that the reference genome consists of 17 sequences, while in our assembly includes only 16 chromosomes. This is due to the fact that the reference genome also includes the sequence of the mitochondrial DNA, which consists of 85,779 bp. The remaining statistics exhibit very similar values (fig. 16b).

![Figure 16: Comparison reference genome](../../images/vgp_assembly/hi-c_pretext_conclusion.png "Comparison bwetween contact maps generated by using the final assembly (a) and the reference genome (b).")

If we compare the contact map of our assembled genome (fig. 17a) with the reference assembly (fig. 17b), we can see that the two are essentially identical. This means that we have achieved an almost perfect assembly at the chromosome level.

# References
{% bibliography %}
        
{:.no_toc}
