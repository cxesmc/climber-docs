# Getting started

Here you can find the basic information and steps needed to get **CLIMBER-X** running.

For a quick start, see [Quick-start](quick-start.md).

There are currently four different flavors of **CLIMBER-X** that can be set up:

- `climber-clim`: minimal climate model configuration with atmosphere, ocean, sea ice and land (including dynamic vegetation)
- `climber-clim-bgc`: coupled climate-carbon cycle model configuration; clim plus with ocean biogeochemistry
- `climber-clim-ice`: coupled climate-ice sheet model configuration; clim plus with ice sheets
- `climber-clim-bgc-ice`: fully coupled model configuration; clim plus with ocean biogeochemistry and ice sheets

The model dependencies vary according to the desired model configuration:

- Dependencies are: NetCDF, coordinates, Python3.x, runner, CDO
- Additional dependencies if using coupled ice sheets are: Yelmo, LIS

See: [Dependencies](dependencies.md) for more details.

Follow the steps in [Installation](installation.md) to get the code and compile the model and then you are ready to run
your first simulations following the instructions in [Usage](usage.md)

