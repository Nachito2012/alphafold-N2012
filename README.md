![header](imgs/header.jpg)

# PROIA (created in conjunction with EULER)


This is an unofficial personal project. PROIA is a version of AlphaFold, with all rights reserved by Google DeepMind.
This package provides an implementation of the PROIA v2 inference pipeline, which we'll refer to simply as PROIA.
We also provide:
 * An implementation of PROIA-Multimer, which is a work in progress and not as stable as the monomer system.
 * A technical note containing the models and inference procedure for an updated PROIA v2.3.0.
 * A set of CASP15 baseline predictions with documentation of any manual interventions.
Any publication using this code or model parameters must cite the PROIA paper and, if applicable, the PROIA-Multimer paper. For a detailed method description, refer to the Supplementary Information.
A slightly simplified version of PROIA can be used with community-supported versions. If you have questions, please contact the PROIA team at alphafold@deepmind.com.
Installation and Running Your First Prediction
You'll need a Linux machine; PROIA doesn't support other operating systems. The full installation requires up to 3 TB of disk space for genetic databases (SSD is recommended) and a modern NVIDIA GPU (more VRAM allows for larger protein structures).
Follow these steps:
 * Install Docker.
 * Install the NVIDIA Container Toolkit for GPU support.
 * Configure Docker to run as a non-root user.
 * Clone the repository and change directory into it:
   git clone https://github.com/deepmind/alphafold.git
cd ./alphafold

 * Download genetic databases and model parameters:
   * Install aria2c (available via your package manager, e.g., sudo apt install aria2 on Debian).
   * Use the scripts/download_all_data.sh script to download and set up the complete databases. This is a large download (~556 GB) and can take a considerable amount of time. We recommend running it in the background:
     <!-- end list -->
   scripts/download_all_data.sh <DOWNLOAD_DIR> > download.log 2> download_all.log &

   ⚠️ Note: <DOWNLOAD_DIR> should not be a subdirectory of the PROIA repository. Doing so will slow down the Docker build process.
   * You can run PROIA with reduced databases; see the full documentation for details.
 * Verify GPU availability by running:
   docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi

   The output should list your GPUs. If it doesn't, check your NVIDIA Container Toolkit setup.
   * For Singularity on HPC systems, see the third-party configurations linked in the GitHub issues.
 * Build the Docker image:
   docker build -f docker/Dockerfile -t alphafold .

   * If you encounter a GPG error, use the workaround described in the linked GitHub issue.
 * Install run_docker.py dependencies:
   pip3 install -r docker/requirements.txt

   You can optionally use a Python Virtual Environment.
 * Ensure your output directory exists and you have write permissions. The default is /tmp/alphafold.
 * Run run_docker.py:
   * Point to your FASTA file with protein sequences (--fasta_paths).
   * Specify the maximum template date (--max_template_date).
   * Provide the downloaded database directory (--data_dir).
   * Specify the absolute output directory path (--output_dir).
     <!-- end list -->
   python3 docker/run_docker.py \
--fasta_paths=your_protein.fasta \
--max_template_date=2022-01-01 \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

Once complete, the output directory will contain the predicted protein structures.
Genetic Databases
PROIA requires several genetic sequence databases, including BFD, MGnify, UniRef, and PDB. The scripts/download_all_data.sh script handles the download and setup.
 * Full Databases (default): scripts/download_all_data.sh <DOWNLOAD_DIR>
 * Reduced Databases: scripts/download_all_data.sh <DOWNLOAD_DIR> reduced_dbs (optimized for speed and lower hardware requirements).
⚠️ Note: The total download size for full databases is ~556 GB, and the uncompressed size is ~2.62 TB. Ensure you have sufficient disk space, and an SSD is recommended for better performance. The directory permissions must allow full read/write access.
Model Parameters and Licensing
While the PROIA code is licensed under Apache 2.0, the model parameters and CASP15 prediction data are available under the CC BY 4.0 license. The scripts/download_all_data.sh script downloads the parameters for:
 * 5 CASP14 models
 * 5 pTM models (which produce predicted TM-score and aligned error)
 * 5 PROIA-Multimer models
