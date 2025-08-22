# Running Demux and DADA2 
We also provide additional images for inferring species from 16S data using [DADA2](https://benjjneb.github.io/dada2/). The inputs are identical to those for demux-only, so please refer to the [main readme](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md) 
for instructions on formulating inputs and designing/specifying indexes. Installation, testing, and running with the Docker/Singularity images for the demux and dada2 process is similar to that for the demux-only images, but is repeated below with a few key changes. 


**Contents:**
* [Databases:]()
* [Using Docker](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#using-singularityapptainer)
* [Building with Singularity/Apptainer](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#building-with-singularityapptainer)
* [Using Singularity](https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/README.md#inputs)

## Databases:
The following databases are included in the images, but others can be added:
* GreenGenes ('gg_13_8_train_set_97.fa.gz')
* Silva 1? ('SILVA_138_SSURef_NR99_05_01_20_opt.arb.gz')
* Silva 2? ('SILVA_138_SSURef_NR99_tax_silva.fasta.gz')
* Silva 3? ('silva_nr_v132_train_set.fa.gz')
* 
By default, the [GreenGenes database](https://greengenes2.ucsd.edu/) is run, but this can be changed within 'workflow/scripts/summary_taxa.R' in line 14.




## Using Docker
To analyze our code to analyze dual-indexed sequencing data, first ensure that Docker is installed. Docker is a handy platform for sharing not only code but also coding environments, which enables easy code sharing without hours of installing dependencies. You can find instructions on Docker installation here {https://docs.docker.com/engine/install/}. Once Docker is installed, open the Docker desktop application or run ‘systemctl start docker’ to launch the Docker Daemon. 


1. **Installation and setting up the Docker image** \
  There are two ways to acquire the docker image needed for analysis, either by pulling directly from docker hub (the easiest approach) or using the dockerfile and other inputs provided in 'XXX.zip' to build the docker image.\
  **Pulling image:** \
  Launch Docker. Then in a terminal, run `docker pull rlporter24/XXXXXX:1.0`. This will make a local copy of the Docker image 'rlporter24/XXXXXX, which contains the code and environment needed to process the data, as well as example input files for running a test.\
  **Building image:**\
  Instead of pulling the docker image from the Docker hub, the docker image can also be built from the Dockerfile. This process takes longer and is not recommended. If you choose to build from the Dockerfile, follow the instructions provided on the main readme, using the input file XXX.zip.\


3. **Run the test analysis**
  To check that the set up was successful, run a quick analysis using provided test data. Start by running\
  `docker run -it rlporter24/XXXXX:1.0`\
  to open a container from the image rlporter24/dualindex-demux in an interactive mode (specified by the flags -it). If you built your own image, replace 'rlporter24/dualindex-demux' with the  '{name}:{version}' you provided for the build. In this mode, we can enter a series of commands, step by step within this container. All of the necessary input files are already included within the container, so no files need to be imported. To run the test, navigate to the 16S-demux-edits directory and edit 'Snakefile’ so that line 4 reads:\
  `configfile: "config/test_config.yaml"`\
  rather than:\
  `configfile: "config/config.yaml"`\
  If line 4 already reads\
  `configfile: “config/test_config.yaml”`\
  then no edit is necessary. Next, run:\
  `snakemake --cores 1`\
   replacing 1 with the desired number of cores. This should take about XXXXX minutes or fewer, and will run a test analysis using ‘config/test_fastq.txt’, ‘config/test_samplesheet.txt’ and test files included in /fastq_data/test/. The output files will be generated in the ‘workflow/test_out/’ directory. If the run is successful, the following outputs should be generated in ‘workflow/test_out/trimmed’:\
 <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/testSuccessOutputs.png?raw=true" alt="Alt Text" width="400" height="1000">\
  REPLACE IMAGE!!!\
  If these files are generated in 'workflow/test_out/' the run has been successful! 
  To exit the container, simply type ‘exit’, or stay in the container to run the actual analysis.

5. **Run the actual analysis**\
   If you do not already have an open container, launch an interactive container by running:\
  `docker run -it rlporter24/XXXXXX:1.0` .\
  If you built your own image, replace 'rlporter24/dualindex-demux:1.0' with the  '{name}:{version}' you provided for the build. Next, we can move in the input files (The essential inputs will be the raw fastq data, a fastq file list, and a samplesheet. Details on creating these inputs for your actual analysis are included in the main readme). Transfering input files can be done from inside or outside of the container, as described here {https://docs.docker.com/reference/cli/docker/container/cp/}. From outside of the container (in a separate terminal), run:\
  `docker cp {local_path} {CONTAINER}:{container_path}`\
  The {local_path} can be to an individual file or a directory. The {CONTAINER} should be the container name, not the image name. The container name can be found by running 
  `docker container ls` to list all the current containers, or by looking at the containers in the docker decktop GUI. The {container_path} should be provided relative to the '16s-demux' directory, which is the home directory within the container.\  
  Navigate to the 16S-demux directory and if necessary, edit line 4 so that it reads:\
  `configfile: “config/config.yaml”`\
  rather than\
  `configfile: “config/test_config.yaml”`\
  In the ‘config’ directory, update ‘config.yaml’ so that `samplesheet:` and `fastqlist:` in lines 2 and 4 are followed by the paths to your input samplesheet and fastqlist, respectively (more details in the __Inputs__ section). If you are using custom primers or indexes, you may need to adjust the input file for ‘indicies:’ or the lengths of read1 and read2 indexes and primers (lines 3, 5, 6, 7, 8). For more details on custom primers, see the __Custom Primers__ section.\
  Once the inputs and paths are updated, run:\
  `snakemake --cores 1`\
  within the 16S-demux directory (replace 1 with the desired number of cores). The analysis time will vary with the number and size of input files as well as the machine used. If the run is successful, a message similar to that below should be output:\
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/snakemakeSuccessOutput.png?raw=true" alt="Alt Text" width="750" height="400">\
  __Note:__ Before starting the actual run, you can use the command ‘snakemake -n’ to do a dry run. This is helpful for ensuring that names and file locations are correct before starting the full run. 
7. **Transfer Output Files Out**\
  Once the run is completed, output files must be transfered out of the container. To move files located in /16s-demux/workflow/out/ to a local destination, run the following:\
  `docker cp {CONTAINER:/16s-demux/workflow/out/} {local_path}`.\
8. **Clean Up**\
  After the run is completed and outputs have been moved out and saved, exit from the container. The remnants of the container created can be removed with the following code:\
  `docker rm  {container name}`\
  This will not remove the image - only the container that was just now built using this image. Be sure any important data or intermediates are transferred out of the container before removing it.\



## Building with Singularity/Apptainer:
To build a singularity/Apptainer image, ensure the necessary files are arranged properly and build from the '.def' file:

1. All necessary files are included in ‘demux.zip’ in **location**. Download and unzip the file. The following file structure should be created:
   <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/sing_demux_fileStructure.png?raw=true" alt="Alt Text" width="400" height="400">\
  REPLACE IMAGE!\
  ‘16s-demux.def’ and ‘requirements.txt’ are both needed for the building process, and the fastq_data contains example data for the test. All of the code is contained within the ‘16S-demux’ directory, and outputs will be generated there as well.
  __Note:__ The following step will likely require more resources than are available on a HPC login node, so be sure you are on a compute node or use a job manager to allocate resources.
3. Move to the directory containing the def file (16s-demux.def) and run the following:
  `singularity build demux-image.sif 16s-demux.def`
  The image will be created locally and a file ‘demux-image.sif’ will be created in the current working directory. The build should take less than 10 minutes, and if it is completed successfully, the the final output will look something like this:
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/buildSuccessOutput.png?raw=true" alt="Alt Text" width="750" height="400">\
  REPLACE IMAGE!\
  Once completed, the file 'demux-image.sif' should be found in the current working directory.

5. Now that the image has been built, you can run the demultiplexing analysis within it as described in ‘> Using Singularity/Apptainer’ using the image ‘demux-image.sif’. 




## Using Singularity/Apptainer
Most HPC are not compatible with Docker use, but do support Singularity/Apptainer. To run using Apptainer, follow these steps:

1. **Setting up the Singularity/Apptainer image**\
   First, ensure that Singularity/Apptainer is installed. The pre-built Singularity image is not currently available for direct download, but the Singularity image can be built locally fairly quickly ( ~10 minutes) and only requires that some specific files be made available locally. Instructions are included in the ‘Building Images’ section below. Once the build is successfully completed, you should have a file named ‘both.sif’ in the directory from which it was built.

3. **Open a shell in the container**\
   To open a shell, run the following command from the directory containing the ‘both.sif’:\
   `singularity shell both.sif` \
   This will bring you to a shell within the container where you can interactively run processes.\


5. **Run the test analysis**\
   To check that the set up was successful, run a quick analysis using provided test data. To do this, edit the Snakefile so that line 4 reads:\
  `configfile: "config/test_config.yaml"`\
  rather than:\
  `configfile: "config/config.yaml"`\
  Then run:\
  `snakemake --cores 1`
  within the ‘16S_demux_edits’ directory, replacing 1 with the desired number of cores. This should take about XX minutes depending on your system, and will run a test analysis using ‘config/test_fastq.txt’,   ‘config/test_samplesheet.txt’ and the test files included in /fastq_data/test/. The output files will be generated in the ‘workflow/test_out/’ directory. If the run is successful, the following outputs should be generated within the test_out/trimmed directory:\
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/testSuccessOutputs.png?raw=true" alt="Alt Text" width="400" height="1000">\
  REPLACE IMAGE!\
  If the files are generated in the ‘XXX’ directory are all present, the run has been successful.\
  To exit the container, simply type ‘exit’.

7. **Run the actual analysis**\
  To run the actual analysis, ensure your input files are accurate and located in the correct directory (see >Inputs for details), and ensure that the paths within ‘config/config.yaml’ are correct. Next edit the Snakefile so that line 4 reads:\
  `config/config.yaml`\
  rather than:\
  `configfile: "config/test_config.yaml`.\
  If you aren’t using a job manager like slurm, you can use the command:\
  `snakemake --cores 1`\
  in the 16S-demux directory to start the analysis (Replace 1 with the desired number of cores). If you are using a job manager, you will need to submit a job to run this command. An example script “submitSnakemake.sh” is included, which will launch an interactive shell within the container and run the entire snakemake pipeline.\
  __Note:__ To use this submission script, be sure to change the SBATCH parameters at the top as needed, and ensure that the image name in the line:\
`singularity shell ../both.sif`\
  matches the image name from your build. The analysis time will vary with the number and size of input files. If the run is successful, a message similar to that below should be output:\
  <img src="https://github.com/rlporter24/Amplicon-dual-index-demux/blob/main/images/snakemakeSuccessOutput.png?raw=true" alt="Alt Text" width="750" height="400">\
  __Note:__ Before starting the actual run, you can use the command `snakemake -n` to do a dry run. This is helpful for ensuring that names and file locations are correct before starting the full run.






