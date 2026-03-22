---
title: "Study Guide to Virtual Cell"
date: 2026-03-21
tags: ["virtual cell", "foundation models", "single-cell biology", "study guide"]
excerpt: "A structured reading guide for understanding the Virtual Cell vision — from foundation models trained on single-cell data to in silico biological simulation."
draft: false
---

The idea of a Virtual Cell has been gaining momentum: build foundation models trained on massive single-cell atlases so we can simulate cellular behavior in silico. It's ambitious, and the literature is growing fast. This post is my attempt to organize the key papers and concepts into a coherent study path.

## RegulatoryGen Papers

Before diving into foundation models, it helps to understand the regulatory genomics landscape these models are trying to capture. These papers establish core concepts around how genetic variation shapes gene expression and disease.

- [The role of regulatory variation in complex traits and disease](https://doi.org/10.1038/nrg3891) — Albert & Kruglyak (2015), *Nature Reviews Genetics*. Comprehensive review linking regulatory variants to phenotypic variation and disease risk.
- [Effects of cis and trans Genetic Ancestry on Gene Expression in African Americans](https://doi.org/10.1371/journal.pgen.1000294) — Price et al. (2008), *PLOS Genetics*. Demonstrates that ~12% of heritable variation in gene expression is due to cis variants, using admixture mapping in African Americans.
- [Impact of regulatory variation from RNA to protein](https://doi.org/10.1126/science.1260793) — Battle et al. (2015), *Science*. Shows that eQTL effects are attenuated at the protein level, revealing post-transcriptional buffering of genetic variation.
- [The GTEx Consortium atlas of genetic regulatory effects across human tissues](https://doi.org/10.1126/science.aaz1776) — GTEx Consortium (2020), *Science*. The definitive multi-tissue eQTL atlas from 49 tissues and 838 donors, characterizing tissue specificity of regulatory effects.
- [RNA splicing is a primary link between genetic variation and disease](https://doi.org/10.1126/science.aad9417) — Li et al. (2016), *Science*. Identifies splicing QTLs as major contributors to complex traits, on par with expression QTLs. Introduces the LeafCutter method.
- [Long-range enhancer–promoter contacts in gene expression control](https://doi.org/10.1038/s41576-019-0128-0) — Schoenfelder & Fraser (2019), *Nature Reviews Genetics*. Reviews how 3D genome architecture facilitates enhancer–promoter communication over large genomic distances.

*Under construction...*
