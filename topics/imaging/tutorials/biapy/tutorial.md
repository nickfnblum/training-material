---
layout: tutorial_hands_on
title: "Execute a BiaPy workflow in Galaxy using a YAML configuration file"
zenodo_link: https://zenodo.org/records/10973241
level: Intermediate
subtopic: advanced
questions:
- "How can we make Deep Learning (DL) models accessible to a broader audience ?"
- "What is BiaPy and how does it streamline deep learning workflows for bioimage analysis?"
- "How can I execute a BiaPy workflow directly within the Galaxy platform?"
- "How do I utilize pre-trained models from the BioImage.IO repository to perform inference on image data?"
objectives:
- Learn to configure a BiaPy workflow by editing a YAML file to define hardware settings, data paths, and model selection.
- Execute an inference workflow in Galaxy using two different pre-trained models sourced from BioImage.IO
time_estimation: 2H
key_points:
- BiaPy is an open-source tool designed to lower the technical barriers for using DL in bioimage analysis.
- Pre-trained models hosted on BioImage.IO can be used directly in Galaxy thanks to the BiaPy tool
- The BiaPy workflow can be controlled via a YAML configuration file, which specifies the task type and model source
contributors:
- rmassei
- danifranco
tags: 
  - Image segmentation
  - Image annotation
  - Deep learning
---

