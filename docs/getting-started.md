# Getting started

Here you can find the basic information and steps needed to get **CLIMBER-X** running.

## Super-quick start

A summary of commands to get started is given below, assuming you are installing the PIK cluster using the `ifort` compiler. For more detailed information see subsequent sections.

```bash
### Download the climber code ###

# Clone repository
git clone git@github.com:cxesmc/climber-x.git

# Enter directory and run configuration script
cd climber-x
python config.py config/pik_ifort 

### Download and configure additional libraries ###

# fftw
cd src/utils/
wget https://www.fftw.org/fftw-3.3.10.tar.gz
tar -xvf fftw-3.3.10.tar.gz
rm fftw-3.3.10.tar.gz
cd fftw-3.3.10
./configure --prefix=$PWD --enable-openmp
make
make install
cd ..

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
./job_climber -s -o RUNDIR
```

Note, if you would also like to run with an interactive ice sheet, then the `lis`
library and the **Yelmo** ice-sheet code must also be downloaded and configured before compiling, as well as the :

```bash
# lis
cd src/utils/lis
git clone git@github.com:anishida/lis.git lis-2.1.5
cd lis-2.1.5
CC=icc FC=ifort ./configure --prefix=$PWD/../2.1.5 --enable-omp --enable-f90
make
make install
cd ..

# yelmo
cd src
git clone git@github.com:palma-ice/yelmo.git
cd yelmo
git checkout climber-x  # Get climber-x branch
python config.py config/pik_ifort
cd ..

# Compile the model
make cleanall
make climber-ice

# Run
./job_climber -s -o RUNDIR
```

## Dependencies

Dependencies are: NetCDF, FFTW, coordinates, LIS
Optional dependencies are: CDO, runner

See: [Dependencies](dependencies.md) for installation tips.

## Directory structure

```fortran
    config/
        Configuration files for compilation on different systems.
    input/
        Location of any input data needed by the model.
    output/
        Default location for model output.
    maps/
        Location of the maps that will be generated to map/interpolate between different grids.
    nml/
        Default parameter namelists that manage the model configuration.
    restart/
        Location of the restart files needed to continue previous model simulations.
    src/
        Source code for CLIMBER-X.
```

## Usage

Follow the steps below to (1) obtain the code, (2) configure the Makefile for your system,
(3) compile an executable program and (4) run a test simulation.

### 1. Get the code.

