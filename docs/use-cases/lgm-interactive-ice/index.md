# LGM simulations with interactive ice sheets

All of the sections below assume we have compiled the `climber-clim-ice` configuration.

A few different relevant configurations are provided below.

## Pre-industrial snapshot with interactive ice sheets

```bash
./runme -rs -q medium --omp 32 -o output/pi_ice_nh -p ctl.nyears=100000 ctl.n_accel=10 ctl.flag_geo=T ctl.flag_ice=T ctl.flag_smb=T ctl.flag_bmb=T ctl.ice_domain_name=NH-32KM ctl.ice_model_name=yelmo
```

## Default simulation for LGM without interactive ice sheets

This command was taken from the run_bench.sh script.

```bash
./runme -rs -q short -w 24:00:00 --omp 32 -o output/$outdir/lgm -p ctl.nyears=10000 ctl.iorbit=1 ctl.ecc_const=0.018994 ctl.obl_const=22.949 ctl.per_const=114.42 ctl.fake_geo_const_file=input/geo_ice_tarasov_lgm.nc ctl.fake_ice_const_file=input/geo_ice_tarasov_lgm.nc ctl.co2_const=190 ctl.ch4_const=375 ctl.n2o_const=200 lnd.lithology_uhh_file=input/Lithology_lgm_UHH.nc
```

## LGM snapshot with interactive ice sheets

```bash
# TO DO!
```