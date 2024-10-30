# Getting started

Here you can find the basic information and steps needed to get **CLIMBER-X** running.

Some step-by-step commands are given for specific environments.
To see how to install CLIMBER-X on the HPC2024 cluster at PIK, see: [Running-at-PIK](running-at-pik.md) for detailed instructions.

There are currently four different flavors of **CLIMBER-X** that can be set up:

- `climber-clim`: minimal climate model configuration with atmosphere, ocean, sea ice and land (including dynamic vegetation)
- `climber-clim-bgc`: coupled climate-carbon cycle model configuration; clim plus with ocean biogeochemistry
- `climber-clim-ice`: coupled climate-ice sheet model configuration; clim plus with ice sheets
- `climber-clim-bgc-ice`: fully coupled model configuration; clim plus with ocean biogeochemistry and ice sheets

The model dependencies vary according to the desired model configuration:

- Dependencies are: NetCDF, coordinates, Python3.x, runner, CDO
- Additional dependencies if using coupled ice sheets are: Yelmo, LIS

See: [Dependencies](dependencies.md) for more details.

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

### 1. Get the code

Clone the repository from [https://github.com/cxesmc/climber-x](https://github.com/cxesmc/climber-x):

```bash
# Clone code repository
git clone https://github.com/cxesmc/climber-x.git
git clone git@github.com:cxesmc/climber-x.git # via ssh

cd climber-x

# Clone input file directory
git clone git@gitlab.pik-potsdam.de:cxesmc/climber-x-input.git input
```

If you plan to make changes to the code, it is wise to check out a new branch:

```bash
git checkout -b user-dev
```

You should now be working on the branch `user-dev`.

### 2. Create the system-specific Makefile

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

### 3. Compiling **CLIMBER-X**

Assuming the source has been downloaded and configured, and all [dependencies](dependencies.md) have also been compiled, now you are ready to compile **CLIMBER-X**:

```bash
make clean
make climber-clim
```

There are currently four different flavors of **CLIMBER-X** that can be compiled:

- `climber-clim`: minimal climate configuration with atmosphere, ocean, sea ice and land
- `climber-clim-bgc`: clim plus with ocean biogeochemistry
- `climber-clim-ice`: clim plus with ice sheets
- `climber-clim-bgc-ice`: clim plus with ocean biogeochemistry and ice sheets

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

If not using the cluster, **CLIMBER-X** can also be run manually by entering the `RUNDIR` and running the following:

```bash
cd RUNDIR
./climber.x > out.out
```

Either of the above will run climber in `RUNDIR` with all simulation output stored in the same directory. Because the directory includes the executable and is self-contained, assuming the contents of the linked directories do not change, the simulation can be re-run at any time.

To perform all of the above steps manually for multiple simulations is very tedious and time-consuming, so a "run" script called `runme` is used to handle the process.

### Using `runme`

Before using `runme`, the user should store a configuration file with some personal choices. To get started, copy the template config file to the main directory:

```bash
cp .runme/runme_config .runme_config
```

Next edit `.runme_config` so that the choices match your configuration. Mostly, this means setting `hpc` to the name of your current system and `account` to the default account to be used for your jobs submitted via SLURM. Also you can add your email address if you would like to receive notifications from SLURM about your jobs.

Now `runme` is ready for use. See `./runme -h` for details on possible arguments.

Note that aside from possible optional arguments, `runme` is always called with the required argument `-o RUNDIR` that specifies the output directory. So, to run a `climber.x` simulation as a job on the cluster in `RUNDIR`, run the command:

```bash
./runme -rs -o RUNDIR
```

where `RUNDIR` is the desired run directory. The option `-r` says that the job should actually be run (instead of just prepared) and `-s` specifies that the job should be run on the cluster. If `-r` is used alone, then the job is simply run as a background process. Other options include `-q, --queue` for the queue alias (short, priority, etc.), `-w, --wall` for the maximum wall clock time to allow in format HH:MM:SS, `--part` to name the processor partition (priority, standard, smp, etc), `--omp` to specify the number of processors, and others. See `./runme -h` for all options.

When called as above, this script will run a simulation using the parameters as they are specified in the namelist parameter files in the `nml` directory. In addition, it is possible to modify the parameters of one simulation at the command line using the argument `-p KEY=VAL KEY=VAL ...`. So, for example, the following command:

```bash
./runme -s -o RUNDIR -p ctl.n_accel=10
```

will run `climber.x` on the cluster in the output directory `RUNDIR` with the control parameter `control.n_accel` set to `10`. Note that `ctl` is a convenient alias for the namelist group `control`, as defined in `.runme/climberx_info.json`.

To perform a simulation an ensemble of simulations with modified parameter values, `runme` should be called via `jobrun` (see below).

#### Using `jobrun` with `runme`

`jobrun` is a command that is part of the Python `runner` library, found here:
[https://github.com/cxesmc/runner](https://github.com/cxesmc/runner). This command facilitates running ensembles of simulations, or simulations with modified parameters via a convenient command-line interface. See [Dependencies](dependencies.md) for its installation instructions.

Using `jobrun`, the following command would produce the same simulation as `./runme -s RUNDIR`:

```bash
jobrun ./runme -rs -o OUTDIR
```

The difference here is that now we don't specify the specific `RUNDIR`, but rather an encapsulating `OUTDIR` that will contain one or more `RUNDIR`'s. In the above example, no parameters are changed, so the simulation is saved in the `default` directory: `OUTDIR/default`.

If we want to change a parameter, this can be done as with the `runme` script via the `-p` option:

```bash
jobrun ./runme -rs -o OUTDIR -p ctl.n_accel=10
```

This will produce one simulation with the parameter `control.n_accel=10`. Since we have changed a parameter for this simulation, `jobrun` treats this as an ensemble, so the output is saved in `OUTDIR/0` for simulation 0. In short, the above command is equivalent to `./runme -s -o OUTDIR -p ctl.n_accel=10`, but in the former case, the output is stored in `OUTDIR/0` and in the latter case, it is stored directly in `OUTDIR`.

The power of `jobrun` comes when we want to run an ensemble:

```bash
jobrun ./runme -rs -o OUTDIR -p ctl.n_accel=1,5,10
```

This ensemble of simulations will appear in `OUTDIR/0`, `OUTDIR/1` and `OUTDIR/2`, respectively.

A more informative output directory can be made using the option `-a` along with `-o`:

```bash
jobrun ./runme -rs -a -o OUTDIR -p ctl.n_accel=1,5,10
```

In this case, the run directories are `OUTDIR/ctl.nccl.1`, `OUTDIR/ctl.nccl.5` and `OUTDIR/ctl.nccl.10`, respectively.

General information about the ensemble can be found in the main ensemble directory `OUTDIR`:

- `params.txt` : contains a table of the parameter combinations set on the command line (can be used to run a new ensemble).
- `info.txt` : the same parameter table as `params.txt`, but also including an index of the `runid` (0,1,2, etc) and the `RUNDIR`:

`info.txt`:

```python
  runid    ctl.n_accel  rundir
      0              1  ctl.nccl.1
      1              5  ctl.nccl.5
      2             10  ctl.nccl.10
```

It is of course possible to define multiple parameter permutations:

```bash
jobrun ./runme -rs -o OUTDIR -p ctl.n_accel=1,5,10 smb.alb_ice=0.3,0.4
```

To generate a more complex ensemble, using e.g. Latin-Hypercube sampling, then a two step approach is often better. First, use the `runner` command `job sample` to build the ensemble, then use `jobrun` to run it:

```bash
# Generate ensemble parameters
job sample -o lhs.txt --seed 4 -N 100 atm.c_trop_2=0.8,1.2 smb.alb_ice=0.3,0.4

# Run ensemble
jobrun ./runme -rs -o OUTDIR -i lhs.txt
```

This two-step method facilitates checking that the ensemble was generated properly and improves reproducibility, since the exact parameter values are available in the table.

### 5. Test simulation

A simple climate-only test simulation for pre-industrial conditions can be run as:

```bash
./runme -rs -o OUTDIR
```
