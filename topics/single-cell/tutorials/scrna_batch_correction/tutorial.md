---
layout: tutorial_hands_on

title: "Batch Correction and Integration with Seurat or Scanpy"
subtopic: tricks
priority: 4
answer_histories:
  - label: Using Seurat
    history: https://singlecell.usegalaxy.eu/u/marisa_jl/h/batch-correction--integration-with-seurat---answer-key
    date: 2025-01-21
questions:
- What is the difference between batch correction and integration?
- How can we perform batch correction or integration using the Scanpy and Seurat pipelines?
objectives:
- Understand what batch correction and integration are and how they are different
- Know when to perform batch correction or integration on single cell data
- Perform batch correction or integration using either the Scanpy or Seurat pipelines
time_estimation: 2H
key_points:
- Batch correction can remove some of the technical differences between experimental groups
- Integration enables us to combine different datasets
- Both the Scanpy and Seurat pipelines include tools that can be used for batch correction or integration
requirements:
-
    type: "internal"
    topic_name: single-cell
    tutorials:
        - scrna-scanpy-pbmc3k
        - scrna-seurat-pbmc3k
tags:
- single cell
contributions:
  authorship:
    - MarisaJL
  editing:
    - nomadscientist
  funding:
    - biofair

---


# Introduction

Single cell analyses can be complex. We may have data from different experimental batches, perhaps because we ran our experiments at different times, in different labs, or using different sequencing p[...]

If we simply run these different batches or combined datasets through a clustering pipeline such as Scanpy or Seurat, we might not get useful results. Clustering prioritises the genes that show the bi[...]

In order to look beyond these technical differences, we can perform batch correction or integration. Both the Scanpy and Seurat pipelines include tools that can be used to correct for differences betw[...]

