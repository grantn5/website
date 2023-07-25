---
title: 'Bytesize 30: nf-core/pgdb'
subtitle: Yasset Perez-Riverol - EMBL-EBI, United Kingdom
type: talk
start_date: '2021-12-07'
start_time: '13:00 CET'
end_date: '2021-12-07'
end_time: '13:30 CET'
embed_at: 'pgdb'
youtube_embed: https://www.youtube.com/watch?v=vbyX3xmTT38
location_url:
  - https://youtu.be/vbyX3xmTT38
  - https://www.bilibili.com/video/BV1db4y1i7GM
  - https://doi.org/10.6084/m9.figshare.17136131.v1
---

# nf-core/bytesize

Join us for a special pipeline-focussed episode of our **weekly series** of short talks: **“nf-core/bytesize”**.

Just **15 minutes** + questions, we will be focussing on topics about using and developing nf-core pipelines.
These will be recorded and made available at <https://nf-co.re>
It is our hope that these talks / videos will build an archive of training material that can complement our documentation. Got an idea for a talk? Let us know on the [`#bytesize`](https://nfcore.slack.com/channels/bytesize) Slack channel!

## Bytesize 30: nf-core/pgdb

This week, Yasset Perez-Riverol ([@ypriverol](https://github.com/ypriverol/)) will tell us all about the nf-core/pgdb pipeline.

nf-core/pgdb is a bioinformatics pipeline to generate proteogenomics databases. pgdb allows users to create proteogenomics databases using EMSEMBL as the reference proteome database.

<details markdown="1"><summary>Video transcription</summary>
:::note
The content has been edited to make it reader-friendly
:::

[0:01](https://www.youtube.com/watch?v=vbyX3xmTT38&t=1)
(host) Hi, everyone. As usual, I'd like to begin by thanking you for joining us and the Chan Zuckerberg Initiative for funding. We're excited to be joined today by Yasset Perez-Riverol from the EBI in the UK, and he will be presenting the nf-core/pgdb pipeline, which is focused on helping users create proteogenomics databases using Ensembl as the reference proteome database. Yasset will be telling us more during the talk. If you have any questions for Yasset, either unmute yourself at the end of the talk or use the chat function, and I will relay the questions over to him. Thanks very much for agreeing to present for us today, Yasset. I'd like to hand over to you now so that you can start your talk.

[0:42](https://www.youtube.com/watch?v=vbyX3xmTT38&t=42)
Okay, thank you for the presentation. I hope this can be seen now. Today I will be talking about proteomics again, nf-core, and specifically one use case is proteogenomics analysis of non-canonical peptides. My name is Yasset Perez-Riverol. I am team coordinator of PRIDE, Proteomics Services at EMBL-EBI. I think this talk will be mainly a use case about how we have been using nf-core and Nextflow for more than a year right now to analyze our data. Rather than the technical talk about insights of nf-core, I guess you guys have more talks in the past talking about the DSL2 modules and so on, but this will be more about what you can do right now with nf-core pipelines to do proteogenomics analysis.

[1:54](https://www.youtube.com/watch?v=vbyX3xmTT38&t=114)
First, what is mass spec, especially bottom up proteomics? In summary, you start by extraction of your sample, and you try to separate the protein from that sample. After some preparation, especially digestion, you convert that into peptides. Then there is mainly two instruments that give you finally millions of mass spectra. Each spectra corresponds to a peptide. All those spectra are a relation between a mass and an intensity for each mass and these fragments of that particular peptide, correspond to one of each peak in the mass spectra. Until that you have the analytical part, the instrument part, but then is when you have bioinformatic tools that need to first take this result from the mass spectrometer, the spectra you need to assign to each of these spectrum peptide with some sort of scoring system there is multiple ones. Then when you have a lease of peptide, you need to infer which protein is the one you are interested in for biology. Finally, you need to do the quantitation of the proteins, how much of that particular protein is in the sample. This is a summary, what bottom up mass spectrometry is. Mainly we'll be talking about this part, what we have done in this part.

[3:59](https://www.youtube.com/watch?v=vbyX3xmTT38&t=239)
What is proteogenomics? Proteogenomics is using mass spectrometry to identify peptides and proteins, but using genomics and transcriptomic information. For example, to improve gene annotation, or to do prioritization of genes by using the protein expression information. In proteogenomics, if you have identified mutations or a variant, you want to see it by mass spectrometry, if that particular mutation can be seen or is expressed and how much of that protein is actually present in the cell, the protein expression of that particular protein for that particular mutation. That's what we want to do with proteogenomics. It's actually the bridge between genomics, transcriptomic and proteomics becoming really popular. There is a really nice review from Alex Seines-Biskey in nature methods about what is proteogenomics and what are the challenges in proteogenomics. Then to perform proteogenomics, the main task is that you need to... after, as I explained before, you have the mass spectrometry data, most of the peptide sequence identification is that you assign the mass spectra to a peptide, using a protein database to do this identification. When you work with a normal proteomics experiment or common proteomics experiment, you use Uniprot or Ensembl to do that. However, when you do proteogenomics, you need to add to these proteogenomics there, a proteogenomics database, you need to add the novel peptide, what we call non-canonical peptides in this case, which is basically a translation from the genomic information, lets say from a mutation, a variant, from a cell the gene, into a peptide information.

[6:07](https://www.youtube.com/watch?v=vbyX3xmTT38&t=367)
This can be summarizing this. I mean, you have the canonical peptides from Ensembl, this is the reference database from Ensembl, and then you need to start adding alternative or non-canonical peptides, alternative reading frames, the non-canonical RNA to that particular database. You can add the variant information from different sources like COSMIC and so on, or you can add patient specific data from the VCF data, you can add that and translate that into a proteomics database. This is actually what we have done this year. We have developed a tool using nf-core, and I will explain a little bit about how it's done, to generate the proteomics database.

[7:03](https://www.youtube.com/watch?v=vbyX3xmTT38&t=423)
As you can see, there is multiple steps if you want to build a proteomics database. Multiple steps are involved to finally have a proteomics database, including the last step, which is a decoy generating step where you take the target database or the sequence database, and then you attach to the database the decoy generation for the peptide identification steps to a computed FDR score for each peptide. First, before any development related to Nextflow or nf-core, we developed a tool in Python that does the simple steps, like for example, performing the translation from the genome transcript into the protein databases. For example, for pseudogenes, or non-coding RNA, and so on. Or for the most common mutations, databases like COSMIC or cBioPortal that translate that into protein databases. Also simple algorithms inside to generate the decoy sequence for large proteomics databases, or for removing stop calls and so on. This is really important for proteomics. I will not go into in detail, but mainly to process the proteomics database. Then that tool enables to do those independent steps.

[8:40](https://www.youtube.com/watch?v=vbyX3xmTT38&t=520)
However, if you want a proteogenomics database, and this is the element of the PGDB, you really need to combine all these steps, depending on which type of database the user wants to generate. You can see in this graph, we developed an nf-core pipeline called PGDB, which basically enabled the user to combine all the tasks that are provided by the Python tool to generate the database that the user wants. In this case, for example, starting from downloading the FASTA file from Ensembl for a specific proteome, then there is a way for the user to attach to the database, for example alternative open reading frames if they want, for example pseudo genes and so on. These are, for those familiar with Nextflow, all conditional steps. This is controlled by parameters in Nextflow, and the user can decide if they want to attach to the final database all these steps. There is another big block here where the user can attach also variants and mutations to the database. This is also controlled by... let me see if I can move this here... okay, this is also controlled by the pipeline, but actually one of the cool things is that you don't need to do anything manual. You don't need to download the data from COSMIC. The pipeline allows you to provide a user in COSMIC and also on FTP, and the pipeline itself allows you to download the original data and to translate into a protein sequence database.

[10:39](https://www.youtube.com/watch?v=vbyX3xmTT38&t=639)
Finally, what we call decoy generation database cleaning, where we remove known sequences that are not interesting and also generate the final decoy database. That's what PGDB does. We have tested the pipeline with multiple samples. For that study, we performed a re-analysis of public proteomics data, including six proteomics data sets, more than 65 cancer cell lines, around 20 million spectra, and we identified more than 5 million canonical peptides. These are the most common ones, and almost 300,000 non-canonical, 2,000 mutations, and 21,000 variants. You can see here the non-canonical mutation we have identified by each of the cell line.

[11:38](https://www.youtube.com/watch?v=vbyX3xmTT38&t=698)
Interestingly, the PGDB as a pipeline also allows you to generate, for example, tissue specific or cancer cell line specific databases. This is only possible because we have a pipeline like this where we can filter out for each specific cell line or cancer cell line, which are the mutations in COSMIC or CBAR portal that you want to use. In PRIDE, we are interested to map all this information to Ensemble coordinates, and we have an independent tool that does that after the re-analysis. You can see in Ensemble for one specific region of the genome what peptide has been seen and in which kind of cell line or sample condition.

[12:28](https://www.youtube.com/watch?v=vbyX3xmTT38&t=748)
Then until here is PGDB and how do we generate the databases, why PG is important for us, and what kind of task it enables for proteomics database generation. As you can see, you have multiple tasks, all of them combining, and Nextflow and nf-core allow us to generate customized databases. You can, for example, generate the reference proteome from Ensembl plus using only COSMIC variants, or you can attach a part of a COSMIC variant. You can also attach open reading frames, or you can add novel open reading frames, or attach the product of pseudogenes and so on. This is highly customizable. If you have a BCF from patient data, you can attach also the variants from the genomic side. That's what we do.

[13:30](https://www.youtube.com/watch?v=vbyX3xmTT38&t=810)
But how do we analyze the data? I think it's not the goal of this presentation to talk in detail about how you can do mass spec, but I want to basically highlight what a group of people in nf-core is trying to achieve also for data analysis. How you can go from mass spec and the protegenomics database to peptides identification and protein quantification. Quantms is an all-in-one DSL2 proteomics pipeline. Originally, we developed proteomics LFQ, released in nf-core, but it's DSL1. But working with DSL2, one of the features that we saw is that you can reuse a huge part of the pipeline where two types of analysis are possible. Then they reuse this part of the pipeline. The module base has allowed us to move into one, let's say, looks like a heavy pipeline, but most customizable pipeline called quantms that allowed us to do TMT labor-free and DIA-LFQ data analysis. The DIA part is still under development, but the TMT and LFQ is done, hasn't been released, but it's done. We are now benchmarking this with existing data.

[14:55](https://www.youtube.com/watch?v=vbyX3xmTT38&t=895)
The most prominent feature is DSL2 based. It allows us to do labor-free quantification data analysis and TMT data analysis. I think it's really relevant because this is not common in proteomics. It's based on a standard file format, sdrf, mzML, mzTab, and it's based on OpenMS and open source tools. This is also the first pipeline, I think, out there that is based on open source tools completely, which is actually not really common either in proteomics. For input and output of this pipeline, we use recently developed by this community, the main people that have been working in this pipeline also, an SDRF for proteomics, which is a tab-delimited sample metadata and experimental design. We actually know that in order to be able to reanalyze data with Nextflow, we really need some kind of input, tab-delimited, that organizes sample data and shows how the experimental design is done. By working on this, with the experience of Nextflow and nf-core, we develop our own representation also with experience with other fields arise press and so on. We developed a tab-delimited file format that when people submit their data to ProteomeXchange, they need to express this into a tab-delimited file format, that will enable us in the future to reanalyze the data.

[16:35](https://www.youtube.com/watch?v=vbyX3xmTT38&t=995)
The second input file is mzML, which contains the raw data, and we have a step for some instrument to convert to mzML if it's possible. The output is mzTab, an additional tab-delimited file format to help the downstream data analysis. This is completely new in proteomics. Nothing in proteomics has been done, at least in a workflow manner, based on standard file format from the input to the output. There is a common pipeline for peptide and protein identification for TMT and label-free. I will not go into details, but mainly it used two search engines, Comet and MSGF+. It used Percolator to boost the identification of both search engines, then they combined both search engines using the consensus ID tool, and then finally performe for localization analysis Luciphor.

[17:36](https://www.youtube.com/watch?v=vbyX3xmTT38&t=1056)
After that, and this is where both branches play, if the user is using an LFQ pipeline, a label-free pipeline, a tool called proteomicsLFQ will perform protein quantification, inference, feature detection, and match between runs. If the user is using a TMT, we would be using a IsobaricAnalyzer to perform quantification between TMT channels. Most important, all this data is exported into mcTAB, which is a standard file format for Proteomics. We have tested already that the QuantMS pipeline actually can export directly into ProteomeXchange, meaning that when you finalize your experiment, you can perform a complete submission into the main database in Proteomics, which is PRIME, but also in ProteomeXchange, because it's working with a standard file format. Then people can download the data, visualize it, and so on.

[18:40](https://www.youtube.com/watch?v=vbyX3xmTT38&t=1120)
One problem that we have in Proteomics is that we don't have something like MultiQC. We don't have something that enables us to visualize the QC reports of a Proteomics experiment. We have worked on a tool, it's called PMultiQC. I haven't talked with Phil about branding, but I hope it's fine. It's actually an extension of MultiQC, but it's mainly working for Proteomics. You can have really nice plots like this where you see the number of peaks per MS-MS, the number of peptide identification by each search engine, and the final result by each of the MS-ROC. You can see one example here. This is active development. We have a really nice feature that enables to search peptides identified in the web by using SAQL Lite in the backend hosted by HTML. This is quite nice and powerful tool. We are doing a lot of work there.

[19:48](https://www.youtube.com/watch?v=vbyX3xmTT38&t=1188)
As anyone knows in nf-core, you can see also all the reports of the pipeline. We have analyzed a lot of data set with NF-quantms, and you can see all the steps here, the CPU usage and also memory usage. This is more for people that are now arriving for the first time at nf-core and want to see what kind of reports you can see when you have your final reports. For those that know nf-core, they know that this is one of the great features of having the pipeline in your pipeline and in your export. The DIA part is under construction, but the TMT and neighbor-free are done.

[20:36](https://www.youtube.com/watch?v=vbyX3xmTT38&t=1236)
I want to thank three teams that have been actively working on this pipeline. First is the OpenMS team. Julianus and Timo, they have been working on every tool that works within the pipeline. Our team that actually has been working in both pipelines and from the Karolinska team, Husen has been working with me on the development of the PGDB and also the re-analysis of the cancer cell line data sets. Thanks for the opportunity and questions.

</details>