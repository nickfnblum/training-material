---
layout: tutorial_hands_on
level: Intermediate
title: GTEx Tissue Modeling with Galaxy Image Learner
questions:
  - "How can GTEx gene expression profiles be transformed into image-like inputs for Image Learner?"
  - "How do GTEx sample annotations provide tissue labels for supervised classification?"
  - "How can this workflow be run on any Galaxy server with Image Learner installed?"
objectives:
  - "Import a prepared GTEx v11 Image Learner metadata table and image ZIP archive."
  - "Optionally rebuild the prepared files from GTEx v11 gene TPM and sample annotation files."
  - "Train and evaluate a multi-class tissue classifier in Galaxy."
time_estimation: "1h"
key_points:
- "GTEx v11 gene TPM profiles can be represented as fixed-size grayscale images, one image per sample."
- "`SMTSD` from the GTEx sample attributes file is used as the tissue label."
- "This tutorial can be run on any Galaxy instance where Image Learner is available or installable."
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

Deep learning models are especially powerful for image classification because they can learn visual patterns directly from pixels. Instead of asking a researcher to define every important feature by hand, a deep learning model can discover combinations of edges, textures, shapes, and higher-level image structures that help separate one class from another. Galaxy's Image Learner tool makes this strategy accessible from a web interface: users provide images, labels, and training settings, and Galaxy runs the deep learning workflow.

In this tutorial, we use that image-learning strategy in an unusual but useful way. GTEx gene expression data are normally stored as tables, where each sample has thousands of gene-level TPM values. We transform each sample's expression vector into a grayscale image, then train Image Learner to classify the tissue of origin. This lets us explore two techniques at once: preparing image datasets for deep learning in Galaxy, and converting high-dimensional tabular biological data into an image-like representation that can be modeled with image classifiers.

The fastest path uses prepared files from Zenodo, so you can focus on running Image Learner and interpreting the results. An optional section shows how the images were generated from the raw GTEx v11 files, so you can rebuild the dataset, change the selected tissues, or test different sample sizes.

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# GTEx Dataset

> <comment-title>Background</comment-title>
>
> GTEx is a large reference resource for studying human gene expression across tissues {% cite GTEx2020 %}. The modeling idea used here treats each sample's vector of gene TPM values as a structured image: values are log-transformed, padded to a square, and saved as a grayscale image. A tissue classifier can then learn tissue-specific expression patterns using Galaxy Image Learner.
>
{: .comment}

