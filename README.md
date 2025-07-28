# AMREnterobacter
Honours Project 2024 - Antimicrobial Resistance in Environmental Enterobacter Species
## Conda & Mamba

Currently using conda and mamba together, but primarily mamba due to its efficiency

To run initial terminal for any program:
#To activate conda/mamba/program
conda_path
conda activate base
mamba activate "program"

#To install new program
conda_path
conda activate base
mamba create -n envname -c bioconda envname

# Abricate

ABRicate screens contigs for antimicrobial resistance genes or virulence genes or plasmids. Helps to build context around the genes

For initial analysis of files do the following:
#use for initial fasta files --> "resfinder" finds the antimicrobial resistant genes
abricate *.fasta --db resfinder > amr.tab #(name it whatever you like as the tab)

#use for initial fasta files --> "plasmidfinder" finds the plasmids within bacteria
abricate  *.fasta --db plasmidfinder > plasmid.tab #(name it whatever you like as the tab)

#use for initial fasta files --> "vfdb (virulence factor database) finds the virulence factors
abricate *.fasta --db vfdb > virulence.tab #(name it whatever you like as the tab)

#then to summarise and tabulate this data in a clean entry. simply: 
abricate --summary tabname.tab > newtabsummary.tab
#Generally recommended to represent this data in a bar graph for summary of amr particularly.

## Prokka

Prokka is a software tool used for annotating bacterial, archaeal, and viral genomes. It rapidly annotates features of the genome (like coding sequences, rRNA, tRNA, etc.) and assigns functional information to gene products.

For conversion into .gff files
for f in *.fasta; do
  prokka "$f" --compliant --addgenes --cpus 16 --prefix "${f%.*}" --outdir "${f%.*}_prokka"
done

## Panaroo

Panaroo is a tool designed for pan-genome analysis, which involves comparing the genomes of multiple members of a bacterial species or genus. It identifies core and accessory genes within a collection of genomes and can be used for evolutionary studies and identifying species-specific genes.

Once you have the .gff (general feature format) of the isolates/strains. You will want to use panaroo. 

Place all the .gff into one folder. Make a folder within that .gff folder named "panaroo".

Open a terminal and then run this code:
panaroo -i *.gff -o panaroo/ --clean-mode strict -t 16 -a core

## IQtree

IQ-TREE is a bioinformatics software designed for phylogenetic analysis. It employs efficient algorithms to infer evolutionary relationships among sequences or species on a phylogenetic tree, using various models of evolution.

Once you have the panaroo files complete, use: 
iqtree -s core_gene_alignment_filtered.aln --prefix mbov -m MFP -B 1000 -nt AUTO


# Blast

BLAST is a widely used program for comparing an amino acid or nucleotide sequence to a database of sequences to identify regions of similarity. It helps in finding homologous sequences and inferring functional and evolutionary relationships.

For screening of specific sequences in the genome, first make a FASTA file for that sequence, then make a blast database for that sequence.
makeblastdb -in YOURFILENAME.fasta -dbtype nucl -out YOURFILENAME_db

Then run this code to check through each isolate for that sequence using blast.

echo "Creating summary file..."
# Empty the summary file if it already exists
> summary.txt

for isolate in [0-9][0-9][0-9].fasta; do
    echo "Processing $isolate"
    blastn -query $isolate -db YOURFILENAME_db -out "${isolate%.fasta}_results.txt" -outfmt 6
    if [ $? -ne 0 ]; then
        echo "BLAST failed for $isolate"
    else
        echo "Results for $isolate:" >> YOURFILENAME.txt
        cat "${isolate%.fasta}_results.txt" >> YOURFILENAME.txt
        echo "" >> YOURFILENAME.txt # Add a blank line for better readability between sections
    fi
done

echo "BLAST processing complete. Summary file created."

#An easier method to blast:
Easier method to blast:
makeblastdb -in YOURFILENAME.fasta -dbtype nucl -out YOURFILENAME_db

blastn -query query.fasta -db database_file -out results.csv -max_target_seqs 5 -outfmt 6 

# MLST

MLST is a method used for the typing of multiple housekeeping genes to assign isolates of microbial species to specific sequence types. It's a tool for characterizing strains of microorganisms for epidemiological studies.
mlst --csv *.fasta > choose_the_name_of_your_file.csv
