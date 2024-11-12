# OpenFOAM-on-GenomeDK 

# Description
This tutorial describes how to 1) Prepare the necessary packages for code compilation, 2) Install the Intel HPC Toolkit, and 3) Install OpenFOAM. It will also touch on structuring a "run" bash script.

# Author

All the credit goes to [Sina Nozarian](https://scholar.google.com/citations?user=xxgnX4EAAAAJ&hl=en), a PhD student at the time of writing. The author only contributed to creating and documenting this GitHub repository to facilitate OpenFOAM use on GenomeDK.

# Target platform
The procedure has been rigorously tested and verified to be fully compatible with OpenFOAM v-2112 and v-2312, ensuring smooth integration and reliable performance with these specific releases.

# Prepare the necessary packages for code compilation 

#### 1. Download the Intel HPC Toolkit using the [link](https://www.intel.com/content/www/us/en/developer/tools/oneapi/hpc-toolkit-download.html). Choose `Linux` and `Offline Installer` as the operating system and distribution, respectively. 

#### 2. Place the installer in your project folder (e.g., DeiC-AU-XX-XXXXXXX).
  
#### 3. Install the Intel HPC Toolkit 

- Navigate to the project folder using the command

    `cd /faststorage/project/<Project Number>`
  
- Install the toolkit using the following command
  
    `sh ./l_HPCKit_p_2024.X.X.XXX_offline.sh -a --silent --eula accept`

- Source it using

    `source ~/intel/oneapi/setvars.sh`
  
- Also run this command

    `export I_MPI_OFI_PROVIDER=verbs`

#### 4. Download OpenFOAM and place the directories in your home address in the _OF_ folder. 

#### 5. Go to ` ~/OF/OpenFOAM-vXXXX/etc/bashrc` and replace `export WM_MPLIB=SYSTEMOPENMPI` with `export WM_MPLIB=INTELMPI`.

#### 6. Source _setvars.sh_, export _I_MPI_OFI_PROVIDER_, and source _bashrc_. Then, go to the OpenFOAM directory through `foam`, and execute `./Allwmake -j <number of CPUs> -s -q -l`.

#### 7. Check if OpenFOAM works by executing solver names and utilities. If not known, you may consider recompiling them.

# Structure bash script
Here is a simple example of a "run" bash script you can use to run the code (Notice the use of "mpiexec" for running the code).

```
#!/bin/bash
#SBATCH --job-name=test_run
#SBATCH --ntasks=32
#SBATCH --ntasks-per-core=1
#SBATCH --mincpus=32
#SBATCH --time=120:00:00
#SBATCH --mem=10gb  
#SBATCH --account=DeiC-AU-XX-XXXXXXX
#SBATCH -p normal

source /home/User/intel/oneapi/setvars.sh --force
export I_MPI_OFI_PROVIDER=verbs
source ~/OF/OpenFOAM-vXXXX/etc/bashrc

cp -R 0.orig 0
touch sim.foam
blockMesh
decomposePar -force 
mpiexec -n $SLURM_NTASKS simpleFoam -parallel 
```
