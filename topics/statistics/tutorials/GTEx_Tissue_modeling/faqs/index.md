---
layout: faq-page
---

## What is the origin of GTEx?

GTEx collected postmortem biospecimens from many human tissues and measured gene expression with bulk RNA sequencing. In the GTEx expression matrix, each sample represents one tissue biospecimen from a donor, each row represents a gene, and each value is a normalized TPM expression measurement. The sample annotation file records the tissue source for each sample.

GTEx releases are versioned because the project updates data processing, sample sets, annotations, and reference gene models over time. Earlier releases, such as v8 and v10, are still useful for reproducible analyses, but this tutorial uses GTEx v11. When this tutorial was written, GTEx v11 was the latest Adult GTEx release available for download.

## Could converting tabular data to images lose information?

Yes. This representation preserves the expression values, but it imposes an artificial spatial layout based on gene order. It is useful for this Image Learner tutorial, but it should not be interpreted as a natural biological image.

## Can I use other images?

Yes. Image Learner can be used with other image datasets, as long as the ZIP archive and metadata table follow the required format.

## Can I change the tool parameters to test different results?

Yes. You can adjust parameters such as the model, epochs, batch size, learning rate, and data split to compare results.