Clone the repository from [https://github.com/cxesmc/climber-x](https://github.com/cxesmc/climber-x):

```bash
# Clone repository
git clone https://github.com/cxesmc/climber-x.git
git clone git@github.com:cxesmc/climber-x.git # via ssh

cd climber-x
```

If you plan to make changes to the code, it is wise to check out a new branch:

```bash
git checkout -b user-dev
```

You should now be working on the branch `user-dev`.


### 2. Create the system-specific Makefile.

To compile **CLIMBER-X**, you need to generate a Makefile that is appropriate for your system. In the folder `config`, you need to specify a configuration file that defines the compiler and flags, including definition of the paths to the `NetCDF`, `FFTW`, `coordinates`, and `LIS`,  libraries. Note that it can be convenient to install `FFTW`, `coordinates` and `LIS` as subdirectories of the `src/` folder, to be sure they are compiled consistently with **CLIMBER-X**.

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

### 3. Compiling **CLIMBER-X**

Assuming the source has been downloaded and configured, and all [dependencies](dependencies.md) have also been compiled, now you are ready to compile **CLIMBER-X**:

```bash
make clean
make climber-clim
```

There are currently four different versions of **CLIMBER-X** that can be compiled:

- `climber-clim`: minimal configuration with ocn,atm,lnd,sic
- `climber-clim-bgc`: clim plus with bgc
- `climber-clim-ice`: clim plus with ice
- `climber-clim-bgc-ice`: clim plus with bgc and ice

These can be compiled by calling the individual names, e.g. `make climber-clim` or `make climber-clim-bgc-ice`. By default, it is also possible to call `make climber` as a shorter alias for `make climber-clim-bgc-ice`. 

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

### 4. Running CLIMBER-X

After **CLIMBER-X** has been compiled, several steps must be completed to run the executable `climber.x`:

1. Create a run directory (`RUNDIR`).
2. Copy namelist parameter files to `RUNDIR`.
3. Make links to the `input`, `maps` and `restart` directories in `RUNDIR`.
4. Copy `VILMA` restart files to `RUNDIR` (since these are eventually modified by `climber.x`).
5. Copy the executable file `climber.x` to `RUNDIR`.
6. To run on the cluster, create a job submission script (e.g. `job.submit`) in `RUNDIR` to manage various computing options (number of processors, etc).

When these steps are completed, `climber.x` should be run directly from `RUNDIR` using the job submission script. E.g., using SLURM:

```bash
cd RUNDIR
sbatch job.submit
```

If not using the cluster, **CLIMBER-X** can also be run manually by entering the `RUNDIR` and specifying the current directory as an argument:

```bash
cd RUNDIR
./climber.x ./ > out.out
```

Either of the above will run climber in `RUNDIR` with all simulation output stored in the same directory. Because the directory includes the executable and is self-contained, assuming the contents of the linked directories do not change, the simulation can be re-run at any time.

To perform all of the above steps manually for multiple simulations is very tedious and time-consuming, so a "job" script has been created to handle the process.

Currently two job-script methods are available (`job_climber` and `runcx`), which are described in the sections below.

#### Using `job_climber`

Aside from possible options, `job_climber` is always called with the required argument `-o RUNDIR` that specifies the output directory. So, to run a `climber.x` simulation as a job on the cluster in `RUNDIR`, run the command:

```bash
./job_climber -s -o RUNDIR
```

where `RUNDIR` is the desired run directory. The option `-s` specifies that the job should be run on the cluster. Alternatively, use `-r` instead of `-s` to simply run it as a background process. Other options include `-c` for the queue name (short, priority, etc), `-w` for the maximum wall clock time to allow in hours, `-j` to specify whether a serial or parallel job is desired (serial or parallel), `-n` to specify the number of processors, and others.

### Using `runcx`

Unlike `job_climber`, the newer script `runcx` was designed only to handle running one simulation, while leaving the task of ensemble generation to the Python module `runner`, developed for that task.

See `./runcx -h` for details on possible arguments. Aside from possible options, `runcx` is always called with the required argument `-o RUNDIR` that specifies the output directory. So, to run a `climber.x` simulation as a job on the cluster in `RUNDIR`, run the command:

```bash
./runcx -s -o RUNDIR
```

where `RUNDIR` is the desired run directory. The option `-s` specifies that the job should be run on the cluster. Alternatively, use `-r` instead of `-s` to simply run it as a background process. Other options include `-q, --qos` for the queue name (short, priority, etc), `-w, --wall` for the maximum wall clock time to allow in hours, `--part` to name the processor partition (haswell, broadwell, etc), `--omp` to specify the number of processors, and others. See `./runcx -h` for all options.

Note that various default options can be specified in the script's json-format configuration file `runcx.js`, so as to avoid having to specifying them every time.

Run as a command as above, this script will run a simulation using the parameters as they are specified in the namelist parameter files in the `nml` directory. In addition, it is possible to modify the parameters of one simulation at the command line using the argument `-p KEY=VAL KEY=VAL ...`. So, for example, the following command:

```bash
./runcx -s -o RUNDIR -p ctl.n_accel=10
```

will run `climber.x` on the cluster in the output directory `RUNDIR` with the control parameter `control.n_accel` set to `10`. Note that `ctl` is a convenient alias for the namelist group `control`, as defined in `runcx.js`.


To perform a simulation an ensemble of simulations with modified parameter values, `runcx` should be called via `jobrun` (see below).

#### Using `jobrun` with `runcx`

`jobrun` is a command that is part of the Python `runner` library, found here:
[https://github.com/alex-robinson/runner](https://github.com/alex-robinson/runner). This command facilitates running ensembles of simulations, or simulations with modified parameters via a convenient command-line interface. See the above `runner` page for its installation instructions.

using `jobrun`, the following command would produce the same simulation as `./runcx -s RUNDIR`:

```bash
jobrun ./runcx -s -- -o OUTDIR
```

The difference here is that all options that follow the `--` are `jobrun` options. So now we don't specify the specific `RUNDIR`, but rather an encapsulating `OUTDIR` that will contain one or many `RUNDIR`'s. In the above example, no parameters are changed, so the simulation is saved in the `default` directory: `OUTDIR/default`.

If we want to change a parameter, this can be done as with the `runcx` script via the `-p` option:

```bash
jobrun ./runcx -s -- -o OUTDIR -p ctl.n_accel=10
```

This will produce one simulation with the parameter `control.n_accel=10`. Since we have changed a parameter for this simulation, `jobrun` treats this as an ensemble, so the output is saved in `OUTDIR/0` for simulation 0. In short, the above command is equivalent to `./runcx -s -o OUTDIR -p ctl.n_accel=10`, but in the former case, the output is stored in `OUTDIR/0` and in the latter case, it is stored directly in `OUTDIR`.

The power of `jobrun` comes when we want to run an ensemble:

```bash
jobrun ./runcx -s -- -o OUTDIR -p ctl.n_accel=1,5,10
```

This ensemble of simulations will appear in `OUTDIR/0`, `OUTDIR/1` and `OUTDIR/2`, respectively.

A more informative output directory can be made using the option `-a` along with `-o`:

```bash
jobrun ./runcx -s -- -a -o OUTDIR -p ctl.n_accel=1,5,10
```

In this case, the run directories are `OUTDIR/ctl.nccl.1`, `OUTDIR/ctl.nccl.5` and `OUTDIR/ctl.nccl.10`, respectively.

General information about the ensemble can be found in the main ensemble directory `OUTDIR`:

- `params.txt` : contains a table of the parameter combinations set on the command line (can be used to run a new ensemble).
- `info.txt` : the same parameter table as `params.txt`, but also including an index of the `runid` (0,1,2, etc) and the `RUNDIR`:

`info.txt`:

```
  runid    ctl.n_accel  rundir
      0              1  ctl.nccl.1
      1              5  ctl.nccl.5
      2             10  ctl.nccl.10
```

It is of course possible to define multiple parameter permutations:

```bash
jobrun ./runcx -s -- -o OUTDIR -p ctl.n_accel=1,5,10 smb.alb_ice=0.3,0.4
```

To generate a more complex ensemble, using e.g. Latin-Hypercube sampling, then a two step approach is often better. First, use the `runner` command `job sample` to build the ensemble, then use `jobrun` to run it:

```bash
# Generate ensemble parameters
job sample -o lhs.txt --seed 4 -N 100 atm.c_trop_2=0.8,1.2 smb.alb_ice=0.3,0.4

# Run ensemble
jobrun ./runcx -s -- -o OUTDIR -i lhs.txt
```

This two-step method facilitates checking that the ensemble was generated properly and improves reproducibility, since the exact parameter values are available in the table.


### 5. Test simulation

A simple climate-only test simulation for pre-industrial conditions can be run as:

```bash
./job_climber -s -f -o OUTDIR
```

