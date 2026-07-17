# BIP_Project_T2DM_Heterogenous_KG
The wider project studies the comorbidity between type 2 diabetes mellitus and infectious disease, with a specific narrative angle: diabetes → recurrent infection → increased antibiotic exposure → antimicrobial resistance (AMR). The goal is to build a small, interpretable, mechanism-aware knowledge graph prototype organised into three key layers.

## Preprocessing data from CARD for AMR layer: 

Corresponding file: data_extraction_cleaning_AMR_layer.ipynb

Data download
1) Downloaded the CARD-R Resistomes, Variants, & Prevalence data folder
2) Further downloaded and extracted "card_prevalence.txt.gz"


Script input: card_prevalence.csv


To work with this dataset, the columns titled 'Model_ID', 'Model_Type', 'NCBI_Plasmid', 'NCBI_WGS', 'NCBI_Chromosome', 'NCBI_Genomic_Island', and 'Criteria' were dropped. 
The column 'Name' was renamed to ‘Resistance_Gene ’, and  'ARO Categories’ was split into ‘Resistance_Mechanism’ and ‘Drug_Class’.

Data from the CARD database was further restricted to those of the pathogens of interest. These are grouped into core and extended as outlined below.


Core: Klebsiella pneumoniae, Streptococcus pneumoniae, Escherichia coli, Acinetobacter baumannii, Mycobacterium tuberculosis
Extended: Enterococcus faecalis, Staphylococcus aureus, Pseudomonas aeruginosa, Burkholderia pseudomallei, Porphyromonas gingivalis


We provide unique IDs for pathogens, resistance mechanisms, and drug classes. We further create the relation and display relationship tags: 
ARGs confers_resistance_to Drug class (relation: arg_drugclass)
ARGs employs Resistance Mechanism (relation: arg_mechanism)
ARGs associates_with Pathogen (relation: arg_pathogen)
Drug_Class targets Pathogen (relation: drugclass_pathogen)


The final dataset from preprocessing has columns: ‘x_index’, ‘x_name’, ‘x_type’, ' y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’. 


Script output: amr_layer_data.csv

## Preprocessing data from PrimeKG for host layer: 

Corresponding file: data_extraction_cleaning_host_layer.ipynb

Data download
1) Main files from Harvard Dataverse: nodes.csv, edges.csv, kg.csv

Script input: nodes.csv, kg.csv


Preprocessing involved:
Removing duplicate relationship since Prime KG has two edges for the same relation(x to y & y to x)
Removing nodes we would not immediately need: ‘Drugs’, ‘Exposure’, ‘Cellular_Component ', ' Molecular_Function ', ' Anatomy '.
Removing ‘phenotype_absent’ relation, since we are interested in effect/phenotype that are actually expressed as a result of a disease. 




Considering our curated schema, our selected key disease/infection related to Type 2 diabetes mellitus are Pneumonia, Urinary Tract Infection, Cellulitis, Tuberculosis, Diabetic foot infection/ulcer, Folliculitis, Osteomyelitis, Otitis externa, Meliodiosis, Periodontitis, Bacteremia.
Key effect/phenotype: Hyperglycemia, Oxidative stress/Increased ROS production, Chronic inflammation, Immune dysregulation/immunodeficiency, Insulin resistance, Obesity, Peripheral neuropathy. 
Out of these, Pneumonia, Urinary Tract Infection, Folliculitis, Otitis externa, Periodontitis, Bacteremia, Hyperglycemia, Immunodeficiency, Peripheral neuropathy and Type 2 Diabetes Mellitus itself occurred as duplicate nodes with both diseases and effect/phenotypes labels. 
This required reconciliation to centralise the information tied to each of these key nodes. 


Note that:
- Bacteremia was strictly saved as an effect/phenotype. This was converted to disease.
- Penetrating foot ulcers was strictly saved as an effect/phenotype. This was converted to disease.
- UTI (disease) was merged with Recurrent UTIs (effect/phenotype)
-  obesity disorder was strictly saved as a disease. We converted to effect/phenotype.


Manual inspection revealed that some of these entities (all except Insulin resistance) connected to Type 2 diabetes after 3 or more indirect connections, making it difficult to incorporate naturally. We built shorter connections between T2DM and the core effects and comorbid infections of interest, with justification from the literature. 


Subgraph extraction was then done with the following (one-hop) algorithm.
1) We find all relations where T2DM is either a source node or a target node.
2) If T2DM is connected to a disease, we find and include only the disease-phenotype/gene relations. This adds indirect phenotype and gene nodes through connected diseases.
3) If T2DM is connected to a phenotype, we find the phenotype-phenotype/gene relations and concatenate. This adds indirect other phenotype and gene nodes through connected phenotypes.
4) If T2DM is connected to a gene, find the gene-gene/pathway/BP/phenotype relations and concatenate.
None of these includes connections back to diseases, as this is not a direct comorbidity and may complicate the subgraph. 
In summary, there is only one more non-disease indirect layer to avoid an unending loop and to avoid recycling information.
Node reconciliation and extraction resulted in duplicates, which were duly taken care of. 
To highlight the immune dysfunction associated with Type 2 diabetes mellitus, we defined an immune keyword list with the help of the literature and https://www.ncbi.nlm.nih.gov/books/NBK230991/ as :

keywords = ['lymph', 'NK',' T cell', 'immune', 'immuno', 'antibody', 'B cells', 'CD', 'cytokines', 'IgA', 'IgG', 'IgM',
'IgE', 'IgD',' IL-','inflammation','inflammatory','leuko','Phago','Macrophages' ,'neutrophil','monocyte','monokine','basophil',
'eosinophil','TNF','histio','antigen','immunodeficiency','rheumatoid factor','interleukin','interferon','IFN','TCR']


Phenotypes/effects which contained at least one of these words were retagged as immune_effect/phenotype. 


The final dataset from preprocessing has columns: ‘x_index’, ‘x_name’, ‘x_type’, ' y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’. 


Script output: host_layer_data.csv



## Further preprocessing: 
### For Drug_class-gene connection

Corresponding file: drug_drug_class.ipynb

Script input: host_layer_data.csv, kg.csv, drug_data.csv, antibiotics_list.csv


Using a reliable Kaggle dataset (https://doi.org/10.34740/kaggle/dsv/7850792) on Antibiotics and their classes, we identified the genes/proteins associated with antibiotics in the drug classes present in the AMR layer. This allowed us to create a connection between gene/protein entities in Layer 2 and Drug_class entities in Layer 3. 


The final dataset from preprocessing has columns: ‘x_index’, ‘x_name’, ‘x_type’, ' y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’. 


Script output: drug_class_gene_data.csv

### Pathogen-infection connection
Recall that key infections were carefully selected in advance to align with the objective of this project, findings in the literature and the pathogens of interest. These infections and pathogens of interest were then manually connected with the relation: Pathogen ‘infectious_agent for’ infection. 

### KG creation
The final dataset for the KG was simply a concatenation of the dataframes from host_layer_data.csv,  amr_layer_data.csv, drug_class_gene_data.csv and the pathogen_disease_infection dataframe. 



