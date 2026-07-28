# BIP_Project_T2DM_Heterogenous_KG
The wider project studies the comorbidity between type 2 diabetes mellitus and infectious disease, with a specific narrative angle: diabetes -> recurrent infection -> increased antibiotic exposure -> antimicrobial resistance (AMR). The goal is to build a small, interpretable, mechanism-aware knowledge graph prototype organised into three key layers.

## Preprocessing data from CARD for AMR layer

Corresponding file: data_extraction_cleaning_AMR_layer.ipynb

Data download
1) Downloaded the CARD-R Resistomes, Variants, & Prevalence data folder (version 4.0.2) from https://card.mcmaster.ca/download
2) Further downloaded and extracted "card_prevalence.txt.gz"


Script input: card_prevalence.csv


To work with this dataset, the columns titled 'Model_ID', 'Model_Type', 'NCBI_Plasmid', 'NCBI_WGS', 'NCBI_Chromosome', 'NCBI_Genomic_Island', and 'Criteria' were dropped. 
The column 'Name' was renamed to ‘Resistance_Gene’, and  'ARO Categories’ was split into ‘Resistance_Mechanism’ and ‘Drug_Class’.

Data from the CARD database was further restricted to those of the pathogens of interest. These are grouped into core and extended as outlined below.


Core: Klebsiella pneumoniae, Streptococcus pneumoniae, Escherichia coli, Acinetobacter baumannii, Mycobacterium tuberculosis
Extended: Enterococcus faecalis, Staphylococcus aureus, Pseudomonas aeruginosa, Burkholderia pseudomallei, Porphyromonas gingivalis


We provide unique IDs for pathogens, resistance mechanisms, and drug classes. We further create the relation and display relationship tags: 
ARGs confer_resistance_to Drug class (relation: arg_drugclass)
ARGs employs Resistance Mechanism (relation: arg_mechanism)
ARGs associates_with Pathogen (relation: arg_pathogen)
Drug_Class targets Pathogen (relation: drugclass_pathogen)


The final dataset from preprocessing has the following columns: ‘x_index’, ‘x_name’, ‘x_type’, ‘y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’. 


Script output: amr_layer_data.csv

## Preprocessing data from PrimeKG for host layer

Corresponding file: data_extraction_cleaning_host_layer.ipynb

Data download from https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/IXA7BM
1) Main files from Harvard Dataverse (version 2.1): nodes.csv, edges.csv, kg.csv

Script input: nodes.csv, kg.csv


Preprocessing involved:
Removing duplicate relationships since Prime KG has two edges for the same relation(x to y & y to x)
Removing nodes we would not immediately need: ‘Drugs’, ‘Exposure’, ‘Cellular_Component ', ' Molecular_Function ', ' Anatomy '.
Removing the ‘phenotype_absent’ relation, since we are interested in effect/phenotype that are actually expressed as a result of a disease. 




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
2) If T2DM is connected to a disease, we find and include the disease-phenotype or disease-gene relations of those diseases. Further disease-disease relations are included only if the disease is a key disease entity. This adds indirect phenotype and gene nodes through connected diseases.
3) If T2DM is connected to a phenotype, we find the phenotype-phenotype or phenotype-gene relations of that phenotype and concatenate. This adds indirect other phenotype and gene nodes through connected phenotypes.
4) If T2DM is connected to a gene, find the gene-gene, gene-pathway, gene-biological process or gene-phenotype relations and concatenate.
None of these includes connections back to non-core diseases, as these are not direct comorbidities and may complicate the subgraph. 
In summary, there is only one more indirect layer to avoid an unending loop and information recycling. Node reconciliation and extraction resulted in duplicates, which were duly dealt with.

To highlight the immune dysfunction associated with Type 2 diabetes mellitus, we defined a relevant immune keyword list with the help of the literature and https://www.ncbi.nlm.nih.gov/books/NBK230991/ as :

keywords = ('lymph', 'NK',' T cell', 'immune', 'immuno', 'antibody', 'B cells', 'CD', 'cytokines', 'IgA', 'IgG', 'IgM', 'IgE', 'IgD',' IL-', 'inflammation', 'inflammatory', 'leuko', 'Phago', 'Macrophages', 'neutrophil', 'monocyte', 'monokine', 'basophil', 'eosinophil', 'TNF','histio', 'antigen', 'immunodeficiency', 'rheumatoid factor', 'interleukin', 'interferon', 'IFN', 'TCR')


Phenotypes/effects which contained at least one of these words were retagged as immune_effect/phenotype. 


The final dataset from preprocessing has columns: ‘x_index’, ‘x_name’, ‘x_type’, ' y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’. 


Script output: host_layer_data.csv



## Further processing
### For Drug_class-gene connection

Corresponding file: drug_drug_class.ipynb

Script input: host_layer_data.csv, kg.csv, drug_data.csv, antibiotics_list.csv


Using a reliable Kaggle dataset (https://doi.org/10.34740/kaggle/dsv/7850792) on Antibiotics and their classes, we identified the genes/proteins associated with antibiotics in the drug classes present in the AMR layer. This allowed us to create a connection between gene/protein entities in Layer 2 and Drug_class entities in Layer 3. 


The final dataset from preprocessing has columns: ‘x_index’, ‘x_name’, ‘x_type’, ' y_index ', ' y_name ', ' y_type ', ' relation' and ‘display_relation’. 


Script output: drug_class_gene_data.csv

### Pathogen-infection connection
Recall that key infections were carefully selected in advance to align with the objective of this project, findings in the literature and the pathogens of interest. These infections and pathogens of interest were then manually connected with the relation: Pathogen ‘infectious_agent for’ infection. 

## Curating a causal layer
To formally connect the host layer with the AMR layer, an intermediate layer was created to link the varied 'recurrent infection' nodes which were related to Type 2 diabetes mellitus via immunodeficiency to the concept of antimicrobial resistance and hence the AMR layer. Before this, however, we created some edges which we considered important. Each 'recurrent infection' could be and hence was matched with a key disease that we had already manually connected to the Type 2 diabetes node. 

Now, from the literature, some key components of immunodeficiency were connected to increased susceptibility to specific pathogens. Using this, relations between components of immunodeficiency and pathogen nodes were created. For example, 'Neutropenia' increases_susceptibility_to 'Klebsiella pneumoniae'. 

Further manual edges were included: 
'Recurrent bacterial infections' associated_with (phenotype_phenotype) 'increased antibiotic exposure' 

'Hyperglycemia' associated_with (phenotype_phenotype)  ' increased risk of pathogen mutation'  associated_with (phenotype_phenotype)  'Increased antibiotic exposure'. 

'Increased antibiotic exposure' increases _selection_for all antibiotic resistance genes

## KG creation
The final dataset for the KG was simply a concatenation of the dataframes from host_layer_data.csv,  amr_layer_data.csv, drug_class_gene_data.csv, the pathogen_disease_infection dataframe and all the dataframes arising from the necessary manual connections and the curated bridge layer. 



