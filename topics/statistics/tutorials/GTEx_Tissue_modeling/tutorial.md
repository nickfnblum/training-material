---
layout: tutorial_hands_on
level: Intermediate
title: GTEx Tissue Modeling with Galaxy Image Learner
questions:
  - "How can GTEx gene expression profiles be transformed into image-like inputs for Image Learner?"
  - "How do GTEx sample annotations provide tissue labels for supervised classification?"
  - "How can this workflow be run on any Galaxy server with Image Learner installed?"
objectives:
  - "Prepare GTEx v11 gene TPM and sample annotation files for tissue classification."
  - "Create an Image Learner input table and ZIP archive of grayscale expression images."
  - "Train and evaluate a multi-class tissue classifier in Galaxy."
time_estimation: "2h"
key_points:
- GTEx v11 gene TPM profiles can be represented as fixed-size grayscale images, one image per sample.
- `SMTSD` from the GTEx sample attributes file is used as the tissue label.
- This tutorial can be run on any Galaxy instance where Image Learner is available or installable.
contributors:
- paulocilasjr
- allissadillman
- nakucher
- jgoecks
tags:
- GTEx
- Tissue Classification
- Gene Expression
- Image Learner
- Deep Learning
---

This tutorial uses GTEx v11 expression and sample metadata files from the GTEx public Google Cloud Storage bucket:

1. [`GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz`](https://storage.googleapis.com/adult-gtex/bulk-gex/v11/rna-seq/GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz)
2. [`GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt`](https://storage.googleapis.com/adult-gtex/annotations/v11/metadata-files/GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt)

The first file contains gene-level TPM expression values. The second file contains sample metadata, including tissue labels. We subset samples by tissue, convert each expression profile into a grayscale image, create a metadata CSV with `image_path` and `label`, and train a tissue classifier with Galaxy Image Learner.

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

> <comment-title>Background</comment-title>
>
> GTEx is a large reference resource for studying human gene expression across tissues {% cite GTEx2020 %}. The modeling idea used here treats each sample's vector of gene TPM values as a structured image: values are log-transformed, padded to a square, and saved as a grayscale image. A tissue classifier can then learn tissue-specific expression patterns using Image Learner.
>
{: .comment}

# Input Data

Download the two required GTEx v11 files:

| File | Direct URL | Use |
|---|---|---|
| `GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz` | [GTEx v11 gene TPM GCT](https://storage.googleapis.com/adult-gtex/bulk-gex/v11/rna-seq/GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz) | Gene TPM matrix |
| `GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt` | [GTEx v11 sample attributes](https://storage.googleapis.com/adult-gtex/annotations/v11/metadata-files/GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt) | Sample labels and QC metadata |

The GCT expression matrix has genes as rows and samples as columns. The sample attributes file includes:

| Column | Meaning |
|---|---|
| `SAMPID` | GTEx sample identifier. This must match expression matrix sample columns. |
| `SMTS` | Broad tissue category. |
| `SMTSD` | Detailed tissue label used as the classification target in this tutorial. |

# Workflow Overview

The workflow uses the following steps:

1. Load the GTEx TPM GCT file.
2. Transpose the matrix so samples are rows and genes are columns.
3. Join sample IDs with GTEx sample attributes.
4. Select a manageable set of tissues and a balanced number of samples per tissue.
5. Convert each sample's TPM vector into a grayscale image.
6. Train Image Learner to predict tissue label from the generated image.

The tutorial is written for any Galaxy instance that has Image Learner installed, or where an administrator can install Image Learner from the ToolShed.

# Prepare the Image Learner Inputs

Image Learner expects a metadata table and an image ZIP archive. The metadata table must include one row per image and at least two columns:

| Column | Description |
|---|---|
| `image_path` | Image filename inside the ZIP archive. |
| `label` | Tissue label, derived from `SMTSD`. |

The script below creates:

- `expression_matrix.csv`: selected samples by genes
- `metadata_base.csv`: selected sample IDs and labels
- `gtex_tissue_images.zip`: grayscale expression images
- `gtex_image_learner_input.csv`: Image Learner metadata table

> <hands-on-title>Generate GTEx expression images locally</hands-on-title>
>
> 1. Place the two downloaded GTEx files in the same directory.
>
> 2. Save the following script as `prepare_gtex_image_learner.py`.
>
>    ```python
>    import math
>    import os
>    import zipfile
>
>    import numpy as np
>    import pandas as pd
>    from matplotlib import pyplot as plt
>
>    TPM_GCT = "GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz"
>    SAMPLE_ATTRIBUTES = "GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt"
>
>    SELECTED_TISSUES = [
>        "Brain - Cortex",
>        "Heart - Left Ventricle",
>        "Liver",
>        "Lung",
>        "Muscle - Skeletal",
>        "Adipose - Subcutaneous",
>        "Skin - Sun Exposed (Lower leg)",
>    ]
>    SAMPLES_PER_TISSUE = 200
>    RANDOM_SEED = 42
>    IMAGE_DIR = "gtex_tissue_images"
>
>
>    def load_gct(path):
>        df = pd.read_csv(path, sep="\t", skiprows=2, index_col=0, compression="gzip")
>        df = df.iloc[:, 1:].transpose()
>        df.index.name = "sample_id"
>        return df
>
>
>    def load_metadata(path):
>        return pd.read_csv(path, sep="\t")
>
>
>    def select_samples(expr, annot):
>        annot = annot[annot["SAMPID"].isin(expr.index)]
>        annot = annot[annot["SMTSD"].isin(SELECTED_TISSUES)]
>        selected = []
>        for tissue, group in annot.groupby("SMTSD"):
>            n = min(SAMPLES_PER_TISSUE, len(group))
>            selected.append(group.sample(n=n, random_state=RANDOM_SEED))
>        meta = pd.concat(selected).drop_duplicates(subset=["SAMPID"])
>        meta = meta.sort_values(["SMTSD", "SAMPID"])
>        filtered = expr.loc[meta["SAMPID"]]
>        labels = meta[["SAMPID", "SMTSD"]].rename(columns={"SAMPID": "sample_id", "SMTSD": "label"})
>        return filtered, labels
>
>
>    def row_to_image(row):
>        values = np.log1p(row.to_numpy(dtype=np.float32))
>        size = math.ceil(math.sqrt(values.size))
>        padded = np.zeros(size * size, dtype=np.float32)
>        padded[: values.size] = values
>        return padded.reshape((size, size))
>
>
>    def write_images(expr, image_dir):
>        os.makedirs(image_dir, exist_ok=True)
>        for sample_id, row in expr.iterrows():
>            image = row_to_image(row)
>            plt.imsave(os.path.join(image_dir, f"{sample_id}.jpg"), image, cmap="gray", format="jpg")
>
>
>    def zip_images(image_dir, zip_path):
>        with zipfile.ZipFile(zip_path, "w", compression=zipfile.ZIP_DEFLATED) as handle:
>            for filename in sorted(os.listdir(image_dir)):
>                if filename.endswith(".jpg"):
>                    handle.write(os.path.join(image_dir, filename), arcname=filename)
>
>
>    expr = load_gct(TPM_GCT)
>    annot = load_metadata(SAMPLE_ATTRIBUTES)
>    expr, labels = select_samples(expr, annot)
>
>    expr.to_csv("expression_matrix.csv")
>    labels.to_csv("metadata_base.csv", index=False)
>    write_images(expr, IMAGE_DIR)
>
>    image_input = labels.copy()
>    image_input["image_path"] = image_input["sample_id"] + ".jpg"
>    image_input = image_input[["image_path", "label", "sample_id"]]
>    image_input.to_csv("gtex_image_learner_input.csv", index=False)
>    zip_images(IMAGE_DIR, "gtex_tissue_images.zip")
>
>    print(f"Expression matrix: {expr.shape[0]} samples x {expr.shape[1]} genes")
>    print(labels["label"].value_counts().sort_index())
>    print("Created gtex_image_learner_input.csv and gtex_tissue_images.zip")
>    ```
>
> 3. Run the script.
>
>    ```bash
>    python prepare_gtex_image_learner.py
>    ```
>
{: .hands_on}

> <tip-title>Choosing tissues and sample counts</tip-title>
>
> `SELECTED_TISSUES` and `SAMPLES_PER_TISSUE` are intentionally explicit. Start with a small balanced task so the tutorial runs quickly, then expand the tissue list or increase the sample count once the workflow is working on your Galaxy server.
>
{: .tip}

# Upload Data to Galaxy

> <hands-on-title>Upload prepared files</hands-on-title>
>
> 1. Create a new history. A useful name is `GTEx v11 Tissue Modeling`.
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Upload:
>
>    - `gtex_image_learner_input.csv`
>    - `gtex_tissue_images.zip`
>
>    {% snippet faqs/galaxy/datasets_upload.md %}
>
> 3. Confirm datatypes:
>
>    - `gtex_image_learner_input.csv`: `csv` or `tabular`
>    - `gtex_tissue_images.zip`: `zip`
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
{: .hands_on}

# Train the Tissue Classifier

> <hands-on-title>Run Image Learner</hands-on-title>
>
> 1. {% tool [Image Learner](toolshed.g2.bx.psu.edu/repos/goeckslab/image_learner/image_learner/0.1.5) %} with the following parameters:
>
>    - {% icon param-file %} *The metadata csv containing image_path column, label column*: `gtex_image_learner_input.csv`
>    - {% icon param-file %} *Image zip*: `gtex_tissue_images.zip`
>    - {% icon param-select %} *Task Type*: `Multi-class Classification`
>    - {% icon param-select %} *Overwrite label and/or image column names?*: `Yes`
>    - {% icon param-text %} *Target/label column name*: `c2: label`
>    - {% icon param-text %} *Image column name*: `c1: image_path`
>    - {% icon param-select %} *Select a model for your experiment*: `CAFormer S18 384`
>    - {% icon param-select %} *Customize Default Settings*: `Yes`
>    - {% icon param-text %} *Epochs*: `20`
>    - {% icon param-text %} *Early Stop*: `10`
>    - {% icon param-text %} *Train split*: `0.7`
>    - {% icon param-text %} *Validation split*: `0.1`
>    - {% icon param-text %} *Test split*: `0.2`
>    - {% icon param-text %} *Random seed*: `42`
>
> 2. Execute the tool.
>
{: .hands_on}

> <tip-title>Recommended configuration</tip-title>
>
> | Parameter | Value | Rationale |
> |---|---|---|
> | Task type | Multi-class classification | Each image belongs to one GTEx tissue label. |
> | Label column | `label` | Derived from `SMTSD` in GTEx sample attributes. |
> | Image column | `image_path` | Filename inside `gtex_tissue_images.zip`. |
> | Model | `CAFormer S18 384` | Strong general-purpose image backbone available in Image Learner. |
> | Split | `70/10/20` | Train, validation, and held-out test evaluation. |
> | Random seed | `42` | Reproducible split and training setup. |
>
{: .tip}

# Interpret the Outputs

Image Learner creates a trained model, an HTML report, and a predictions/statistics collection.

Focus on:

- **Dataset overview**: confirm that all selected tissues are represented.
- **Training and validation curves**: check whether training converged and whether validation performance diverges from training performance.
- **Test metrics**: use the held-out test split for the final performance estimate.
- **Confusion matrix**: identify tissues with similar expression patterns that are difficult to separate.
- **Per-class precision, recall, and F1**: inspect whether strong overall accuracy hides weak performance for specific tissues.

For a balanced subset, accuracy is easy to interpret. If you use naturally imbalanced GTEx tissue counts, prefer weighted and per-class metrics in addition to accuracy.

# Limitations

This workflow is a teaching example and a convenient Image Learner benchmark. It is not a claim that expression profiles are naturally images. Gene order in the matrix determines pixel adjacency, so spatial image patterns are an engineered representation rather than biological anatomy. Treat high performance as evidence that tissue-specific expression patterns are learnable from this representation, then validate with appropriate biological and statistical controls for research use.

# Conclusion

You prepared GTEx v11 gene TPM data and sample attributes, generated grayscale images from tissue expression profiles, and trained a Galaxy Image Learner model to classify GTEx tissues. The workflow is suitable for any Galaxy server with Image Learner.