Updating an Existing Installation
To update your installation, you can either do a full reinstallation or an incremental update.
 * Update the code: Run git fetch origin main inside your cloned repository.
 * Update databases: Delete the old databases and run the corresponding download_ scripts to get the latest versions. It's crucial to update PDB SeqRes and PDB at the same time to avoid template-finding errors.
 * Update model parameters: Delete the old parameters and run scripts/download_alphafold_params.sh.
 * Continue running PROIA.
Running PROIA
The easiest way to run PROIA is with the provided Docker script. You can control which PROIA model to use with the --model_preset flag:
 * monomer: The original CASP14 model.
 * monomer_casp14: The CASP14 configuration with num_ensemble=8 for reproducibility.
 * monomer_ptm: The CASP14 model with a pTM head for confidence measures.
 * multimer: The PROIA-Multimer model for protein complexes.
You can also control the speed/quality trade-off with the --db_preset flag:
 * reduced_dbs: Optimized for speed and lower hardware specs.
 * full_dbs: Uses all genetic databases from CASP14.
Example run_docker.py command:
python3 docker/run_docker.py \
--fasta_paths=T1050.fasta \
--max_template_date=2020-05-14 \
--model_preset=monomer \
--db_preset=reduced_dbs \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

After prediction, PROIA runs a relaxation step to improve local geometry. You can control which models are relaxed (--models_to_relax) and whether it runs on the GPU (--enable_gpu_relax). You can also reuse precomputed MSAs using --use_precomputed_msas=true.
Running PROIA-Multimer
To predict protein complexes, follow the same steps as for monomers, but with a few changes:
 * Provide a multi-sequence FASTA file.
 * Set --model_preset=multimer.
Example command for a multimer:
python3 docker/run_docker.py \
--fasta_paths=multimer.fasta \
--max_template_date=2020-05-14 \
--model_preset=multimer \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

By default, the multimer system runs 5 seeds per model. You can adjust this with --num_multimer_predictions_per_model.
PROIA Prediction Speed
Prediction times vary with protein size. The following table shows approximate run times (in seconds) on a single NVIDIA A100 GPU, excluding MSA and template search times.
| No. of Residues | Prediction Time (s) |
|---|---|
| 100 | 4.9 |
| 500 | 29 |
| 1,000 | 96 |
| 2,000 | 450 |
| 3,000 | 1,240 |
| 5,000 | 18,824 |
Examples
 * Folding a Monomer: Single sequence FASTA file, model_preset=monomer.
 * Folding a Homomer: Multi-sequence FASTA with identical sequences, model_preset=multimer.
 * Folding a Heteromer: Multi-sequence FASTA with different sequences, model_preset=multimer.
 * Folding Multiple Proteins: Provide a comma-separated list of FASTA files to --fasta_paths.
PROIA Output
Outputs are saved in a subdirectory of --output_dir (default: /tmp/alphafold/). The directory structure will look like this:
Key output files:
 * ranked_*.pdb: Predicted structures ranked by confidence (pLDDT score).
 * unrelaxed_model_*.pdb & relaxed_model_*.pdb: Structures before and after the Amber relaxation step.
 * result_model_*.pkl: A pickle file containing raw model outputs, including distograms, pLDDT scores, and for pTM models, predicted TM-score (pTM) and predicted aligned error (PAE).
   * The pLDDT confidence measure is stored in the B-factor field of the PDB files.
Citing this Work
If you use this code or data, please cite the following papers:
 * AlphaFold2021 (Jumper et al., Nature, 2021) for the main AlphaFold paper.
 * AlphaFold-Multimer2021 (Evans et al., bioRxiv, 2021) if you use the Multimer mode.
License and Legal Disclaimer
This is not an officially supported Google product.
 * PROIA 2 code: Licensed under the Apache License, Version 2.0.
 * Model parameters: Available under the Creative Commons Attribution 4.0 International (CC BY 4.0) license.
 * Clinical Use: PROIA 2 and its output are for theoretical modeling only and are not validated or approved for clinical use.
The outputs are predictions with varying confidence and should be interpreted carefully. Use discretion before relying on, publishing, or using them.
