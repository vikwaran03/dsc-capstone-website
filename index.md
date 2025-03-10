---
layout: default
title: Characterizing Extrachromosomal DNA Regions with Graph Neural Networks
---

<h1 style="text-align: center;">
  <strong> Characterizing Extrachromosomal DNA Regions with Graph Neural Networks </strong>
</h1>

<br>

## **Background and Research Goals**
- Extrachromosomal DNA (ecDNA) are amplified fragments of the genome that exist away from the chromosomes. ecDNA has been shown to be found in aggressive forms of cancer.
- <strong>We aim to characterize ecDNA regions, while also being able distinguish them from more benign amplified regions called Homogeneously Staining Regions (HSRs).</strong>

<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/ec_structure.png" alt="Image 1" width="45%">
    <img src="figures/hsr_structure.png" alt="Image 2" width="45%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 1: ecDNA (left) and HSR (right) 3d Structure </p>

<br>

## **Data: Hi-C Matrices and RNA-seq Reads**
- Aligned sequencing reads from the GBM39 cell line to the hg38 (human) genome, aggregating gene and read counts per 5000 bp genomic regions.
- Mapped binned reads to edges defined by Hi-C matrices, constructing 2 connected graphs where nodes represent genomic regions and weighted edges capture interaction strength. 
- Represented each Hi-C matrix as a 251×251 adjacency matrix for both ecDNA and HSR, with values indicating interaction frequency. For modeling, the top 25\% of edges by distance (binned into 14 bins of length 0.05) were pruned and binarized, with 1s marking significant connections [1].

### **Interaction Graphs**
<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/full_graph.png" alt="Image 1" width="45%">
    <img src="figures/graph3.png" alt="Image 2" width="45%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 2: Full interaction graph (left) and Interaction graph with certain regions selected (right) </p>

## **Methods**
### **Clustering**
We develop a pipeline as below to cluster our data. We take our ecDNA interaction graph, as defined above; apply node2vec [2] on it, giving us 16-dimensional vectors for each node; append the read counts and gene counts to the embeddings; reduce the data to 2 dimensions using PCA; and lastly cluster the data using DB-SCAN. We then repeat the process on the HSR graph. DB-SCAN allows us to find an unspecified number of clusters because we did not have a pre-specified amount of classes that we were looking for.

![Node2Vec Clustering Pipeline](figures/clustering_pipeline.png)
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 1: Clustering Pipeline </p>

### **Classification**

GraphSAGE is an inductive graph learning algorithm that simultaneously learns the graphical structure of the neighborhood of a node and the distribution of their features to create aggregated embeddings for each node in the training set of a graph [3]. For each node in the training set, denoted as the target node in the figure below, GraphSAGE samples its neighbors and aggregates their node features to update node embeddings. The vector containing the aggregated information of the neighbors is concatenated to the current state of the embedding for the target node. The final set of node embeddings contain information about the node features of its neighbors and the structure of the graph around it.

<div style="text-align: center;">
  <img src="figures/graphsagevis.png" alt="GraphSAGE Algorithm" width="600">
  <p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 2: GraphSAGE Algorithm Visualization <a href="https://github.com/pyg-team/pytorch_geometric/discussions/3799" target="_blank"> Link to Source </a></p>
</div>

GraphSAGE is useful for our classification problem due to differences in genomic interaction and genetic expression between HSR and ecDNA. In terms of genomic interaction, ecDNA has a circular structure [5], whereas HSR does not, allowing regions within ecDNA to uniquely interact. In terms of genetic expression, ecDNA is more expressive than HSR. Its existence off of the chromosome amplifies the genes it contains. Genomic interaction is represented by the edges in our graph, and expression is captured in the graph through the RNA-seq read count feature of the node vector. Notable differences in interactions and expression, captured by the structure and node features of the graph, naturally lead to the use of GraphSAGE for the classification task.

