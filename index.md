---
layout: default
title: Characterizing Extrachromosomal DNA Regions with Graph Neural Networks
---

# Characterizing Extrachromosomal DNA Regions with Graph Neural Networks
## Background and Research Goals
Explain...

## Data
### HI-C Matrices
Explain...
### RNA-seq Reads
Explain...

## Methods
### Node2Vec
Explain...
### GraphSAGE
![GraphSAGE Clasification Pipeline](figure/Sage%20Process.png)



![GraphSAGE Algorithm](figure/graphsagevis.png)

## Interaction Graphs
Display graphs here...

## Results
### Node2Vec Embeddings and Clusters
Explain...
### GraphSAGE Classification Results
We trained our GraphSAGE classification model on the combined graph G, obtaining the results in the table below. We split the graph into train, validation, and test sets using masks with a split of 70%/15%/15%. The test metrics in the table display the results of predictions on both the validation and test sets. Early stopping was implemented to capture the best performing model during training.

| **Metric**   | **Train**  | **Test**   |
|-------------|-----------|-----------|
| Accuracy    | 0.92308   | **0.81333** |
| Precision   | 0.92327   | 0.82799   |
| Recall      | 0.92294   | **0.82719** |
| F1 Score    | 0.92304   | 0.82744   |

![Confusion Matrix](figure/sage_confusion_matrix%20(1).png)

## Discussion
### Node2Vec
Explain...
### GraphSAGE
Explain...

## References
List references...
