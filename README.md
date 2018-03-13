AStrap R package
====================

Identification of alternative splicing from transcriptome sequences without a reference genome

About
====================
AStrap implements a de novo approach to detect alternative splicing (AS) from transcript sequences without a reference genome, including identification of AS events by extensive pair-wise alignments of transcript sequences from SMRT sequencing data and prediction of AS types by a machine-learning model integrating more than 500 assembled features. AS events of four types including intron retention (IR), exon skipping (ES), alternative donor sites (AltD), and alternative acceptor sites (AltA) were considered. AStrap consists of four main stages: data preprocessing, feature construction, classification model building, identification of AS events and prediction of AS types. AStrap could be a valuable addition to the community for the study of AS in non-model organisms with limited genetic resources.

Installing AStrap
=============
Mandatory 
---------

* R (>3.1). [R 3.3.3](https://www.r-project.org/) is recommended.

Required R Packages
---------
* [stringr](https://CRAN.R-project.org/package=stringr), [ROCR](https://cran.r-project.org/web/packages/ROCR/index.html), [BSgenome](http://www.bioconductor.org/packages/release/bioc/html/BSgenome.html), [rtracklayer](http://www.bioconductor.org/packages/release/bioc/html/rtracklayer.html), [BioSeqClass](http://www.bioconductor.org/packages/release/bioc/html/BioSeqClass.html), [igraph](https://cran.r-project.org/web/packages/igraph/index.html), [Biostrings](http://www.bioconductor.org/packages/release/bioc/html/Biostrings.html), 

Suggested R Packages
---------
* [RWeka](https://cran.r-project.org/web/packages/RWeka/index.html), [ggplot2](https://cran.r-project.org/web/packages/ggplot2/index.html), [Gviz](http://www.bioconductor.org/packages/release/bioc/html/Gviz.html), [e1071](https://CRAN.R-project.org/package=e1071), [adabag](https://CRAN.R-project.org/package=adabag), [randomForest](https://CRAN.R-project.org/package=randomForest), 

Installation
---------
* Install the R package using the following commands on the R console:
```
install.packages("devtools")
library(devtools)
install_github("BMILAB/AStrap")
library(AStrap)
```

Using AStrap
=============
In order to facilitate user understanding, we use the provided example dataset to illustrate the standard analysis work-flow of AStrap.

Section 1 Data loading
---------
For identification AS events and prediction AS types, first the user should load data into AStrap.
* Use function "readDNAStringSet" to read transcriptome sequences (FASTA format).
```
##Loading transcript sequences
trSequence.path <- system.file("extdata","example_TRsequence.fasta",package = "AStrap")
trSequence <-  readDNAStringSet(trSequence.path,format = "fasta")
```
* Use function "readCDHIT" to read a table of list of clusters generated by [CD-HIT](http://weizhongli-lab.org/cd-hit/).
```
##Loading the file of a list of clusters generated by CD-HIT-EST
cdhit.path <- system.file("extdata","example_cdhitest.clstr",package = "AStrap")
raw.cluster <- readCDHIT(cdhit.path)
```
* Use function "readGMAP" to load pairwise sequence alignments generated by [GMAP](http://research-pub.gene.com/gmap/) (GFF3 format). Meanwhile, this function will adjust clustering result if the parameter recluster is TRUE (default). 
```
##Loading the alignment file in GFF3 format generated by GMAP
gmap.path <- system.file("extdata","example_gmap.gff3",package = "AStrap")
cluster.align <- readGMAP(gmap.path,raw.cluster, recluster = TRUE, recluster.identity = 0.7,recluster.coverage = 0.7)
#Pairwise alignment of isoforms in the same cluster
alignment <- cluster.align$alignment
#Adujust  clusters
rew.cluster <- cluster.align$cluster
```
* In addition, pairwise alignments of isoforms of the same cluster can be visualized by the function "plotCluster" and "plotAlign".
```
##Plotting a network graph
gg1 <- plotCluster(raw.cluster,cluster.id=c("7"))
plot(gg1)
gg2 <- plotAlign(alignment,cluster.id=c("7"))
plot(gg2)
```


Section 2 Feature construction
---------
 In AStrap, we have compiled a compendium of 511 unique features that covers major factors known to shape introns and/or exons. In fact, feature construction has been embedded in the function AStrap (see below), users therefore don’t need to carry out this step.
* Use function "extract_IsoSeq_tr" to extract sequence around splice sites based on the transcript sequences.
```
##Loading example data
load(system.file("data","sample_Aligndata.Rdata",package = "AStrap"))
##Extracting sequence around splice sites based on the transcript sequences
Aligndata <- extract_IsoSeq_tr(Aligndata,trSequence)
```
* Use function "getFeature" to construct the feature space. 
```
##Loading the consensus matrix of sequences of the [-2,+3] region of acceptor sites.
load(system.file("data","example_PWM_acceptor.Rdata",package = "AStrap"))
##Loading the consensus matrix of the sequences of the [-2,+3] region of donor sites
load(system.file("data","example_PWM_donor.Rdata",package = "AStrap"))
##Constructing the feature space
feature <- getFeature(Aligndata)
```

Section 3 Model building and performance evaluation
---------
Two classification models trained on collected AS data from rice and human were integrated in AStrap, which could be directly applied for distinguishing among AS types for other species. For classification of AS types, we applied and compared three widely used machine-learning techniques, including support vector machine (SVM), random forests (RF), and adaptive boosting (AdaBoost). According to our analysis (see our paper), the RF-based model performed the best, followed by the AdaBoost-based model, and the SVM-based model performed the worst. Therefore, it is recommended that users adopt RF-based model for prediction of AS types.
* Use rice classification model, including SVM, RF, AdaBoost.
```
rice_model<- load(system.file("data","rice_model.Rdata",package = "AStrap"))

```
* Use human classification model, including SVM, RF, AdaBoost.
```
human_model<- load(system.file("data","human_model.Rdata",package = "AStrap"))
```
Meanwhile, users can also train a specific classification model on their own data sets.
* Use function "extract_IsoSeq_ge" to extract sequence around splice sites based on genome.
```
##Loading example alternative splicing data
path <- system.file("extdata","sample_riceAS.txt",package = "AStrap")
rice_ASdata <-read.table(path,sep="\t",head = TRUE,stringsAsFactors = FALSE)
##Loading genome using the package BSgenome
library("BSgenome.Osativa.MSU.MSU7")
##Extracting sequence around splice sites based on the genome
rice_ASdata<- extract_IsoSeq_ge(rice_ASdata,Osativa)
```
* Use function "buildTrainModel" to build model. The classification method can be chosen using parameter classifier, including SVM, RF (default), and AdaBoost. This function returns a list, including training set, test set, fitted model, predicted classification results, evaluation matrix of the fitted model and an ROC curve. 
```
library(randomForest)
library(ROCR)
library(ggplot2)
model <- buildTrainModel(rice_ASdata, chooseNum = 100,
                          proTrain = 2/3, proTest = 1/3, ASlength =0,
                          classifier = "rf", use.all = FALSE)
```

Section 4 Identification of AS events and prediction of AS types
---------
This section describes the identification of AS events based on pairwise alignment of isoforms of the same cluster and prediction of AS types based on the fitted model.
*  User function "AStrap" to identify AS events and predict AS types.
```
##Loading rice model
rice_model<- load(system.file("data","rice_model.Rdata ",package = "AStrap"))   
##Identification and prediction based on RF-based model of rice
result <- AStrap(alignment,trSequence,rice_RFmodel)

```

