# Running CLIMBER-X on the HPC2015 at PIK

Here you can find the basic information and steps needed to get **CLIMBER-X** running on the (old) HPC2015  at PIK.

## Get the code

### CLIMBER-X climate model 

```bash

### Download the CLIMBER-X code ###

# Clone repository
git clone git@github.com:cxesmc/climber-x.git

# Enter directory 
cd climber-x

# Clone input file directory
git clone git@gitlab.pik-potsdam.de:cxesmc/climber-x-input.git input

# Run configuration script
python config.py config/pik_ifort

### Download and configure additional libraries ###

# fftw
cd src/utils/
wget https://www.fftw.org/fftw-3.3.10.tar.gz
tar -xvf fftw-3.3.10.tar.gz
rm fftw-3.3.10.tar.gz
mv fftw-3.3.10 fftw
cd fftw
./configure --prefix=$PWD --enable-openmp CC=icc F77=ifort 
make
make install
cd ../../..  # Return to climber-x parent directory

# coordinates
cd src/utils/
git clone git@github.com:cxesmc/coordinates.git
cd coordinates
python config.py config/pik_ifort 
cd ../../..  # Return to climber-x parent directory

### Compile and run ###

# Compile the climate model 
make cleanall
make climber-clim

# Run a pre-industrial equilibrium climate-only test simulation
./job_climber -s -f -o output/clim -c short -j parallel -n 16
```

### CLIMBER-X climate and carbon cycle model

If you would also like to run CLIMBER-X with an interactive carbon cycle, then the **HAMOCC** 
ocean biogeochemistry (`bgc`) code must also be downloaded:

```bash
# bgc
cd src/
git clone git@github.com:cxesmc/bgc.git
cd ..
```
Since the HAMOCC model source code is not open source, the `bgc` repository is private at the moment and 
you need to be given permission in order to access it. HAMOCC is covered by the Max Planck Institute for 
Meteorology software licence agreement as part of the MPI-ESM ([https://code.mpimet.mpg.de/attachments/download/26986/MPI-ESM_SLA_v3.4.pdf](https://code.mpimet.mpg.de/attachments/download/26986/MPI-ESM_SLA_v3.4.pdf)).
A pre-requisite to access the `bgc` repository is therefore that you agree to the MPI-ESM license
by following the steps outlined here: [https://code.mpimet.mpg.de/projects/mpi-esm-license](https://code.mpimet.mpg.de/projects/mpi-esm-license).
Once you have done so, send us (whom?) an email and you will be granted permission to access 
the `bgc` repository.
Note that you will need a GitHub account for that.

```bash
# Compile the climate and carbon cycle model 
make clean
make climber-clim-bgc

# Run a pre-industrial equilibrium simulation with ocean biogeochemistry
./job_climber -s -f -o output/clim-bgc -c short -j parallel -n 16 \&control="flag_bgc=T"
```

### CLIMBER-X climate and ice sheet model

If you would also like to run with an interactive ice sheet, then the `lis`
library must be installed, the **Yelmo** ice-sheet code must be downloaded and configured 
and the solid Earth model **VILMA** libraries must be downloaded before compiling:

```bash
# lis
cd src/utils
git clone git@github.com:anishida/lis.git lis-2.1.5
cd lis-2.1.5
./configure --prefix=$PWD/../lis --enable-omp --enable-f90 CC=icc FC=ifort 
make
make install
cd ../../..

# yelmo
cd src
git clone git@github.com:palma-ice/yelmo.git
cd yelmo
git checkout climber-x  # Get climber-x branch
python config.py config/pik_ifort
cd ../..

# vilma
cd src/
git clone git@github.com:cxesmc/vilma.git  # private repository, premission needed
cd ..

# Compile the climate and ice sheet model
make clean
make climber-clim-ice

# Run pre-industrial equilibrium simulation with interactive Greenland ice sheet
./job_climber -s -f -o output/clim-ice -c short -j parallel -n 16 \&control="flag_ice=T flag_geo=T flag_smb=T flag_imo=T ice_model_name=yelmo ice_domain_name=GRL-16KM"
```

### Fully coupled CLIMBER-X configuration

If you have followed all steps above you will also be ready to run fully coupled simulations:

```bash
# Compile the fully coupled model
make clean
make climber-clim-bgc-ice  # or equivalently make climber

# Run pre-industrial equilibrium simulation with ocean biogeochemistry and interactive Greenland ice sheet
./job_climber -s -f -o output/clim-bgc-ice -c short -j parallel -n 16 \&control="flag_bgc=T flag_ice=T flag_geo=T flag_smb=T flag_imo=T ice_model_name=yelmo ice_domain_name=GRL-16KM"
```

