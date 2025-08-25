![header](imgs/header.jpg)

>PROIA: An Educational Project 📝<
This project, named PROIA, is an educational initiative that acknowledges the original work and intellectual property of Google DeepMind and its dedicated employees. It does not intend to plagiarate or steal anyone's work.
What is AlphaFold? 🧬
AlphaFold is a software package that provides an implementation of the inference pipeline for AlphaFold v2, a highly accurate model for predicting protein structures. This document also covers AlphaFold-Multimer, a version for predicting protein complexes.
Key Features:
 * Protein Structure Prediction: Predicts the 3D structure of proteins.
 * Multimer Support: Predicts the structure of complexes with multiple protein chains.
 * Model Options: Includes various models like monomer (original), monomer_ptm (with confidence scores), and multimer.
 * Confidence Metrics: Provides scores like pLDDT and predicted aligned error (PAE) to measure the accuracy of predictions.
 * Output: Generates PDB files, JSON files with metrics, and other data related to the prediction.
Installation and Setup 💻
To run AlphaFold, you need a machine with Linux, a modern NVIDIA GPU, and substantial disk space (up to 3 TB for databases).
 * Install Prerequisites:
   * Install Docker and the NVIDIA Container Toolkit.
   * Set up Docker to run as a non-root user.
   * Install aria2c for downloading databases.
 * Get the Code and Data:
   * Clone the repository:
     git clone https://github.com/deepmind/alphafold.git
cd ./alphafold

   * Download the genetic databases and model parameters. This can take a significant amount of time:
     scripts/download_all_data.sh <DOWNLOAD_DIR>

 * Build the Docker Image:
   * Build the Docker image with the provided Dockerfile:
     docker build -f docker/Dockerfile -t alphafold .

 * Install Python Dependencies:
   * Install the dependencies required to run the main script:
     pip3 install -r docker/requirements.txt

How to Use AlphaFold with Python 🐍
The simplest way to run AlphaFold is with the run_docker.py script. Here are some common use cases with Python examples.
1. Folding a Monomer
This is for predicting the structure of a single protein chain. Create a FASTA file (e.g., monomer.fasta) with your sequence.
>sequence_name

Then, use the following Python command to run the prediction:
python3 docker/run_docker.py \
--fasta_paths=monomer.fasta \
--max_template_date=2021-11-01 \
--model_preset=monomer \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

2. Folding a Homomer
For a protein complex with multiple identical chains (e.g., a trimer). The FASTA file (homomer.fasta) will have the same sequence repeated for each chain.
>sequence_1
>sequence_2
>sequence_3

Then, run the prediction using the multimer model preset:
python3 docker/run_docker.py \
--fasta_paths=homomer.fasta \
--max_template_date=2021-11-01 \
--model_preset=multimer \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

3. Folding a Heteromer
For a protein complex with different chains (e.g., an A2B3 heteromer). The FASTA file (heteromer.fasta) will contain the sequences for each unique chain, listed in the correct order.
>sequence_1
>sequence_2
>sequence_3
>sequence_4
>sequence_5

Run the command using the multimer model preset:
python3 docker/run_docker.py \
--fasta_paths=heteromer.fasta \
--max_template_date=2021-11-01 \
--model_preset=multimer \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

4. Folding Multiple Proteins
You can fold multiple proteins one after another by passing a comma-separated list of FASTA files.
python3 docker/run_docker.py \
--fasta_paths=monomer1.fasta,monomer2.fasta \
--max_template_date=2021-11-01 \
--model_preset=monomer \
--data_dir=$DOWNLOAD_DIR \
--output_dir=/home/user/absolute_path_to_the_output_dir

Output and Interpretation 📊
The results will be saved in a subdirectory within your --output_dir. . The key files include:
 * ranked_*.pdb: Predicted structures ordered by confidence (pLDDT score).
 * ranking_debug.json: A JSON file with the pLDDT scores for each model.
 * timings.json: A JSON file showing the time taken for each step of the pipeline.
 * relaxed_model_*.pdb: The predicted structure after an Amber relaxation step to improve local geometry.
The pLDDT score is stored in the B-factor field of the output PDB files, with a higher value indicating greater confidence.
