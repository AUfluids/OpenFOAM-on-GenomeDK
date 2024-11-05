# OpenFOAM-on-Genome

[![Paper: Author](https://img.shields.io/badge/Author-green.svg)](https://sites.google.com/view/zehtabiyan/home)

# Description
This tutorial describes how to 1) Prepare the necessary packages for code compilation, 2) Install the Intel HPC Toolkit , and 3) Install OpenFOAM. It will also touch on how to structure a "run" bash script.

# Target platform
The procedure has been rigorously tested and verified to be fully compatible with OpenFOAM v-2112 and v-2312, ensuring its smooth integration and reliable performance with this specific release.

# Author
[Navid Zehtabiyan-Rezaie](https://sites.google.com/view/zehtabiyan/home)

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


#### 4. Download OpenFOAM and place the directories in your home address in _OF_ folder. 

#### 5. Go to ` ~/OF/OpenFOAM-v2112/etc/bashrc` and replace `export WM_MPLIB=SYSTEMOPENMPI` with `export WM_MPLIB=INTELMPI`.

#### 6. Source _setvars.sh_, export _I_MPI_OFI_PROVIDER_, and source _bashrc_. The, go to OpenFOAM directory through `foam`, and execute `./Allwmake -j <number of CPUs> -s -q -l`.

#### 7. Check if OpenFOAM works by executing solver names and utilities. If not known, you may consider compiling them again.

# Structure bash script
Here is an example of a "run" bash script you can use to run the code (Notice the use of "mpiexec" for running the code).

```
#!/bin/bash
#SBATCH --job-name=test_run
#SBATCH --ntasks=32
#SBATCH --ntasks-per-core=1
#SBATCH --mincpus=32
#SBATCH --time=120:00:00
#SBATCH --mem=10gb  
#SBATCH --account=DeiC-AU-N2-2023009
#SBATCH -p normal
##SBATCH --nodes=1

source /home/nzr/intel/oneapi/setvars.sh --force
export I_MPI_OFI_PROVIDER=verbs
source ~/OF/OpenFOAM-v2112/etc/bashrc


rm -rf constant/extendedFeatureEdgeMesh > /dev/null 2>&1
rm -f constant/triSurface/rotatingZone.eMesh > /dev/null 2>&1
rm -f constant/triSurface/fixed.eMesh > /dev/null 2>&1
cp -R 0.orig 0
touch foam.foam
blockMesh
checkMesh
topoSet
decomposePar -force 
mpiexec -n $SLURM_NTASKS simpleFoam -parallel 
reconstructPar -latestTime

postProcess -func linesample -latestTime
mv log.postProcess log.postProcesslineSample
postProcess -funcs '(writeCellCentres)'
mv log.postProcess log.postProcesswriteCellCentres
postProcess -func surfaces -latestTime
mv log.postProcess log.postProcesssurfaces
foamToVTK -cellSet actuationDisk1CellSet

rm -r process*
```
