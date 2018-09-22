# BCB546_UNIX_Assignment
- UNIX_ASSIGMENT FOLDER HAS ORIGINAL DATA FOR ASSIGNEMT AND INSTRUCTION 
- PROCESSED_FILES FOLDER HAS THE DATA PROCESSING OR INTERMEDIATE FILES
- FINAL_FILES FOLDER HAS THE REQUIRED FILES FOR THE ASSIGNMENT 
File inspection 

for s in *.txt ; do echo " $(wc -l $s) lines"; echo  "$(awk -F "\t" '{print NF; exit}' $s) $s Number of column"; echo " $(du -h $s) file size"; echo $(file $s) "looking for ASCII and non-ASCII characters in the files" ; done

output for fang_et_al_genotype
2783 fang_et_al_genotypes.txt lines
986 fang_et_al_genotypes.txt Number of column
 11M	fang_et_al_genotypes.txt file size
fang_et_al_genotypes.txt: ASCII text, with very long lines looking for ASCII and non-ASCII characters in the files
output for snp_position
 984 snp_position.txt lines
15 snp_position.txt Number of column
 84K	snp_position.txt file size
snp_position.txt: ASCII text looking for ASCII and non-ASCII characters in the files

processing data 
all files used for processing data will be saved in the processed_file directory

creating Teosinte file only # grouping teosite genotype by ZMPBA, ZMPIL, ZMPJA without headers
grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > ../Processed_file/teosinte_genotype.txt

creating Maize file only # grouping maize genotype by ZMMIL, ZMMLR, ZMMMR without headers
grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > ../Processed_file/Maize_genotype.txt

Extraction of header from the fang_et_al_genotype
Extract header from the fang_et_al_genotype and add it to the maize and teosinte genotype file separately 
# extract header 
head -n 1 fang_et_al_genotypes.txt > ../Processed_file/fang_header
# adding header to the maize and teosinte genotype file
cat fang_header teosinte_genotype.txt > headed_teosinte_genotype.txt
cat fang_header Maize_genotype.txt > headed_maize_genotype.txt

Remove 1 and 2 using cut because there are unnecessary columns
cut -f 3-968 headed_teosinte_genotype.txt > Editheaded_teosinte_genotype.txt
cut -f 3-968 headed_maize_genotype.txt > Editheaded_Maize_genotype.txt

Transpose the data files 
Transpose Editheaded_teosinte_genotype.txt and Editheaded_Maize_genotype.txt since they have headers 
for i in Editheaded_*; do awk -f ../UNIX_Assignment/transpose.awk $i > transposed_"$i"; done
 
sorting the files
sorting the snp_position.txt and the transposed files by column 1 or the SNP ID.
for i in transposed_Editheaded_* snp_position.txt; do sort -k1,1 $i > sorted_"$i" ; done 
check to see if the files are sorted and all outputs for all the files is zero implying sorted 
for i in sorted_*; do sort -k1,1 -c  $i |echo $? ; done

joining the files
i.e. joining the sorted snp_position.txt with sorted transposed maize and teosite genotype files separately
for i in sorted_transposed_Editheaded_*; do join -1 1 -2 1 sorted_snp_position.txt $i > joined_"$i" ; done
checking the joined 
for i in sorted_transposed_Editheaded_* sorted_snp_position.txt joined_sorted_transposed_Editheaded_* ; do echo "number of lines at: $(wc -l $i)"; done
outputs 
number of lines at: 966 sorted_transposed_Editheaded_Maize_genotype.txt
number of lines at: 966 sorted_transposed_Editheaded_teosinte_genotype.txt
number of lines at: 984 sorted_snp_position.txt
number of lines at: 965 joined_sorted_transposed_Editheaded_Maize_genotype.txt
number of lines at: 965 joined_sorted_transposed_Editheaded_teosinte_genotype.txt

sorting by chromosome i.e. sorting by column 3 which has chromosome numbers
for i in joined_sorted_transposed_Editheaded_*; do sort -k3,3n $i > chr_sort_"$i" ; done

check the sorted files and the outputs for all the files are zero implying they are sorted
for i in chr_sort_joined_sorted_transposed_Editheaded_*; do sort -k3,3n -c  $i |echo $? ; done
substitution with questmark (?) or dash (-) using sed command
replacing missing values (tab) with ?
for i in chr_sort_joined_sorted_transposed_Editheaded_*; do sed 's/-t/?/g' $i > Qmark_"$i" ; done
replace the missing data (?) with -
for i in chr_sort_joined_sorted_transposed_Editheaded_*; do sed 's/?/-/g' $i > Dash_"$i" ; done

Extracting chromosomes 
for i in {1..10}; do awk '$3=='$i'' Qmark_chr_sort_joined_sorted_transposed_Editheaded_Maize_genotype.txt > maize_chr"$i"_Qmark.txt; done
for i in {1..10}; do awk '$3=='$i'' Qmark_chr_sort_joined_sorted_transposed_Editheaded_teosinte_genotype.txt > teosinte_chr"$i"_Qmark.txt; done
for i in {1..10}; do awk '$3=='$i'' Dash_chr_sort_joined_sorted_transposed_Editheaded_Maize_genotype.txt > maize_chr"$i"_dash.txt; done
for i in {1..10}; do awk '$3=='$i'' Dash_chr_sort_joined_sorted_transposed_Editheaded_teosinte_genotype.txt > teosinte_chr"$i"_dash.txt; done

Last part of the assignments stored these files in the directory named as Final_files

sorting chromosomes based increasing SNP position
for i in {1..10}; do sort -k4,4n maize_chr"$i"_Qmark.txt > ../Final_files/maize_chr"$i"_Qmark_Increase_snp_position.txt; done
for i in {1..10}; do sort -k4,4n teosinte_chr"$i"_Qmark.txt > ../Final_files/teosinte_chr"$i"_Qmark_Increase_snp_position.txt; done



•	Sorting chromosomes by decreasing snp position
for i in {1..10}; do sort -k4,4nr maize_chr"$i"_dash.txt > ../Final_files/maize_chr"$i"_dash_decrease_snp_position.txt; done
for i in {1..10}; do sort -k4,4nr teosinte_chr"$i"_dash.txt > ../Final_files/teosinte_chr"$i"_dash_decrease_snp_position.txt; done


SNPs with unknown positions

for i in chr_sort_joined_sorted_transposed_Editheaded_*; do grep "unknown" $i > ../Final_files/Unknown_position_"$i" ; done

SNPs with multiple positions
for i in chr_sort_joined_sorted_transposed_Editheaded_*; do grep "multiple" $i > ../Final_files/multiple_position_"$i" ; done
