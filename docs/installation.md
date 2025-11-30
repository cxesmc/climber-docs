# Installing CLIMBER-X

Here you can find the basic information and steps needed to install **CLIMBER-X**.

There are currently four different flavors of **CLIMBER-X** that can be set up:

- `climber-clim`: minimal climate model configuration with atmosphere, ocean, sea ice and land (including dynamic vegetation)
- `climber-clim-bgc`: coupled climate-carbon cycle model configuration; clim plus with ocean biogeochemistry
- `climber-clim-ice`: coupled climate-ice sheet model configuration; clim plus with ice sheets
- `climber-clim-bgc-ice`: fully coupled model configuration; clim plus with ocean biogeochemistry and ice sheets

The model dependencies vary according to the desired model configuration:

- Dependencies are: NetCDF, coordinates, Python3.x, runner, CDO
- Additional dependencies if using coupled ice sheets are: Yelmo, LIS

See: [Dependencies](dependencies.md) for more details.

Follow the steps below to (1) obtain the code, (2) configure the Makefile(s) for your system,
(3) compile an executable program.

## 1. Get the code

Clone the repository from [https://github.com/cxesmc/climber-x](https://github.com/cxesmc/climber-x):

```bash
# Clone code repository
git clone https://github.com/cxesmc/climber-x.git
git clone git@github.com:cxesmc/climber-x.git # via ssh

cd climber-x

# Clone input file directory
git clone https://gitlab.pik-potsdam.de/cxesmc/climber-x-input.git input
git clone git@gitlab.pik-potsdam.de:cxesmc/climber-x-input.git input    # via ssh

```

If you plan to make changes to the code, it is wise to check out a new branch:

```bash
git checkout -b user-dev
```

You should now be working on the branch `user-dev`.

If you would also like to run CLIMBER-X with an interactive carbon cycle, then the **HAMOCC**
ocean biogeochemistry (`bgc`) code must also be downloaded:

```bash
# bgc
cd src/
git clone git@github.com:cxesmc/bgc.git
cd bgc/
git submodule update --init --recursive     # for submodule M4AGO 
cd ../..
```

Since the HAMOCC model code is not open source, the `bgc` repository is private at the moment and
you need to be given permission in order to access it. HAMOCC is covered by the Max Planck Institute for
Meteorology software licence agreement as part of the MPI-ESM ([https://code.mpimet.mpg.de/attachments/download/26986/MPI-ESM_SLA_v3.4.pdf](https://code.mpimet.mpg.de/attachments/download/26986/MPI-ESM_SLA_v3.4.pdf)).
A pre-requisite to access the `bgc` repository is therefore that you agree to the MPI-ESM license
by following the steps outlined here: [https://code.mpimet.mpg.de/projects/mpi-esm-license](https://code.mpimet.mpg.de/projects/mpi-esm-license).
Once you have done so, send an email to [Matteo Willeit](mailto:matteo.willeit@gmail.com?subject=[GitHub]%20bgc%20source%20code) and you will be granted permission to access the `bgc` repository.

If you would also like to run with an interactive ice sheet, the **Yelmo** ice-sheet code
must be downloaded and configured and the solid Earth model **VILMA** libraries must be
downloaded before compiling:

```bash
# yelmo
cd src
git clone git@github.com:palma-ice/yelmo.git
cd yelmo
git checkout climber-x                     # Get climber-x branch
cd ../..            # Return to climber-x parent directory

# vilma
cd src/
git clone git@github.com:cxesmc/vilma.git  # private repository, premission needed
cd ..
```

## 2. Create the system-specific Makefile

To compile **CLIMBER-X**, you need to generate a Makefile that is appropriate for your system. In the folder `config`, you need to specify a configuration file that defines the compiler and flags, including definition of the paths to the `NetCDF`, `FFTW`, `coordinates`, `Yelmo` and `LIS` libraries. Note that it can be convenient to install `FFTW`, `coordinates`, `Yelmo` and `LIS` as subdirectories of the `src/` folder, to be sure they are compiled consistently with **CLIMBER-X**.

You can use another configuration file in the config folder as a template, e.g.,

```bash
cd config
cp pik_ifort myhost_mycompiler
```

Then you would modify the file `myhost_mycompiler` to match your paths. Back in `climber-x`, you can then generate your Makefile with the provided python configuration script:

```bash
cd ../climber-x
python config.py config/myhost_mycompiler
```

The result should be a Makefile that is ready for use.

If you want to use the ice sheet model you have to similarly create the Makefile for Yelmo:

```bash
cd src/yelmo/
python config.py config/myhost_mycompiler 
cd ../..
```

## 3. Compiling **CLIMBER-X**

Assuming the source has been downloaded and configured, and all [dependencies](dependencies.md) have also been compiled, now you are ready to compile **CLIMBER-X**.

There are currently four different flavors of **CLIMBER-X** that can be compiled:

- `climber-clim`: minimal climate configuration with atmosphere, ocean, sea ice and land
- `climber-clim-bgc`: clim plus with ocean biogeochemistry
- `climber-clim-ice`: clim plus with ice sheets
- `climber-clim-bgc-ice`: clim plus with ocean biogeochemistry and ice sheets

These can be compiled by calling the individual names, e.g. `make climber-clim` or `make climber-clim-bgc-ice`. By default, it is also possible to call `make climber` as a shorter alias for `make climber-clim-bgc-ice`, e.g.:

```bash
make clean
make climber
```

The climate only version `climber-clim` corresponds to the version described by Willeit et al. (2022). This particular model setup does not require non-climate source code or the LIS library for compilation.

That's it. The executable `climber.x` should now be available in the main directory.

To compile the model with debug flags enabled use:

```bash
make climber debug=1
```

By default, the model is compiled with `openmp`. To compile the model without openmp use:

```bash
make climber openmp=0
```

This version should typically not be used.
