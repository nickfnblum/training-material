# Tutorial - GTEx Tissue Modeling with Galaxy Image Learner

A Galaxy tutorial for converting GTEx v11 gene expression profiles into grayscale images and training an Image Learner tissue classifier.

## Overview

This tutorial shows how to load GTEx gene TPM values, join sample attributes, subset tissues, convert each expression vector into an image, and classify tissue labels with a no-code Galaxy machine learning tool. It uses GTEx v11 files from public Google Cloud Storage URLs and can run on any Galaxy instance that has Image Learner installed.

## Required GTEx Files

- `GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz`: https://storage.googleapis.com/adult-gtex/bulk-gex/v11/rna-seq/GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz
- `GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt`: https://storage.googleapis.com/adult-gtex/annotations/v11/metadata-files/GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt

GTEx Portal source pages:

- Bulk tissue expression: https://gtexportal.org/home/downloads/adult-gtex/bulk_tissue_expression
- Metadata: https://gtexportal.org/home/downloads/adult-gtex/metadata

## Outputs Created Before Galaxy Upload

- `gtex_image_learner_input.csv`
- `gtex_tissue_images.zip`
- Optional audit files: `expression_matrix.csv`, `metadata_base.csv`

## Galaxy Task

- **Task**: Multi-class tissue classification
- **Input image representation**: log-transformed TPM vector padded to a square grayscale image
- **Label**: `SMTSD` tissue label from GTEx sample attributes
- **Tool**: Galaxy Image Learner

## Files

- `tutorial.md` - Main hands-on tutorial
- `tutorial.bib` - Bibliography
- `data-library.yaml` - GTEx raw file URL references
- `workflows/` - Image Learner workflow skeleton
- `faqs/` - Tutorial FAQ
