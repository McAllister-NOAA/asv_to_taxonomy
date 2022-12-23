# ASV to Taxonomy

Written by Sean McAllister, v.1 12/2022

## Purpose

This stand-alone script is meant to take a formatted btab blastn file and parse the results to assign taxonomy for each ASV. The basic standpoint is that from a list of taxIDs from the best blast hits for each ASV and a confidence value (based on percent identity), we should be able to assign the lowest/deepest appropriate taxonomy shared by all best blast hits.

## Options
```
-a = ASV counts table (make sure there is text in the upper left)
-s = Formatted blastn results (w/ headers ASV Perc Len TaxID correction)
-t = reformatted taxonkit output (TaxID simplified taxonomy)
-f = filtering options Species,Genus,Family,Order,Class,Phylum (e.g. 97,95,90,80,70,60)
-n = Allin Output basename
-c = Location of common names file (grep "genbank common name" from names.dmp NCBI taxonomy file)
-d = List of ASVs to ignore (one per line) for outputs ignoring contaminants and/or unknowns
-o = List of samples (one per line) in the order you want them exported. Must be exact matches to ASV counts table. Does not have to include all samples.
-h = This help message
```

##### ASV counts table (-a, tab delimited)
```
x	sample1	sample2	sample3
ASV_1	914	365	3519
ASV_2	4897	33097	13934
ASV_3	0	2124	0
```
##### Formatted blastn results (-s, tab delimited)
Create from ```-outfmt '6 qseqid pident length staxids'```

```
ASV	percent	length	taxid	correction
ASV_1	100	313	102976	313
ASV_2	82.372	313	207955	257.824
ASV_3	100	313	67591, 67591;134455, 360400	313
```

Note: Taxonomic assignments like ```67591;134455``` above indicate two taxonomic IDs assigned to the same sequence. These assignments are ignored unless they are the only assignment for the best blast hits.

##### Reformatted taxonkit output (-t)
Create list of taxid from each formatted blastn result file and run through taxonkit

```
taxonkit lineage taxids.txt | awk '$2!=""' > taxonkit_out.txt
taxonkit reformat taxonkit_out.txt | cut -f1,3 > reformatted_taxonkit_out.txt
```
Produces list of taxIDs (column one) and their taxonomy assignment (K;P;C;O;F;G;S, second column)

```
102976	Eukaryota;Arthropoda;Malacostraca;Euphausiacea;Euphausiidae;Euphausia;Euphausia pacifica
207955	Eukaryota;Arthropoda;Hexanauplia;Calanoida;Centropagidae;Centropages;Centropages abdominalis
67591	Eukaryota;Oomycota;Peronosporales__c;Peronosporales;Peronosporaceae;Phytophthora;Phytophthora macrochlamydospora
134455	Eukaryota;Oomycota;Peronosporales__c;Peronosporales;Peronosporaceae;Phytophthora;Phytophthora quininea
360400	Eukaryota;Oomycota;Peronosporales__c;Peronosporales;Peronosporaceae;Phytophthora;Phytophthora captiosa
```

Note: Gaps in the NCBI taxonomy at each level are filled from the lower taxonomic assignment. I.E. ```Peronosporales__c``` in the above example means that it is the class containing the order Peronosporales. These tags help to keep track of individual taxa so that they are not relegated to "Unknown" or "NA" on filtering.

##### Filtering options (-f)
Use this comma delimited string to set the boundaries for assignment confidence based on percent identity. For ```97,95,90,80,70,60```: species assignment for ```x≥97%```, genus assignment for ```97%>x≥95%```, family assignment for ```95%>x≥90%```, order assignment for ```90%>x≥80%```, class assignment for ```80%>x≥70%```, phylum assignment for ```70%>x≥60%```, and anything lower than ```60%``` is assigned to "Unknown."

##### Other options
```-n``` is the outname of your choice

```-c``` is created from the names.dmp NCBI taxonomy file. ```grep "genbank common name" names.dmp > commonNames.txt```
 
```-d``` (optional) List of ASVs to ignore (one per line)
```-o``` (optional) List of samples in the appropriate order for outfiles (one per line, must match the ASV counts file.

### Example assignment
From the examples given, you would get the following assignments:

```
ASV_1	Eukaryota;Arthropoda;Malacostraca;Euphausiacea;Euphausiidae;Euphausia;Euphausia pacifica
ASV_2	Eukaryota;Arthropoda;Hexanauplia
ASV_3	Eukaryota;Oomycota;Peronosporales__c;Peronosporales;Peronosporaceae;Phytophthora
```

## Dependencies

To create files prior to running will need:

1. ```blastn``` (https://blast.ncbi.nlm.nih.gov/Blast.cgi?PAGE_TYPE=BlastDocs&DOC_TYPE=Download)
2. ```taxonkit``` (https://bioinf.shenwei.me/taxonkit/)

The script itself requires:

1. ```perl``` ```List::MoreUtils```
2. ```taxonkit``` (https://bioinf.shenwei.me/taxonkit/)
3. ```ImportText.pl``` KronaTools (https://hpc.nih.gov/apps/kronatools.html)


## Outputs
This program produces a lot of useful output files.

1. ```outname_asvTaxonomyTable.txt``` Main taxonomy table with assignments per ASV. (+IGNORE file when given)
2. ```outname_barchart.txt``` Terminal taxonomy assignments with counts for easy barchart manipulation in Excel. (+IGNORE +NoUnknowns).
3. ```outname_barchart_forR.txt``` Terminal taxonomy assignments with counts for easy barchart creation in R. (+IGNORE)
4. ```outname_master_krona.html``` Krona Plot showing hierarchical taxonomy browser per sample. (+IGNORE)
5. ```outname_wholeKRONA.html``` Krona Plot showing hierarchical taxonomy browser will all sample counts summed.
6. ```outname_singleBlastHits_with_MULTItaxid.txt``` Prints hits with two taxIDs per sequence (e.g. ```67591;134455```) and whether they were chosen for downstream output.
7. ```outname_unique_terminaltaxa.txt``` List all terminal taxa.
6. ```outname_taxid_to_commonname_ALL.txt``` List of common names associated with taxIDs.
12. ```outname_heatmap_multiASV.txt``` List of ASVs, and their count tables, for each taxa at each hierarchy. Best used for the lower/deeper taxa to highlight ASVs with the same taxonomic assignment where one might be real (have large counts) and one ASV might have a different length/contain slight erroneous bases. Could help with filtering ASVs. (+IGNORE)
13. ```outname_unknown_asvids.txt``` A list of all the ASVs assigned to "Unknown" and the reason why (```NO TAXID ASSIGNMENT```, ```TaxaStringDeleted_or_DoesNotExist```, ```NOCONFIDENCE```)



**Legal Disclaimer**

*This repository is a software product and is not official communication of the National Oceanic and Atmospheric Administration (NOAA), or the United States Department of Commerce (DOC). All NOAA GitHub project code is provided on an 'as is' basis and the user assumes responsibility for its use. Any claims against the DOC or DOC bureaus stemming from the use of this GitHub project will be governed by all applicable Federal law. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation, or favoring by the DOC. The DOC seal and logo, or the seal and logo of a DOC bureau, shall not be used in any manner to imply endorsement of any commercial product or activity by the DOC or the United States Government.*
