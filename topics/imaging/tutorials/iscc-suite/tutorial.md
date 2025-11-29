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

For example:
```
K4AOMGOGQJA4Y46PAC4YPPA63GKD5RVFPR7FU3I4OOEW44TYXNYOTMY
```

Files with similar content will have similar Data-Code components, while the Instance-Code verifies file integrity.


> <agenda-title></agenda-title>
>
> In this tutorial, we will deal with:
>
> 1. TOC
> {:toc}
>
{: .agenda}


# Prepare your data

For this tutorial, we'll use a simple dataset with images that demonstrate different use cases.

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
>

This would demonstrate both verification and similarity detection.

# Generate ISCC Codes

The first step is generating ISCC codes for your input files. This creates a content fingerprint that can be used throughout your workflow.

> <hands-on-title>Generate ISCC codes for input files</hands-on-title>
>
> 1. {% tool [Generate ISCC hash]( https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum) %} with the following parameters:
>    - {% icon param-file %} *"Input File"*: Select your first example image
>
>    This will generate a 55-character ISCC code for the file.
>
> 2. Inspect the output by clicking on the dataset.
>    
>    You should see a single line containing the ISCC code. For the first example image the code is expected to be:
> ```
> K4AGSPOSB5SS2X427WZ27QASTSBVTS55DXLMFDF7WOJKEOSTDEI3OXQ
> ```
> 3. Repeat for the other example image to generate codes for comparison.
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
> 1. {% tool [Verify ISCC hash](toolshed.g2.bx.psu.edu/repos/bgruening/iscc_sum_verify/iscc_sum_verify) %} with the following parameters:
>    - {% icon param-file %} *"File to verify"*: Select the first example image
>    - *"Source of expected ISCC code"*: `Enter manually`
>        - *"Expected ISCC code"*: Paste the ISCC code you generated in the previous step
>
> 2. Check the verification report in the output.
>    
>    The report shows:
>    - The filename
>    - Expected ISCC code
>    - Generated ISCC code
>    - Status: OK (match) or FAILED (mismatch)
>
{: .hands_on}

## Workflow Integration

A more powerful use case is connecting verification directly in workflows:

> <hands-on-title>Automated verification in workflows</hands-on-title>
>
> 1. Create a simple workflow:
>    {% snippet faqs/galaxy/workflows_create_new.md %}
>    - Create an **Input Dataset** tool
>    - Add **Generate ISCC hash** tool for your input file
>    - Add **Verify ISCC hash** tool
>        - Connect the processed file as input
>        - Connect the ISCC output from step 1 to "File containing expected ISCC code"
>
> This creates an automated check that your processing didn't unexpectedly alter the content.
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
> 1. {% tool [Compare ISCC hash similarity]( https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum_compare) %} with the following parameters:
>    - *"Input type"*: `Compare two files`
>        - {% icon param-file %} *"First file"*: Select your original image
>        - {% icon param-file %} *"Second file"*: Select a similar or converted version
>    - *"Similarity threshold (Hamming distance)"*: `12` (default)
>
> 2. Examine the output report.
>    
>    The report shows the Hamming distance between files. Lower numbers indicate higher similarity:
>    - 0-5: Nearly identical
>    - 6-12: Likely similar
>    - 13-20: Possibly related
>
{: .hands_on}

## Find Similar Files in Collections

When working with many files, you can identify all similar items:

> <hands-on-title>Find similar files in a collection</hands-on-title>
>
> 1. Create a dataset collection with your test images
>
>    {% snippet faqs/galaxy/collections_build_list.md %}
>
> 2. {% tool [Compare ISCC hash similarity]( https://toolshed.g2.bx.psu.edu/view/imgteam/iscc_sum_compare/iscc_sum_compare) %} with the following parameters:
>    - *"Input type"*: `Find similar files in collection`
>        - {% icon param-collection %} *"File collection to compare"*: Select your collection
>    - *"Similarity threshold"*: `12`
>
> 3. Review the similarity groupings.
>    
>    Files are grouped by similarity, with reference files listed first and similar files indented with their distance (~N).
>
{: .hands_on}

> <question-title></question-title>
>
> 1. Why would you want to detect similar files in your workflow?
> 2. What threshold would you use to find near-duplicates?
>
> > <solution-title></solution-title>
> >
> > 1. Similar file detection helps to:
> >    - Identify duplicate data that wastes storage and compute
> >    - Track how files are transformed through processing
> >    - Find related samples or experimental replicates
> >    - Detect unexpected modifications in supposedly identical files
> >
> > 2. For near-duplicates (files that should be essentially identical), use a threshold of 5 or lower. For broader similarity (e.g., different versions of the same content), 12-15 works well.
> >
> {: .solution}
>
{: .question}

# Practical Example: Integrating with Image Analysis

Let's see how ISCC tools can be benificial in a typical image analysis workflow with quality control checkpoints.

> <hands-on-title>ISCC-enhanced image processing workflow</hands-on-title>
>
> 1. Start with an input image and generate its ISCC code
>
> 2. Process the image
>
> 3. Add verification checkpoints:
>    - After format conversion: verify the Data-Code remains similar
>    - After filtering: document the new ISCC code for provenance
>    - Before final output: generate ISCC codes for all results
>
> 4. Store ISCC codes alongside your results for future reference
>
{: .hands_on}

Example workflow structure:

```
Input Image
    ↓
[Generate ISCC] → Store original code
    ↓
[Image Processing]
    ↓
[Generate ISCC] → Store processed code
    ↓
Output + Provenance Data
```

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