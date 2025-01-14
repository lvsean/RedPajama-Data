### ArXiv

Follow these instructions to create the Arxiv dataset. These steps assume that you are in the `prep/arxiv` directory.

#### Setup

Install the dependencies specified in `arxiv_requirements.txt`:

```bash
pip install -r arxiv_requirements.txt
```

You also need to download a fasttext model for language identification and store it in the models directory:

```bash
mkdir -p models
wget https://dl.fbaipublicfiles.com/fasttext/supervised-models/lid.176.bin -P models
```

You also need an AWS account and have setup access keys to download the arxiv data.

Finally, setup the following folder structure (or create corresponding symlinks) to store data:

```bash
mkdir -p data/arxiv
mkdir ./work
mkdir -p logs/arxiv
```

#### Data Download

We download arxiv source files, available on S3 in the `arxiv` requester pays bucket. This means that the party sending
the request pays for the download. To find out more about the arxiv bulk data access check out
[this link](https://info.arxiv.org/help/bulk_data_s3.html). The downloading of arxiv data is implemented in the script
`run_download.py` and can be run in parallel using the the `arxiv_kickoff_download.sh` script in the scripts folder.

To get started, first write your AWS credentials to `aws_config.ini`:

```ini
[DEFAULT]
ACCESS_KEY = <access-key>
SECRET_KEY = <secret-key>
```

On a slurm cluster, you can download the files in parallel using:

```bash
bash scripts/arxiv-kickoff-download.sh
```

This will partition the the arxiv urls on S3 into 100 chunks and download them in parallel with 100 workers.

#### Data Preparation

The processing of arxiv data is implemented in the script `run_clean.py` and applies the following set of cleaning
operations:

- Remove everything before the first occurence of a section
- Remove everything after and including the bibliograhy
- Remove all comments (both at the and of a line and multiline)
- inline-expand user defined macros defined via `\newcommand` and `\def`, that have no arguments

Use the following script to run the cleaning in parallel on the downloaded files:

```bash
bash scripts/arxiv-kickoff-cleaning.sh
```

which submits a job-array to slurm with 100 jobs, each of which calls `run_clean.py` on the files in its input
partition.

#### Token Count

You can count the tokens generated by the cleaning process using the `token_count.py` script:

```bash
python token_count.py --data_dir <path-to-cleaned-data>
```

Alternatively, you can also submit a slurm job using the `arxiv-token-count-slurm.sbatch` script in the scripts folder.
Note that this uses the `EleutherAI/pythia-6.9b-deduped` tokenizer from Huggingface.
