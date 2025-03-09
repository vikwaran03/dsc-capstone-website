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
![GraphSAGE Classification Pipeline](figures/Sage%20Process.png)

GraphSAGE is an inductive graph learning algorithm that simultaneously learns the graphical structure of the neighborhood of a node and the distribution of their features to create aggregated embeddings for each node in the training set of a graph \citep{hamilton2018inductiverepresentationlearninglarge}. For each node in the training set, GraphSAGE samples its neighbors and aggregates their features to update node embeddings. The vector containing the aggregated information of the neighbors is concatenated to the current state of the embedding for the target node. The final result is a set of node embeddings that contain information about the node features of its neighbors and the structure of the graph that produced them.

![GraphSAGE Algorithm](figures/graphsagevis.png)

*Figure 1: GraphSAGE Algorithm*

GraphSAGE is useful for our classification problem due to differences in genomic interaction and genetic expression between HSR and ecDNA. In terms of genomic interaction, ecDNA has a circular structure \citep{wu2019circular}, whereas HSR does not, allowing regions within ecDNA to uniquely interact. In terms of genetic expression, ecDNA is more expressive than HSR. Its existence off of the chromosome amplifies the genes it contains. Genomic interaction is represented by the edges in our graph, and expression is captured in the graph through the RNA-seq read count feature of the node vector. Notable differences in interactions and expression, captured by the structure and node features of the graph, naturally lead to the use of GraphSAGE for the classification task.

We trained GraphSAGE on a graph containing both the ecDNA and HSR portions, with each being a disjoint subgraph of a larger graph. Our model followed an encoder decoder paradigm. The encoder consisted of two GraphSAGE layers that generated 16-dimensional embeddings for each node. The decoder consisted of two linear fully connected layers with ReLU activation, reducing the dimensionality to a 2 dimensional output layer. Softmax was applied to the output layer for classification, and Cross Entropy loss was used to evaluate predictions and train the model.  

*Figure 2: GraphSAGE Classification Pipeline*

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

*Table 1: GraphSAGE Performance Metrics*

![Confusion Matrix](figures/sage_confusion_matrix%20(1).png)

*Figure 3: Confusion Matrix of GraphSAGE Predictions on Test Set*

## Discussion
### Node2Vec
Explain...
### GraphSAGE
Explain...

## References
[1] **Hamilton, William L., Rex Ying, and Jure Leskovec.** 2018. “Inductive Representation Learning on Large Graphs.” [Link](https://arxiv.org/abs/1706.02216)
[2] **Wu, Sihan, Kristen M Turner, Nam Nguyen, Ramya Raviram, Marcella Erb, Jennifer Santini, Jens Luebeck, Utkrisht Rajkumar, Yarui Diao, Bin Li et al.** 2019. “Circular ecDNA promotes accessible chromatin and high oncogene expression.” Nature 575 (7784): 699–703 [Link](https://www.nature.com/articles/s41586-019-1763-5)
