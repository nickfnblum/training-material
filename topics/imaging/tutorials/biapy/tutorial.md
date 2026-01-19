---
layout: tutorial_hands_on
title: Training Custom YOLO Models for Segmentation of Bioimages
level: Intermediate
subtopic: advanced
zenodo_link: https://zenodo.org/records/16096782
questions:
- Why should I use ByaPy for deep learning on bioimages?
- How can I run BiaPy workflows using Galaxy?
objectives:
- Run inference on a set of images using a pre-trained model from BioImage.IO and a pre-configured BiaPy file
- Fine-tune two existing BioImage.IO models by creating a BiaPy configuration file from scratch and run inference on a set of 3D images
time_estimation: 2H
key_points:
- A configuration file can be run in parallel to test different pre-trained models from BioImage.IO
- DL models can be fine-tuned in Galaxy to adapt to specific datasets
contributors:
- rmassei
- danifranco
tags: 
  - Image segmentation
  - Image annotation
  - Deep learning
---

The application of supervised and unsupervised **Deep Learning (DL)** methods in bioimage analysis have been constantly increasing in biomedical 
researcg in the last decades. DL algorithms allow automatically classifying complex biological structures by learning complex patterns and features directly from large-scale imaging data, medical scans, or high-throughput biological datasets. Furthermore, trained models can be easily
share on online repositories [(BioImage.IO)](https://bioimage.io/#/models) to be reused by other scientists and support open science.

However, running DL models often require high-level programming skills which can be often be a barrier to general audience especially the 
one without a proper computation background. Additionally, many DL models require GPU acceleration, which is not always accessible to all researchers. 
Such obstacoles might the practical and routine adoption of DL models in bioimaging.

*How to make DL models accessible to a larger audience?* Well, [BiaPy](https://biapy.readthedocs.io/en/latest/) is an open source library and application that streamlines the use of common deep-learning workflows for a large variety of bioimage analysis tasks, including 2D and 3D semantic segmentation, instance segmentation, object detection, image denoising, single image super-resolution, self-supervised learning (for model pretraining), image classification and image-to-image translation. 

In this training, you will learn how to execute a BiaPy worflow directly in Galaxy. In particular, we will run [inference](https://en.wikipedia.org/wiki/Deep_learning) on a set of images using a pre-trained model from BioImage.IO. Furthermore, we fine-tune two existing BioImage.IO models and run inference on a set of 3D images obtained from the CartoCell dataset available on Zenodo.

Said so... Let's start to train these models!

## Get the image data and the YAML configuration file

> <hands-on-title>Data Upload</hands-on-title>
>
> 1. Create a new history for this tutorial.
>
> 2. Download the following image and import it into your Galaxy history.
>    - [`example_image.tiff`](workflows/test-data/example_image.tiff)
>    - [`example_image_ground_truth.tiff`](workflows/test-data/example_image_ground_truth.tiff)
>    
>    If you are importing the image via URL:
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>    If you are importing the image from the shared data library:
>
>    {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
>
> 3. Rename the datasets appropriately if needed (e.g. `"original_image"`, `"ground_truth"`)
>
> 4. Confirm the datatypes are correct (`tiff` for both images)
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
> 
>    {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
{: .hands_on}

## Prepare the image datasets for processing

## Run inference on a set of images using a pre-trained model

## Fine-tune two existing BioImage.IO models by creating a BiaPy configuration file from scratch

## Visualize the result from the inference

## Compare different pre-trained models 

## Worflow to test different models in one run

## Conclusions