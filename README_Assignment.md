# UNIX Assignment
## Data inspection
### Attributes of fang_et_al_genotypes.txt data

The following snipet comprises the UNIX commands I used for data inspection of the Fang-et_al_genotypes.txt file.

```
$ wc fang_et_al_genotypes.txt
$ grep -v "^#" fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
$ du -h fang_et_al_genotypes.txt
```

To learn about the diffent nucleotide pairs in the file, I ran a `less` command and used the checker `#/nucleotide pair` for each of the 16 possible combinations. I also inspected the file for any ACII characters and there was none.

```
$ less fang_et_al_genotypes.txt
$ file fang_et_al_genotypes.txt
```

By inspecting this file, I learned that 

 1. The file has 2783 rows, 2744038 words, 11051939 characters, and 986 columns. 
 2. The file has data for the nucleotides 'A, T, G, C" in the  A/A, T/T, G/T, C/T, G/G and C/C pairs. 
 3. The file is 6.5Mb in size 
 4. The file has ASCII texts with very long lines.
 
To know how much of the different `Groups` of maize and teosinte we have in the Fang_et_al_genotypes.txt file, I used.

```
$ grep -v "^#" fang_et_al_genotypes.txt | cut -f3 | sort | uniq -c | sort -n
```

This showed me that we have 
 
- **For maize**
  - 290 (ZMMIL)
  - 1256 (ZMMLR)
  - 27 (ZMMMR)
  
- **For teosinite;**
  - 900 (ZMPBA)
  - 41 (ZMPIL)
  - 34 (ZMPJA)

### Attributes of snp_position.txt data

The following snipet comprises the UNIX commands I used for data inspection of the snp_position.txt file

```
$ wc snp_position.txt
$ file snp_position.txt
$ du -h snp_position.txt
$ grep -v "^#" snp_position.txt | awk -F "\t" '{print NF; exit}'
```

By inspecting this file, I learned that

 - The file is made of 984 rows, 13198 words and 82763 characters.
 - The file has 15 columns.
 - the file is 41Kb big in size.
 - The file is encoded as ASCII text.

I wanted to learn about the number of Chromosomes whose data we have and I ran, so I grepped the `snp_position.txt` file and piped the result to `cut` through the third column, `sort` its and count for the number of `uniq` characters in it.

```
$ grep -v "^#" snp_position.txt | cut -f3 | sort | uniq -c | sort -n
```

This showed me that our data is for 10 chromosomes with most SNPs appearing on Chromosome 1 while the least SNPs are on chromosome 10. This same command showed that there are 27 SNPs whose location is unknown while 6 of them appear on more than one chromosome.

# Data processing
## snp file

I started this by first preparing the snp_position.txt file for further operations. I removed the second colum that had the **cdv_marker_id** such that the second column becomes **Chromosome**. I saved its header in a separate file and created another file with the `snp_position` file that doesn't have headers. I sorted this unheaded file anc echoed it to confirm the sorting. The "0" indicated that it had been successfully sorted.

```
$ cut -f1,3,4,5,6,7,8,9,10,11,12,13,14,15 snp_position.txt > snp_less_cdv.txt
$ head -n 1 snp_less_cdv.txt > headers_snp_position.txt
$ tail -n +2 snp_less_cdv.txt > unheaded_snp.txt
$ sort -k1,1 unheaded_snp.txt > sorted_unheaded_snp.txt
$ sort -k1,1 -c sorted_unheaded_snp.txt
$ echo $?
```

The "0" indicated that it had been successfully sorted.

## Maize data

Having had the "snp-file" ready for merging, I went on to organize the maize dataset. I first created a separate file having headers of the fang_et_al_genotypes.txt.

```
$ head -n 1 fang_et_al_genotypes.txt > headers_fang.txt
```

I then separated the "ZMM" data from the `fang_et_al_genotypes.txt` into a separate file, attached headers to it and transposed it. 

