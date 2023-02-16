


# Data Inspection


## Check if ASCII characters

	$ file fang_et_al_genotypes.txt snp_position.txt 
		fang_et_al_genotypes.txt: ASCII text, with very long lines
		snp_position.txt:         ASCII text
	
	
## Check size
	
	$ ls -lh fang_et_al_genotypes.txt snp_position.txt 
		-rw-r--r--. 1 yueliu domain users 11M Feb 15 13:34 fang_et_al_genotypes.txt
		-rw-r--r--. 1 yueliu domain users 81K Feb 15 13:34 snp_position.txt
	
	$ du -h fang_et_al_genotypes.txt snp_position.txt 
		6.1M	fang_et_al_genotypes.txt
		38K	snp_position.txt
	
	
## Check number of rows
	$ wc -l fang_et_al_genotypes.txt snp_position.txt 
	 	2783 fang_et_al_genotypes.txt
     	984 snp_position.txt
    	3767 total

## check number of columns
	$ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt 
	$ awk -F "\t" '{print NF; exit}' snp_position.txt 

## Inspect content
	$ head -n 20 fang_et_al_genotypes.txt | cut -f 1-10 | column -t
	$ head -n 20 snp_position.txt | cut -f 1-10 | column -t



	
# Data Processing

## Extract maize and teosinte data seperately from fang_et_al_genotypes.txt, transpose, remove headers, and output as new files
	$ grep -E "Group|ZMMIL|ZMMLR|ZMMMR" fang_et_al_genotypes.txt |  awk -f transpose.awk | tail -n +4 > maize_genotypes.txt
	$ grep -E "Group|ZMPBA|ZMPIL|ZMPJA" fang_et_al_genotypes.txt |  awk -f transpose.awk | tail -n +4 > teosinte_genotypes.txt



## Extract columns from snp_position.txt, remove headers, and output as new files

	$ cut -f 1,3,4 snp_position.txt | tail -n +2 > snpid.txt
	
	
## Check and sort genotype data & SNP ID before ***join***

### Check the commone column

	$ sort -k1,1 snpid.txt | uniq | wc -l
		983
	
	$ sort -k1,1 maize_genotypes.txt | uniq | wc -l
		983
	
	$ sort -k1,1 teosinte_genotypes.txt | uniq | wc -l
		983
### Creat sorted files and check if sorted

	$ sort -k1,1 snpid.txt > snpid_sorted.txt
	$ echo $?
		0
	
	$ sort -k1,1 maize_genotypes.txt | uniq > maize_genotypes_sorted.txt
	$ echo $?
		0
	
	$ sort -k1,1 teosinte_genotypes.txt | uniq > teosinte_genotypes_sorted.txt
	$ echo $?
		0

## Join SNP position with genotypes with tab-delimited, keep unpairable liknes,and check the number of join lines
	$ join -1 1 -2 1 -t -a 1 $'\t' snpid_sorted.txt maize_genotypes_sorted.txt > maize_joint.txt
	
	$ join -1 1 -2 1 -t $'\t' -a 1 snpid_sorted.txt teosinte_genotypes_sorted.txt > teosinte_joint.txt
	
	$ wc -l maize_joint.txt teosinte_joint.txt snpid_sorted.txt  maize_genotypes_sorted.txt teosinte_genotypes_sorted.txt
		983 maize_joint.txt
     	983 teosinte_joint.txt
     	983 snpid_sorted.txt
     	983 maize_genotypes_sorted.txt
     	983 teosinte_genotypes_sorted.txt
    	4915 total



## Select SNPs with multiple or unknown position respectively

	$ for i in {multiple,unknown}; do awk '$2 ~ /'$i'/ || $3 ~ /'$i'/ {print $0}' maize_joint.txt > maize_position_$i.txt; done
	 
	$ for i in {multiple,unknown}; do awk '$2 ~ /'$i'/ || $3 ~ /'$i'/ {print $0}' teosinte_joint.txt > teosinte_position_$i.txt; done
	
	

## Select rows based on chromosome value and output as new files 

	$ for i in {1..10}; do awk '$2 ~ /'$i'$/ {print $0}' maize_joint.txt > maize_chr_$i.txt; done
	$ for i in {1..10}; do awk '$2 ~ /'$i'$/ {print $0}' teosinte_joint.txt > teosinte_chr_$i.txt; done

### Check if the selection of maize is right
	$ cut -f 2 maize_joint.txt | sort -V | uniq -c
    155 1
    127 2
    107 3
     91 4
    122 5
     76 6
     97 7
     62 8
     60 9
     53 10
      6 multiple
     27 unknown
     
	$ wc -l maize_chr_* 
     53 maize_chr_10.txt
    155 maize_chr_1.txt
    127 maize_chr_2.txt
    107 maize_chr_3.txt
     91 maize_chr_4.txt
    122 maize_chr_5.txt
     76 maize_chr_6.txt
     97 maize_chr_7.txt
     62 maize_chr_8.txt
     60 maize_chr_9.txt
    950 total
  
  
### Check if the selection of teiosinte is right
	$ cut -f 2 teosinte_joint.txt | sort -V | uniq -c
    155 1
    127 2
    107 3
     91 4
    122 5
     76 6
     97 7
     62 8
     60 9
     53 10
      6 multiple
     27 unknown
	
	$ wc -l teosinte_chr_*
     53 teosinte_chr_10.txt
    155 teosinte_chr_1.txt
    127 teosinte_chr_2.txt
    107 teosinte_chr_3.txt
     91 teosinte_chr_4.txt
    122 teosinte_chr_5.txt
     76 teosinte_chr_6.txt
     97 teosinte_chr_7.txt
     62 teosinte_chr_8.txt
     60 teosinte_chr_9.txt
    950 total


## Order SNPs on each chromose with increasing position and replace missing data with ? 


	$ for i in {1..10}; do cat maize_chr_$i.txt | sort -k3,3n > maize_increase_chr$i.txt; done 
	
	$ for i in {1..10}; do cat teosinte_chr_$i.txt | sort -k3,3n > teosinte_increase_chr$i.txt; done 


## Order SNPs on each chromose with decreasing position and replace missing data with - 

	$ for i in {1..10}; do cat maize_chr_$i.txt | sort -k3,3nr | sed 's/\?/\-/g' > maize_decrease_chr$i.txt; done
	
	$ for i in {1..10}; do cat teosinte_chr_$i.txt | sort -k3,3nr | sed 's/\?/\-/g' > teosinte_decrease_chr$i.txt; done
	


## Summary



