# Project 2: MD Simulations of TCF4 AlphaFold Predicted Structured Regions

## Motivation & Background

**Pitt-Hopkins Syndrome (PTHS)** is a rare neurodevelopmental disorder characterized by intellectual disability, developmental delay, distinct facial features, and autonomic nervous system dysfunction (such as hyperventilation followed by apnea). 

PTHS is caused by haploinsufficiency, point mutations, or deletions in the **TCF4** gene (Transcription Factor 4). Currently, there is no experimentally resolved structure via X-ray crystallography, NMR, or neutron diffraction. However, its sequence is known and found at UniProt (Universal Protein Resource), the world's leading, freely accessible database of protein sequence and functional information. From the UniProt Knowledgebase (UniProtKB), we can obtain the human TCF4 sequence here: [UniProtKB TCF4](https://rest.uniprot.org/uniprotkb/P15884.fasta)

In the absence of an experimentally resolved structure, we can use an AI Protein Structure Prediction Server:
1) [AlphaFold Server](https://alphafoldserver.com/)
2) [Protenix Server](https://protenix-server.com/login)

Using AlphaFold, we will select a structured piece of TCF4 as a starting point for our MD simulations.

---

## Prerequisites & Requirements
* An active account on the DEAC cluster.
* Access to a web browser (for CHARMM-GUI and the RCSB PDB).
* An SSH client / terminal to access the cluster.
* Access to class [DEAC OnDemand Dashboard](https://phy320.deac.wfu.edu/pun/sys/dashboard).
* A file transfer tool (e.g., `scp`, [FileZilla](https://filezilla-project.org/)) to upload/download files to the cluster.

---

## Part 1 Instructions

### Step 0: Create an account on Github and clone this repository
1. Go to the [Github](https://github.com/) website and create an account.
2. Use an SSH client / terminal to log into the DEAC cluster using your username (same as your email before @wfu.edu):
    ```bash
    ssh -Y [username]@artemis.deac.wfu.edu
3. Go to your directory and clone this repository:
    ```bash
    cd /deac/phy/classes/phy320-fa-2026/[username]
    git clone https://github.com/samcho/Project-2-PHY320620-Fall2026.git
    cd Project-2-PHY320620-Fall2026/

### Step 1: Select the Structured Section of TCF4 for MD Simulations
1. Using the enclosed Jupyter Notebook (TCF4 PDB Generator.ipynb)
2. Make a directory named "helix[num]" where [num] is replaced with your assigned helix number
3. Copy your helix to your directory. An example command is:
   ```bash
   mkdir helix1
   cp extracted_helices/helix_1*pdb helix 1

### Step 2: Prepare Input Files with CHARMM-GUI
1. Navigate to [CHARMM-GUI](https://www.charmm-gui.org/), register, and log in.
2. Select **Input Generator** $\rightarrow$ **Solution Builder**.
3. **PDB Option:** Upload your PDB file and click **Next**.
4. **Model/Chain Selection:** 
   * Select the TCF4 protein chain. (default)
5. **Solvation & Ions:**
   * Select a **Rectangular** water box. (default)
   * Set the edge distance (minimum 10 Å padding around the protein). (default)
   * Add ions $\text{KCl}$) to neutralize the system and set physiological concentration (e.g., $0.15\text{ M}$). (default)
6. **Input Generator Options:**
   * Select **CHARMM36m** as the force field (optimized for intrinsically disordered proteins). (default)
   * Under **MD Software**, select **NAMD**. (not default)
   * Set target temperature (e.g., $310.15\text{ K}$ for physiological conditions). (not default)
7. Click **Generate Input Files** and download the resulting archive (`charmm-gui.tgz` or `.zip`).

### Step 3: Transfer & Unpack Files on the Cluster
1. Transfer the archive to your cluster directory:
   ```bash
   scp charmm-gui.tgz username@artemis.deac.wfu.edu:/deac/phy/classes/phy320-fa-2026/[username]
2. Unpack files and create directories for 3 independent trajectories:
   ```bash
   ssh -Y username@artemis.deac.wfu.edu
   cd /deac/phy/classes/phy320-fa-2026/username/Project-2-PHY320620-Fall2026/
   tar -xvfz charmm-gui.tgz; mv charmm-gui-* 1
   tar -xvfz charmm-gui.tgz; mv charmm-gui-* 2
   tar -xvfz charmm-gui.tgz; mv charmm-gui-* 3

### 4. How to use the SLURM queuing system: Three main commands:

    sbatch -> submit to the queue.
    squeue -> check the status of the queue.
    scancel -> cancel a submission to the queue.

1. Copy the slurm scripts to your directory:
   ```bash
   cp ../../../md*.slurm .
2. To submit your md-equil.slurm script to the DEAC cluster, type “sbatch md-equil.slurm”. The check on whether everything is okay, type “squeue –u [username]” where [username] is your username. 
3. Wait until md-equil.slurm is finished by using the `squeue` command and verify by opening and checking the `step4_equilibration.out` file to see that the simulation continued until the end (look for "End of program" message at the end of the file). 
4. Submit the md-dyna.slurm script to the DEAC cluster. This should take about 5 days. Every 4 hours or so, your directory will be filled with about 1 ns of MD simulation. Check to make sure that each trajectory completed successfully. 
5. After the first md-dyna.slurm script is finished, use `nano` to edit lines 27 and 28 to reflect the next set of MD simulations. Re-submit the md-dyna.slurm script to the DEAC cluster, wait until it ends, and check to make sure that each trajectory completed successfully. Keep repeating this step until 250 ns of MD simulations has completed.