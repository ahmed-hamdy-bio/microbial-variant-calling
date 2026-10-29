# Microbial genome variant calling and bioinformatics file formats

A reproducible pipeline that follows bacterial sequencing data through every standard format:
**GenBank, FASTA, FASTQ, SAM/BAM, VCF and Newick.**

**Reference:** NCBI RefSeq NC_005816, *Yersinia pestis* plasmid pPCP1 (9,609 bp, 10 annotated genes, including the
virulence gene *pla*).

**Data:** four strains and their paired-end reads are **simulated** from the real reference with known mutations. This is
deliberate: knowing the true variants lets the pipeline be validated. Real sequencing data would add indels, repeats and
mixed populations, so the accuracy below describes an idealised case.

| Step | Format | Tool |
|---|---|---|
| Reference with gene annotation | GenBank, FASTA | Biopython |
| Sequencing reads | FASTQ | Python (simulated, 2 x 100 bp, about 60x) |
| Read alignment | SAM / BAM | bwa mem, samtools |
| Variant calls | VCF | bcftools |
| Strain relationships | Newick | Biopython (neighbour joining) |

## Pipeline
1. Parse the GenBank record, list genes, export FASTA
2. Simulate four strains on a known phylogeny and generate FASTQ reads
3. Align with `bwa mem`, sort and index BAM files, inspect SAM fields, compute coverage
4. Call haploid SNPs with `bcftools mpileup` and `bcftools call`, filter (QUAL >= 30, depth >= 10)
5. Validate calls against the simulated truth (precision and recall)
6. Annotate each SNP with its gene and codon change (synonymous, missense, nonsense)
7. Build a neighbour-joining tree from SNP distances and write it in Newick format

## Run it (easiest: Google Colab, no installation)
1. Open https://colab.research.google.com and choose **File > Upload notebook**, then select `analysis.ipynb`.
2. Choose **Runtime > Run all**. The notebook installs `samtools`, `bcftools` and `bwa` itself.
3. When it finishes, `pipeline_outputs.zip` (data, results, figures) is downloaded.

Local Linux, macOS (Homebrew) or Windows WSL:
```bash
sudo apt install samtools bcftools bwa
pip install -r requirements.txt
jupyter nbconvert --to html --execute analysis.ipynb
```

## Results
- **Reference:** 9,609 bp, 45.3% GC, 10 annotated genes.
- **Reads:** 2,882 read pairs per strain (2 x 100 bp). All reads mapped and were properly paired; mean depth 60x, with 99.2% to 99.6% of sites covered at 10x or more.
- **Variants:** 9, 10, 8 and 2 SNPs called in S1 to S4. Against the simulated truth, precision and recall were both 1.0 for all four strains (29 true positives, no false positives, no missed SNPs).
- **Annotation:** of the 29 SNP calls, 17 were missense, 5 synonymous and 7 intergenic; none were nonsense. SNPs fell in several genes, including *pim* (pesticin immunity), *pst* (pesticin) and *pla* (outer membrane protease).
- **Tree:** the neighbour-joining tree (`results/strains.nwk`) matches the simulated relationships: S1 and S2 share a 6-SNP branch, with private branches of 3 and 4 SNPs, while S3 (8 SNPs) and S4 (2 SNPs) branch separately from the reference. Estimated pairwise SNP distances equal the true distances.

## Limitations
- Simulated data: uniform coverage, substitution errors only, no indels, repeats or contamination. Perfect precision and recall describe this easy case, not real-world accuracy.
- A 9.6 kb plasmid is much smaller than a bacterial chromosome.
- The tree uses SNPs only and assumes no recombination.
- Filtering thresholds follow convention and were not tuned; no read trimming or QC step was needed for simulated reads.

## Files
`data/` (GenBank, FASTA, FASTQ, BAM, VCF), `results/` (annotated variants, validation table, Newick tree), `figures/`.
