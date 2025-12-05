# dDocent_demux
version 1.05
## Illumina Demultiplexing Script

A flexible Python script for demultiplexing Illumina paired-end sequencing reads based on header indices (i7/i5) and/or inline barcodes with configurable mismatch tolerance.

## Features

- **Auto-detection of demultiplexing mode**: Automatically detects whether to use header indices, inline barcodes, or both based on input file format
- **Mismatch tolerance**: Configurable Hamming distance threshold for fuzzy matching
- **Barcode offset detection**: Automatically checks inline barcodes at position 0 and position 1 to handle extra leading bases
- **Automatic barcode trimming**: Removes inline barcodes from output sequences (optional)
- **Progress tracking**: Real-time progress updates with processing rate
- **Comprehensive logging**: Automatic log file generation with detailed statistics
- **Debug mode**: Optional diagnostic output showing first 100 reads for troubleshooting
- **Unmatched read tracking**: Reports top 10 most common non-matched barcode/index combinations

## Requirements

- Python 3.6+
- No external dependencies (uses only Python standard library)

## Installation

Simply download the script and make it executable:
`chmod +x dDocent_demux`

## Quick Start
`dDocent_demux -r1 reads_R1.fastq.gz -r2 reads_R2.fastq.gz -s samples.txt -o output_prefix -m 1`

## Sample File Format

The script automatically detects the demultiplexing mode based on column headers in your tab-delimited sample file.

### Header Indices Only
```
Sample  i7  i5
Sample1  AGGCTATA  GAGATTCC
Sample2  TCCGGAGA  CTCTCTAT
```


### Inline Barcodes Only
```
Sample  Barcode_R1  Barcode_R2
Sample1  ATCACG  CGATGT
Sample2  TTAGGC  TGGCCA
```


### Both Header Indices and Inline Barcodes
```
Sample  i7  i5  Barcode_R1  Barcode_R2
Sample1  AGGCTATA  GAGATTCC  ATCACG  CGATGT
Sample2  TCCGGAGA  CTCTCTAT  TTAGGC  TGGCCA

```


**Notes:**
- File must be tab-delimited
- Column headers are case-insensitive
- Alternative barcode column names: `BC_R1`, `BC_R2`, `Barcode_F`, `Barcode_R`

## Usage

### Basic Usage
`dDocent_demux -r1 <R1.fastq.gz> -r2 <R2.fastq.gz> -s <samples.txt> -o <prefix>`

### Common Options
# Allow up to 2 mismatches
`dDocent_demux -r1 R1.fq.gz -r2 R2.fq.gz -s samples.txt -o output -m 2`

# Discard undetermined reads (don't save them)
`dDocent_demux -r1 R1.fq.gz -r2 R2.fq.gz -s samples.txt -o output --discard-undetermined`

# Keep inline barcodes in output (don't trim)
`dDocent_demux -r1 R1.fq.gz -r2 R2.fq.gz -s samples.txt -o output --no-trim`

# Enable debug mode (check first 100 reads)
`dDocent_demux -r1 R1.fq.gz -r2 R2.fq.gz -s samples.txt -o output --debug`


## Command Line Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `-r1, --read1` | Yes | R1 FASTQ file (gzipped or uncompressed) |
| `-r2, --read2` | Yes | R2 FASTQ file (gzipped or uncompressed) |
| `-s, --samples` | Yes | Sample file with indices/barcodes |
| `-o, --output` | Yes | Output prefix for undetermined reads and log file |
| `-m, --mismatches` | No | Maximum mismatches allowed (default: 1) |
| `--no-trim` | No | Do not trim inline barcodes from output |
| `--discard-undetermined` | No | Discard undetermined reads instead of saving |
| `--debug` | No | Enable debug mode (shows first 100 reads) |

## Output Files

### Per-Sample Files

- `{SampleName}.F.fastq.gz` - Forward reads for each sample
- `{SampleName}.R.fastq.gz` - Reverse reads for each sample

### Undetermined Reads (if not discarded)

- `{prefix}_undetermined.F.fastq.gz` - Unmatched forward reads
- `{prefix}_undetermined.R.fastq.gz` - Unmatched reverse reads

### Log File

- `{prefix}_demultiplex_{timestamp}.log` - Detailed run statistics and diagnostics

## Output Statistics

The log file includes:

1. **Demultiplexing mode** detected
2. **Sample combinations** loaded from input file
3. **Barcode offset statistics** (when using inline barcodes)
4. **Per-sample read counts** and percentages
5. **Top 10 non-matched combinations** for troubleshooting

### Example Log Output
```
==================================================

Total reads processed: 1,234,567

Total time: 5.2m

Average rate: 3,950 reads/sec

Barcode offset statistics:

Position 0: 1,100,000 reads (89.09%)

Position 1: 134,567 reads (10.91%)

Per-sample counts:

Sample1             :    500,000 ( 40.49%)

Sample2             :    600,000 ( 48.59%)

undetermined        :    134,567 ( 10.91%)
```


## How It Works

### Mismatch Tolerance

The script uses Hamming distance to allow a configurable number of mismatches:
- Compares observed indices/barcodes to expected values
- Allows up to `-m` total mismatches across all indices and barcodes
- Rejects ambiguous matches (reads that match multiple samples equally well)

### Barcode Offset Detection

When using inline barcodes, the script automatically checks two positions:
- **Position 0**: Barcode starts at the first base
- **Position 1**: Barcode starts at the second base (1bp offset)

This handles cases where library prep includes an extra leading base.

### Barcode Trimming

By default, inline barcodes are removed from output sequences:
- Adjusts for detected offset position
- Trims both sequence and quality strings
- Use `--no-trim` to keep barcodes in output

## Troubleshooting

### No Reads Matching

1. Enable debug mode to see first 100 reads:
`dDocent_demux ... --debug`

2. Check that indices in FASTQ headers match expected format
3. Verify barcode lengths match between sample file and actual reads
4. Check if index order is swapped (i7+i5 vs i5+i7)

### Low Match Rate

- Increase mismatch tolerance: `-m 2` or `-m 3`
- Check "Top 10 Non-Matched Combinations" in log file
- Verify sample file has correct index/barcode sequences

### High Undetermined Rate

- Review unmatched combinations in log file
- May indicate sequencing quality issues
- Check if additional samples are missing from sample file

## Performance

Typical processing rates:
- ~4,000-6,000 reads/second on standard hardware
- Gzipped input/output supported natively
- Memory usage: minimal (processes reads sequentially)

## Examples

### Example 1: Standard Dual-Index Demultiplexing
```
dDocent_demux \

-r1 Lane1_R1.fastq.gz \

-r2 Lane1_R2.fastq.gz \

-s indices.txt \

-o Lane1 \

-m 1
```

### Example 2: Inline Barcode Demultiplexing with Trimming
```
dDocent_demux \

-r1 reads_R1.fastq.gz \

-r2 reads_R2.fastq.gz \

-s barcodes.txt \

-o demux \

-m 2
```

### Example 3: Combined Indices and Barcodes

```
dDocent_demux \

-r1 data_R1.fastq.gz \

-r2 data_R2.fastq.gz \

-s combined_samples.txt \

-o output \

-m 1 \

--discard-undetermined
```

## Citation

If you use this script in your research, please cite this repository.


## Author

Jonathan Puritz, Puritz Lab of Marine Evolutionary Ecology, University of Rhode Island

## Support

For issues or questions, please submit an issue.