The raw data used to create this tutorial dataset were downloaded from the [GTEx Portal](https://gtexportal.org/). The GTEx Portal provides multiple GTEx releases and many file types, including expression matrices, metadata, eQTL files, histology resources, and other analysis outputs. For this image-classification workflow, we need two specific pieces of information:

- A gene-level expression matrix, so each sample can be represented by a vector of expression values.
- Sample metadata, so each sample can be assigned a tissue label for supervised learning.

The GTEx v11 gene TPM file and v11 sample attributes file provide exactly those inputs. TPM values are normalized expression measurements that are appropriate for comparing gene expression profiles across samples in this tutorial context. The sample attributes file links each sample ID to tissue metadata, including the detailed tissue label (`SMTSD`) used as the class label.

| File | Direct URL | Use |
|---|---|---|
| `GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz` | [GTEx v11 gene TPM GCT](https://storage.googleapis.com/adult-gtex/bulk-gex/v11/rna-seq/GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz) | Gene TPM matrix |
| `GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt` | [GTEx v11 sample attributes](https://storage.googleapis.com/adult-gtex/annotations/v11/metadata-files/GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt) | Sample labels and QC metadata |

The raw GCT expression matrix has genes as rows and samples as columns. The sample attributes file includes:

| Column | Meaning |
|---|---|
| `SAMPID` | GTEx sample identifier. This must match expression matrix sample columns. |
| `SMTS` | Broad tissue category. |
| `SMTSD` | Detailed tissue label used as the classification target in this tutorial. |

The raw expression matrix is large: it contains many genes and many GTEx samples, so loading the full table into memory can be expensive. Each sample column contains the expression profile for one biospecimen. Each row corresponds to a gene, and the values summarize RNA-seq abundance as TPM. The annotation table contains technical and biological metadata for the samples, including identifiers, tissue categories, and quality-related fields.

This structure makes GTEx useful for many machine learning tasks. In this tutorial, the task is tissue classification: given the expression pattern of a sample, can a model identify the tissue it came from? Because different tissues express different sets of genes at different levels, GTEx provides a strong biological signal for this type of classification problem.

## GTEx dataset transformation

We will train the model with Image Learner, a Galaxy tool available on Galaxy servers such as [usegalaxy.org](https://usegalaxy.org/) when the tool is installed. Image Learner expects image-based training data in a specific format:

- A metadata table with one row per image.
- An `image_path` column that names the image file inside the ZIP archive.
- A `label` column that contains the class to predict.
- A ZIP archive containing all image files referenced by the metadata table.

To convert tabular gene expression data into images, we treat each sample's gene expression values as a long list of numbers. First, the values are log-transformed to reduce the effect of very large expression values. Then the vector is padded with zeros until it can fill a square. Finally, the square array is saved as a grayscale image, where darker and lighter pixels represent lower and higher transformed expression values. The image is not a photograph or a biological tissue picture; it is a structured representation of the sample's expression profile that an image model can process.

We have already made the generated Image Learner inputs available on Zenodo, and those prepared files are the recommended starting point for this tutorial. However, you can also generate the files inside Galaxy with the JupyterLab Interactive Tool. By changing parameters such as `SELECTED_TISSUES` and `SAMPLES_PER_TISSUE`, you can create different tissue-classification tasks and test how the model behaves.

> <hands-on-title>Optional: Generate GTEx expression images in Galaxy JupyterLab</hands-on-title>
>
> 1. Create or switch to the Galaxy history where you want the rebuilt files to appear.
>
> 2. In the Galaxy tool panel, open **Interactive Tools** and select **JupyterLab Notebook**.
>
> 3. Use the default JupyterLab environment. Click **Run Tool** to start the JupyterLab Interactive Tool.
>
>    The JupyterLab job will appear in your Galaxy history. Wait until it is running, then open it from the history dataset or from **User** > **Active Interactive Tools**.
>
> 4. In JupyterLab, open a terminal with **Others** > **Terminal**.
>
> 5. Check that the Python packages needed by the script are available:
>
>    ```bash
>    python -c "import numpy, pandas, matplotlib, psutil; print('Python environment is ready')"
>    ```
>
>    If this command reports a missing package, install the missing dependencies in the JupyterLab environment:
>
>    ```bash
>    python -m pip install --user numpy pandas matplotlib psutil
>    ```
>
> 6. Download the GTEx v11 expression matrix and sample annotation file directly from the GTEx links:
>
>    ```bash
>    curl -L -o GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz https://storage.googleapis.com/adult-gtex/bulk-gex/v11/rna-seq/GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz
>    curl -L -o GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt https://storage.googleapis.com/adult-gtex/annotations/v11/metadata-files/GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt
>    ```
>
> 7. Create the Python script in JupyterLab:
>
>    - Click **File** > **New** > **Python File**.
>    - Paste the script below into the editor.
>    - Save the file, then rename it in the JupyterLab file browser to `gtex_v11_to_images_adaptive.py`.
>    - Keep it in the same folder as the two downloaded GTEx files.
>
> 8. Before running the script, edit the variables near the top if you want to change the dataset:
>
>    - `SELECTED_TISSUES`: choose which GTEx tissue labels to model. The names must match `SMTSD` values in the GTEx annotation file.
>    - `SAMPLES_PER_TISSUE`: choose how many samples to use per tissue. Use a small value first if the Galaxy server has limited memory.
>    - `RANDOM_SEED`: keep this fixed for reproducible sampling, or change it to select a different random subset.
>    - `NORMALIZATION_METHOD`: use `log` for the recommended default, or test `minmax`, `zscore`, or `none`.
>    - `LUDWIG_OUTPUT` and `ZIP_OUTPUT`: change these only if you want different output filenames.
>
>    > <tip-title>Choosing tissues and sample counts</tip-title>
>    >
>    > `SELECTED_TISSUES` and `SAMPLES_PER_TISSUE` are intentionally explicit. Start with a small balanced task so the tutorial runs quickly, then expand the tissue list or increase the sample count once the workflow is working on your Galaxy server.
>    >
>    {: .tip}
>
>    ```python
>    # gtex_v11_to_images_adaptive.py
>    #
>    # Memory-adaptive GTEx V11 image pipeline for Galaxy JupyterLab.
>    #
>    # What this script does:
>    #   1. Checks available CPU and RAM.
>    #   2. Chooses safer/faster settings based on the computer.
>    #   3. Loads GTEx annotation metadata.
>    #   4. Selects samples from selected tissues.
>    #   5. Reads the large TPM matrix in chunks.
>    #   6. Creates one grayscale JPG image per sample.
>    #   7. Creates ludwig_input.csv.
>    #   8. Zips all images for Galaxy upload.
>
>    import math
>    import os
>    import zipfile
>    from pathlib import Path
>
>    import matplotlib
>    import numpy as np
>    import pandas as pd
>
>    matplotlib.use("Agg")
>    from matplotlib import pyplot as plt
>
>
>    TPM_FILE = "GTEx_Analysis_2025-08-22_v11_RNASeQCv2.4.3_gene_tpm.gct.gz"
>    ANNOTATION_FILE = "GTEx_Analysis_v11_Annotations_SampleAttributesDS.txt"
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
>    NORMALIZATION_METHOD = "log"
>    IMAGE_DIR = "output_images"
>    METADATA_OUTPUT = "metadata_base.csv"
>    LUDWIG_OUTPUT = "ludwig_input.csv"
>    ZIP_OUTPUT = "output_images.zip"
>
>
>    def get_available_ram_gb():
>        try:
>            import psutil
>
>            return psutil.virtual_memory().available / (1024 ** 3)
>        except ImportError:
>            print("psutil is not installed. Using conservative memory settings.")
>            return 4.0
>
>
>    def get_cpu_count():
>        return os.cpu_count() or 1
>
>
>    def choose_runtime_settings():
>        ram_gb = get_available_ram_gb()
>        cpu_count = get_cpu_count()
>
>        if ram_gb >= 32:
>            profile = "high-resource"
>            chunksize = 5000
>        elif ram_gb >= 16:
>            profile = "medium-resource"
>            chunksize = 2500
>        elif ram_gb >= 8:
>            profile = "low-resource"
>            chunksize = 1000
>        else:
>            profile = "very-low-resource"
>            chunksize = 250
>
>        print("Computer resource profile:")
>        print(f"  Available RAM: {ram_gb:.2f} GB")
>        print(f"  CPU cores: {cpu_count}")
>        print(f"  Selected profile: {profile}")
>        print(f"  TPM read chunksize: {chunksize}")
>
>        return {"ram_gb": ram_gb, "cpu_count": cpu_count, "profile": profile, "chunksize": chunksize}
>
>
>    def load_and_select_metadata():
>        print("Loading annotation file...")
>        annot = pd.read_csv(ANNOTATION_FILE, sep="\t")
>
>        required = {"SAMPID", "SMTSD"}
>        missing = required - set(annot.columns)
>        if missing:
>            raise ValueError(f"Missing required annotation columns: {missing}")
>
>        annot = annot[annot["SMTSD"].isin(SELECTED_TISSUES)].copy()
>        annot = annot.drop_duplicates(subset=["SAMPID"])
>
>        selected = []
>        for tissue, group in annot.groupby("SMTSD"):
>            n = min(SAMPLES_PER_TISSUE, len(group))
>            selected.append(group.sample(n=n, random_state=RANDOM_SEED))
>
>        if not selected:
>            raise ValueError("No samples found for SELECTED_TISSUES.")
>
>        meta = pd.concat(selected).reset_index(drop=True)
>        labels = meta[["SAMPID", "SMTSD"]].rename(columns={"SAMPID": "sample_id", "SMTSD": "label"})
>        labels.to_csv(METADATA_OUTPUT, index=False)
>
>        print("Selected samples per tissue:")
>        print(labels["label"].value_counts().sort_index())
>        return labels
>
>
>    def get_available_selected_samples(selected_sample_ids):
>        print("Reading GCT header...")
>        header = pd.read_csv(TPM_FILE, sep="\t", skiprows=2, nrows=0, compression="gzip")
>
>        available_samples = [sample_id for sample_id in selected_sample_ids if sample_id in header.columns]
>        missing = sorted(set(selected_sample_ids) - set(available_samples))
>        if missing:
>            print(f"Warning: {len(missing)} selected samples were not found in TPM matrix.")
>
>        if not available_samples:
>            raise ValueError("None of the selected samples were found in the TPM matrix.")
>
>        print(f"Samples found in TPM matrix: {len(available_samples)}")
>        return available_samples
>
>
>    def read_selected_expression(sample_ids, chunksize):
>        print("Reading selected TPM expression values in chunks...")
>        sample_vectors = {sample_id: [] for sample_id in sample_ids}
>        total_genes = 0
>        usecols = ["Name"] + sample_ids
>
>        reader = pd.read_csv(
>            TPM_FILE,
>            sep="\t",
>            skiprows=2,
>            compression="gzip",
>            usecols=usecols,
>            chunksize=chunksize,
>        )
>
>        for chunk_index, chunk in enumerate(reader, start=1):
>            chunk_values = chunk[sample_ids].astype(np.float32)
>            for sample_id in sample_ids:
>                sample_vectors[sample_id].extend(chunk_values[sample_id].to_numpy())
>            total_genes += len(chunk)
>            if chunk_index % 10 == 0:
>                print(f"Processed {total_genes} genes...")
>
>        print(f"Finished reading {total_genes} genes.")
>        return sample_vectors, total_genes
>
>
>    def normalize_values(values, method):
>        values = np.asarray(values, dtype=np.float32)
>        if method == "log":
>            return np.log1p(values)
>        if method == "minmax":
>            return (values - np.min(values)) / (np.max(values) - np.min(values) + 1e-8)
>        if method == "zscore":
>            return (values - np.mean(values)) / (np.std(values) + 1e-8)
>        if method == "none":
>            return values
>        raise ValueError(f"Unknown normalization method: {method}")
>
>
>    def vector_to_image(values, total_genes):
>        values = normalize_values(values, NORMALIZATION_METHOD)
>        image_size = math.ceil(math.sqrt(total_genes))
>
>        padded = np.zeros(image_size * image_size, dtype=np.float32)
>        padded[:total_genes] = values
>        return padded.reshape((image_size, image_size))
>
>
>    def save_images(sample_vectors, total_genes):
>        Path(IMAGE_DIR).mkdir(parents=True, exist_ok=True)
>        total_samples = len(sample_vectors)
>
>        for i, (sample_id, values) in enumerate(sample_vectors.items(), start=1):
>            image = vector_to_image(values, total_genes)
>            output_path = Path(IMAGE_DIR) / f"{sample_id}.jpg"
>            plt.imsave(output_path, image, cmap="gray", format="jpg")
>            plt.close("all")
>
>            if i % 50 == 0 or i == total_samples:
>                print(f"Saved {i}/{total_samples} images.")
>
>
>    def create_ludwig_csv(labels, available_samples):
>        labels = labels[labels["sample_id"].isin(available_samples)].copy()
>        labels["image_path"] = labels["sample_id"] + ".jpg"
>        labels[["image_path", "label"]].to_csv(LUDWIG_OUTPUT, index=False)
>        print(f"Created {LUDWIG_OUTPUT}")
>
>
>    def zip_images():
>        jpg_files = sorted(filename for filename in os.listdir(IMAGE_DIR) if filename.endswith(".jpg"))
>        if not jpg_files:
>            raise ValueError(f"No JPG images found in {IMAGE_DIR}")
>
>        with zipfile.ZipFile(ZIP_OUTPUT, "w", compression=zipfile.ZIP_DEFLATED) as handle:
>            for filename in jpg_files:
>                handle.write(Path(IMAGE_DIR) / filename, arcname=filename)
>
>        print(f"Created {ZIP_OUTPUT}")
>
>
>    def main():
>        settings = choose_runtime_settings()
>        labels = load_and_select_metadata()
>        selected_sample_ids = labels["sample_id"].tolist()
>        available_samples = get_available_selected_samples(selected_sample_ids)
>        labels = labels[labels["sample_id"].isin(available_samples)].copy()
>
>        sample_vectors, total_genes = read_selected_expression(
>            sample_ids=available_samples,
>            chunksize=settings["chunksize"],
>        )
>
>        save_images(sample_vectors=sample_vectors, total_genes=total_genes)
>        create_ludwig_csv(labels=labels, available_samples=available_samples)
>        zip_images()
>
>        print("Done.")
>        print(f"Images folder: {IMAGE_DIR}")
>        print(f"Ludwig CSV: {LUDWIG_OUTPUT}")
>        print(f"Images ZIP: {ZIP_OUTPUT}")
>
>
>    if __name__ == "__main__":
>        main()
>    ```
>
> 9. Run the script in the JupyterLab terminal.
>
>    ```bash
>    python gtex_v11_to_images_adaptive.py
>    ```
>
>    The script prints the detected memory profile, selected samples per tissue, progress while reading the TPM matrix, and progress while saving images.
>
> 10. Confirm that the expected output files were created:
>
>    ```bash
>    ls -lh metadata_base.csv ludwig_input.csv output_images.zip
>    ```
>
>    The script creates:
>
>    - `metadata_base.csv`: selected sample IDs and labels
>    - `ludwig_input.csv`: Image Learner metadata table with `image_path` and `label`
>    - `output_images.zip`: grayscale expression image ZIP archive
>
> 11. Push the two Image Learner inputs back to your Galaxy history.
>
>    Open the Python notebook in JupyterLab by double-clicking the file named `ipython_galaxy_notebook.ipynb`, or open a new one with **File** > **New** > **Notebook**. Choose the Python kernel, then run the following commands.
>    The `put()` function is available in Galaxy JupyterLab and exports files from the JupyterLab workspace into the Galaxy history. Copy and paste the following commands into a Jupyter notebook cell:
>
>    ```python
>    put("ludwig_input.csv")
>    put("output_images.zip")
>    ```
>
>    Look for the play icon at the top of the window and click it to execute the commands.
>    The files will appear as new datasets in the Galaxy history that launched JupyterLab. Wait for both datasets to turn green before using them in Image Learner.
>
> 12. Return to Galaxy and use `ludwig_input.csv` as the metadata table and `output_images.zip` as the image ZIP.
>
{: .hands_on}

# Workflow

## Overview

The hands-on workflow uses the following steps:

1. Import the previously prepared metadata CSV and image ZIP from Zenodo.
2. Configure Image Learner with `label` as the target column and `image_path` as the image column.
3. Configure the hyperparameters.
4. Run the tool to train Image Learner to predict tissue label from the generated expression image.
5. Review the model report, test metrics, and confusion matrix.

The tutorial is written for any Galaxy instance that has Image Learner installed, or where an administrator can install Image Learner from the ToolShed.

## Environment and Data Upload

> <tip-title>Image Learner tool requirement</tip-title>
>
> Image Learner expects, by default, a metadata table and an image ZIP archive. The metadata table must include one row per image and at least two columns:
>
> | Column | Description |
> |---|---|
> | `image_path` | Image filename inside the ZIP archive. |
> | `label` | Tissue label, derived from `SMTSD`. |
>
> The prepared Zenodo metadata CSV already contains these columns.
>
{: .tip}

> <hands-on-title> Environment and Data Upload </hands-on-title>
>
> 1. Create a new history for this tutorial. If you are not inspired, you can name it *GTEx v11 Tissue Modeling*.
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Import the prepared files from Zenodo or from the shared data library:
>
>    ```
>    https://zenodo.org/records/19963477/files/selected_gtex_v11_tpm_image_tissue_labels.csv
>    https://zenodo.org/records/19963477/files/selected_gtex_v11_tpm_image_tissue_dataset.zip
>    ```
>
>    > <tip-title>Data Type</tip-title>
>    > Leave the `Type` field as `Auto-Detect` when uploading the metadata CSV. For the image archive, set or confirm the datatype as `zip`.
>    >
>    {: .tip}
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
> 3. Check that the data formats are assigned correctly:
>
>    - `selected_gtex_v11_tpm_image_tissue_labels.csv`: `csv` or `tabular`
>    - `selected_gtex_v11_tpm_image_tissue_dataset.zip`: `zip`
>
>    If they are not, follow the Changing the datatype tip.
>
>    {% snippet faqs/galaxy/datasets_change_datatype.md datatype="datatypes" %}
>
> 4. Add a tag (`GTEx v11 tissue model dataset`) to both datasets. This is important to trace back which dataset the model was built on.
>
>    {% snippet faqs/galaxy/datasets_add_tag.md %}
>
{: .hands_on}

> <tip-title>Using locally rebuilt files instead</tip-title>
>
> If you ran the optional script, upload `ludwig_input.csv` and `output_images.zip` instead of the Zenodo files. The Image Learner settings are the same: label column `c2: label` and image column `c1: image_path`.
>
{: .tip}

## Train the Tissue Classifier

> <hands-on-title>Run Image Learner</hands-on-title>
>
> 1. {% tool [Image Learner](toolshed.g2.bx.psu.edu/repos/goeckslab/image_learner/image_learner/0.1.5) %} with the following parameters:
>
>    - {% icon param-file %} *The metadata csv containing image_path column, label column*: `selected_gtex_v11_tpm_image_tissue_labels.csv`
>    - {% icon param-file %} *Image zip*: `selected_gtex_v11_tpm_image_tissue_dataset.zip`
>    - {% icon param-select %} *Task Type*: `Multi-class Classification`
>    - {% icon param-select %} *Select a model for your experiment*: `CAFormer S18 384`
>    - {% icon param-select %} *Customize Default Settings*: `Yes`
>    - {% icon param-text %} *Epochs*: `30`
>    - {% icon param-text %} *Early Stop*: `30`
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
> | Image column | `image_path` | Filename inside the image ZIP archive. |
> | Model | `CAFormer S18 384` | Strong general-purpose image backbone available in Image Learner. |
> | Epochs | `30` | Maximum number of training passes over the dataset. More epochs can improve learning, but too many can overfit. |
> | Early Stop | `30` | Stop training if the validation metric does not improve for this many epochs. This helps avoid wasting time and can reduce overfitting. |
>
{: .tip}

## Interpret the Outputs

After training the model, the Image Learner generates a detailed report summarizing its performance across the training, validation, and test datasets.  

To access this report, click on the **eye icon** next to the dataset in your Galaxy history (as shown in the figure below).
![Open image learner report](figures/open_report.png)

### Overall Performance Overview

Before diving into each tab, let’s first interpret the key performance metrics:

| Metric | Train | Validation | Test |
|--------|------|------------|------|
| Accuracy | 1.0000 | 1.0000 | 0.9890 |
| Micro Accuracy | 1.0000 | 1.0000 | 0.9915 |
| Hits at K | 1.0000 | 1.0000 | 1.0000 |
| Loss | 0.0000 | 0.0000 | 0.0473 |
| ROC-AUC | 1.0000 | 1.0000 | 1.0000 |

These results indicate strong model performance:

- Perfect training and validation performance suggests the model has captured patterns in the dataset very effectively.
- Test performance remains very high, with only a slight drop in accuracy and a small increase in loss, indicating good generalization.
- ROC-AUC of 1.0 across all splits shows excellent class separability.
- Low test loss (0.0473) suggests only minor prediction uncertainty on unseen data.

However, such near-perfect results should be interpreted carefully, as they may indicate:
- A very clean and well-separated dataset, or  
- Potential overfitting if the dataset is small or not diverse enough  

To better understand these results and assess model reliability, we now explore each section of the report in detail. Each tab provides deeper insights into model behavior and helps determine whether adjustments are needed.

---

### Config and Overall Performance Summary

#### 1. Dataset Overview

![Dataset Overview](figures/dataset_overview.png)

This table shows how samples are distributed across:
- Training set  
- Validation set  
- Test set  

When interpreting this table:
- A balanced dataset (similar number of samples per class) helps the model learn fairly across categories.
- An imbalanced dataset may bias the model toward dominant classes.

---

#### 2. Training Configuration

![Training configuration](figures/training_config.png)

This section summarizes how the model was trained, for example:

- Validation Metric  
  The metric used to monitor performance during training and select the best model.

- Loss Function  
  Measures how far predictions are from true labels.

- Optimizer  
  Controls how model weights are updated (e.g., Adam, SGD).

---

### Training and Validation Results

#### 1. Accuracy Across Epochs

![Accuracy across epochs](figures/accuracy_epochs.png)

This curve shows how accuracy evolves over training epochs:

- Training accuracy: performance on training data  
- Validation accuracy: generalization performance  

Interpretation:
- Both curves increasing → good learning  
- Training high but validation drops → overfitting  
- Both low → underfitting  

---

#### 2. ROC-AUC Across Epochs

![ROC-AUC across epochs](figures/rocauc_epochs.png)

ROC-AUC measures class separability:

- Close to 1.0 → excellent performance  
- Close to 0.5 → random guessing  

A stable high curve indicates strong classification ability.

---

#### 3. Overfitting Gap

![Overfitting gap: ROC-AUC across epochs](figures/overffiting_gap.png)

This curve shows the difference between training and validation:

- Small gap → good generalization  
- Large gap → overfitting  

---

### Test Results

![Test Performance Summary](figures/test_metrics.png)

#### 1a. Macro, Micro, and Weighted Metrics

- Macro: treats all classes equally  
- Micro: aggregates all predictions globally  
- Weighted: accounts for class frequency  

#### 1b. Key Metrics Explained

- Accuracy: overall correctness  
- Precision: correctness of positive predictions  
- Recall: ability to find positives  
- F1-score: balance between precision and recall  
- ROC-AUC: class separability  

---

#### 2. Grad-CAM Heatmaps

![Grad-CAM heatmaps](figures/gradcam.png)

Grad-CAM highlights which regions influenced predictions, providing insight into whether the model is focusing on meaningful patterns rather than noise or artifacts.

- Bright areas → important regions  
- Random patterns → possible uncertainty  

These are essential for interpreting and validating model behavior.

---

## Summary

Overall, the model demonstrates **excellent performance**, achieving near-perfect results on both training and validation datasets and maintaining **very high accuracy and ROC-AUC on the test set**. This suggests that the model has learned strong and meaningful patterns from the data and is able to generalize well to unseen samples.

However, the near-perfect scores—especially on training and validation—should be interpreted with caution. They may indicate that:
- The dataset is highly clean and well-separated, or  
- The model may be slightly overfitting, particularly if the dataset is small or lacks diversity  

By examining the detailed metrics, training curves, and Grad-CAM visualizations, we can confirm whether the model is:
- Learning relevant features  
- Generalizing appropriately  
- Reliable for downstream analysis  

In this case, the results suggest a **high-performing model with strong predictive capability**, but further validation on additional or more diverse data may still be beneficial to ensure robustness.

# Limitations

This workflow is designed as a teaching example and a convenient Image Learner benchmark. It does not imply that gene expression profiles are inherently image-like data.

The transformation from expression vectors to images depends on gene ordering, meaning spatial relationships in the images are engineered rather than biologically meaningful. As a result, model performance reflects the ability to learn patterns in this representation, not necessarily true spatial biological structure.

Additionally, the near-perfect performance observed in this tutorial may be influenced by dataset characteristics such as class separability or limited variability. For research applications, results should be validated using additional datasets, alternative representations, and appropriate statistical controls.

# Conclusion

In this tutorial, you used prepared GTEx v11 expression-derived images and metadata to train a Galaxy Image Learner model for tissue classification. The model achieved strong performance across training, validation, and test datasets, demonstrating that tissue-specific expression patterns can be effectively learned using this image-based representation.

At the same time, the near-perfect evaluation metrics highlight the importance of careful interpretation and validation. While the results suggest strong predictive capability, additional testing on more diverse datasets would help confirm robustness and real-world applicability.

Overall, this workflow demonstrates how high-dimensional biological data can be transformed and modeled using deep learning tools in Galaxy, providing both a practical pipeline and a foundation for further experimentation.
