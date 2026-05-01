---
layout: faq-page
---

## What GTEx files are used?

This tutorial uses `GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz` and `GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt`, downloaded manually from the GTEx Portal.

## What is the label?

The target label is `SMTSD`, the detailed tissue label in the GTEx sample attributes file.

## Why convert expression data into images?

Image Learner trains image models. The preprocessing step makes one grayscale image per sample by log-transforming the sample's TPM vector, padding it to a square, and saving it as a JPEG. This preserves the expression values in a fixed-size input format that Image Learner can consume.

## Is this workflow AnVIL-specific?

No. It is based on the AnVIL_Galaxy rationale, but this version uses manually downloaded GTEx v11 files and any Galaxy server with Image Learner installed.

## Which Galaxy server can run it?

Any Galaxy instance can run the tutorial if Image Learner is installed from the ToolShed and the instance has enough compute for the selected sample count and model.

## Can I use more tissues?

Yes. Edit `SELECTED_TISSUES` in the preprocessing script. Start with a small balanced tissue set, confirm the workflow runs, then scale up.

## Why use balanced sampling?

Balanced sampling makes tutorial metrics easier to interpret and prevents highly represented tissues from dominating accuracy. For research, compare balanced and naturally distributed tissue sets.

## What metrics should I inspect?

Use held-out test accuracy, weighted precision, weighted recall, weighted F1, per-class metrics, and the confusion matrix. Per-class metrics are especially important when tissue counts are imbalanced.
