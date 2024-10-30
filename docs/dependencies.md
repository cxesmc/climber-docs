# Dependencies

**CLIMBER-X** is dependent on the following libraries:

- NetCDF: [NetCDF library](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)
- coordinates: [coordinates](https://github.com/cxesmc/coordinates), a module to handle grid/points definition, interpolation mapping and subsetting. The library will have to be compiled from the original source code.

- Python 3.x, which is only needed for automatic configuration of the Makefile
and the use of the `runme` script for job preparation and submission.
- runner: ['runner' Python library (cxesmc fork)](https://github.com/cxesmc/runner)
- CDO: [Climate Data Operators](https://code.mpimet.mpg.de/projects/cdo/), used for more efficient
creation of maps to transform between different coordinate grids.

Needed only if running with coupled ice sheets:

- Yelmo: [Yelmo ice sheet model](https://github.com/palma-ice/yelmo). The library will have to be compiled from the original source code.
- LIS: [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/). The library will have to be compiled from the original source code.

Installation tips for each dependency can be found below.

## Installing NetCDF (preferably version 4.0 or higher)

The NetCDF library is typically available with different distributions (Linux, Mac, etc).
Along with installing `libnetcdf`, it will be necessary to install the package `libnetcdf-dev`.
Installing the NetCDF viewing program `ncview` is also recommended.

If you want to install NetCDF from source, then you must install both the
`netcdf-c` and subsequently `netcdf-fortran` libraries. The source code and
installation instructions are available from the Unidata website:

[https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)

## Installing coordinates

Download the coordinates source:
[https://github.com/cxesmc/coordinates](https://github.com/cxesmc/coordinates).
Configure the package, and install it in the location
of your choice (below defined as `$COORDROOT`):

```bash
git clone git@github.com:cxesmc/coordinates.git $COORDROOT
cd $COORDROOT
python config.py config/pik_hpc2024_ifx
make clean
make coord-static openmp=1
```

## Installing LIS

Download the LIS source:
[https://www.ssisc.org/lis/](https://www.ssisc.org/lis/)
Configure the package, and install it in the location
of your choice (below defined as `$LISROOT`). Also, make sure to
enable the Fortran90 and openmp interface:

```bash
git clone git@github.com:anishida/lis.git $LISROOT
./configure --prefix=$LISROOT --enable-omp --enable-f90
make
make install
```

Note: make sure to set the environment variables `CC` and `FC`, in order to set
a specific compiler, for example for gcc/gfortran use the following configure command:

```bash
CC=gcc FC=gfortran ./configure --prefix=$LISROOT --enable-f90 --enable-omp
```

## Installing runner

1. Install `runner` to your system's Python installation via `pip`.

```bash
pip install https://github.com/cxesmc/runner/archive/refs/heads/master.zip
```

That's it! Now check that system command `job` is available by running `job -h`.
If the command is not found, it means that the Python bin directory is not available in your `PATH`. To add it, typically something like this is needed in your .profile or .bashrc file:

```bash
PATH=${PATH}:${HOME}/.local/bin
export PATH
```
