# Dependencies 

**CLIMBER-X** is dependent on the following libraries:

- NetCDF: [NetCDF library](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)
- FFTW: [Fastest Fourier Transform in the West](https://www.fftw.org/). The library will have to be compiled from the original source code.
- coordinates: [coordinates](https://github.com/alex-robinson/coordinates), a module to handle grid/points definition, interpolation mapping and subsetting. The library will have to be compiled from the original source code.
- LIS: [Library of Iterative Solvers for Linear Systems](http://www.ssisc.org/lis/). The library will have to be compiled from the original source code.

OPTIONAL:
- Python 3.x, which is only needed for automatic configuration of the Makefile
and the use of the scripts `job_climber` and `runcx` for job preparation and submission.
- CDO: [Climate Data Operators](https://code.mpimet.mpg.de/projects/cdo/), used for more efficient
creation of maps to transform between different coordinate grids.
- runner: ['runner' Python library (alex-robinson fork)](https://github.com/alex-robinson/runner)


Installation tips for each dependency can be found below.

## Installing NetCDF (preferably version 4.0 or higher)

The NetCDF library is typically available with different distributions (Linux, Mac, etc).
Along with installing `libnetcdf`, it will be necessary to install the package `libnetcdf-dev`.
Installing the NetCDF viewing program `ncview` is also recommended.

If you want to install NetCDF from source, then you must install both the
`netcdf-c` and subsequently `netcdf-fortran` libraries. The source code and
installation instructions are available from the Unidata website:

[https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html](https://www.unidata.ucar.edu/software/netcdf/docs/getting_and_building_netcdf.html)

## Installing FFTW

1. Download and configure the FFTW source:
[https://www.fftw.org/download.html](https://www.fftw.org/download.html)

```bash
wget https://www.fftw.org/fftw-3.3.10.tar.gz
tar -xvf fftw-3.3.10.tar.gz
rm fftw-3.3.10.tar.gz
cd fftw-3.3.10
./configure --prefix=$FFTWROOT --enable-openmp
make
make install
```

## Installing coordinates

1. Download the coordinates source:
[https://github.com/cxesmc/coordinates](https://github.com/cxesmc/coordinates).
Configure the package, and install it in the location 
of your choice (below defined as `$COORDROOT`):

```bash
git clone git@github.com:cxesmc/coordinates.git $COORDROOT
cd $COORDROOT
python config.py config/pik_ifort
make clean
make coord-static parallel=1
```

## Installing LIS

1. Download the LIS source:
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

1. Install `runner` to your system's Python installation via `pip`, along with dependency `tabulate`.

```bash
pip install https://github.com/alex-robinson/runner/archive/refs/heads/master.zip
pip install tabulate
```

That's it! Now check that system command `job` is available by running `job -h`. 

Note that install method `python setup.py install` should be avoided if possible to maintain Python system integrity.
