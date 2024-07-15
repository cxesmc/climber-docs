# Running CLIMBER-X on the albedo cluster at AWI

To run **CLIMBER-X** on the `albedo` cluster at AWI, essentially the same instructions apply as for [running at PIK](running-at-pik.md).

The only differences are the following:

1. A different set of modules needed to be loaded into the environment to get the right compilers and NetCDF libraries etc. installed. For `albedo` use the following:

```bash
    module load intel-oneapi-compilers/2022.1.0
    module load netcdf-c/4.8.1-openmpi4.1.3-oneapi2022.1.0
    module load netcdf-fortran/4.5.4-oneapi2022.1.0
    module load udunits/2.2.28
    module load ncview/2.1.8
    module load cdo/2.2.0
    module load python/3.10.4
```

2. When installing the `climber-x-exlib` libraries, use the `dkrz` specific script:

```bash
    cd climber-x-exlib
    ./install_dkrz.sh ifx
```

All other instructions should be the same.
