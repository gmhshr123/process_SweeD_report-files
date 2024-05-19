
# Summary of Steps for Processing SweeD Report Files

This document summarizes the steps and scripts used to process SweeD report files, starting from extracting relevant columns and adding headers to filtering specific chromosome data.

## Step 1: Process `.txt` Files with `vcftools`

First, we processed `.txt` files to create VCF files.

### Script: `process_files.sh`
```bash
#!/bin/bash

# Loop through all .txt files in the current directory
for file in *.txt; do
  # Extract the base name of the file (without extension)
  base_name=$(basename "$file" .txt)
  
  # Run the vcftools command with the current file
  vcftools --vcf all.females.noasterisk.vcf --keep "$file" --recode --out "${base_name}.female"
done
```

## Step 2: Run `SweeD` on Each Generated VCF File

We then ran the `SweeD` command on each generated `.recode.vcf` file.

### Script: `process_and_sweed.sh`
```bash
#!/bin/bash

# Step 1: Process .txt files with vcftools
for file in *.txt; do
  base_name=$(basename "$file" .txt)
  vcftools --vcf all.females.noasterisk.vcf --keep "$file" --recode --out "${base_name}.female"
done

# Step 2: Run SweeD on each generated .recode.vcf file
for recode_file in *.recode.vcf; do
  base_name=$(basename "$recode_file" .recode.vcf)
  /home/minguo/sweed/SweeD -name "${base_name}.10k" -input "$recode_file" -grid 10000
done
```

## Step 3: Format SweeD Report Files

Next, we formatted the SweeD report files to include only lines starting with numbers.

### Script: `format_sweed_reports.sh`
```bash
#!/bin/bash

# Loop through all SweeD_Report.*.10k files in the current directory
for file in SweeD_Report.*.10k; do
  # Run the awk command on the current file and redirect output to a new file with .format extension
  awk '/^[0-9]/' "$file" > "${file}.format"
done
```

## Step 4: Add Chromosome Information

We added chromosome information based on line numbers.

### Script: `add_chromosome_column.sh`
```bash
#!/bin/bash

# Loop through all SweeD_Report.*.10k.format files in the current directory
for file in SweeD_Report.*.10k.format; do
  # Define the output file name with .addcol extension
  output_file="${file}.addcol"
  
  # Run the awk command to add chromosome information
  awk -v OFS="	" '{
    if (NR <= 10000) chr="Chr01";
    else if (NR <= 20000) chr="Chr02";
    else if (NR <= 30000) chr="Chr03";
    else if (NR <= 40000) chr="Chr04";
    else if (NR <= 50000) chr="Chr05";
    else if (NR <= 60000) chr="Chr06";
    else if (NR <= 70000) chr="Chr07";
    else if (NR <= 80000) chr="Chr08";
    else if (NR <= 90000) chr="Chr09";
    else if (NR <= 100000) chr="Chr10";
    else if (NR <= 110000) chr="Chr11";
    else if (NR <= 120000) chr="Chr12";
    else if (NR <= 130000) chr="Chr13";
    else if (NR <= 140000) chr="Chr14";
    else if (NR <= 150000) chr="Chr15";
    else if (NR <= 160000) chr="Chr16";
    else if (NR <= 170000) chr="Chr17";
    else if (NR <= 180000) chr="Chr18";
    else if (NR <= 190000) chr="Chr19";
    print $0, chr
  }' "$file" > "$output_file"
done
```

## Step 5: Reorder Columns

We reordered the columns so the 6th column becomes the 1st.

### Script: `reorder_columns.sh`
```bash
#!/bin/bash

# Loop through all SweeD_Report.*.10k.format.addcol files in the current directory
for file in SweeD_Report.*.10k.format.addcol; do
  # Define the output file name with .reordered extension
  output_file="${file}.reordered"
  
  # Run the awk command to rearrange columns
  awk -v OFS="	" '{print $6, $1, $2, $3, $4, $5}' "$file" > "$output_file"
done
```

## Step 6: Add Header

We added a header to the reordered files.

### Script: `add_header_and_reorder_columns.sh`
```bash
#!/bin/bash

# Define the header
header="Chr	Position	Likelihood	Alpha	StartPos	EndPos"

# Loop through all SweeD_Report.*.10k.format.addcol files in the current directory
for file in SweeD_Report.*.10k.format.addcol; do
  # Define the output file name with .final extension
  output_file="${file}.final"
  
  # Print the header to the output file
  echo -e "$header" > "$output_file"
  
  # Run the awk command to rearrange columns and append to the output file
  awk -v OFS="	" '{print $6, $1, $2, $3, $4, $5}' "$file" >> "$output_file"
done
```

## Step 7: Extract Rows with `Chr07` and Include Header

Finally, we extracted rows where the `Chr` column value is `Chr07` and included the header.

### Script: `extract_chr07_with_header.sh`
```bash
#!/bin/bash

# Loop through all SweeD_Report.*.10k.format.addcol.final files in the current directory
for file in SweeD_Report.*.10k.format.addcol.final; do
  # Define the output file name with Chr07_only prefix
  base_name=$(basename "$file" .10k.format.addcol.final)
  output_file="Chr07_only.${base_name}"

  # Extract the header and rows where the first column (Chr) is Chr07, then save to the new file
  awk -v OFS="	" 'NR==1 || $1 == "Chr07"' "$file" > "$output_file"
done
```