```
$ grep 'ZMM' fang_et_al_genotypes.txt > maize_data.txt
$ cat headers_fang.txt maize_data.txt > headed_maize.txt
$ awk -f transpose.awk headed_maize.txt > transposed_maize.txt
```

I then went on to create a transposed maize file that doesnt have headers so that I could sort it. I sorted this and it was ready for merging with the `snp-file`. I confirmed that it had been merged by checking on its number of rows with the `wc -ls` command and also echoed it to comfirm it had been successfully sorted. The "0" echo confirmed it had been sorted while the `wc -l` giving me 983 rows, which was the same number of rows for the snp-file confirmed it had been transposed and ready for merging.

```
$ tail -n +4 transposed_maize.txt > unheaded_transposed_maize.txt
$ sort -k1,1 unheaded_transposed_maize.txt > sorted_unheaded_maize.txt
$ sort -k1,1 -c sorted_unheaded_maize.txt
$ echo $?
$ wc -l sorted_unheaded_maize.txt
```

 I then merged the snp-file with this maize file to formed one composite data file and attached snp-headers to check that the joint file is having SNP_ID, Chromosome and position in the first, second and third columns respectively. I confirmed this cutting the first 4 columns. I navigated this joined-file to confirm the merging was successfull by grepping the number of columns it had, which must be the sum of the columns of the two separate file less by one (the common SNP_ID column). 
 
 ```
 $ join -1 1 -2 1 -t $'\t' sorted_unheaded_snp.txt sorted_unheaded_maize.txt > joined_maize.txt
 $ cut -f1,2,3,4 joined_maize.txt | head -n 4
 $ tail -n +5 joined_maize.txt | awk -F "\t" '{print NF; exit}'
 ```
 
 I then attached headers to this file
 
 ```
 $ cat headers_snp_position.txt joined_maize.txt > head_joined_maize.txt
 ```
 
 ## 22 snp files for maize
 ### SNPs ordered based on increasing position values (maize)
 
 I loaded and used the `bioawk` command with the `-c hdr` that identified column named **"Chromosome"**, selected values equal to the chromosome number and then piped the results for sorting according to column 3, by treating them as numerics.
 
 Since the original file had missing values represented with "?", I did not do anything to these at this stage. I then sent the result in a separate file I named according to the chromosome I was dealing with. 
 
 I did the same task for all the 10 chromosomes.
 
 ```
 $ module load bioawk
 $ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr1_maize.txt
 $ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr2_maize.txt
 $ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr3_maize.txt
 $ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr4_maize.txt
 $ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr5_maize.txt
 $ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr6_maize.txt
 $ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr7_maize.txt
 $ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr8_maize.txt
 $ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr9_maize.txt
 $ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3n > chr10_maize.txt
 ```
 
 ### SNPs ordered based on decreasing position values and missing data encoded by the symbol: - (maize)
 
 I used the same `bioawk` command but sorted in the reversed order using `rn` in specifications.
 
 I then piped the results of `sort` to `sed` to identify and replace the "?", which represented missing values in the original files, with "-".
 
 I then sent the final results to a separate file. I used the same command for all the chromosomes
 
 ```
  $ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr1_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr2_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr3_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr4_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr5_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr6_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr7_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr8_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr9_rev_maize.txt
  $ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_maize.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr10_rev_maize.txt
  ```
Just to be sure that the substitution was successfull with `sed`, I could `less` the created file and then search for th new substitute. This would show me the points where it had been placed. I did this for all the files as I created them.

```
$ less chr3_rev_maize.txt
#/-
```

### SNPs with unknown positions (maize)

Just to get familiar with other programs, I decided to gather these with `awk`. This command looks for the specific column and searched for the specified component  in the file of interest then send results to the file you name. In order to check if the file I had was the one I wanted, I `cut` the first 5 columns and looked at the first 20 rows using `head` and the last 20 rows using `tail`. 