The application of supervised and unsupervised **Deep Learning (DL)** methods in bioimage analysis have been constantly increasing in biomedical research in the last decades ([Esteva, A. et al.](https://www.nature.com/articles/s41746-020-00376-2)). DL algorithms allow automatically classifying complex biological structures by learning complex patterns and features directly from large-scale imaging data, medical scans, or high-throughput biological datasets ([Franco-Barranco et al., 2025](https://www.nature.com/articles/s41592-025-02699-y)). Furthermore, trained models can be easily
share on online repositories [(BioImage.IO)](https://bioimage.io/#/models) to be reused by other scientists and support open science. 

However, running DL models often require high-level programming skills which can be often be a barrier to general audience especially the 
one without a proper computation background. Additionally, many DL models require GPU acceleration, which is not always accessible to all researchers. 
Such obstacoles might the practical and routine adoption of DL models in bioimaging. 

*So, how to make DL models accessible to a larger audience?* Well, [BiaPy](https://biapy.readthedocs.io/en/latest/) is an open source library and application that streamlines the use of common deep-learning workflows for a large variety of bioimage analysis tasks, including 2D and 3D semantic segmentation, instance segmentation, object detection, image denoising, single image super-resolution, self-supervised learning (for model pretraining), image classification and image-to-image translation. 

In this training, you will learn how to execute a BiaPy worflow directly in Galaxy. We learn how to run [inference](https://en.wikipedia.org/wiki/Deep_learning) on a set of images using two pre-trained models from BioImage.IO defined in a 
BiaPy YAML configuration file. A BiaPy YAML configuration file includes information about the hardware to be used, such as the number of CPUs or GPUs, the specific task or workflow, the model to be used, optional hyperparameters, the optimizer, and the paths for loading and storing data. 

![example-yaml.png](../../images/biapy/example-yaml.png "Example of a BiaPy YAML file where the model merry-water-buffalo is defined (red box)")

For this training we will use two configuration file with two different models from BioImage.IO.

Let's start with BiaPy!

> <agenda-title></agenda-title>
>
> In this tutorial, we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}

## Getting test data and the BiaPy YAML configuration file

The dataset required for this tutorial is available from [Zenodo]({{ page.zenodo_link }}). The CartoCell dataset containts whole epithelial cysts acquired at low resolution with minimal human intervention ([more information]({{ page.zenodo_link }})). The dataset is divided test, train and validation data each folder containg images and associated masks.

In order to simplify the your upload, we already prepared the test images and YAML files in the **Data Library** which is easy to be accessed on the left pannel in Galaxy.

{% snippet faqs/galaxy/datasets_import_from_data_library.md %}

You should then have in your history the following files:
- `01_raw_image.tiff`
- `01_raw_mask.tiff`
- `02_raw_image.tiff`
- `02_raw_mask.tiff`
- `conf_cartocell_model1.yaml`
- `conf_cartocell_model2.yaml`

## Run inference using the BioImage.IO pre-trained model

Now we can set up the BiaPy tool with the [venoumus-swam model](https://bioimage.io/#/artifacts/venomous-swan) which is defined in `conf_cartocell_model1.yaml`

> <hands-on-title>Configure the BiaPy Tool with venoumus-Swam</hands-on-title>
>
> 1. {% tool [Build a workflow with BiaPy](toolshed.g2.bx.psu.edu/repos/iuc/biapy/biapy/3.6.5+galaxy0) %} with the following parameters to extract metadata from the image:
>
>- *Do you have a configuration file?* : `Yes, I have one and I want to run BiaPy directly`
>
>- *Select a configuration file*: `conf_cartocell_model1.yaml`
>
>- *Specify the test raw images*: `01_raw_image.tiff` and `02_raw_image.tiff`
>
>- *Specify the test ground truth/target images*: `01_raw_mask.tiff` and `02_raw_mask.tiff`
>
>- On *Select output* check the boxes:
>     - [x]  `Test predictions (if exist)`
>     - [x]  `Post-processed test predictions (if exist)` 
>     - [x]  `Evaluation metrics (if exist, on test data)`
{: .hands_on}

Once the tool finish its run you will have three different datasets in your history.

**Test predictions**: Full-size output images produced by the model on the test set. Because the model predicts small, overlapping patches, these patch outputs are merged back together to form one prediction per original image.

**Post-Processed Test Prediction**: Test predictions after automatic “clean-up” steps defined in the configuration. These steps can refine the raw output (for example, removing small spurious regions or separating touching objects). In this tutorial, Voronoi tessellation is applied to help split instances.

**Test metrics:** Numerical scores that measure how well the predictions match the ground truth (if provided). In instance segmentation, the report typically includes:

- IoU per output channel (how well pixel regions overlap), and

- Matching metrics (how well individual predicted objects match true objects), shown for raw predictions and post-processed predictions.

...but you can find more info in [BiaPy documentation](https://biapy.readthedocs.io/en/latest/)!

## Visualize the results

As first step, we can visualize one slice of the segmentation on the original image. We will work with `02_raw_image.tiff`

> <hands-on-title>Extract 2D results from the BiaPy output</hands-on-title>
>
> 1. [Extract Dataset](__EXTRACT_DATASET__) %} with the following parameters:
>- Input List: '"Build a workflow with BiaPy on dataset 2, 3, and others: Post-processed test predictions"'
>- How should a dataset be selected?:
>     - Select by Index
>         - Element index: `1`
>
> 2. Rename {% icon galaxy-pencil %} the dataset to `biapy_prediction.tiff`
>
> 3. {% tool [Convert image format](toolshed.g2.bx.psu.edu/repos/imgteam/bfconvert/ip_convertimage/6.7.0+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `biapy_prediction.tiff`
>    - *"Extract series"*: `All series`
>    - *"Extract timepoint"*: `All timepoints`
>    - *"Extract channel"*: `All channels`
>    - *"Extract z-slice"*: `Extract z-slice`
>        - *"Z-slice id"* `22`
>    - *"Extract range"*: `All images`
>    - *"Extract crop"*: `Full image`
>    - *"Tile image"*: `No tiling`
>    - *"Pyramid image"*: `No Pyramid`
>
> 4. Rename {% icon galaxy-pencil %} the dataset to `stack_biapy_prediction.tiff`
>
> 5. {% tool [Convert image format](toolshed.g2.bx.psu.edu/repos/imgteam/bfconvert/ip_convertimage/6.7.0+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `01_raw_image.tiff`
>    - *"Extract series"*: `All series`
>    - *"Extract timepoint"*: `All timepoints`
>    - *"Extract channel"*: `All channels`
>    - *"Extract z-slice"*: `Extract z-slice`
>        - *"Z-slice id"* `22`
>    - *"Extract range"*: `All images`
>    - *"Extract crop"*: `Full image`
>    - *"Tile image"*: `No tiling`
>    - *"Pyramid image"*: `No Pyramid`
>
> 6. Rename {% icon galaxy-pencil %} the dataset to `stack_raw_image.tiff`
>
> 7. {% tool [Overlay images](toolshed.g2.bx.psu.edu/repos/imgteam/overlay_images/ip_overlay_images/0.0.4+galaxy4) %} with the following parameters to convert the image to PNG:
>    - *"Type of the overlay"*: `Segmentation contours over image`
>    - {% icon param-file %} *"Intensity image"*: `stack_raw_image.tiff` file
>    - {% icon param-file %} *"Label map"*: `stack_biapy_prediction.tiff` file (output of {% tool [Convert binary image to label map](toolshed.g2.bx.psu.edu/repos/imgteam/binary2labelimage/ip_binary_to_labelimage/0.5+galaxy0) %})
>    - *"Contour thickness"*: `1`
>    - *"Contour color"*: `green`
>    - *"Show labels"*: `no`
>8. Rename {% icon galaxy-pencil %} the dataset to `2D-overlay_biapy.tiff`
{: .hands_on}

The segmentation results for the 22th z-stack are shown below:

![comparison_2D_biapy2.png](../../images/biapy/comparison_2D_biapy2.png "Segmentation results with the venoumus-swam model"){: width="50%"}

We can also do better and visualize the full 3D segmentation using the [LibCarna](https://github.com/kostrykin/LibCarna) tool in Galaxy!

> <hands-on-title>Visual 3D</hands-on-title>
>
> 1. {% tool [Render 3-D image data](toolshed.g2.bx.psu.edu/repos/imgteam/libcarna_render/libcarna_render/0.2.0+galaxy2) %} with the following parameters:
>    - {% icon param-file %} *"Input image (3-D)"*: `01_raw_image.tiff` 
>    - *"Unit of the intensity values"*: `No unit`
>    - *"Coordinate system"*: `Point Y to the Top`
>    - *"Rendering mode"*: `Maximum Intensity Projection (MIP)`
>    - *"Color map"*: `gist_gray`
>    - *"Camera parameters"*:
>      - *"Distance"*: `200`
>    - *"Video parameters"*:
>      - *"Frames"*: `400`
>    - *"Render mask overlay"*:
>      - {% icon param-file %} *"Mask overlay (3-D) *"*: `biapy_prediction.tiff`
{: .hands_on}

<video loop="true" autoplay="autoplay" muted width="75%">
    <source src="../../images/biapy/carto_segm.mp4" type="video/mp4"/>
</video>

Pretty cool, eh? 

We can do the same for also for `02_raw_image.tiff`:

<video loop="true" autoplay="autoplay" muted width="75%">
    <source src="../../images/biapy/carto_segm_2.mp4" type="video/mp4"/>
</video>

## Compare different pre-trained models 

Lets now run the BiaPy tool again but this time with the [merry-water-buffalo](https://bioimage.io/#/artifacts/merry-water-buffalo) model:

> <hands-on-title>Configure the BiaPy Tool for Merry-Water-Buffalo</hands-on-title>
>
> 1. {% tool [Build a workflow with BiaPy](toolshed.g2.bx.psu.edu/repos/iuc/biapy/biapy/3.6.5+galaxy0) %} with the following parameters to extract metadata from the image:
>
>- *Do you have a configuration file?* : `Yes, I have one and I want to run BiaPy directly`
>
>- *Select a configuration file*: `conf_cartocell_model2.yaml`
>
>- *Specify the test raw images*: `01_raw_image.tiff` and `02_raw_image.tiff`
>
>- *Specify the test ground truth/target images*: `01_raw_mask.tiff` and `02_raw_mask.tiff`
>
>- On *Select output* check the boxes:
>     - [x]  `Test predictions (if exist)`
>     - [x]  `Post-processed test predictions (if exist)` 
>     - [x]  `Evaluation metrics (if exist, on test data)`
{: .hands_on}

We can visualize again the results using the previous approach:

> <hands-on-title>Extract the results from the BiaPy output</hands-on-title>
>
> 1. [Extract Dataset](__EXTRACT_DATASET__) %} with the following parameters:
>- Input List: '"Build a workflow with BiaPy on dataset 2, 3, and others: Post-processed test predictions"'
>- How should a dataset be selected?:
>     - Select by Index
>         - Element index: `1`
>
> 2. Rename {% icon galaxy-pencil %} the dataset to `biapy_prediction.tiff`
>
> 3. {% tool [Convert image format](toolshed.g2.bx.psu.edu/repos/imgteam/bfconvert/ip_convertimage/6.7.0+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `biapy_prediction.tiff`
>    - *"Extract series"*: `All series`
>    - *"Extract timepoint"*: `All timepoints`
>    - *"Extract channel"*: `All channels`
>    - *"Extract z-slice"*: `Extract z-slice`
>        - *"Z-slice id"* `22`
>    - *"Extract range"*: `All images`
>    - *"Extract crop"*: `Full image`
>    - *"Tile image"*: `No tiling`
>    - *"Pyramid image"*: `No Pyramid`
>
> 4. Rename {% icon galaxy-pencil %} the dataset to `stack_biapy_prediction.tiff`
>
> 5. {% tool [Convert image format](toolshed.g2.bx.psu.edu/repos/imgteam/bfconvert/ip_convertimage/6.7.0+galaxy3) %} with the following parameters:
>    - {% icon param-file %} *"Input Image"*: `01_raw_image.tiff`
>    - *"Extract series"*: `All series`
>    - *"Extract timepoint"*: `All timepoints`
>    - *"Extract channel"*: `All channels`
>    - *"Extract z-slice"*: `Extract z-slice`
>        - *"Z-slice id"* `22`
>    - *"Extract range"*: `All images`
>    - *"Extract crop"*: `Full image`
>    - *"Tile image"*: `No tiling`
>    - *"Pyramid image"*: `No Pyramid`
>
> 6. Rename {% icon galaxy-pencil %} the dataset to `stack_raw_image.tiff`
>
> 7. {% tool [Overlay images](toolshed.g2.bx.psu.edu/repos/imgteam/overlay_images/ip_overlay_images/0.0.4+galaxy4) %} with the following parameters to convert the image to PNG:
>    - *"Type of the overlay"*: `Segmentation contours over image`
>    - {% icon param-file %} *"Intensity image"*: `stack_raw_image.tiff` file
>    - {% icon param-file %} *"Label map"*: `stack_biapy_prediction.tiff` file (output of {% tool [Convert binary image to label map](toolshed.g2.bx.psu.edu/repos/imgteam/binary2labelimage/ip_binary_to_labelimage/0.5+galaxy0) %})
>    - *"Contour thickness"*: `1`
>    - *"Contour color"*: `green`
>    - *"Show labels"*: `no`
>8. Rename {% icon galaxy-pencil %} the dataset to `2D-overlay_biapy.tiff`
{: .hands_on}

Results will look like this:

![comparison_2D_biapy3.png](../../images/biapy/comparison_2D_biapy3.png "Segmentation results with the merry-water-buffalo model"){: width="50%"}

Just by visually inspecting the results, it seems that **venomous-swan** model predict more sharp contours and cells, but **merry-water-buffalo** seems to capture better the cells with less merges... however segmentation is a bit noiser.

It is hard to say that a prediction is better than other by just looking at a slice when working in 3D! 

The **Test metrics** will give us a better overview on which model is performing better!

**Venoumus-swam** struggles mainly with missing objects (low recall)

At IoU ≥ 0.5:

    Precision: 0.328
    Recall: 0.171
    F1: 0.222
    PQ: 0.157

**Water-buffalo** is clearly stronger at instance detection and segmentation

At a commonly used matching threshold (IoU ≥ 0.5), averaged across the 2 test images:

    Precision: 0.761
    Recall: 0.631
    F1: 0.689
    PQ: 0.473

So **Water-buffalo** is better both in:

- Finding objects (higher recall)
- Keeping predictions correct (higher precision).


## Conclusions

This need to be done...

# Optional: Extract complete training workflow from history
As an optional step, you can extract a complete workflow from your Galaxy history. This allows you to save and reuse the entire training process as a reproducible and shareable workflow.

{% snippet faqs/galaxy/workflows_extract_from_history.md %}

