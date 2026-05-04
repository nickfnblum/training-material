# Tutorial - GTEx Tissue Modeling with Galaxy Image Learner

A Galaxy tutorial for converting GTEx v11 gene expression profiles into grayscale images and training an Image Learner tissue classifier.

## Overview

This tutorial shows how to classify GTEx tissues with a no-code Galaxy machine learning tool. The recommended path uses prepared metadata and image files from Zenodo that are ready for Galaxy Image Learner. An optional path shows how to rebuild those files from raw GTEx v11 expression and sample metadata.

## Recommended Zenodo Files

- `selected_gtex_v11_tpm_image_tissue_labels.csv`: https://zenodo.org/records/19963477/files/selected_gtex_v11_tpm_image_tissue_labels.csv
- `selected_gtex_v11_tpm_image_tissue_dataset.zip`: https://zenodo.org/records/19963477/files/selected_gtex_v11_tpm_image_tissue_dataset.zip

## Optional Raw GTEx Files

- `GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz`: https://storage.googleapis.com/adult-gtex/bulk-gex/v11/rna-seq/GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz
- `GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt`: https://storage.googleapis.com/adult-gtex/annotations/v11/metadata-files/GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt

GTEx Portal source pages:

- Bulk tissue expression: https://gtexportal.org/home/downloads/adult-gtex/bulk_tissue_expression
- Metadata: https://gtexportal.org/home/downloads/adult-gtex/metadata

## Optional Outputs Created Before Galaxy Upload

- `ludwig_input.csv`
- `output_images.zip`
- Optional audit file: `metadata_base.csv`

## Galaxy Task

- **Task**: Multi-class tissue classification
- **Input image representation**: log-transformed TPM vector padded to a square grayscale image
- **Label**: `SMTSD` tissue label from GTEx sample attributes, pre-mapped into the metadata table
- **Tool**: Galaxy Image Learner

## Files

- `tutorial.md` - Main hands-on tutorial
- `tutorial.bib` - Bibliography
- `data-library.yaml` - Zenodo and raw GTEx URL references
- `workflows/` - Image Learner workflow skeleton
- `faqs/` - Tutorial FAQ
