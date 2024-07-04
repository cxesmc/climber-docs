# Running CLIMBER-X on the HPC2024 (FOOTE) at PIK

Here you can find the basic information and steps needed to get **CLIMBER-X** running on the HPC2024 (FOOTE) at PIK.

## Loading required modules

The following modules have to be loaded in order to compile and run the model. 
For convenience you can also add those commands to your `.profile` file in your home directory.

```bash
    module purge
    module use /p/system/modulefiles/compiler \
               /p/system/modulefiles/gpu \
               /p/system/modulefiles/libraries \
               /p/system/modulefiles/parallel \
               /p/system/modulefiles/tools

    module load intel/oneAPI/2024.0.0
    module load netcdf-c/4.9.2
    module load netcdf-fortran-intel/4.6.1
    module load udunits/2.2.28
    module load json-c/0.17
    module load openssl/3.0.12
    module load curl/8.4.0
    module load expat/2.5.0
    module load ncview/2.1.10
    module load cdo/2.4.1
```

## Get the code

### CLIMBER-X climate model 

```bash
### Download the CLIMBER-X code ###

# Clone repository
git clone git@github.com:cxesmc/climber-x.git

# Enter directory and run configuration script
cd climber-x
python3 config.py config/pik_hpc2024_ifx

### Download and configure additional libraries ###

# fftw
cd src/utils/
wget https://www.fftw.org/fftw-3.3.10.tar.gz
tar -xvf fftw-3.3.10.tar.gz
rm fftw-3.3.10.tar.gz
mv fftw-3.3.10 fftw
cd fftw
./configure --prefix=$PWD --enable-openmp CC=icx F77=ifx 'FFLAGS=-Ofast -march=core-avx2 -mtune=core-avx2 -traceback' 'CFLAGS=-Ofast -march=core-avx2 -mtune=core-avx2 -traceback'
make
make install
cd ../../..  # Return to climber-x parent directory

# coordinates
cd src/utils/
git clone git@github.com:cxesmc/coordinates.git
cd coordinates
python3 config.py config/pik_hpc2024_ifx 
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
module load intel/oneAPI/2023.2.0 #(error when compiling with most recent intel OneAPI 2024.0)
./configure --prefix=$PWD/../lis --enable-omp --enable-f90 CC=icc FC=ifort 'FFLAGS=-Ofast -march=core-avx2 -mtune=core-avx2 -traceback' 'CFLAGS=-Ofast -march=core-avx2 -mtune=core-avx2 -traceback'
make
make install
module load intel/oneAPI/2024.0.0 #(to revert previous change)
cd ../../..

# yelmo
cd src
git clone git@github.com:palma-ice/yelmo.git
cd yelmo
git checkout climber-x  # Get climber-x branch
# in src/yelmo_defs.f90 set:  integer,  parameter :: wp = dp  # set working precision to double precision
python3 config.py config/pik_hpc2024_ifx
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
./job_climber -s -f -o output/climb-bgc-ice -c short -j parallel -n 16 \&control="flag_bgc=T flag_ice=T flag_geo=T flag_smb=T flag_imo=T ice_model_name=yelmo ice_domain_name=GRL-16KM"
```

