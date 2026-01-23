---
layout: tutorial_hands_on
title: Content Tracking and Verification in Galaxy Workflows with ISCC-SUM
level: Intermediate
zenodo_link: https://zenodo.org/records/XXXXXXX
questions:
- What is an ISCC code and why is it useful for data management?
- How can I generate content hashes for files in Galaxy?
- How can I verify file integrity in my workflows?
- How can I detect similar content across different files?
objectives:
- Understand the purpose and structure of ISCC codes
- Generate ISCC codes for files at different workflow stages
- Verify file integrity using ISCC codes
- Detect content similarity between files
time_estimation: 1H
key_points:
- ISCC codes provide content-based identifiers for any file type
- Content hashing helps track data provenance and verify file integrity
- Similarity detection can identify related or duplicate content
- ISCC codes can be integrated into any Galaxy workflow for quality control
contributions:
  authorship:
    - maartenpaul
---

# File content and integrity validation with International Standard Content Code (ISCC)

In scientific workflows, ensuring data integrity and tracking modifications to data content is crucial for reproducibility. Traditional checksums (like MD5 or SHA) can verify if files are identical, but they cannot detect similar content or survive format conversions.

The **International Standard Content Code (ISCC)** is a content-derived identifier that provides both:
- **Identity verification**: Checksum functionality to verify exact file matches
- **Similarity detection**: Ability to detect similar content even across different file formats

The **Galaxy ISCC-suite** allows you to integrate content tracking into any Galaxy workflow, providing quality control and provenance tracking for your data analysis pipelines.

## ISCC Code Structure

An ISCC-SUM code is a 55-character identifier with two main components, which are combined into one code:

- **Data-Code**: Content-based hash that allows similarity comparison
- **Instance-Code**: A fast checksum or cryptographic hash

For example the ISCC hash for this file [`example_image.tiff`](workflows/test-data/example_image.tiff):
```
ISCC:K4AOMGOGQJA4Y46PAC4YPPA63GKD5RVFPR7FU3I4OOEW44TYXNYOTMY
```
Which is composed of the Data and Instance hashes.
``` 
    ISCC:GAD6MGOGQJA4Y46PAC4YPPA63GKD5T7NDBHDOY5EY2CLX35RBVPF55I
    ISCC:IAD4NJL4PZNG2HDTRFXHE6F3ODU3HP3RQ75QJDH6RJSEGXFSUQVBUNI
```
Files with similar content will have similar Data-Code components, but their Instance-Code will be different. Hence the Instance-Code allows to verify file integrity.

> <agenda-title></agenda-title>
>
> In this tutorial, we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}


# Prepare your data

For this tutorial, we'll use a simple dataset with microscope images that demonstrate different use cases. However, the ISCC SUM tools can ben used for any type of digital content.

## Get the data

> <hands-on-title>Data Upload</hands-on-title>
>
> 1. Create a new history for this tutorial in Galaxy.
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Download the following image and import it into your Galaxy history.
>    - [`example_image.tiff`](workflows/test-data/example_image.tiff)
>    - [`example_image2.tiff`](workflows/test-data/example_image.tiff)
>    - [`example_thresholded1.tiff`](workflows/test-data/example_image.tiff)
>    
>    If you are importing the image via URL:
>
>    {% snippet faqs/galaxy/datasets_import_via_link.md %}
> 
{: .hands_on}

# Generate ISCC Codes

The first step is generating ISCC codes for your input files. This creates a content fingerprint that can be used throughout your workflow.

> <hands-on-title>Generate ISCC codes for input files</hands-on-title>
>
> 1. {% tool [Generate ISCC-CODE]( https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum) %} with the following parameters:
>    - {% icon param-file %} *"Input File"*: Select the first example image (`example_image.tiff`.)
>  
>    Run the tool. This will generate a 55-character ISCC code for the file.
>
> 2. Inspect the output by clicking on the dataset.
>    
>    You should see a single line containing the ISCC code in the output. For the first example image the code is expected to be:
> ```
> K4AGSPOSB5SS2X427WZ27QASTSBVTS55DXLMFDF7WOJKEOSTDEI3OXQ
> ```
> 3. Repeat for the other example image to generate another ISCC code for comparison.
>
{: .hands_on}

> <question-title></question-title>
>
> 1. Will the same file always generate the same ISCC code?
>
> > <solution-title></solution-title>
> >
> > 1. Yes! The same file will always generate the identical ISCC code, making it suitable for integrity verification.
> >
> {: .solution}
>
{: .question}

# Verify File Integrity

During workflow execution, you may want to verify that intermediate files match expected content. The **Verify ISCC hash** tool allows you to check if a file matches a known ISCC code.

## Manual Verification

> <hands-on-title>Verify a file against its ISCC code</hands-on-title>
>
> 1. {% tool [Verify ISCC-CODE](https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum_verify) %} with the following parameters:
>    - {% icon param-file %} *"Dataset to verify"*: Select the first example image
>    - *"Expected ISCC-CODE"*: 
>        - *"Expected ISCC code"*: Paste the ISCC code you generated in the previous step
>
> 2. Check the verification report in the output.
>    
>    The report shows:
>    - Status: OK (match) or FAILED (mismatch)
>    - Expected ISCC code
>    - Generated ISCC code

