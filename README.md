# eMIRNA

eMIRNA is a comprehensive and user-friendly R-based pipeline for predicting and annotating the presence of known and novel microRNAs. This document is intended to give a technical supplementary description about how to run the eMIRNA pipeline through a detailed explanation of all the modules that form part of this program.

&nbsp;
&nbsp;

## Introduction

The eMIRNA pipeline makes use of a Machine Learning approach based on Support Vector Machine (SVM) algorithm to assess whether putative candidate sequences can be predicted as pre-miRNA-like structures. First, the eMIRNA model must be trained by making use of positive (microRNAs) and negative (other ncRNAs) sequence datasets. Once the SVM model has been trained, putative pre-miRNA candidates can be subjected to prediction.

&nbsp;

## Prerequisites

The following R libraries are required for running the eMIRNA pipeline:
+ seqinr (https://CRAN.R-project.org/package=seqinr)
+ stringr (https://CRAN.R-project.org/package=stringr)
+ Biobase (https://bioconductor.org/packages/release/bioc/html/Biobase.html)
+ scales (https://CRAN.R-project.org/package=scales)
+ caret (https://CRAN.R-project.org/package=caret)
+ bimba (https://github.com/RomeroBarata/bimba)
+ LiblineaR (https://CRAN.R-project.org/package=LiblineaR)
+ biomaRt (https://bioconductor.org/packages/release/bioc/html/biomaRt.html)

The following programs are required for running the eMIRNA pipeline:
+ RNAfold [1] (https://www.tbi.univie.ac.at/RNA/)
+ UNAFold [2] (https://github.com/rcallahan/UNAFold)
+ Triplet-SVM [3] (http://www.bioinfo.au.tsinghua.edu.cn/mirnasvm/)
+ BEDTools v2.27.0 [4] (https://bedtools.readthedocs.io/en/latest/)
+ Bowtie [5] (https://mcardle.wisc.edu/mprime/help/bowtie/manual.html)
+ Fasta_ushuffle (https://github.com/agordon/fasta_ushuffle)


All the executables should be stored at computer $PATH in order to be run properly.

&nbsp;

## Positive and Negative Datasets

Training the eMIRNA Classifier requires two FASTA files with **Positive** and **Negative** sequences.

The **Positive Sequences** must correspond to those sequences annotated as microRNA genes in the available Reference Genome for the species under study. GTF annotation and FASTA files for corresponding transcripts can be downloaded from the Ensembl repositories available at http://www.ensembl.org/info/data/ftp/index.html.

The **Negative Sequences** must correspond to non-coding sequences other than microRNA genes in the available Reference Genome for the species under study. GTF annotation and FASTA files for corresponding transcripts cand be downloaded from the Ensembl repositories available at http://www.ensembl.org/info/data/ftp/index.html.

In the event that no Reference Genome or no good microRNA or non-coding transcripts are available for downloading, we strongly recommend to choose sequences from the closer phylogenetically related reference species for training the model, otherwise the results can suffer from low reliability.

Both Positive and Negative datasets must be in linear FASTA format. Should you have multilinear FASTA files, they should be converted to linear FASTA files. Users can use the following perl command:

`perl -pe '/^>/ ? print "\n" : chomp' in.fa | tail -n +2 > out.fa`

where `in.fa` corresponds to multilinear FASTA file, and `out.fa` is the resulting linearized FASTA file ready to use.

&nbsp;

## eMIRNA.Filter.by.Size

The first eMIRNA module makes use of previous Positive and Negative FASTA files, to perform an initial filtering process based on sequence length. Tipically, microRNA genes range from 50 to 150 nucleotides long. Our first aim would be to filter the selected sequences based on expected microRNA genes length. We will apply this function to both Positive and Negative FASTA files. The Positive sequences should not experience any filtering upon this process, if correctly generated. For Negative sequences, all long non-coding sequences will be removed from our Negative FASTA file, retaining only those sequences resembling microRNA genes in length, according to estabished thresholds.

This function requires four arguments:

+ PATH to Positive or Negative FASTA file.
+ String with desired output prefix name.
+ Lower length filtering threshold.
+ Upper length filtering threshold.

We recomend to set 50 nucleotides for lower length threshold, and 150 for the upper, but users can define their own limit thresholds.

Example of usage:

`eMIRNA.Filter.by.Size("PATH to Positive FASTA file", "Pos", 50, 100)`

`eMIRNA.Filter.by.Size("PATH to Negative FASTA file", "Neg", 50, 100)`

Once the eMIRNA.Filter.by.Size function has run, a new folder named `eMIRNA` will have been created at your computer `$HOME`, with a subfolder, inside `eMIRNA/` folder, called `FilterSize_Results/`, in which a FASTA file named `Pos/Neg_filter_size.fa` will be generated with the results of running the function.

&nbsp;

## eMIRNA.Filter.by.Structure

The second eMIRNA module aims to estimate the secondary folding structure of selected filtered sequences both in Positive and Negative datasets, thus filtering out all sequences that do not resemble a pre-miRNA hairpin-like secondary structure. The eMIRNA.Filter.by.Structure function will make use of RNAfold [1] program to calculate the estimated secondary folding structure, which should be available in your computer `$PATH` to be correctly executed. Tipically, microRNA genes have a characteristic secondary structure, composed by two stems joined by complementarity and one terminal loop, forming a hairpin-like secondary structure. Some bubbles or bulges can appear within the two stems, belonging to non-paired nucleotides in the sequence.

This function requires two arguments:

+ PATH to eMIRNA.Filter.by.Size Positive or Negative FASTA output files.
+ String with desired output prefix name.

Example of usage:

`eMIRNA.Filter.by.Structure("~/eMIRNA/FilterSize_Results/Pos_filter_size.fa", "Pos")`

`eMIRNA.Filter.by.Structure("~/eMIRNA/FilterSize_Results/Neg_filter_size.fa", "Neg")`

Once the eMIRNA.Filter.by.Structure has run, a new folder named `FilterSstructure_Results/` will be created inside `eMIRNA/` folder, in which a FASTA file called `Pos/Neg_filter_nloop.fa` will be generated with the results of running the function.

&nbsp;

## eMIRNA.Features

The third eMIRNA module aims to calculate a series of structural, statistical and sequence-derived features from each sequence that had passed previous filterings, in order to obtain an estimated representation of their structural characteristics. Afterwards, these feature matrix will be processed by the prediction software to discriminate between microRNAs and other type of sequences.

The function requires two arguments:

+ PATH to eMIRNA.Filter.by.Structure Positive or Negative FASTA output files.
+ String with desired output prefix name.

Example of usage:

`Pos = eMIRNA.Features("~/eMIRNA/FilterStructure_Results/Pos_filter_nloop.fa”, "Pos")`

`Neg = eMIRNA.Features("~/eMIRNA/FilterStructure_Results/Neg_filter_nloop.fa”, "Neg")`

Once the eMIRNA.Features has run, a new folder named `Features_Results/` will be created inside `eMIRNA/` folder, in which a .csv file called `Pos/Neg.csv` will be generated with the results of running the function.

The eMIRNA.Features function includes the calculation of a total of 6 Sequence Features, comprising 55 variables, 8 Secondary Structure Features, comprising 23 variables, and 24 Structural Statistics. 

Sequence Features:

+ 32 Triplet elements calculated with SVM-Triplets pipeline [3] (T1 - T32).
+ Sequence Length (Length).
+ Guanine+Cytosine/Length (GC).
+ Adenine+Uracil / Guanine+Cytosine ratio (AU.GCr).
+ Adenine, Uracil, Guanine, Cytosine / Length (Ar, Ur, Gr, Cr).
+ Dinucleotide content / Length (AAr, GGr, CCr, UUr, AGr, ACr, AUr, GAr, GCr, GUr, CAr, CGr, CUr, UAr, UGr, UCr).

Secondary Structure Features:

+ Terminal hairpin loop length (Hl).
+ 5’ - 3’ Stems length (Steml5, Steml3).
+ Base pairs in Secondary Structure (BP).
+ Base pairs or matches at 5’ - 3’ Stems (BP5, BP3).
+ Unpaired bases or mismatches at 5’ - 3’ Stems (Mism5, Mism3).
+ Bulges at 5’ - 3’ Stems (B5, B3).
+ Bulges at 5’ - 3’ Stems of type 1,2,3,4 or 5 unpaired bases (BN1.5, BN1.3 … BN5.5, BN5.3).
+ A-U, G-C and G-U Pairings in sequence (AUp, GCp, GUp).

Structural Statistics:

+ Minimum Free Energy estimated by RNAfold [1] (MFE).
+ Ensemble Free Energy (EFE).
+ Centroid Free Energy (CFE).
+ Centroid Distance to Ensemble (CDE).
+ Maximum Expected Accuracy of Free Energy (MEAFE).
+ Maximum Expected Accuracy (MEA).
+ BP / Length (BPP).
+ MFE Ensemble Frequency (EFreq).
+ Ensemble Diversity (ED).
+ MFE / Length (MFEadj).
+ EFE / Length (EFEadj).
+ CDE / Length (Dadj).
+ Shannon Entropy / Length (SEadj).
+ MFE – EFE / Length (DiffMFE.EFE).
+ MFEadj / GC (MFEadj.GC).
+ MFEadj / BP (MFEadj.BP).
+ MFE estimated by UNAFold [2] (dG).
+ dG / Length (dGadj).
+ Structural Entropy estimated by Melt function from UNAFold [2] (dS).
+ dS / Length (dSadj).
+ Structural Enthalpy estimated by Melt function (dH).
+ dH / Length (dHadj).
+ Fusion Temperature estimated by Melt function (dT).
+ dT / Length (dTadj).

&nbsp;

## eMIRNA.Train

The fourth eMIRNA module aims to perform the training process of a Machine Learning-based Support Vector Machine (SVM) algorithm, making use of Feature representation previously calculated, to construct a SVM model capable to distinguish between microRNAs and other non-coding sequences. The SVM algorithm is built by using a 10 k-fold cross-validation over a training set randomly selected from analyzed features.

This function requires three arguments:

+ Positive Features calculated by eMIRNA.Features, saved in R object.
+ Negative Features calculated by eMIRNA.Features, saved in R object.
+ Imbalance correction algorithm.

Example of usage:

`SVM = eMIRNA.Train(Pos, Neg, imbalance=”smote”)`

It is important that when running the SVM training process, both Positive and Negative matrices have a balanced number of sequences to evaluate, keeping the number of positive and negative sequences to be similar, in order to avoid an overrepresentation of one of the two classes. To overcome this problem, eMIRNA.Trains implements a series of imbalance correction methods available. If required, eMIRNA.Train will first perform a Noise Reduction A Priori Synthetic correction (NRAS) of input features, as reported by Rivera W [6], followed by the preferred method to over-sampling the minority class to correct class-imbalance biases. Available methods are (adasyn, bdlsmote1, bdlsmote2, mwmote, ros, rwo, slsmote, smote):

+ ADASYN: Adaptive Synthetic Sampling [7]
+ BDLSMOTE: borderline-SMOTE1 and borderline-SMOTE2 [8]
+ MWMOTE: Majority Weighted Minority Over-Sampling TEchnique [9]
+ ROS: Random Over-Sampling
+ RWO: Random Walk Over-Sampling [10]
+ SLSMOTE: Safe-Level-SMOTE [11]
+ SMOTE: Synthetic Minority Over-Sampling TEchnique [12]

By default, eMIRNA.Train will not perform any class-imbalance correction, but users are imperiously advised to do so, otherwise the training process could suffer.

Once the function has run, eMIRNA.Train will create a SVM classifier capable to differentiate between microRNAs and other structurally microRNA-like non-coding RNAs.

&nbsp;

## eMIRNA.Hunter

Once we have trained our model for predicting new microRNA candidates, we will have to test its performance and discovery ability to classify non previously annotated microRNAs in our species of interest. The eMIRNA.Hunter module is an auxiliar BASH script developed to obtain pre-miRNA candidate sequences to classify, by making use of an homology-based recovery from previously annotated microRNAs in reference species, in order to find orthologous sequences in our less annotated species under study.

The eMIRNA.Hunter script implements Bowtie [5] for the alignment of mature microRNA annotated sequences in reference species like humans or rodents, to find orthologous regions in the genome of our species of interest, reconstructing pre-miRNA sequences from mature microRNAs and generating a FASTA and BED files for the candidates to be classified by the previously trained SVM algorithm.

This module requires four arguments:

+ PATH to Reference Genome Bowtie Index of our species of interest.
+ PATH to FASTA file of selected reference organism mature microRNAs.
+ PATH to desired output.
+ Desired output prefix name.

A detailed explanation of each variable can be accessed with -h (help) option:

```
eMIRNA.Hunter Usage Instructions:
eMIRNA.Hunter [options]
Input:
 -r	                             PATH to Referenfe Genome Bowtie Index
 -f	                             PATH to Reference Genome FASTA file
 -o	                             PATH to desired output folder
 -x	                             Desired Name string for output files
 -u	                             Upwards number of bases for pre-miRNA reconstruction (30-60 bp recommended)
 -b	                             Backwards number of bases for pre-miRNA reconstruction (1-10 bp recommended)
 -h	                             Display help page
Output:
 <x>.sam			                      SAM file output from Bowtie miRNA alignment
 <x>.log				                     LOG file output from Bowtie miRNA alignment
 <x>_homolog_miRNAs.bed	         BED file output with candidate pre-miRNAs for prediction
 <x>_homolog_miRNAs.fa	          FASTA file output with candidate pre-miRNAs for prediction
 
 ```

For achieving a successful cross-species alignment, it is very important that mature microRNA sequences FASTA file from reference organism are in DNA code, with Ts for Thymine and no Us for Uracil in RNA code. Please make sure that your mature microRNA sequences are in DNA code, otherwise the alignment process will fail.

For generating Bowtie Index for your Reference Genome, please refer to Bowtie Manual [5].

After successfully running eMIRNA.Hunter script, four files will have been created at predefined output PATH:

+ SAM file with aligned sequences.
+ Log file with alignment statistics.
+ BED file with positions of reconstructed premiRNA homologous candidates.
+ FASTA file with reconstructed premiRNA homologous candidates.

Once the FASTA file with pre-miRNA candidates has been generated, users must process this sequences by following the previously described steps for eMIRNA pipeline, in order to obtain a Feature matrix representing those candidate sequences that will then be subjected to classification by the SVM trained algorithm.

&nbsp;

eMIRNA.Hunter_denovo
The eMIRNA.Hunter_denovo module is a modified versión of eMIRNA.Hunter module, developed to obtain pre-miRNA candidate sequences to perform a de novo discovery of novel putative miRNAs from smallRNA-seq files.

Users should provide a properly collapsed FASTA file with smallRNA-seq sequences from canonical FASTQ sequence files. The FASTQ files should be quality-check filtered and sequencing adaptors trimmed before running the collapser tool from FASTX-toolkit (http://hannonlab.cshl.edu/fastx_toolkit/index.html) for collapsing FASTQ files into FASTA files with uniquely represented sequences.

Users are encouraged to perform a pre-filtering process of the collapsed FASTA file to retain sequences between 18-25 nucleotides in length, corresponding to the average length of mature miRNAs.

A detailed explanation of each variable can be accessed with -h (help) option:

```
eMIRNA.Hunter_denovo Usage Instructions:
eMIRNA.Hunter_denovo [options]
Input:
  -r                      PATH to Referenfe Genome Bowtie Index
  -f                      PATH to Collapsed FASTA file
  -o                      PATH to desired output folder
  -x                      Desired Name string for output files
  -u                      Upwards number of bases for pre-miRNA reconstruction (30-80 bp recommended)
  -b                      Backwards number of bases for pre-miRNA reconstruction (1-10 bp recommended)
  -h                      Display help page
Output:
  <x>.sam                 SAM file output from Bowtie miRNA alignment
  <x>.log                 LOG file output from Bowtie miRNA alignment
  <x>_miRNAs.bed          BED file output with candidate pre-miRNAs for prediction
  <x>_miRNAs.fa           FASTA file output with candidate pre-miRNAs for prediction
  
  ```

&nbsp;
 
## eMIRNA.Predict

The fifth eMIRNA module aims to perform microRNA classification by making use of the previously trained SVM algorithm and candidates sequences generated by eMIRNA.Hunter.

This module requires three arguments:

+ SVM trained algorithm object.
+ Feature matrix representing candidates sequences to evaluate.
+ String with desired output prefix name.

Example of usage:

`eMIRNA.Predict(SVM, Candidates_Feature_Matirx, “Candidates”)`

Once the eMIRNA.Predict has run, a new folder named `Prediction_Results/` will be created inside `eMIRNA/` folder, in which a .txt file called `Candidates.txt` will be generated, containing a list of Sequence candidates names classified as putative pre-miRNAs by the SVM trained algorithm.

&nbsp;

## eMIRNA.Refiner

After generating a list of putative new microRNAs by implementing the eMIRNA pipeline, users should filter these new candidates in order to select those more likely to be new putative microRNA candidates in the species of interest. The eMIRNA.Refiner module has been specifically designed to cover this task. This BASH script complements eMIRNA.Hunter and eMIRNA.Predict modules by selecting those candidate sequences that have more probability to correspond to new non-annotated microRNA sequences orthologous to the reference organism annotated microRNAs in which candidate sequences reconstruction has been performed.

The eMIRNA.Refiner module makes use of an auxiliar R script called eMIRNA_biomaRt_calc.R, which should be located at the same $PATH where eMIRNA.Refiner is found. The functionality of eMIRNA.Refiner is based on comparing the microRNA gene neighbourhood both in the reference species and in the species under study. Many shared orthologous microRNA sequences will not only share a conserved sequence between species, but also the genome location in which they are placed, making them more reliable if sharing both sequence and location. The candidate microRNA genes neighbourhood is estimated by establishing a comparison window around the sequence (tipically 2-10 Mb) and contrasting its homologous gene similarity with the corresponding microRNA in the reference organism. By doing so, the eMIRNA.Refiner module can calculate a Neighbourhood Score, measuring the ratio of similarity among annotated genes in the locus of interest between the reference organism and our species of interest. The higher this ratio would be, the most realiable the new microRNA candidate could be considered for further analyses, meaning that not only the sequence could be conserved between species, but also its genetic topology, highlighting the possible metabolic Importance of this miRNA gene.

This module requires nine arguments:

+ PATH to GTF annotation file from the species of interest.
+ PATH to GFF miRNA annotation file from selected reference species.
+ PATH to list of candidate sequences generated by eMIRNA.Predict module.
+ PATH to BED file with positions of reconstructed pre-miRNA orthologous candidates by eMIRNA.Hunter.
+ Window size in Mb for Neighbouring Genes between-species contrast.
+ Name of the species of interest.
+ Name of the reference species to contrast.
+ PATH to desired output.
+ Desired output prefix name.

Users should check genome availability at biomaRt repositories by using the the following R code:

`mart = useMart('ensembl'), followed by listDatasets(mart)`

A detailed explanation of each variable can be accessed with -h (help) option:

```
eMIRNA.Seeker Usage Instructions:
eMIRNA.Seeker [options]
Input:
 -g	                                      PATH to Referenfe Genome GTF Annotation file
 -i	                                      PATH to Model Organism microRNA GFF Annotation file
 -p	                                      PATH to List of Predicted microRNA candidates by eMIRNA
 -b	                                      PATH to BED Homolog microRNAs output file from eMIRNA.Hunter
 -d	                                      Mb window for between-species Neighbouring Genes Contrast (2-5 recommended)
 -s	                                      Name of species of interest (scientific name in lower case, e.g. sscrofa)
 -m	                                      Name of reference species for contrast (scientific name in lower case, e.g. hsapiens)
 -o	                                      PATH to desired output folder
 -x	                                      Desired Name string for output files
 -h	                                      Display help page
Output:
<x>_Predicted_miRNAs_annotated.bed		      BED file output with eMIRNA Predicted already Annotated miRNAs
<x>_Predicted_miRNAs_NON_annotated.bed	   BED file output with eMIRNA Predicted Novel miRNAs
<x>_Putative_Predicted_miRNAs.txt		       TXT file output with Putative miRNA Candidates filtered by Neighbouring Score

```

After successfully running the eMIRNA.Refiner script, one .txt file will have been created at predefined output `PATH`, containing the most relevant putative new non-annotated microRNAs, their estimated positions and corresponding Neighbouring Score. Besides, two BED files, containing novel and annotated candidates will be generated.

&nbsp;

## eMIRNA.Refiner_denovo

The eMIRNA.Refiner_denovo module is a modified versión of eMIRNA.Refiner module, developed to obtain pre-miRNA candidate sequences to perform a de novo discovery of novel putative miRNAs from smallRNA-seq files.

This module six nine arguments:

+ PATH to GTF annotation file from the species of interest.
+ PATH to list of candidate sequences generated by eMIRNA.Predict module.
+ PATH to BED file with positions of reconstructed pre-miRNA candidates by eMIRNA.Hunter_denovo.
+ PATH to FASTA file with sequences of reconstructed pre-miRNA candidates by eMIRNA.Hunter_denovo.
+ PATH to desired output.
+ Desired output prefix name.

A detailed explanation of each variable can be accessed with -h (help) option:

```
eMIRNA.Refiner_novel Usage Instructions:
eMIRNA.Refiner_novel [options]
Input:
  -g                                             PATH to Referenfe Genome GTF Annotation file
  -p                                             PATH to List of Predicted microRNA candidates by eMIRNA
  -b                                             PATH to BED microRNAs output file from eMIRNA.Hunter_denovo
  -f                                      PATH to FASTA microRNAs output file from eMIRNA.Hunter_denovo
  -o                                             PATH to desired output folder
  -x                                             Desired Name string for output files
  -h                                             Display help page
Output:
  <x>_Predicted_miRNAs_annotated.bed             BED file output with eMIRNA Predicted already Annotated miRNAs
  <x>_Predicted_miRNAs_NON_annotated.bed         BED file output with eMIRNA Predicted Novel miRNAs
  <x>_Predicted_miRNAs_NON_annotated.fa          FASTA file output with eMIRNA Predicted Novel miRNAs
  
  ```

After successfully running the eMIRNA.Refiner_denovo script, a BED file will have been created at predefined output PATH, containing the most relevant putative new non-annotated microRNAs, and their estimated positions. Besides, a BED files containing already annotated detected candidates and FASTA file with correspondent candidate sequences will be generated.

&nbsp;

## eMIRNA.Structural.Pvalues

Finally, after having obtained a list of putative novel pre-miRNA sequences by the aforementioned eMIRNA functions, users can analyse if the structural integrity of predicted pre-miRNAs can achieve a stable conformation at a statistically significant level. The eMIRNA.Structural.Pvalues function implements a n-randomization of provided sequences while mantaining k-let counts as described by Jiang et al. [13], using the fasta_ushuffle wrapper available at https://github.com/agordon/fasta_ushuffle. 

This module requires three arguments:

+ PATH to FASTA file of putative novel miRNA candidates generated by eMIRNA.Refiner_denovo.
+ String with desired output prefix name.
+ Number of iterations to perform.

Example of usage:

`eMIRNA.Structural.Pvalues(“Candidates_Predicted_miRNAs_NON_annotated.fa”, “Candidates”, 100)`

By default, eMIRNA.Structural.Pvalues will perform 100 random shuffling interations over each provided sequence. Users cand set their desired number of iterations, but should be aware of computing times required for iterating and folding of secondary structures for each sequence. As computing costs can exponentially increase with higher number of iterations, we encourge users to set their desired range of iterations between 100 and 1000, depending on the number of candidates sequences to be analysed.

Once the eMIRNA.Structural.Pvalues has run, a new .csv file called `Candidates_Structural_Pvalues.csv` will be generated at `/Prediction_Results` folder, containing calculated MFE and EFE Structural Pvalues for each candidates sequence. Users can then select those sequences with a significantly stable secondary structure folding as sequences having higher probability of have been properly profiled and predicted.

