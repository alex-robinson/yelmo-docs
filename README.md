
# Yelmo

Welcome to Yelmo, an easy to use continental ice sheet model.

## Directory structure

    docs/
        Model documentation.
    input/
        Location of any input data needed by the model.
        The REMBOv2_data respository should be available here as:
        `input/REMBOv2_data` which contains some useful boundary datasets.
    libs/
        Auxiliary libraries nesecessary for running the model.
    maps/
        Generated maps and grids.
    output/
        Default location for model output.
    src/
        Source code for Yelmo.


## Getting the code

Yelmo is managed in a git repository on the PIK cluster:

    /iplex/01/tumble/robinson/repos/yelmo

You must have the correct group permissions to be able to clone a repository
and push changes. 

1. Clone the repository to a local path (replace `robinson` with your username):

        git clone robinson@cluster.pik-potsdam.de:/iplex/01/tumble/robinson/repos/yelmo LOCAL_PATH

2. Create your own branch so that your changes do not overwrite the master branch:
        
        git checkout -b your_branch

3. Push your branch back to the repository, to make sure your changes are also committed to the cluster:

        git push -u origin your_branch

Note use the command `git branch` at any time to see which branches are available and which branch is currently checked out (marked by the asterisk).

That's it! You now have all of the Yelmo code in the local path `LOCAL_PATH`.

## Compiling Yelmo programs

Yelmo uses a Makefile to manage program compilation. Currently one test program exists for Yelmo development: `test_yelmo.x`. Compilation can be acheived via make:

    make test_yelmo    : compiles the program test_yelmo.x (ice sheet development program)
    make clean         : removes all compiled files

By default, the Makefile will choose gfortran as the compiler and use the `-O3` compiler options. To compile on the PIK cluster using ifort add `ifort=1` to the end of the command, eg.:

    make test_yelmo ifort=1

Likewise to include debugging options, use `debug=1`:

    make test_yelmo debug=1 

If compiled on the PIK cluster with ifort, the library paths given in the Makefile (mainly to find the netcdf libraries) should be correct and the programs should compile without any problems. For local compilation, typical Mac locations are currently specified that may need to be modified for other computers. To do so, find these lines in the Makefile:

    netcdf_inc = /opt/local/include
    netcdf_lib = /opt/local/lib

## Running the programs

Yelmo is designed so that the source code is located in one place and each simulation produces output in a new directory. In this way, there is only one copy of the source code and compiled program, and one output directory for each program call.

By default, if no output directory is specified, the program sends output directly to the path `output` and can be run from the command line:

    ./test_yelmo.x

Otherwise the output directory can be specified as a command-line argument. In this case, the output directory should already contain a copy of the namelist files for the domains to be simulated.
    
    mkdir output/simulation1
    cp Greenland.nml output/simulation1/
    ./test_yelmo.x output/simulation1
    
_Note: In the near future, the automatic generation of output directories, as well as batch simulations and parameter changes will be handled using the python __job__ script._

## Parameter modification

Yelmo depends on a namelist file with default parameter values located in the `src` directory (`src/yelmo.nml`). This file should remain unchanged during production runs, it is only used to initially load all parameter values. 

Next, in the main directory, there should be one namelist file per modeled domain (e.g., `Greenland.nml`), with each group of parameters corresponding to each model component. The name of this namelist file corresponds to the name of the domain given in the code when the domain was initialized. As the program runs, it first loads parameters from the default namelist file, then it checks if any parameters should be overwritten from the domain-specific namelist file. 

This two step procedure is convenient in that usually the default values do not change and can be stored for reference in the model component namelist file. Meanwhile in the domain-specific namelist file, only the parameters of interest that need to be modified for a given run need to be specified. 

# Libraries

Yelmo relies on some external libraries (some of which have been built specifically for use with it), however the Yelmo repository is mostly self-contained. If the libraries are portable, like fortran modules, then they are stored in the `libs/` subdirectory. The only external dependency is on the NetCDF libraries, which must be preinstalled on the system. 

Below is a list of the libraries used in Yelmo:

__nml__ is a custom library for reading namelist parameters into a Fortran program. It has the same functionality as native nml reading, however it does not require the definition of a common block. 

__coordinates__ handles the definition of grids and points on user-specified coordinate systems. It has the ability to map between different coordinate systems, interpolation and oblique stereographic projection.

__subset__ depends on the coordinates module and allows subsets of points to be generated from pre-defined grids. The subset of points can be of higher resolution than the original grid, and if so, mapping to and from the original grid is handled internally.

__insol__ provides functions to calculate the daily insolation at any latitude for any time up to 5 Ma BP or 1 Ma AP (after present). 

__solvers__ provides some basic differential equation solvers. 



