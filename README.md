# VDJ Cross-Reactivity & T-Cell Receptor Homology Pipeline

[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/mit-license.php)
[![NCBI BLAST+](https://img.shields.io/badge/NCBI-BLAST%20v2.17%2B-green.svg)](https://blast.ncbi.nlm.nih.gov/)

A high-throughput, multi-virus, multi-cancer computational bioinformatics pipeline designed to screen and quantify T-cell receptor beta (TRB) CDR3 sequence homology against multi-viral reference databases (e.g., EBV, CMV, HIV) using optimized local protein alignment (`blastp-short`).

---

## Features

* **Criss-Cross Matrix Processing**: Automatically pairs and executes parallelized homology scans across all detected virus references and cancer target cohorts (e.g., TCGA cohorts, private RNA-Seq files).
* **Strict Quality Filtration**: Auto-Filter non-accepted formats in the files such as the names, CDR3s, amino acids, etc.
* **Randomized Trial Execution**: Based on "k" number of samples which pulls "n" number based on cross-matrix of viral CDR3 sample size or a baseline n-viral # set.
* **Optimized Parallel Execution**: Leverages multi-threading (`ThreadPoolExecutor`) and chunking to prevent loss of data during interruptions and memory drops. (Memory pooling)

ALL FILES ORGANIZED BASED ON TRIAL OR NON-TRIAL MODE.

---

## Directory Structure

```text
BRCA_LUAD/
│
├── Targets/
│   ├── Virus_Reference.xlsx        # Reference database containing viral TRB CDR3 sequences
│   ├── PAAD_Target.xlsx            # Cancer target repertoire dataset 1
│   └── LGG_Target.xlsx             # Cancer target repertoire dataset 2
│
├── Total_Homology/
│   └── HIT_TABLES/                 # Structured output directory
│       ├── EBV/
│       │   ├── Non-Trial/
│       │   └── Trial/              # Trial_1, Trial_2, Trial_3 outputs
│       └── CMV/
│           ├── Non-Trial/
│           └── Trial/
│
└── execution.py                    # Main pipeline execution script
```

## Execution Method:
1. Clean all necessary files and upload all INPUT files into /Targets.
 - Viral Reference File: MUST BE NAMED "Virus_Reference.xlsx". The script auto-organizes and parses columns to find all valid viral reference CDR3s. 
     - Column headers must match pattern: "CDR3, TRB [Virus Name], removed duplicates" (case sensitive)
     - if not followed, that column of CDR3s are blocked and ignored from proceeding.
 - Cancer Target Files: MUST HAVE NAME FORMAT "[Cancer Appreviated/Prefix]_Target.xlsx"
     - Must have a "Case ID" column; if missing the script defaults to using the first column as a key worker.
     - Column must have string "cdr3" anywhere in the name.
 - Strict Validation rules for the sequencing on pBLAST:
     - Accepted Formats: Standard IUPAC 20-letter amino acid sequences using uppercase or lowercase characters
     - Rejected / Skipped Formats:
        - Any sequence containing non-standard biological characters such as tildes (~), asterisks (* representing stop codons), spaces, or numeric digits.
        - Empty fields, null entries, or literal "NAN" values.
        - Behavior: The pipeline automatically discards invalid rows in memory before execution without permanently modifying your raw source Excel files.
2. PRE-EXECUTION/before script fires:
 - Install all required packages on Python: ```pip install pandas openpyxl```
 - Verify NCBI BLAST+ binaries (```blastp.exe``` and ```makeblastdb.exe```) installed on system (the script defaults to looking into the directory: ```C:\Program Files\NCBI\blast-2.17.0+\bin```)
 - Open ```execution.py``` and adjust the configuration block at the TOP to match desired params.
3. Execute ```execution.py``` and wait. (run ```python execution.py``` if on terminal)
 - Note, script *will* check existing files within structured ```HIT_TABLES/[Virus Name]/[Mode (trial/non-trial)]/``` directories before execution. 
    - if complete hit tables already exist for given viral-cancer combiation, script skips BLAST execution on that pair (prints skip).
    - If interrupted, you can re-run the script and it boots to that section (note partial blocks are NOT saved for corruption prevention).
4. See your results live as they print and in the end:
 - Folder naming: Auto-named after Virus Name codename from the Reference Sheet (```Virus_Reference```, example: EBV).
    - Inside each folder of virus contains the ```Non-Trial``` and ```Trial``` modes respectively.
 - CSV naming scheme:
    - Non-Trial Mode: ```{CancerName}_HIT_TABLE_part{N}.csv``` (eg., ```PAAD_Trial_1_HIT_TABLE_part1.csv```)
    - Trial Mode: ```{CancerName}_Trial_{TrialNum}_HIT_TABLE_part{N}.csv``` (eg.,```PAAD_Trial_1_HIT_TABLE_part1.csv```)