{: .hands_on}

## Workflow Integration

Integration of ISCC-CODES in 
A more powerful usecase is integrating verification directly in workflows:

> <hands-on-title>Automated verification in workflows</hands-on-title>
>
> 1. Create a simple workflow:
>
> {% snippet faqs/galaxy/workflows_create_new.md %}
>
>    - Create an **Input Dataset** tool
>    - Add **Generate ISCC hash** tool for your input file
>    - Add **Parse parameter value** tool to get the ISCC code that you generated.
         - Connect its input to the output of `Generate ISCC-CODE`
>    - Add **Verify ISCC hash** tool
>        - Connect the input file as input
>        - Connect the "Parse parameter value" from step 1 to "File containing expected ISCC code"
>
> When placing this in a full workflow this can help to validate that your processing didn't unexpectedly alter the content.
> ![fetch_data.jpg](../../images/iscc-suite/verify_wf1.png)
{: .hands_on}

## Image analysis workflow Integration

This can be applied in an image analysis workflow to verify an image processing tool provides the expected reproducible output.

> <hands-on-title>Image analysis verification in workflows</hands-on-title>
>
> 1. Create a workflow:
>    - Create an **Input Dataset** tool. Rename to 'Original image'.
>    - Connect the "**Treshold image** with scikit-image" tool
>      - At the Tool Parameters step make sure to select the Otsu threshold.
>    - Add a second input. Rename to 'Segmented image'.
>    - Add **Generate ISCC hash** tool to the second input file
>    - Add **Verify ISCC hash** tool
>        - Connect the processed file as input
>
> This creates an automated check that your processing didn't unexpectedly alter the content.
> ![fetch_data.jpg](../../images/iscc-suite/verify_wf.png)
{: .hands_on}

> <comment-title>When to Use Verification</comment-title>
>
> Verification is particularly useful:
> - After file transfers or storage operations
> - To confirm correct input files in complex workflows
> - As quality control checkpoints in processing pipelines
> - To detect unintended data modifications
{: .comment}

# Detect Similar Content

One of ISCC's unique features is detecting similar content, even across different formats. This is useful for finding duplicates, tracking content transformations, or identifying related files.

## Compare Two Files

> <hands-on-title>Compare two files for similarity</hands-on-title>
>
> 1. {% tool [Find datasets with similar ISCC-CODEs]( https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum_similarity) %} with the following parameters:
>    - *"Input type"*: `Datasets to compare`
>        - {% icon param-file %} Select multiple datasets (or a collection, see below)
>    - *"Similarity threshold (Hamming distance)"*: `12` (default)
>
> 2. The tool will create tabular output which indicates which datasets are similar.
>    The table will list all the files that has been set as input. For the files which have a similar file, below the set threshold, the similar file is listed.

{: .hands_on}

## Find Similar Files in Collections

When working with a collection of files, you can identify all similar items:

> <hands-on-title>Find similar files in a collection</hands-on-title>
>
> 1. Create a dataset collection with your test images
>
>    {% snippet faqs/galaxy/collections_build_list.md %}
>
> 2. {% tool [Find datasets with similar ISCC-CODEs]( https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum_similarity) %} with the following parameters:
>    - *"Input type"*: `Datasets to compare`
>        - {% icon param-file %} Select a collection
>    - *"Similarity threshold (Hamming distance)"*: `12` (default)
>
> 3. Review the similarity groupings. As explained above.
>    
>
{: .hands_on}

> <question-title></question-title>
>
> 1. Why would you want to detect similar files in your workflow?
>
> > <solution-title></solution-title>
> >
> > 1. Similar file detection helps to:
> >    - Identify duplicate data that wastes storage and compute
> >    - Track how files are transformed through processing
> >    - Find related samples or experimental replicates
> >    - Detect unexpected modifications in supposedly identical files
> 
> >
> {: .solution}
>
{: .question}


# Practical use cases:

## Use Case 1: Quality Control in High-Throughput Workflows

For processing thousands of images:

- Generate ISCC codes for all inputs at the start
- Add verification steps after critical transformations
- Flag unexpected changes automatically
- Maintain full provenance chain

## Use Case 2: Collaborative Data Sharing

When sharing data between institutions:

- Generate and publish ISCC codes alongside data
- Recipients verify data integrity upon receipt
- Track data lineage across organizations

# Conclusion

The Galaxy ISCC-suite provides powerful tools for content tracking and verification in scientific workflows:

- **Generate ISCC hash**: Create content identifiers for any file
- **Verify ISCC hash**: Check file integrity at workflow checkpoints
- **Compare ISCC similarity**: Detect related or duplicate content

By integrating these tools into your Galaxy workflows, you can:
- Improve data quality control
- Track content provenance
- Detect duplicates and similar content
- Enhance workflow reproducibility

The ISCC standard works with any file type, making these tools universally applicable across different research domains.

# References

- ISCC - International Standard Content Code: https://iscc.codes/
- ISCC-SUM Implementation: https://github.com/iscc/iscc-sum
- ISCC-SUM Quick Start: https://sum.iscc.codes/quickstart/