```
$ awk -F "\t" '$3 ~ /unknown/' head_joined_maize.txt > unknown_position_maize.txt
$ cut -f1,2,3,4,5 unknown_position_maize.txt | head -n 20
$ cut -f1,2,3,4,5 unknown_position_maize.txt | tail -n 20
```

### SNPs with multiple positions (maize)

I also used `awk` to obtain this file and as well checked it using the `cut` command.

```
$ awk -F "\t" '$3 ~ /multiple/' head_joined_maize.txt > multiple_position_maize.txt
$ cut -f1,2,3,4,5 multiple_position_maize.txt | head -n 20
$ cut -f1,2,3,4,5 multiple_position_maize.txt | tail -n 20
```

## Teosinte

In order to process the data file and obtain 22 files for Teosinte, I followed the same steps and used the same command lines as used for the Maize data above. For this reason, I will eliminate the wording for the respective command lines and simply write my entire snippet below.

### Making the joint Teosinte file was done as follows

```
$ grep 'ZMP' fang_et_al_genotypes.txt > teosinite_data.txt
$ cat headers_fang.txt teosinite_data.txt > headed_teosinite.txt
$  awk -f transpose.awk headed_teosinite.txt > transposed_teosinite.txt
$ tail -n +4 transposed_teosinite.txt > unheaded_transposed_teosinite.txt
$ sort -k1,1 unheaded_transposed_teosinite.txt > sorted_unheaded_teosinite.txt
$ sort -k1,1 -c sorted_unheaded_teosinite.txt
$ echo $?
$ wc -l sorted_unheaded_teosinite.txt
$ join -1 1 -2 1 -t $'\t' sorted_unheaded_snp.txt sorted_unheaded_teosinite.txt > joined_teosinite.txt
$ cut -f1,2,3,4 joined_teosinite.txt | head -n 4
$ tail -n +5 joined_teosinite.txt | awk -F "\t" '{print NF; exit}'
$ cat headers_snp_position.txt joined_teosinite.txt > head_joined_teosinite.txt
```

## 22 snp files for teosinte
### SNPs ordered based on increasing position values (teosite)

```
$ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr1_teosinite.txt
$ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr2_teosinite.txt
$ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr3_teosinite.txt
$ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr4_teosinite.txt
$ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr5_teosinite.txt
$ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr6_teosinite.txt
$ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr7_teosinite.txt
$ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr8_teosinite.txt
$ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr9_teosinite.txt
$ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3n > chr10_teosinite.txt
```

### SNPs ordered based on decreasing position values and missing data encoded by the symbol: - (teosinte)

```
$ bioawk -c hdr '$Chromosome == "1" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr1_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "2" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr2_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "3" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr3_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "4" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr4_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "5" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr5_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "6" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr6_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "7" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr7_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "8" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr8_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "9" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr9_rev_teosinite.txt
$ bioawk -c hdr '$Chromosome == "10" {print $0}' head_joined_teosinite.txt | sort -t $'\t' -k3,3rn | sed 's/?/-/g' > chr10_rev_teosinite.txt
```
  
To check if the substitution was successful, I used `less` and searched for the "-" using `#/-`
  
```
$ less chr3_rev_teosinite.txt
#/-
```

### SNPs with unknown positions (teosinte)

```
$ awk -F "\t" '$3 ~ /unknown/' head_joined_teosinite.txt > unknown_position_teosinite.txt
$ cut -f1,2,3,4,5 unknown_position_teosinite.txt | head -n 20
$ cut -f1,2,3,4,5 unknown_position_teosinite.txt | tail -n 20
```

### SNPs with multiple positions (teosinte)

```
$ awk -F "\t" '$3 ~ /multiple/' head_joined_teosinite.txt > multiple_position_teosinite.txt
$ cut -f1,2,3,4,5 multiple_position_teosinite.txt | head -n 20
$ cut -f1,2,3,4,5 multiple_position_teosinite.txt | tail -n 20
```
