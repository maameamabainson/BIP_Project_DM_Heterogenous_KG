# BIP_Project_T2DM_Heterogenous_KG
The wider project studies the comorbidity between type 2 diabetes mellitus and infectious disease, with a specific narrative angle: diabetes → recurrent infection → increased antibiotic exposure → antimicrobial resistance (AMR). The goal is to build a small, interpretable, mechanism-aware knowledge graph prototype organised into three key layers.

## Preprocessing data from CARD for AMR Layer: 

Corresponding file: data_extraction_cleaning_AMR_layer.ipynb

Data download
1) Downloaded the CARD-R Resistomes, Variants, & Prevalence data folder
2) Further downloaded and extracted "card_prevalence.txt.gz"


Note:
- Original file in .txt format.
- Converted to .csv using online converter (https://mconverter.eu/convert/txt/csv/).
- Cleaned in Excel to place mechanisms and drug classes in the right columns


Description of original data from CARD Variants Download README file: 
"card_prevalence.txt.gz": tab-separated; indicates the prevalence of each detected AMR
determinant by assembly type (chromosome, plasmid, wgs) and RGI confidence criteria
(Perfect or Strict). Available and viewable on the web at https://card.mcmaster.ca/prevalence.


The headers included are:
- "ARO Accession": the unique ARO identifier from CARD for each ontology term
- "Name": the name for this accession as it appears in CARD
- "Model ID": the AMR detection model ID used to predict this determinant
- "Model Type": the AMR detection model type used to predict this determinant
- "Pathogen": the pathogen/species being described by prevalence statistics
- "NCBI Plasmid / NCBI Chromosome / NCBI WGS / NCBI Genomic Island": the prevalence (as %) of this determinant across all analysed assemblies for this data type
- "Criteria": the RGI criteria (perfect or strict) used to calculate the prevalence of this determinant
- "ARO Categories": semi-colon-separated list of ARO categories listed for this determinant (AMR Gene (ARG) Family, Resistance Mechanism, Drug Class)

Script input: card_prevalence.csv

To work with this dataset, the columns titled 'Model_ID', 'Model_Type', 'NCBI_Plasmid', 'NCBI_WGS', 'NCBI_Chromosome', 'NCBI_Genomic_Island', and 'Criteria' were dropped. 
The column 'Name' was renamed to ‘Resistance_Gene ’, and  'ARO Categories’ was split into ‘Resistance_Mechanism’ and ‘Drug_Class’.

Data from the CARD database was further restricted to those of the pathogens of interest. These are grouped into core and extended as outlined below.


Core: Klebsiella pneumoniae, Streptococcus pneumoniae, Escherichia coli, Acinetobacter baumannii, Mycobacterium tuberculosis
Extended: Enterococcus faecalis, Staphylococcus aureus, Pseudomonas aeruginosa, Burkholderia pseudomallei, Porphyromonas gingivalis


We provide unique IDs for pathogens, resistance mechanisms, and drug classes. We further create the relation and display relationship tags: 
- ARGs confers_resistance_to Drug class (relation: arg_drugclass)
- ARGs employs Resistance Mechanism (relation: arg_mechanism)
- ARGs associates_with Pathogen (relation: arg_pathogen)
- Drug_Class targets Pathogen (relation: drugclass_pathogen)


The final dataset from preprocessing has columns: ‘x_index’, ‘x_name’, ‘x_type’, ' y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’.

Script output: amr_layer_data.csv