We trained GraphSAGE on a graph containing both the ecDNA and HSR portions, with each being a disjoint subgraph of a larger graph. Our model followed an encoder decoder paradigm. The encoder consisted of two GraphSAGE layers that generated 16-dimensional embeddings for each node. The decoder consisted of two linear fully connected layers with ReLU activation, reducing the dimensionality to a 2 dimensional output layer. Softmax was applied to the output layer for classification, and Cross Entropy loss was used to evaluate predictions and train the model.  

![GraphSAGE Classification Pipeline](figures/Sage%20Process.png)
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 3: Classification Pipeline </p>


## **Results**
### **Clustering Results**
These are the results of PCA and subsequent clustering for ecDNA (left) and HSR (right).

<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/ec_clusters (4).png" alt="Image 1" width="45%">
    <img src="figures/hsr_clusters.png" alt="Image 2" width="45%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 4: ecDNA (left) and HSR (right) PCA clusters </p>

<br>

We visualized these clusters by overlaying their locations on top of the heatmap of the corresponding HI-C matrix. We also attached a gene track under each of the plots to visualize which genes were present within our loci range. It allows us to compare whether clusters match up with the presence of genes.

<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/ec_hic_clusters (2).png" alt="Image 1" width="45%">
    <img src="figures/hsr_hic_clusters (3).png" alt="Image 2" width="45%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 5: ecDNA (left) and HSR (right) HI-C heatmap with clusters and gene track </p>

<br>

After computing these clusters, we checked if these clusters created significant differences in terms of multiple graph properties. Specifically, we applied a Mann-Whitney U-Test on all pairs clusters for each property that we were interested in. The following heatmap shows the p-values for each U-Test that we conducted. We defined significance at p < 0.05.

<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/ec_pval_heatmap (1).png" alt="Image 1" width="90%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 6: ecDNA U-Test P-Values </p>

<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/hsr_pval_heatmap (1).png" alt="Image 1" width="90%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 7: HSR U-Test P-Values </p>


<br>

### **Classification Results**
We trained our GraphSAGE classification model on the combined graph G, obtaining the results in the table below. We split the graph into train, validation, and test sets using masks with a split of 70%/15%/15%. The test metrics in the table display the results of predictions on both the validation and test sets. Early stopping [4] was implemented to capture the best performing model during training.

<div style="display: flex; justify-content: center; align-items: center; gap: 40px;">
    <div>
        <table>
            <thead>
                <tr>
                    <th>Metric</th>
                    <th>Train</th>
                    <th>Test</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>Accuracy</td>
                    <td>0.92308</td>
                    <td><b>0.81333</b></td>
                </tr>
                <tr>
                    <td>Precision</td>
                    <td>0.92327</td>
                    <td>0.82799</td>
                </tr>
                <tr>
                    <td>Recall</td>
                    <td>0.92294</td>
                    <td><b>0.82719</b></td>
                </tr>
                <tr>
                    <td>F1 Score</td>
                    <td>0.92304</td>
                    <td>0.82744</td>
                </tr>
            </tbody>
        </table>
    </div>
    <div>
        <img src="figures/sage_confusion_matrix%20(1).png" alt="Confusion Matrix" style="width: 100%;">
    </div>
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 8: GraphSAGE Results Table (left) and Confusion Matrix (right) </p>

## **Takeaways**
### **Clustering**
We defined three different types of clusters given our results - the sparse outliers, the dense core, and the more tightly clustered regions. We examined the nature of these three and came away with three conclusions:

- Differences in ecDNA and HSR clusters suggest distinct sets of similarly behaving genes specific to each structure. We hypothesize that these differences in clusters from ecDNA and HSRs could come from their different 3d structures. Specifically, it tells us that in ecDNA, certain genomic regions are being brought together in new spatial neighborhoods, likely exposing different genes to regulatory elements like enhancers or promoters. Conversely, in HSRs, different clusters suggest a different structural pattern, which may be causing the regulation of an alternate set of genes

