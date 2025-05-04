Download GRN file: 

https://www.cell.com/cms/10.1016/j.celrep.2017.10.001/attachment/e7309c03-e579-4119-a95e-376ab2066cbb/mmc2.csv

Enter command in bash in reduce this file to lung tissue-specific GRN and save to a file:

(head -n 1 mmc2.csv; grep 'Lung' mmc2.csv) > grn.csv

Download the GCN file from a paper that looked at specific regulatory mechanism in nomrmal and cancerous human lung tissue:

https://link.springer.com/article/10.1186/s12864-022-08591-9


Translate Gene Identifiers Using Biomart Mapping Tables and an sqlite database. Download the table from https://www.ensembl.org/ using human genes and selecting these attributes and saving them in a CSV file:

Gene stable ID
Gene stable ID version
Transcript stable ID
Transcript stable ID version
Gene name


Prepare the files for database loading (convert to tab-delimited format:

cat mmc2.csv | sed 's/\"//g' | sed 's/,/\t/4;s/,/\t/3;s/,/\t/1;s/,/\t/1' > grn.tab

cat gcn.csv | sed 's/\"//g' | sed 's/,/\t/4;s/,/\t/3;s/,/\t/1;s/,/\t/1' > gcn.tab

#(Change TF column header to Gene A and target gene column to GeneB)



Intall sqlite 3 shell and create a database called mapping:

Sqlite3 mapping.db

Use the shell to load 3 tables into the mapping database(grn,gcn,names):

.separator "\t"
.import grn.tab grn
.separator "\t"
.import gcn.tab gcn
.mode csv
.import mart_export.txt names
.schema grn
.schema gcn
.schema names
.separator "\t"
.headers on
.output GRN-REMAPPED.tsv SELECT grn.TF, names."Gene Name", grn.Tissues, grn.TargetGene FROM grn INNER JOIN names ON names."Gene stable ID" = grn.TargetGene WHERE grn.Tissues LIKE '%Lung%';


Make a merged GRN and GCN edge list with bash:

cat GRN-REMAPPED.tsv| awk '{print $1,$2}' > temp
awk '{print "GRN " $0}' temp   > temp2
sed '1s/GRN/NetworkType/g' temp2  > temp3
sed '1s/"Gene/GeneTarget/g' temp3  > GRN_edges.tab
rm temp temp2 temp3

cat gcn.tab | awk '{print $1,$2}' > temp
awk '{print "GCN " $0}' temp   > temp2
sed '1s/GCN/NetworkType/g' temp2  > GCN_edges.tab
rm temp temp2

#Concatenate the files
(tail -n +2 GRN_edges.tab; tail -n +2 GRN_edges.tab; tail -n +2 GCN_edges.tab) > merged.gcn.grn.tab

#remove duplicate lines
cat merged.gcn.grn.tab | uniq | sed 's/\s/\t/g' > unique.merged.gcn.grn.tab
