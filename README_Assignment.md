# UNIX Assignment
## Data inspection
### Attributes of Fang_et_al_genotypes

The following snipet comprises the UNIX commands I used for data inspection of the Fang-et_al_genotypes.txt file

```javascript
$ wc fang_et_al_genotypes.txt
$ grep -v "^#" fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
$ du -h fang_et_al_genotypes.txt
$ less fang_et_al_genotypes.txt
$ file fang_et_al_genotypes.txt
```

To learn about the diffent nucleotide pairs in the file, I ran a `less` command and used the checker `#/nucleotide pair` for each of the 16 possible combinations

By inspecting this file, I learned that 

 1. The file has 2783 lines, and 986 columns. 
 2. The file has data for the nucleotides 'A, T, G, C" in the  A/A, T/T, G/T, C/T, G/G and C/C pairs. 
 3. The file is 6.5Mb in size 
 4. the file has no non-ASCII characters

The following snipet comprises the UNIX commands I used for data inspection of the snp_position.txt file

```javascript
$ wc snp_positions.txt
$ file snp_position.txt
$ du -h snp_position.txt
$ grep -v "^#" snp_position.txt | awk -F "\t" '{print NF; exit}'
```

By inspecting this file, I learned that

 - The file is made of 984 lines and 15 columns.
 - the file is 41Kb big in size.
 - The file is encoded in ASCII.

To know how much of the different `Groups` of maize and teosinte we have in the Fang_et_al_genotypes.txt file, I used...
```javascript
grep -v "^#" fang_et_al_genotypes.txt | cut -f3 | sort | uniq -c | sort -n
```
...and this showed me that we have 
 - **For maize**
  - 290 (ZMMIL)
  - 1256 (ZMMLR)
  - 27 (ZMMMR)
  
- **For teosinite;**
 - 900 (ZMPBA)
 - 41 (ZMPIL)
 - 34 (ZMPJA)
 
 I wanted to learn about the number of `Chromosomes` whose data we have and I ran...
```javascript
grep -v "^#" snp_position.txt | cut -f3 | sort | uniq -c | sort -n
```
...which sorted the information in colum 3 of the snp_position.txt file and counted how often each chromosome appeared, considering a chromosome as a unique character. 

This showed me that our data is for 10 chromosomes with most SNPs appearing on Chromosome 1 while the least SNPs are on chromosome 10. This same command showed that there are 27 SNPs whose location cannot be traced while 6 of them appear on more than one chromosome.

# Data processing
### Maize data

grepping the fang_et_al_genotypes.txt such that we send data containing `ZMM` and `ZMP` groups to specific files allows us otain maize and teosinite files data only in separate files

```javascript
grep 'ZMM' fang_et_al_genotypes.txt > maize_data.txt
grep 'ZMP' fang_et_al_genotypes.txt > teosinite_data.txt
```

I then attached headers from the original fang_et_al_genotypes.txt file to the created data files above using the snippet below.

```
$ head -n 1 fang_et_al_genotypes.txt > headers_fang.txt
$ cat headers_fang.txt maize_data.txt > headed_maize.txt
$ cat headers_fang.txt teosinite_data.txt > headed_teosinite.txt
```

Just to confirm the action for addition of headers had been successful by checking the first 4 columns for both the maize and teosinite files

```
$ cut -f1,2,3,4 headed_maize.txt | head -n 4
$ cut -f1,2,3,4 headed_teosinite.txt | head
 -n 4
```

I then transposed the created maize and teosinite files and sorted them by the first column, containing the `SNP_ID` after transposition.

```
$  awk -f transpose.awk maize_data.txt > transposed_maize_data.txt
$ awk -f transpose.awk teosinite_data.txt > tran
sposed_teosinite_data.txt
$  sort -k1,1 transposed_maize_data.txt > sorted_transposed_maize.txt
$  sort -k,1 transposed_teosinite_data.txt > sorted_transposed_teosinite.txt
```

To check if the transpositiona and sorting had been performed, I cut the first 3 columns and looked at their first 4 raws with a `head -n 4` and compared that with their original untransposed files

```
$ cut -f1,2,3 sorted_transposed_maize.txt | head
 -n 4
$ cut -f1,2,3 sorted_transposed_teosinite.txt | head
 -n 4
$ cut -f1,2,3 maize_data.txt | head -n 4
```

To further check if the files are propery sorted, I ran a sort check, for which a 0 upon `echo $?` would mean that they were successfully sorted

```javascript
$ sort -k1,1 -c sorted_transposed_maize.txt
$ echo $?
$ sort -k1,1 -c sorted_transposed_teosinite.txt
$ echo $?
```

Since I wanted my second column to be for Chromosome, I created a separate `snp_position` file with the native **cdv_marker_id** removed. This was done by the following command.

```
grep -v "^#" snp_position.txt | cut -f1,3,4,5,6,7,8,9,10,11,12,13,14,15 | head -n 5 > snp_data_less_cdv.txt
```

I confirmed the elimianation of the second column by `cutting` the first four columns of the `snp_data_less_cdv.txt` file

```
cut -f1,2,3,4 snp_data_less_cdv.txt
```
This comfirmed that the order of the columns was "SNP_ID", then "Chromosome" and "position". I then sorted the `snp_data_less_cdv.txt` file and also echoed it to confirm it was sorted before I could join it to the transposed files above.

```
$ sort -k1,1 snp_data_less_cdv.txt > sorted_snp_data.txt
$ sort -k1,1 -c sorted_snp_data.txt
$ echo $?
```

Giving me a `0` showed that it was successfully sorted. So I went on to join the file with the transposed maize and teosinite files