- Less populous clusters within ecDNA reveal structurally related genes. The plot below shows how regions in the same cluster that correspond to genes are related in 3D space. For example, we can see two regions from Cluster 5 that correspond to genes NIPSNAP2 (blue) and ZNF713 (lime). Now from the figure on the right, we can see that these genes lie along a loop in 3d space. This gives us evidence toward the fact these genes may share regulatory elements like promoters or enhancers and be co-regulated/co-expressed in ecDNA.

<div style="display: flex; justify-content: center; align-items: center; gap: 20px;">
    <img src="figures/selected_clusters_3d (1).png" alt="Image 1" width="45%">
    <img src="figures/ec_structure_selected_genes (2).png" alt="Image 2" width="45%">
</div>
<p style="display: flex; justify-content: center; align-items: center; font-size: 10px"> Figure 9: &nbsp;<strong>Left</strong>: ecDNA Structure with Clusters 4 and 5 highlighted &nbsp;<strong>Right</strong>: Same as &nbsp;<strong>Left</strong>&nbsp; but specific genes highlighted </p>
   
- Graphically, the two most populous clusters in ecDNA and HSR behave significantly differently. At the 5% significance level, the only two clusters that were significant in all of the graph properties we looked at were Cluster 1 (outliers) and Cluster 2 (dense core). These two clusters behave distinctly when it comes to HI-C interactions and gene/read counts.

### **Classification**
Of the four metrics we used to evaluate GraphSAGE, accuracy and recall are the most critical for our problem. Recall is particularly important for our task due to the cancerous properties of ecDNA. Recall weights false negatives, which for our problem, means predicting HSR (a benign region) when the region is ecDNA (a cancerous region). Failing to diagnose a patient with cancer is far more harmful than diagnosing a non-cancer patient with cancer, the false positive case. Fortunately, our GraphSAGE is generating balanced predictions, as seen in the confusion matrix.

GraphSAGE performed well on both the train and test sets, but there is a notable discrepancy of about 0.1 in each metric between the two sets. This is likely due to the small size of our dataset. However, strong performance on the train set is a positive indicator that GraphSAGE was able to find patterns in the graph structure and node features differentiating ecDNA and HSR. Obtaining more cell samples on the GBM39 cell line can boost the robustness of our graph learning model. To further extend research on the classification task, GNNExplainer [6] can be attached to the GraphSAGE model to learn key predictive regions of the graph. These subgraphs can be matched to their 3D location, which could uncover key differences in structure between ecDNA and HSR. 

## **References**
[1] **Bigness, Jeremy, Xavier Loinaz, Shalin Patel, Erica Larschan, and Ritambhara Singh**. 2022. “Integrating long-range regulatory interactions to predict gene expression using graph convolutional networks.” Journal of Computational Biology 29(5): 409–424 [Link](https://www.liebertpub.com/doi/10.1089/cmb.2021.0316)

[2] **Grover, Aditya, and Jure Leskovec. 2016.** “node2vec: Scalable Feature Learning for Networks.” [Link](https://arxiv.org/abs/1607.00653)

[3] **Hamilton, William L., Rex Ying, and Jure Leskovec.** 2018. “Inductive Representation Learning on Large Graphs.” [Link](https://arxiv.org/abs/1706.02216)

[4] **Prechelt, Lutz.** 2002. “Early stopping-but when?” In Neural Networks: Tricks of the trade. Springer: 55–69 [Link](https://link.springer.com/chapter/10.1007/3-540-49430-8_3)

[5] **Wu, Sihan, Kristen M Turner, Nam Nguyen, Ramya Raviram, Marcella Erb, Jennifer Santini, Jens Luebeck, Utkrisht Rajkumar, Yarui Diao, Bin Li et al.** 2019. “Circular ecDNA promotes accessible chromatin and high oncogene expression.” Nature 575 (7784): 699–703 [Link](https://www.nature.com/articles/s41586-019-1763-5)

[6] **Ying, Rex, Dylan Bourgeois, Jiaxuan You, Marinka Zitnik, and Jure Leskovec.** 2019. “GNNExplainer: Generating Explanations for Graph Neural Networks.” [Link](https://arxiv.org/abs/1903.03894)