> <comment-title></comment-title>
>
> This tutorial is based on the [Introduction to scRNA-seq integration](https://satijalab.org/seurat/articles/integration_introduction) and [Integrative analysis in Seurat v5](https://satijalab.org/se[...]
>
{: .comment}


> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Important tips for easier analysis

{% snippet topics/single-cell/faqs/single_cell_omics.md %}

{% snippet faqs/galaxy/tutorial_mode.md %}

{% snippet faqs/galaxy/analysis_troubleshooting.md sc=true %}

# Batch Correction or Integration?

We will often need to perform batch correction or integration during single cell analyses. If we are working with different experimental batches, donors, conditions, or datasets, then we need to look [...]

The terms batch correction and integration are closely related and you may see them being used somewhat interchangably, because they both refer to the same process of looking for shared cell subpopula[...]

The only difference is that we tend to talk about batch correction when we are working with groups produced in a single study (e.g. different experimental batches), while we would say integration when[...]

In this tutorial, you can choose whether you want to use the Scanpy or Seurat pipelines for clustering and batch correction. Scanpy and Seurat's integration tools will create a dimensional reduction t[...]

# Scanpy or Seurat?

Scanpy and Seurat are two of the most commonly used pipelines (sets of tools) for analysing single cell data. Both pipelines have all the tools required to perform clustering, which identifies groups [...]

The Scanpy and Seurat pipelines include tools for preprocessing single cell data, performing dimensional reductions such as PCA, constructing a neighbourhood graph and finding clusters in it. Although[...]

The main difference between these two pipelines is that Scanpy is written for Python while Seurat is written for R. If we were working in a Python or R environment, then we would need to choose the ap[...]

## Get data

{% include _includes/cyoa-choices.html option1="Scanpy" option2="Seurat" default="Scanpy" text="Choose the single cell pipeline you want to use." %}

<div class='Scanpy' markdown='1'>

> <hands-on-title> Data Upload </hands-on-title>
>
> 1. Create a new history for this tutorial
>
> 2. Import the files from [Zenodo]({{ page.zenodo_link }})
>
>    ```
>   https://zenodo.org/records/20562556/files/Input_Anndata.h5ad
>    ```
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
> 3. Check that the datatype is `h5ad`
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
{: .hands_on}

</div>

<div class='Seurat' markdown='1'>

> <hands-on-title> Data Upload </hands-on-title>
>
> 1. Create a new history for this tutorial
>
> 2. Import the files from [Zenodo]({{ page.zenodo_link }})
>
>    ```
>   https://zenodo.org/records/14734574/files/Input_SeuratObject.rds
>    ```
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
> 3. Check that the datatype is `rds`
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
{: .hands_on}

</div>

You should now have either an AnnData or SeuratObject dataset in your history. Both datasets contain the same information, but in the different formats required by the Scanpy and Seurat pipelines.

We are using a single cell dataset of human Peripheral Blood Mononuclear Cells (PBMCs) that was also used in Seurat's [Integrative analysis in Seurat v5](https://satijalab.org/seurat/articles/seurat5_[...]

Let's take a look at our data before we begin the analysis to see whether we might need to perform batch correction or integration. While batches or combined datasets often do require correction, we s[...]

Correcting batch effects is important, but we also have to be careful about overcorrecting or overintegrating our data. If we apply batch correction or integration tools when they aren't needed, we co[...]

<div class='Scanpy' markdown='1'>

> <hands-on-title> Inspect the Data </hands-on-title>
>
> 1. {% tool [Inspect AnnData](toolshed.g2.bx.psu.edu/repos/iuc/anndata_inspect/anndata_inspect/0.11.4+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `output` (Input dataset)
>    - *"What to inspect?"*: `General information about the object`
>
{: .hands_on}

> <question-title></question-title>
>
> 1. How many cells and genes are in this dataset?
>
> > <solution-title></solution-title>
> >
> > 1. If we click on the {% icon galaxy-eye %} of the new output in our history, we can see that this dataset contains information about the expression of 33,694 vars (genes) in 10,434 obs (cells).
> >
> {: .solution}
>
{: .question}

Now we'll take a closer look at the metadata describing how the dataset was produced. It can tell us whether the dataset is made up of different batches that might require correction.

> <hands-on-title> Inspect the Cell Metadata </hands-on-title>
>
> 1. {% tool [Inspect AnnData](toolshed.g2.bx.psu.edu/repos/iuc/anndata_inspect/anndata_inspect/0.11.4+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `output` (Input dataset)
>    - *"What to inspect?"*: `Key-indexed observations annotation (obs)`
> 2. {% tool [Count](Count1) %} with the following parameters:
>    - {% icon param-file %} *"from dataset"*: `obs` (output of **Inspect AnnData** {% icon tool %})
>    - *"Count occurrences of values in column(s)"*: `c['11']`
{: .hands_on}

</div>

<div class='Seurat' markdown='1'>

> <hands-on-title> Inspect the Data </hands-on-title>
>
> 1. {% tool [Seurat Data Management](toolshed.g2.bx.psu.edu/repos/iuc/seurat_data/seurat_data/5.0+galaxy0) %} with the following parameters:
>    - *"Method used"*: `Inspect Seurat Object`
>        - *"Display information about"*: `General`
>
{: .hands_on}

> <question-title></question-title>
>
> 1. How many cells and genes are in this dataset?
> 2. How many layers are in this dataset?
>
> > <solution-title></solution-title>
> >
> > 1. If we click on the {% icon galaxy-eye %} of the new output in our history, we can see that this dataset contains information about the expression of 33,694 features (genes) in 10,434 samples (c[...]
> > 2. Data in SeuratObjects are stored in layers. In this case, we only have one layer called `counts`. The `counts` layer is the raw data that we'll be using in this tutorial.
> >
> {: .solution}
>
{: .question}

Now we'll take a closer look at the metadata describing how the dataset was produced. It can tell us whether the dataset is made up of different batches that might require correction.

> <hands-on-title> Inspect the Cell Metadata </hands-on-title>
>
> 1. {% tool [Seurat Data Management](toolshed.g2.bx.psu.edu/repos/iuc/seurat_data/seurat_data/5.0+galaxy0) %} with the following parameters:
>    - *"Method used"*: `Inspect Seurat Object`
>        - *"Display information about"*: `Cell Metadata`
> 2. {% tool [**Count** occurrences of each record](Count1) %} with the following parameters:
>    - *"Count occurrences of values in column(s)"*: `c11` This is the 11th column in your table, which contains the `Method` metadata
{: .hands_on}

</div>

> <question-title></question-title>
>
> 1. What does the `Method` column represent in the cell metadata?
> 2. Do you think batch correction or integration is needed for this analysis?
>
> > <solution-title></solution-title>
> >
> > 1. The dataset that we're using comes from a study that compared different single cell techniques. The `Method` column tells us which technique was used on each cell. Use the {% icon galaxy-eye %}[...]
> > 2. Each experimental technique can be considered as its own experimental batch. Each of these batches was processed independently, which by itself can be enough to require batch correction, even i[...]
> >
> {: .solution}
>
{: .question}

> <comment-title></comment-title>
>
> The cell metadata is any information about the cells that the original authors have included with the dataset. As well as the cell barcode or identifier for each individual cell, the metadata will u[...]
{: .comment}

# Clustering without Batch Correction

We suspect that batch correction will be needed because of the different technologies used to construct this dataset, but we'll try clustering without any correction first. This will confirm whether b[...]

Since our focus is batch correction/integration, we won't go into too much detail on the clustering process. We just want to see how the integration steps fit into the main clustering pipeline and und[...]

<div class='Scanpy' markdown='1'>

We'll follow the default Scanpy pipeline here, except that we'll use `30` PCs to build the neighborhood graph and cluster with a resolution of `2` as these were the parameters used in [the original Se[...]

> <hands-on-title> Cluster without Batch Correction </hands-on-title>
>
> 1. {% tool [Scanpy normalize](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_normalize/scanpy_normalize/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `output` (Input dataset)
>    - *"Method used for normalization"*: `Normalize counts per cell, using 'pp.normalize_total'`
>        - *"Exclude (very) highly expressed genes for the computation of the normalization factor (size factor) for each cell"*: `No`
>
>    > <comment-title></comment-title>
>    >
>    > We will use the output from `Scanpy normalize` in the  following section when we perform batch correction.
>    >
>    > If you're already familiar with the Scanpy clustering pipeline and you just want to try using the  {% icon tool %} **Scanpy remove confounders** tools, then you can skip ahead to the **Clu[...]
>    {: .comment}
>
> 2. {% tool [Scanpy Inspect and manipulate](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_inspect/scanpy_inspect/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy normalize** {% icon tool %})
>    - *"Method used for inspecting"*: `Logarithmize the data matrix, using 'pp.log1p'`
>
> 3. {% tool [Scanpy filter](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_filter/scanpy_filter/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy Inspect and manipulate** {% icon tool %})
>    - *"Method used for filtering"*: `Annotate (and filter) highly variable genes, using 'pp.highly_variable_genes'`
>        - *"Choose the flavor for identifying highly variable genes"*: `Seurat`
>
> 4. {% tool [Scanpy Inspect and manipulate](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_inspect/scanpy_inspect/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy filter** {% icon tool %})
>    - *"Method used for inspecting"*: `Scale data to unit variance and zero mean, using 'pp.scale'`
>        - *"Maximum value"*: `10.0`
>
> 5. {% tool [Scanpy cluster, embed](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_cluster_reduce_dimension/scanpy_cluster_reduce_dimension/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy Inspect and manipulate** {% icon tool %})
>    - *"Method used"*: `Computes PCA (principal component analysis) coordinates, loadings and variance decomposition, using 'pp.pca'`
>        - *"Type of PCA?"*: `Full PCA`
>
> 6. {% tool [Scanpy Inspect and manipulate](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_inspect/scanpy_inspect/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used for inspecting"*: `Compute a neighborhood graph of observations, using 'pp.neighbors'`
>        - *"Number of PCs to use"*: `30`
>
> 7. {% tool [Scanpy cluster, embed](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_cluster_reduce_dimension/scanpy_cluster_reduce_dimension/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy Inspect and manipulate** {% icon tool %})
>    - *"Method used"*: `Cluster cells into subgroups, using 'tl.louvain'`
>        - *"Flavor for the clustering"*: `vtraag (much more powerful than igraph)`
>            - *"Resolution"*: `2.0`
>
> 8. {% tool [Scanpy cluster, embed](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_cluster_reduce_dimension/scanpy_cluster_reduce_dimension/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used"*: `Embed the neighborhood graph using UMAP, using 'tl.umap'`
>
> 9. {% tool [Scanpy plot](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_plot/scanpy_plot/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used for plotting"*: `Embeddings: Scatter plot in UMAP basis, using 'pl.umap'`
>        - *"Keys for annotations of observations/cells or variables/genes"*: `louvain,Method`
>        - *"Show edges?"*: `No`
>
{: .hands_on}

Now let's take a look at our results. We'll plot one version of the UMAP showing the clusters we've just identified and another coloured by `Method` to see if that might be influencing our results.

> <hands-on-title> Visualise the Results </hands-on-title>
> 9. {% tool [Scanpy plot](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_plot/scanpy_plot/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used for plotting"*: `Embeddings: Scatter plot in UMAP basis, using 'pl.umap'`
>        - *"Keys for annotations of observations/cells or variables/genes"*: `louvain,Method`
>        - *"Show edges?"*: `No`
>
{: .hands_on}

![Two UMAP plots showing many small and fragmented clusters of cells. The first UMAP is coloured into 47 clusters. The second UMAP shows many clusters as a single colour of cells analysed with the sam[...]

> <question-title></question-title>
>
> 1. How many clusters did we identify?
> 2. Are the batches well mixed?
>
> > <solution-title></solution-title>
> >
> > 1. The first plot is coloured by cluster. We can see there are 47 clusters (Scanpy numbers the clusters starting from 0). That's a lot - this is partly because we used a relatively high clustering[...]
> > 2. The second plot shows the UMAP coloured by `Method`. Each colour here represents cells that were sequenced by a different experimental technique. We can see lots of clusters and patches of cell[...]
> >
> {: .solution}
>
{: .question}

</div>

<div class='Seurat' markdown='1'>

In the Seurat pipeline, we can use the ability of the SeuratObject to store multiple layers to split our data up before we begin the analysis. We might do this when we have different batches or if we'[...]

Splitting our data into layers means that the Seurat preprocessing tools can work on each layer separately. Seurat can treat each layer as if it were a separate dataset during preprocessing. Each laye[...]

The other tools in the Seurat pipeline, such as `RunPCA` and `FindClusters` will still work on the entire dataset.

Splitting the batches into separate layers could help to address some of the technical differences between them because of the separate preprocessing, but we'll have to wait for the results to see if [...]

> <hands-on-title> Split the Batches into Layers </hands-on-title>
>
> 1. {% tool [Seurat Integrate](toolshed.g2.bx.psu.edu/repos/iuc/seurat_integrate/seurat_integrate/5.0+galaxy0) %} with the following parameters:
>    - *"Method used"*: `Split data into layers using 'split'`
>        - *"Factor or group to use to split data"*: `Method`
>
>    > <comment-title></comment-title>
>    >
>    > We are splitting our data on `Method` as this is the column in our metadata that represents our batches. Each of the methods listed in this column will be split into its own layer.
>    {: .comment}
>
{: .hands_on}

Let's take a look to see what we've done to our data.

> <hands-on-title> Inspect the Data </hands-on-title>
>
> 2. {% tool [Seurat Data Management](toolshed.g2.bx.psu.edu/repos/iuc/seurat_data/seurat_data/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Integrate** {% icon tool %})
>    - *"Method used"*: `Inspect Seurat Object`
>        - *"Display information about"*: `General`
>
{: .hands_on}

> <question-title></question-title>
>
> 1. How many layers do we now have in our dataset?
> 2. What do these layers represent?
>
> > <solution-title></solution-title>
> >
> > 1. We can see that there are now 9 layers in our SeuratObject.
> > 2. We started out with one layer of raw data, called `counts`. That layer has now been split up according to `Method`. We now have nine `counts` layers. Each layer represents one of the batches na[...]
> >
> {: .solution}
>
{: .question}

Now that we have split our data so that each batch is in its own layer, we will cluster it. We won't perform any batch correction, so we'll see if the differences in `Method` are causing any problems [...]

We'll follow the default Seurat pipeline here, except that we'll use `30` PCs to build the neighborhood graph and cluster with a resolution of `2` as these were the parameters used in [the original Se[...]

> <comment-title></comment-title>
> Seurat has another option for preprocessing - rather than use the three separate functions presented below, you can use a single function called `SCTransform` to preform normalisation, identificatio[...]
>
> If you use `SCTransform` for preprocessing, then you'll need to choose `Yes` for `Use SCT as Normalization Method` when you run `IntegrateLayers`. The `SCTransform` normalises the data in its own wa[...]
>
> The next step after identifying clusters would usually be to look for marker genes that are differentially expressed between clusters. If you perform integration/batch correction after using `SCTran[...]
>
> The rest of the workflow will be the same as shown in this tutorial, but you will end up with different results because `SCTransform` handles preprocessing in a slightly different way than the three[...]
{: .comment}

> <hands-on-title> Cluster without Batch Correction </hands-on-title>
>
> 1. {% tool [Seurat Preprocessing](toolshed.g2.bx.psu.edu/repos/iuc/seurat_preprocessing/seurat_preprocessing/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Integrate** {% icon tool %})
>    - *"Method used"*: `Normalize with 'NormalizeData'`
>        - *"Method for normalization"*: `LogNormalize`
>
> 2. {% tool [Seurat Preprocessing](toolshed.g2.bx.psu.edu/repos/iuc/seurat_preprocessing/seurat_preprocessing/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Preprocessing** {% icon tool %})
>    - *"Method used"*: `Identify highly variable genes with 'FindVariableFeatures'`
>        - *"Method to select variable features"*: `vst`
>        - *"Output list of most variable features"*: `No`
>
> 3. {% tool [Seurat Preprocessing](toolshed.g2.bx.psu.edu/repos/iuc/seurat_preprocessing/seurat_preprocessing/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Preprocessing** {% icon tool %})
>    - *"Method used"*: `Scale and regress with 'ScaleData'`
>        - *"Regress out a variable"*: `No`
>        - *"Features to scale"*: `Variable Features`
>
> 4. {% tool [Seurat Run Dimensional Reduction](toolshed.g2.bx.psu.edu/repos/iuc/seurat_reduce_dimension/seurat_reduce_dimension/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Preprocessing** {% icon tool %})
>    - *"Method used"*: `Run a PCA dimensionality reduction using 'RunPCA'`
>
>    > <comment-title></comment-title>
>    >
>    > We will use the output from `RunPCA` in the  following section when we perform batch correction.
>    >
>    > If you're already familiar with the Seurat clustering pipeline and you just want to try using the  {% icon tool %} **Seurat Integrate** tools, then you can skip ahead to the **Clustering a[...]
>    {: .comment}
>
> 5. {% tool [Seurat Find Clusters](toolshed.g2.bx.psu.edu/repos/iuc/seurat_clustering/seurat_clustering/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Run Dimensional Reduction** {% icon tool %})
>    - *"Method used"*: `Compute nearest neighbors with 'FindNeighbors'`
>        - *"Number of dimensions from reduction to use as input"*: `30`
>
> 6. {% tool [Seurat Find Clusters](toolshed.g2.bx.psu.edu/repos/iuc/seurat_clustering/seurat_clustering/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Find Clusters** {% icon tool %})
>    - *"Method used"*: `Identify cell clusters with 'FindClusters'`
>        - *"Resolution"*: `2.0`
>        - *"Algorithm for modularity optimization"*: `1. Original Louvain`
>        - *"Name for output clusters"*: `unintegrated_clusters`
>
>    > <warning-title></warning-title>
>    >
>    > Make sure that you change the default name for the clusters to `unintegrated_clusters`!
>    {: .warning}
>
> 7. {% tool [Seurat Run Dimensional Reduction](toolshed.g2.bx.psu.edu/repos/iuc/seurat_reduce_dimension/seurat_reduce_dimension/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Find Clusters** {% icon tool %})
>    - *"Method used"*: `Run a UMAP dimensional reduction using 'RunUMAP'`
>        - *"UMAP implementation to run"*: `uwot`
>        - *"Run UMAP on dimensions, features, graph or KNN output"*: `dims`
>            - *"Number of dimensions from reduction to use as input"*: `30`
>        - In *"Advanced Options"*:
>            - *"Name for dimensional reduction"*: `umap.unintegrated`
>
>    > <warning-title></warning-title>
>    >
>    > Make sure that you change the default name for the UMAP results to `umap.unintegrated`!
>    {: .warning}
>
{: .hands_on}

Now let's take a look at our results. We'll first plot a UMAP showing the clusters we've just identified. Then, we will colour this plot in by `Method` to see if that might be influencing our results.[...]

> <hands-on-title> Visualise the Results </hands-on-title>
>
> 1. {% tool [Seurat Visualize](toolshed.g2.bx.psu.edu/repos/iuc/seurat_plot/seurat_plot/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Run Dimensional Reduction** {% icon tool %})
>    - *"Method used"*: `Visualize Dimensional Reduction with 'DimPlot'`
>        - *"Name of reduction to use"*: `umap.unintegrated`
>
> 2. {% tool [Seurat Visualize](toolshed.g2.bx.psu.edu/repos/iuc/seurat_plot/seurat_plot/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Run Dimensional Reduction** {% icon tool %})
>    - *"Method used"*: `Visualize Dimensional Reduction with 'DimPlot'`
>        - *"Name of reduction to use"*: `umap.unintegrated`
>        - In *"Advanced Options"*:
>            - *"Factor to group cells by"*: `Method`
>
{: .hands_on}

![Two UMAP plots showing many small and fragmented clusters of cells. Image A is coloured into 48 clusters. Image B shows many clusters as a single colour of cells analysed with the same method.](../.[...]

> <question-title></question-title>
>
> 1. How many clusters did we identify?
> 2. Are the batches well mixed?
>
> > <solution-title></solution-title>
> >
> > 1. The first plot is coloured by cluster. We can see there are 48 clusters (Seurat numbers the clusters starting from 0). That's a lot - this is partly because we used a relatively high clustering[...]
> > 2. The second plot shows the UMAP coloured by `Method`. Each colour here represents cells that were sequenced by a different experimental technique. We can see lots of clusters and patches of cell[...]
> >
> {: .solution}
>
{: .question}

</div>

# Clustering with Batch Correction

It looks like we do need to perform batch correction on our dataset. The Scanpy and Seurat pipelines both provide tools that can reduce the technical differences between batches. If we were combining [...]

<div class='Scanpy' markdown='1'>

Two simple changes will enable us to perform batch corrections within the Scanpy pipeline.

First, we will go back to the step where we identified Highly Variable Genes (HVGs). This time, we will add in `Method` as the batch key. Now, the tool will select the most variable genes within each [...]

Secondly, we will add in one more step using a tool called Harmony. We will use Harmony in between performing the PCA and constructing the neighborhood graph. Harmony will take the principal component[...]

> <hands-on-title> Recluster with Batch Correction </hands-on-title>
>
> 1. {% tool [Scanpy filter](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_filter/scanpy_filter/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy Inspect and manipulate** {% icon tool %})
>    - *"Method used for filtering"*: `Annotate (and filter) highly variable genes, using 'pp.highly_variable_genes'`
>        - *"Choose the flavor for identifying highly variable genes"*: `Seurat`
>        - *"Specify the batch key"*: `Method`
>
>    > <comment-title> short description </comment-title>
>    >
>    > We will specify `Method` as the batch key. This means that Highly Variable Genes will be identified within each batch and then combined.
>    {: .comment}
>
> 2. {% tool [Scanpy Inspect and manipulate](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_inspect/scanpy_inspect/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy filter** {% icon tool %})
>    - *"Method used for inspecting"*: `Scale data to unit variance and zero mean, using 'pp.scale'`
>        - *"Maximum value"*: `10.0`
>
> 3. {% tool [Scanpy cluster, embed](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_cluster_reduce_dimension/scanpy_cluster_reduce_dimension/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy Inspect and manipulate** {% icon tool %})
>    - *"Method used"*: `Computes PCA (principal component analysis) coordinates, loadings and variance decomposition, using 'pp.pca'`
>        - *"Type of PCA?"*: `Full PCA`
>
> 4. {% tool [Scanpy remove confounders](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_remove_confounders/scanpy_remove_confounders/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used for plotting"*: `Integrate multiple single-cell experiments with Harmony, using 'external.pp.harmony_integrate'`
>        - *"The name of the column in adata.obs that differentiates among experiments/batches"*: `Method`
>
> 5. {% tool [Scanpy Inspect and manipulate](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_inspect/scanpy_inspect/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy remove confounders** {% icon tool %})
>    - *"Method used for inspecting"*: `Compute a neighborhood graph of observations, using 'pp.neighbors'`
>        - *"Number of PCs to use"*: `30`
>        - *"Use the indicated representation"*: `X_pca_harmony`
>
>    > <comment-title></comment-title>
>    >
>    > We will use the corrected embedding, 'X_pca_harmony' to calculate the neighborhood graph. Make sure to enter this as the representation to use.
>    {: .comment}
>
> 6. {% tool [Scanpy cluster, embed](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_cluster_reduce_dimension/scanpy_cluster_reduce_dimension/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy Inspect and manipulate** {% icon tool %})
>    - *"Method used"*: `Cluster cells into subgroups, using 'tl.louvain'`
>        - *"Flavor for the clustering"*: `vtraag (much more powerful than igraph)`
>            - *"Resolution"*: `2.0`
>        - *"Key under which to add the cluster labels"*: `louvain_integrated`
>
>    > <comment-title></comment-title>
>    >
>    > We'll use a different name for this clustering so that we don't get confused. Enter 'louvain_integrated' as the key to add the cluster labels under. We'll use this when we plot our integrated c[...]
>    {: .comment}
>
> 7. {% tool [Scanpy cluster, embed](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_cluster_reduce_dimension/scanpy_cluster_reduce_dimension/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used"*: `Embed the neighborhood graph using UMAP, using 'tl.umap'`
>
{: .hands_on}

Now let's visualise the results again to see if the batch correction has worked. As before, we'll make one plot coloured by cluster ans another coloured by batch (`Method`). We're hoping that the batc[...]

> <hands-on-title> Recluster with Batch Correction </hands-on-title>
> 1. {% tool [Scanpy plot](toolshed.g2.bx.psu.edu/repos/iuc/scanpy_plot/scanpy_plot/1.11.5+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Annotated data matrix"*: `anndata_out` (output of **Scanpy cluster, embed** {% icon tool %})
>    - *"Method used for plotting"*: `Embeddings: Scatter plot in UMAP basis, using 'pl.umap'`
>        - *"Keys for annotations of observations/cells or variables/genes"*: `louvain_integrated,Method`
>        - *"Show edges?"*: `No`
>
>    > <comment-title> short description </comment-title>
>    >
>    > Make sure to use 'louvain_integrated' rather than 'louvain' for the cluster annotation to plot. This is the name we used for our integrated clusters.
>    {: .comment}
>
{: .hands_on}

![Two UMAP plots showing three large groups of cells, each made up of multiple clusters. The first UMAP is coloured as 25 clusters. The second UMAP shows a mix of colours across all the clusters.](../[...]

> <question-title></question-title>
>
> 1. How many clusters did we identify?
> 2. How well mixed are the batches?
>
> > <solution-title></solution-title>
> >
> > 1. The first plot shows 25 clusters (remember that Scanpy starts from cluster 0!). Although the high resolution means we still have plenty of clusters, the batch correction has reduced the number.[...]
> > 2. When we colour in the plot by `Method`, we can see that all the colours are mixed together across all of the clusters. We don't have any clusters that contain only one colour. The batch correct[...]
> >
> {: .solution}
>
{: .question}

</div>

<div class='Seurat' markdown='1'>

We will now run Seurat's batch correction tool - it's called `IntegrateLayers`, but despite the name we can use the same tool to address differences between batches as we would for integrating dataset[...]

> <comment-title></comment-title>
>
> {% tool Seurat Integrate %} provides several integration methods, which all perform the integration or batch correction in their own way. You might want to experiment by using different methods to s[...]
{: .comment}

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [Seurat Integrate](toolshed.g2.bx.psu.edu/repos/iuc/seurat_integrate/seurat_integrate/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Run Dimensional Reduction** {% icon tool %})
>    - *"Method used"*: `Apply integration methods with 'IntegrateLayers'`
>        - *"Integration method to use"*: `CCA Integration`
>        - *"Name for new dimensional reduction"*: `integrated.cca`
>
>    > <comment-title> Remember the name </comment-title>
>    >
>    > Make sure you remember the name you've used for the new dimensional reduction - we'll be using this later instead of the PCA we produced previously.
>    {: .comment}
>
{: .hands_on}

It's good practice to rejoin our layers now, so that those separate layers/batches will end up together. We don't actually need to do this now (as it won't affect the clustering results), but it is im[...]

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [Seurat Integrate](toolshed.g2.bx.psu.edu/repos/iuc/seurat_integrate/seurat_integrate/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Integrate** {% icon tool %})
>    - *"Method used"*: `Join layers with 'JoinLayers'`
>
{: .hands_on}

> <question-title></question-title>
>
> 1. How many layers are now in our dataset?
>
> > <solution-title></solution-title>
> >
> > 1. You might think that we should only have one layer in our dataset now, because we split it into nine layers that we have now rejoined. However, if you use {% tool Seurat Data Management %} to c[...]
> >
> >    In fact, if you run {% tool Seurat Data Management %} on the previous dataset in your history, from before we rejoined the layers, you'll see that it actually had 19 layers in it - each of the [...]
> >
> {: .solution}
>
{: .question}

Now let's try clustering our integrated data. We'll repeat the steps we performed earlier, but this time we'll be using the dimensional reduction produced by `Integrate Layers` instead of the PCA. The[...]

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [Seurat Find Clusters](toolshed.g2.bx.psu.edu/repos/iuc/seurat_clustering/seurat_clustering/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Integrate** {% icon tool %})
>    - *"Method used"*: `Compute nearest neighbors with 'FindNeighbors'`
>        - *"Name of reduction to use"*: `integrated.cca`
>        - *"Number of dimensions from reduction to use as input"*: `30`
>
>    > <comment-title></comment-title>
>    >
>    > Make sure to use `integrated.cca` as the reduction, not the `pca` we made previously.
>    {: .comment}
>
> 2. {% tool [Seurat Find Clusters](toolshed.g2.bx.psu.edu/repos/iuc/seurat_clustering/seurat_clustering/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Find Clusters** {% icon tool %})
>    - *"Method used"*: `Identify cell clusters with 'FindClusters'`
>        - *"Resolution"*: `2.0`
>        - *"Algorithm for modularity optimization"*: `1. Original Louvain`
>        - *"Name for output clusters"*: `cca_clusters`
>
>    > <comment-title></comment-title>
>    >
>    > Make sure that you know what name you used for your clusters as we'll use this for the UMAP!
>    {: .comment}
>
> 3. {% tool [Seurat Run Dimensional Reduction](toolshed.g2.bx.psu.edu/repos/iuc/seurat_reduce_dimension/seurat_reduce_dimension/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Find Clusters** {% icon tool %})
>    - *"Method used"*: `Run a UMAP dimensional reduction using 'RunUMAP'`
>        - *"Name of reduction to use"*: `integrated.cca`
>        - *"UMAP implementation to run"*: `uwot`
>        - *"Run UMAP on dimensions, features, graph or KNN output"*: `dims`
>            - *"Number of dimensions from reduction to use as input"*: `30`
>        - In *"Advanced Options"*:
>            - *"Name for dimensional reduction"*: `umap.cca`
>
>    > <comment-title></comment-title>
>    >
>    > Make sure that you know what name you used for your UMAP results as we'll use this for the plots!
>    {: .comment}
>
{: .hands_on}

Let's see how the batch correction has changed our results. As before, we'll make one plot coloured by cluster and then another coloured by batch (`Method`). We're hoping that the batches in that seco[...]

> <hands-on-title> Task description </hands-on-title>
>
> 1. {% tool [Seurat Visualize](toolshed.g2.bx.psu.edu/repos/iuc/seurat_plot/seurat_plot/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Run Dimensional Reduction** {% icon tool %})
>    - *"Method used"*: `Visualize Dimensional Reduction with 'DimPlot'`
>        - *"Name of reduction to use"*: `umap.cca`
>
> 2. {% tool [Seurat Visualize](toolshed.g2.bx.psu.edu/repos/iuc/seurat_plot/seurat_plot/5.0+galaxy0) %} with the following parameters:
>    - {% icon param-file %} *"Input file with the Seurat object"*: `rds_out` (output of **Seurat Run Dimensional Reduction** {% icon tool %})
>    - *"Method used"*: `Visualize Dimensional Reduction with 'DimPlot'`
>        - *"Name of reduction to use"*: `umap.cca`
>        - In *"Advanced Options"*:
>            - *"Factor to group cells by"*: `Method`
>
{: .hands_on}

![Two UMAP plots showing three large groups of cells, each made up of multiple clusters. Image A is coloured as 25 clusters. Image B shows a mix of colours across all the clusters.](../../images/scrna[...]

> <question-title></question-title>
>
> 1. How many clusters did we identify?
> 2. How well mixed are the batches?
>
> > <solution-title></solution-title>
> >
> > 1. The first plot shows 25 clusters (remember that Seurat starts from cluster 0!). Although the high resolution means we still have plenty of clusters, the batch correction has reduced the number.[...]
> > 2. When we colour in the plot by `Method`, we can see that all the colours are mixed together across all of the clusters. We don't have any clusters that contain only one colour. The batch correct[...]
> >
> {: .solution}
>
{: .question}

</div>

# Comparing the Results

Before vs after batch correction
Side by side pictures...
Discuss what batch correction has done to the data


> <comment-title></comment-title>
>
> We've seen from our plots that the batch correction has mixed the different methods together, but this alone isn't enough to convince us that the batch correction has been successful. As always with[...]
>
> In order to do this, we would usually take a closer look at the clusters to work out what they represent, for example by looking for clusters expressing genes that are known to be present in specifi[...]
>
>If you look back at the cell metadata table we created at the beginning of this tutorial, you'll see there is an annotation called `CellType`. We can colour in our UMAPs using this annotation instead[...]
>
> If the cell types are all blended together across the entire UMAP (as with our `Method` plot) then this would be a sign that something has gone wrong. When we are performing batch correction or inte[...]
>
> The `CellType` annotation won't match up exactly with our clusters (remember we used a high resolution to make lots of clusters!) but they certainly shouldn't be scattered across the whole plot!
{: .comment}

# Conclusion

{% icon congratulations %} Well done, you've successfully used Seurat to prepare and cluster single cell data. You might want to check your results against the example histories for the [Scanpy](add l[...]

In this tutorial, we've learned how to perform batch correction or integration when analysing single cell data with either the Scanpy or Seurat pipelines. If you want to learn more about these pipelin[...]
