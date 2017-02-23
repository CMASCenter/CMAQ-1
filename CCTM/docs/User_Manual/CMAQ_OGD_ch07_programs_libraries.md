
[<< Previous Chapter](CMAQ_OGD_ch06_req_lib.md) - [Home](README.md) - [Next Chapter >>](CMAQ_OGD_ch08_input_files.md)
***
7. CMAQ Programs and Libraries
===========================

<a id="Overview"></a>
Overview
--------

The core CMAQ programs that are needed to perform a basic air quality model simulation are [MCIP](#mcip), [ICON](#icon), [BCON](#bcon), [JPROC](#jproc), and [CCTM](#cctm). The relationships among these programs are depicted within the green box in [Figure 7-1](#Figure7-1). The blue boxes represent programs that are not part of the CMAQ distribution package but supply data necessary for an air quality simulation (emissions and meteorology data). The yellow boxes represent the standard CMAQ preprocessors: MCIP, ICON, BCON, and JPROC. The red box represents the CMAQ chemistry-transport model (CCTM), the Eulerian air quality modeling component of CMAQ. Data flows between the CMAQ programs are represented in by arrows. The red arrows illustrate the flow of data from the CMAQ preprocessors and the emissions model to CCTM. The green arrows show the data feedbacks from CCTM to create initial and boundary conditions for nested simulations. The black arrow illustrates the connection between the meteorological model and MCIP. Finally, the blue arrow shows that the output from MCIP can be used to drive an emissions model.

The meteorological model,such as [WRF‑ARW](http://www.wrf-model.org), generates gridded meteorology for input to both CMAQ and the emissions model.

The emissions model converts emissions inventories to gridded, hourly emissions formatted for CMAQ. The [SMOKE](http://www.smoke-model.org) emissions model is currently available for preparing emissions data for CMAQ.

CMAQ includes several "in-line" options to support coupling between meteorology and chemistry processes, and to facilitate operational air quality forecast modeling (see [Chapter 4](CMAQ_OGD_ch04_science.md)). The user can incorporate photolysis rate calculations and emissions processing during a CCTM simulation. There are several advantages of incorporating these processes directly in a CCTM simulation:

1. Photolysis rate calculations use the aerosol concentrations and meteorology from the CCTM simulation, simulating the feedbacks of the input emissions and resulting air quality on photochemistry
2. Emissions are meteorologically modulated at the synchronization (chemistry) time step rather than being linearly time-interpolated within each simulation hour
3. Disk space may be saved, because a 3‑D emissions file is no longer needed for elevated point sources
4. CMAQ can more easily be coupled with a meteorological model, enabling direct emissions modulation by the underlying, freshly computed meteorological variables

<a id=Figure7-1></a>

<center>
![](./images/Figure7-1.png "Figure7-1.png")

</center>
<center>
**Figure 7‑1.CMAQ core programs**

</center>

[MCIP](#mcip) is the first program in the CMAQ distribution package that a user should run when setting up a new simulation. MCIP is used to preprocess the data from a meteorological model for CMAQ and SMOKE.

[ICON](#icon) creates a binary netCDF initial conditions file for input to CCTM. Users have the option to create initial conditions data either from a text file of vertical concentration profiles or from an existing CCTM output file. ICON outputs initial conditions data that are configured for a specific modeling grid and chemical parameterization.

[BCON](#bcon) creates a binary netCDF lateral boundary conditions file for input to CCTM. Users have the option to create boundary conditions from either a text file of vertical concentration profiles or from an existing CCTM or larger-scale (e.g., global-scale) output file. BCON outputs boundary conditions data that are configured for a specific modeling grid and chemical parameterization. If derived from an existing CCTM or larger-scale output file, BCON produces dynamic boundary conditions that vary in time and space. When derived from vertical concentration profiles, BCON produces static boundary conditions for input to CCTM.

[JPROC](#jproc) converts physical information about photoreactive molecules into clear-sky photolysis rate look-up tables for input to CCTM. The in-line photolysis approach allows photolysis rates to be adjusted by simulated gas and aerosol concentrations rather than by climatological values in the off-line approach. JPROC is not required when the in-line photolysis approach is selected.

[CCTM](#cctm) is run last in the sequence of programs. All of the other CMAQ programs, and the emissions and meteorological models, are used to prepare the inputs to CCTM. By using data that are synchronized for a particular modeling time period, model grid, vertical layer configuration, and chemical parameterization, CCTM can produce estimates of pollutant concentrations, wet and dry deposition rates, and visibility metrics.

In addition to the core programs shown in [Figure 7‑1](#Figure7-1), the CMAQ distribution package also includes utilities `($CMAQ_HOME/UTIL/*)` and preprocessors `($CMAQ_HOME/PREP/*)` for utilizing additional technical and diagnostic features in CMAQ. CMAQ utilities include CHEMMECH and CREATE_EBI for editing existing and preparing new chemical mechanisms for CMAQ. CMAQ preprocessors include LTNG_2D_DATA that converts monthly lightning flash counts and ratios of cloud-to-cloud and cloud-to-ground flashes into an input file for CCTM (prior to CMAQv5.2). CMAQv5.2 updates the production of lightning NOx by using hourly NLDN data. The CMAQ preprocessor CALMAP creates maps of the crop calendar for use in estimating windblown dust emissions.

This chapter provides detailed descriptions of the CMAQ programs and utilities. Information about the third-party libraries used by CMAQ—such as I/O API, netCDF, and MPI available in [Chapter 6](CMAQ_OGD_ch06_req_lib.md).

When viewing the tables that list each program’s input and output files, recall that the various I/O API file formats shown are described in [Chapter 6](CMAQ_OGD_ch06_req_lib.md) .

<a id="BCON"></a>
## BCON

### Description

The program BCON prepares lateral chemical boundary conditions (BCs) for CCTM from either ASCII vertical profiles or from an existing CCTM output concentration (CONC) file. The BCs created by BCON can be static in both time and space (i.e., time-invariant with uniform concentrations in all boundary grid cells), dynamic in both time and space, or a combination of the two. The ASCII vertical profiles are primarily used to create static BCs. Dynamic BCs can be extracted from CONC files on either the same horizontal grid spacing (i.e., as a windowed modeling domain) or for a finer-resolution model grid (i.e., for a nested simulation), or they can be interpolated from a larger-scale CTM simulation (which is analogous to defining lateral BCs for WRF‑ARW).

There are two distinct modes of operation for BCON, and the mode used depends on the nature of the input data. When creating BCON executables, the user must specify whether the input data will be ASCII vertical profiles or a CONC file by selecting either “profile” or “m3conc”, respectively, for the setting of the *ModType* variable. This variable determines the input module to use when creating a BCON executable.

CCTM can also be forced with chemical boundary conditions downscaled from global chemistry models (GCMs), such as GEOS-Chem and MOZART. BCON does not support the processing of data from GCMs. BCs derived from GCMs must be calculated with custom codes or scripts that are not available in the CMAQ distribution package. The CAMx developers (Ramboll Environ) have codes available for extracting regional model BCs from both GEOS-Chem and MOZART. Visit the [Support Software section of www.CAMx.com](http://www.camx.com/download/support-software.aspx) to download these utilities.

### Files, configuration, and environment variables

[Figure 7‑2](#Figure7-2) shows the input and output files and configuration options for BCON. A distinction is made between the options that are invoked at compilation versus those invoked at execution of the program. When compiling BCON, the user specifies a chemical mechanism to configure the gas-phase chemistry and aerosol mechanism used to create the chemical BCs. Setting the *ModMech* and *Mechanism* variables in the BCON compile script configures the program to use a specific set of mechanism INCLUDE files to build an executable. Setting the *ModType* variable in the BCON compile script configures the program to input either a text file of static concentrations or a binary netCDF file of time-dependent concentrations for estimating BCs for CCTM. Separate BCON executables must be prepared for different mechanism and input file configurations.

<a id=Figure7-2></a>

<center>
![](./images/Figure7-2.png "Figure7-2.png")<br>

**Figure 7‑2. BCON input and output files**
</center>

When BCON is run, it converts a data file of chemical ambient concentrations to BCs on a predefined model grid. Through the specification of the *ModInpt* variable in the BCON run script, BCON will input either an ASCII vertical profile file (BC_PROFILE) or an existing CCTM concentration file (CTM_CONC_1); the choice depends on how the user compiled the model. The BC input file provided by the user must have chemical speciation that is consistent with the mechanism configuration of the BCON executable. For example, if BCON was compiled to create BCs using the CB05 mechanism, the input BC profile data must be in terms of the CB05 mechanism. CMAQ is distributed with ASCII vertical profiles representing clean continental BCs for North America for the following chemical mechanisms: cb05e51_ae6_aq, saprc07tb_ae6_aq and racm2_aq6_aq. It is the user’s responsibility to generate BC inputs for other mechanism configurations.

The horizontal grid and vertical layer structures for BCON are defined at execution through the input of a grid description (GRIDDESC) file and a meteorology cross-point 3‑D (MET_CRO_3D) file, respectively. BCON interpolates between the input vertical layer structure and output layer structure if they are different.



### BCON input files ###


<a id=Table7-1></a>

<center>**Table 7‑1. BCON input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|BC_PROFILE|ASCII|Vertical chemical profiles from which to derive boundary conditions; this file is created by the user; used only when the BC environment variable is set to “profile”|
|CTM_CONC_1|GRDDED3|CMAQ concentration file from which to derive boundary conditions; this file is output from CCTM; used only when the BC environment variable is set to “m3conc”|
|MET_CRO_3D_CRS|GRDDED3|Name and location of the coarse-grid MET_CRO_3D file that is required for creating the vertical grid structure if this structure changes between nested simulations; this file is output by MCIP|
|MET_CRO_3D_FIN|GRDDED3|Name and location of the fine-grid MET_CRO_3D file that is required if the vertical grid structure changes between nested simulations; this file is output by MCIP|
|GRIDDESC|ASCII|Horizontal grid description file for defining the model grid; this file is output by MCIP or can be created by the user|
|LAYER_FILE|GRDDED3|3-D cross-point meteorology (METCRO3D) file for defining the vertical layer structure of the model grid; this file is output by MCIP|
|gc_matrix.nml|ASCII|Namelist file for defining the gas-phase species that are input to the model through the boundary|
|ae_matrix.nml|ASCII|Namelist file for defining the aerosol species that are input to the model through the boundary|
|nr_matrix.nml|ASCII|Namelist file for defining the non-reactive species that are input to the model through the boundary|
|tr_matrix.nml|ASCII|Namelist file for defining the tracer species that are input to the model through the boundary|

### Compiling and Running

The configuration options listed here are set during compilation of the BCON executable. When these options are invoked they create a binary executable that is fixed to the specified configuration. To change these options you must recompile BCON and create a new executable.

-   Opt: [default: verbose]<br>
    Defines the action to be taken by the program Bldmake when compiling an executable.
    -   *compile_all:* force compile, even if all the object files are current
    -   *clean_up:* remove all source files upon successful compilation
    -   *no_compile:* do everything except compile
    -   *no_link:* do everything except link
    -   *one_step:* compile and link in one step
    -   *parse_only:* check configuration file syntax
    -   *show_only:* show requested commands but do not execute them
    -   *verbose:* show requested commands as they are executed


-   MakeFileOnly<br>
    Uncomment to build a Makefile to compile the executable. Comment out to both create a Makefile and compile.

-   ModType: [default: module profile]<br>
    Defines the format of the boundary conditions input files to be used by BCON.
    -   *m3conc*: input a CCTM CONC file; used for nested simulations or windows of a parent domain
    -   *profile*: input an ASCII vertical profiles file
    -   *tracer*: use the tracer namelist file to create BCs of tagged tracer species


-   Mechanism: [default: cb05e51_ae6_aq]<br>
    Specifies the gas-phase, aerosol, and aqueous-phase chemical mechanisms for which to create boundary conditions. The choices for the *Mechanism* variable are the mechanism directory names under the `$CMAQ_HOME/CCTM/src/MECHS` directory. Also see the [Mechanism Definitions Table](../Release_Notes/CMAQv5.2_Mechanisms.md)). Examples include:
    -   *cb6r3_ae6_aq*: CB6, revision 3 gas-phase mechanism, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM Other, aqueous/cloud chemistry
    -   *cb05e51_ae6_aq*: CB05 gas-phase mechanism with CMAQv5.1 updates, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM Other, aqueous/cloud chemistry
    -   *cb05tucl_ae6_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM Other, aqueous/cloud chemistry
    -   *cb05tump_ae6_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, mercury, and air toxics, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM, aqueous/cloud chemistry; this is the CMAQv5 multipollutant mechanism
    -   *saprc07tb_ae6_aq*: SAPRC-07 gas-phase mechanism with toluene updates and sixth-generation CMAQ aerosol mechanism
    -  *racm2_ae6_aq*: RACM2 gas-phase mechanism with toluene updates and sixth-generation CMAQ aerosol mechanism

#### BCON compilation

[Chapter 5](CMAQ_OGD_ch05_sys_req.md) provides an overview of how to install and compile the CMAQ pre-processor programs for a test simulation. Follow those steps (summarized below) to compile new versions of BCON.

1. Compile Bldmake, the CMAQ source code and compilation management program. This needs to be done only once—the first time CMAQ is installed.
-   Configure the BCON build script to use the available I/O API and netCDF libraries.
-   Configure the BCON build script for your application.
-   Invoke the build script to create an executable:

`cd $CMAQ_HOME/PREP/bcon/scripts/`
`./bldit.bcon`

#### BCON execution options

The environment variables listed here are invoked during execution of the program and are set in the BCON run script.

-   `EXEC: [default: BCON_${APPL}_${EXECID}]`<br>
    Executable to use for the simulation. The variable CFG is set in the BCON run script. The variable EXECID is set in the config.cmaq configuration file.

-   `GRIDDESC: [default: $CMAQ_HOME/scripts/GRIDDESC1]`<br>
    Grid description file for setting the horizontal grid definition.

-   `GRID_NAME: [default:CMAQ-BENCHMARK]`<br>
    Name of the grid definition contained in the GRIDDESC file that specifies the horizontal grid for the current application of the model.

-   `IOAPI_ISPH: [default: 20]`<br>
    I/O API setting for spheroid type. See I/O API documentation for [setsphere](https://www.cmascenter.org/ioapi/documentation/3.1/html/SETSPHERE.html) for more information.

-   `IOAPI_OFFSET_64: [default: NO]`<br>
    I/O API setting for large time-step records. If your output time step is going to produce data that are >2GB per time step, then this needs to be set to YES.

-   `LAYER_FILE:[default: none]`<br>
    Name and location of a MET_CRO_3D file for specifying the vertical layer structure for the current application of the model.

-   `gc_matrix.nml: [default: none]`<br>
    Gas-phase species namelist file. This file is used to configure the gas-phase species that will be output by BCON.

-   `ae_matrix.nml: [default: none]`<br>
    Aerosol-phase species namelist file. This file is used to configure the aerosol-phase species that will be output by BCON

-   `nr_matrix.nml: [default: none]`<br>
    Nonreactive species namelist file. This file is used to configure the nonreactive species that will be output by BCON

-   `tr_matrix.nml: [default: none]`<br>
    Tracer species namelist file. This file is used to configure the tracer species that will be output by BCON

-   `OUTDIR: [default: $CMAQ_HOME/data/bcon]`
    Output data directory.

-   BC:<br>
    Sets the input file type. The setting of this variable determines how the run script sets the input and output environment variables.
    -   *profile*: sets the output file name to include the tag “profile” in the name; uses the variable BC_PROFILE to point to an ASCII vertical profile file for input to BCON. Also optionally looks for the variable MECH_CONV_FILE to point to a user-defined mechanism conversion file.
    -   *m3conc*: used for nested simulations; sets the output file name to include a start date in the name; uses the variable CTM_CONC_1 to point to a CCTM CONC file for input to BCON.


-   DATE:<br>
    Sets the Julian date to use in naming the BCON output file for nested runs.

-   `SDATE:[default: ${DATE}]`<br>
    Julian start date for extracting boundary conditions from a CCTM CONC file for a nested simulation. If SDATE is not set, it will be set automatically from the CTM_CONC_1 file.

-   `STIME: [default: 000000 ]`<br>
    Start time for extracting boundary conditions from a CCTM CONC file for a nested simulation. If STIME is not set, it will be set automatically from the CTM_CONC_1 file.

-   `RUNLEN: [default: 240000]`<br>
    Number of hours of boundary conditions to extract from a CCTM CONC file for a nested simulation. If RUNLEN is not set, it will be set automatically from the CTM_CONC_1 file.

#### BCON output files

<a id=Table7-2></a>

<center>**Table 7‑2. BCON output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|`BNDY_CONC_1`|`BNDARY3`|Name and location of the gridded boundary conditions data output on the model grid defined by `GRID_NAME`|

The default location of the BCON output files is the `$CMAQ_DATA/bcon` directory, controlled by the `OUTDIR` variable in the run script. The default naming convention for all BCON output files uses the `APPL` and `GRID_NAME` environment variables in the file name. For boundary conditions created from existing `CCTM CONC` files, the Julian date is also used in the file name through the `DATE` environment variable. All of the file-naming variables for `BCON` outputs are set in the run script.

<a id="Calmap"></a>
## Calmap

### Description

The preprocessor program Calmap produces gridded planting start dates, planting end dates, and harvesting end dates for different crop types for estimating the impacts of agricultural activities on windblown dust emissions. The CMAQ windblown dust emissions module can optionally use the output from Calmap to simulate the effects of crop cycles on dust emissions.

### Files, configuration, and environment variables

Figure 7-3 shows that Calmap reads five input files to produce a eight outputs, only three of which are used by the CCTM. Calmap uses the GRIDCRO2D file, produced by MCIP, to define the modeling grid. The BELD01 file points to a BELD3 “a” file of gridded land cover/land use data containing coverage for several different crop categories. The BELD3 “a” file is an input to the BEIS emissions model and can be [generated by the Spatial Allocator](http://www.ie.unc.edu/cempd/projects/mims/spatial/smoke_bio_inputs.html). The rest of the inputs to Calmap are crop calendar data for the United States that are packaged with CMAQv5. Calmap converts to the data to I/O API GRDDED3 files on the modeling grid defined in the GRIDCRO2D file. The CROPMAP01 file contains planting start dates for specific crops. The CROPMAP04 file contains planting end dates for specific crops. The CROPMAP08 file contains harvesting end dates for specific crops. Each of these files are input to the CCTM when the erodible agricultural land (CTM_ERODE_AGLAND) feature is turned on.

<a id=Figure7-3></a>

<center>
![](./images/Figure7-3.png "Figure7-3.png")

</center>
<center>**Figure 7‑3.Calmap input and output files**</center>

### Calmap input files

<center>**Table 7‑3. Calmap input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|GRID_CRO_2D|GRDDED3|Name and location of the time-independent 2-D cross-point meteorology file; output by MCIP|
|BELD01|GRDDED3|BELD land use “A” data file for calculating windblown dust emissions; produced with BELD land use tiles and the Spatial Allocator|
|CPCALED|ASCII|Calendar of agricultural activities by state|
|NCROP|ASCII|Number and names of crop species included in the crop calendar|
|USSTATE|ASCII|Two digit US state codes included in the crop calendar|

### Calmap compilation

Calmap is compiled with a Makefile. The configuration options in the Makefile include only the compiler and compiler flags to use for building the executable. The Makefile is located in the directory with the Calmap source code (`$CMAQ_HOME/calmap/src`). To compile Calmap, source the config.cmaq file and invoke the Makefile at the command line:

`./make`

To port Calmap to different compilers, change the compiler names, locations, and flags in the Makefile

### Calmap execution options

The environment variables listed here are invoked during execution of the program and are set in the Calmap run script.

-   `BASE: [default: $CMAQ_HOME/PREP/agdust/scripts]`

Base Calmap installation location.

-   `GRID_CRO_2D: [default: none]`

Directory path and name of the GRID_CRO_2D file for defining the modeling grid.

-   BELD01: [default: none]

Directory path and name of the BELD3 “A” file for defining the gridded land cover/land use in the modeling domain. Note that this is the same file used by the CCTM to compute dust emissions (DUST_LU_1).

-   CROPMAP01-08: [default: none]

Directory path and names of Calmap output files. The CROPMAP01, CROPMAP04, and CROPMAP08 are the only files used by the CCTM for the dust calculation.

### Calmap output files

<center>**Table 7‑4. Calmap output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|CROPMAP01|GRDDED3|Name and location of the gridded planting start dates file.|
|CROPMAP02|GRDDED3|Name and location of the gridded ??? start dates file; not used by the CCTM.|
|CROPMAP03|GRDDED3|Name and location of the gridded ??? dates file; not used by the CCTM|
|CROPMAP04|GRDDED3|Name and location of the gridded planting end dates file.|
|CROPMAP05|GRDDED3|Name and location of the gridded ??? start dates file; not used by the CCTM|
|CROPMAP06|GRDDED3|Name and location of the gridded ??? start dates file; not used by the CCTM|
|CROPMAP07|GRDDED3|Name and location of the gridded harvesting end dates file.|

<a id="CCTM"></a>
## CCTM

### Description

CCTM is the Eulerian chemistry and transport component of CMAQ. It uses input data produced by the other CMAQ programs and from meteorological and emissions models. CCTM produces multiple output files for each simulation. The basic CCTM outputs include instantaneous and average hourly concentration files, wet and dry deposition files, and visibility estimates. Other CCTM outputs can include diagnostic aerosol and cloud files and processes analysis files.

CCTM contains several science configurations for simulating transport, chemistry, and deposition. All of the science configuration options in CCTM, such as the chemical mechanism to be used, are set when compiling the executable. The model grid and vertical layer structure for CCTM are set at execution. The important distinction between selecting the science configura­tion and the model grid/layer configuration is that CCTM does not need to be recompiled when changing model grids/layers but does need to be recompiled when new science options are invoked.

Optional output files are created when their associated processes are invoked in CCTM. For example, when CCTM is compiled with process analysis turned on, additional output files are created.

CCTM includes options for the in-line processing of emissions and photolysis rates. In-line refers to the handling of processes that had previously been accomplished outside of CCTM, such as emissions processing with SMOKE, with algorithms internal to CCTM. The benefits of in-line emissions processing include the integration of higher time-resolution meteorology in the computation of biogenic emissions and plume rise from point sources and the avoidance of the large data storage burden required for emissions data. The benefit of in-line photolysis rate calculations is the inclusion of predicted gas and aerosol concentrations in the rate calculations.

Both in-line emissions and photolysis are invoked through compile-time configuration options for CCTM. When CCTM is instrumented for in-line emissions calculations, a series of additional input files and environment variables are required at execution. The details of these additional inputs are provided below. In-line photolysis does not require any additional inputs as CCTM includes all of the photolysis rate data internal to the in-line instrumented version of the model.

### Files, configuration, and environment variables

[Figure 7‑4](#Figure7-4) shows the input and output files and configuration options for CCTM. A distinction is made between the options that are invoked at compilation time versus those invoked at execution of the program. When compiling CCTM, the user specifies a chemical mechanism to configure the gas-phase chemistry and aerosol mechanism used for the air quality calculations. Setting the *Mechanism* variable in CCTM compile script configures the program to use a specific set of mechanism INCLUDE files to build an executable. All of the science processes simulated by CCTM must also be selected during the compilation step for CCTM. Separate CCTM executables must be prepared for different mechanism and science configurations. During the execution step, or when CCTM is run, the user sets the horizontal and vertical grid definitions and the input files used for the simulation. Different spatial domains, vertical grid structures and input files can be used with a single CCTM executable, as long as the input files are consistent with the scientific configuration built into the executable. For example, with the gas-phase photochemical mechanism configuration built into a CCTM executable, different modeling domains can be simulated with the executable as long as the emissions and IC/BC files are consistent with the photochemical mechanism configuration built into the executable.

<a id=Figure7-4></a>

<center>
![](./images/Figure7-4.png "Figure7-4.png")

</center>
<center>
**Figure 7‑4. CCTM input and output files**

</center>
#### CCTM input files

<a id=Table7-5></a>

<center>**Table7‑5. Required CCTM input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|GRIDDESC|ASCII|Map projection and grid definitions|
|OCEAN_1|GRDDED3|Name and location of the time-independent 2-D file for defining the fraction of each model grid cell covered by open ocean|
|EMIS_1|GRDDED3|Name and location of the time-dependent 2-D or 3-D emission file speciated for a particular gas-phase chemical mechanism and PM model; output from an emission model, such as SMOKE or CONCEPT|
|INIT_[GASC/AERO/NONR/TRAC]_1|GRDDED3|Name and location of the time-dependent, single-time-step, 3-D initial conditions file speciated for a particular gas-phase chemical mechanism and PM model; output from ICON|
|BNDY_[GASC/AERO/NONR/TRAC]_1|BNDARY3|Name and location of the time-dependent, either single-time-step or multi-time-step, 3-D boundary conditions file speciated for a particular gas-phase chemical mechanism and PM model; output from BCON|
|GRID_CRO_2D|GRDDED3|Name and location of the time-independent 2-D cross-point meteorology file; output by MCIP|
|GRID_DOT_2D|GRDDED3|Name and location of the time-independent 2-D dot-point meteorology file; output by MCIP|
|MET_CRO_2D|GRDDED3|Name and location of the time-dependent 2-D cross-point meteorology file; output by MCIP|
|MET_DOT_3D|GRDDED3|Name and location of the time-dependent 3-D dot-point meteorology file; output by MCIP|
|MET_CRO_3D|GRDDED3|Name and location of the time-dependent 3-D cross-point meteorology file; output by MCIP|
|MET_BDY_3D|BNDARY3|Name and location of the time-dependent 3-D boundary meteorology file; output by MCIP|
|XJ_DATA|ASCII|Name and location of the daily clear-sky photolysis rates file speciated for a particular gas-phase chemical mechanism; output from JPROC|
|OMI|ASCII|Ozone Mapping Instrument vertical ozone column data by latitude for different years|
|gc_matrix.nml|ASCII|Namelist file for defining the gas-phase species that are input to the model through the boundary|
|ae_matrix.nml|ASCII|Namelist file for defining the aerosol species that are input to the model through the boundary|
|nr_matrix.nml|ASCII|Namelist file for defining the non-reactive species that are input to the model through the boundary|
|tr_matrix.nml|ASCII|Namelist file for defining the tracer species that are input to the model through the boundary|

<a id=Table7-6></a>

<center>**Table 7‑6. Optional CCTM input files**</center>

|File Name|Format|Description|
|---------|------|-----------|
|STK_GRPS|GRDDED3|Stack parameter file for point source emissions; produced by the SMOKE program Elevpoint|
|GSPRO|ASCII|Speciation profile file. Contains the factors that are used to separate aggregated inventory pollutant emissions totals into emissions of model species required by CCTM|
|B3GRD|GRDDED3|Normalized biogenic emissions for input to BEIS3; produced by the SMOKE program Normbeis3.|
|BIOSEASON|GRDDED3|Indicates the biogenic emissions factors to use for each day in a given year. This file can be created using the SMOKE utility program Metscan. The BIOSEASON file is time-dependent and usually contains data for an entire year (365 or 366 days). It uses one variable, SEASON, which is either 0 (grid cell should use winter factors for current day) or 1 (grid cell should use summer factors for current day.|
|DUST_LU_1|GRDDED3|BELD land use “A” data file for calculating windblown dust emissions; produced with BELD land use tiles and the Spatial Allocator|
|DUST_LU_2|GRDDED3|BELD land use “TOT” data file for calculating windblown dust emissions; produced with BELD land use tiles and the Spatial Allocator|
|CROPMAP01|GRDDED3|Gridded planting start dates for estimating dust emissions from erodible cropland; produced by the CMAQ preprocessor Cropcal|
|CROPMAP04|GRDDED3|Gridded planting end dates for estimating dust emissions from erodible cropland; produced by the CMAQ preprocessor Cropcal|
|CROPMAP08|GRDDED3|Gridded harvesting end dates for estimating dust emissions from erodible cropland; produced by the CMAQ preprocessor Cropcal|
|LTNGNO|GRDDED3|Name and location of a lightning NO emissions file with rate of production (moles/sec) for each model layer at each time step|
|LTNGPARM_FILE|GRDDED3|Name and location of the time-independent, gridded lightning parameters file that includes monthly flash count totals, scaling factors for calculating flashes using the convective precipitation rate, ratio of intercloud to cloud-to-ground flashes, and moles of NO per flash|
|B4LU_file|GRDDED3|BELD4 land use file with fractional crop distributions gridded to the modeling domain|
|E2C_Soilfile|GRDDED3|EPIC soil properties file for each crop type gridded to the modeling domain. This time-independent file contains the soil pH for each agricultural crop being modeled for the 1‑cm surface layer and 10-cm tilled layer|
|E2C_Fertfile|GRDDED3|EPIC crop type file gridded to the modeling domain. This file contains the initial soil ammonium concentrations for the first day of the simulation estimated by EPIC and the fertilizer application depth and rate|
|INIT_MEDC_1|GRDDED3|Previous day ASXfile to initialize the agricultural soil ammonium concentration and pH|

#### CCTM compilation options

The configuration options listed here are set during compilation of the CCTM executable. When these options are invoked they create a binary executable that is fixed to the specified configuration. To change these options you must recompile CCTM and create a new executable.

<!---
-   `Opt: [default: verbose]`
    Defines the action to be taken by the program Bldmake when extracting source code from CVS and compiling an executable.
    -   *compile_all:* force compile, even if all the object files are current
    -   *clean_up:* remove all source files upon successful compilation
    -   *no_compile:* do everything except compile
    -   *no_link:* do everything except link
    -   *one_step:* compile and link in one step
    -   *parse_only:* check configuration file syntax
    -   *show_only:* show requested commands but does not execute them
    -   *verbose:* show requested commands as they are executed

-   `set MakeOpt`
    Uncomment to build a Makefile to compile the executable.
-->

-   `set ParOpt`
    Build an executable for running on multiple processors. Invoking this command requires the availability of the MPI library/INCLUDE files.

-   `set MakeFileOnly`
    Builds a Makefile to make the model, but does not compile

-   `#set build_parallel_io`
     Uncomment to build CMAQ with true parallel I/O feature (requires ioapi3.2 and pnetcdf)

-   `#set build_twoway`
    Uncomment to build WRF-CMAQ twoway - this cannot be set for stand-alone CMAQ

-   `#set potvortO3`
    Uncomment to build CMAQ with potential vorticity free-troposphere O3 scaling

CCTM Science Modules Selection
```Tcsh
NOTE: For the modules with multiple choices, choose by uncommenting.
#> NOTE: For the modules with multiple choices, choose by uncommenting.
#> look in the CCTM source code repository or refer to the CMAQ documentation
#> for other possible options. Be careful. Not all options work together. 
```
-   `ModDriver: [default: driver/wrf]`
    The CCTM generalized -coordinate driver module.
    - `set ModDriver = driver/wrf` : use WRF-based scheme for mass-conserving advection; select this option when using WRF meteorology
    - `#set ModDriver = driver/yamo` : use Yamartino scheme for mass-conserving advection

-   `ModGrid: [default: Cartesian]` 
    The CCTM model grid configuration module. Currently only Cartesian coordinates are supported by CMAQ.
    -   `set ModGrid = grid/cartesian`

-   `ModInit: [default: init/yamo]`
    The CCTM time-step initialization module that uses a Yamartino scheme for mass-conserving advection
    -   `set ModInit = init/yamo`

-   `ModCpl: [default: couple/gencoor_wrf]`
    Mass coupling concentration converstion module options. Unit conversion and concentration coupling module.
    -   `set ModCpl  = couple/gencoor_wrf`  - Coupling scheme compatible with the WRF-based advection scheme; select this option when ModDriver is set to *ctm_wrf*
    -  `#set ModCpl  = couple/gencoor`      - Coupling scheme compatible with the Yamartino advection scheme; select this option when ModDriver is set to *ctm_yamo*.

<!-- -   `ModAdjc: [default: // Yamartino option]`
    Mass conservation error adjustment scheme. Corrects for mass inconsistencies arising from how the input meteorology treats density and wind fields. This adjustment is required only if the air-density-based scheme for mass-conserving advection is selected. Not used in CMAQv5 because the Yamartino advection scheme is mass conserving. -->

-    `ModHadv: [default: hadv/yamo]`
      Horizontal advection module.  Currently only the Yamartino global mass-conserving hoizontal advection scheme is supported.
     -   `set ModHadv  = hadv/yamo`

-   `ModVadv: [default: vadv/wrf]`
    Vertical advection module.
    -   `set ModVadv = vadv/wrf`   : use the WRF omega calculation with the Piecewise Parabolic Method (PPM) to calculate vertical advection; this module should be used only with WRF meteorology
    -   `#set ModVadv = vadv/yamo`  : use the global mass-conserving scheme to calculate vertical advection

-   `ModHdiff: [default: hdiff/multiscale]`
    The only option in CMAQv5 for the horizontal diffusion module is *hdiff/multiscale*, which uses a diffusion coefficient based on local wind deformation.
    -   `set ModHdiff = hdiff/multiscale`

-   `ModVdiff: [default: vdiff/acm2]`
    Vertical diffusion and surface exchange module.
    -   `set ModVdiff = vdiff/acm2`   : calculate vertical diffusion using the Asymmetric Convective Model version 2 (ACM2)
    <!-- -   *acm2_mp*: use ACM2 vertical diffusion instrumented for multipollutant modeling -->

-   `ModDepv: [default: depv/m3dry]`
    Deposition velocity calculation module.
    -   `set ModDepv = depv/m3dry`   : CMAQ dry deposition velocity routine
    <!-- -   *m3dry_mp*: CMAQ dry deposition velocity routine instrumented for multipollutant modeling -->

-   `ModEmis: [default: emis/emis]`
    - `set ModEmis = emis/emis` : CMAQ in-line anthropogenic and natural emissions module.

-   `ModBiog: [default: biog/beis3]`
    - `set ModBiog = biog/beis3` : Calculate biogenic emissions in-line with the BEIS3 model.

-   `ModPlmrs: [default: plrise/smoke]`
    - `set ModPlmrs = plrise/smoke` : Calculate in-line plume rise for large point sources using the Briggs algorithm as it is implemented in SMOKE.

-   `ModCgrds: [default: spcs/cgrid_spcs_nml]`
    CMAQ model species configuration module.
    -   `set ModCgrds = spcs/cgrid_spcs_nml` : namelist files used to configure CMAQ model species
    -   `#set ModCgrds = spcs/cgrid_specs_icl` : use Fortran INCLUDE files used to configure CMAQ model species

-   `ModPhot: [default: phot/inline]`
    Photolysis calculation module.
    -   `set ModPhot = phot/inline` : calculate photolysis rates in-line using simulated aerosols and ozone concentrations
    -   `#set ModPhot = phot/table` : calculate clear-sky photolysis rates off-line using the CMAQ program JPROC; provide daily photolysis rate look-up tables to CCTM

-   `Mechanism: [default: cb05e51_ae6_aq`]
    -  `set Mechanism = cb05e51_ae6_aq` : optimized for the Carbon Bond-05 mechanism with chlorine 
    - `#set Mechanism = cb05e51_ae6nvPOA_aq` : use .. 
    - `#set Mechanism = cb05eh51_ae6_aq` : use ..
    - `#set Mechanism = cb05mp51_ae6_aq` : use ..
    - `#set Mechanism = cb05tucl_ae6_aq`: use the Euler Backward Iterative solver optimized for the Carbon Bond-05 mechanism with chlorine and updated toluene chemistry
    - `#set Mechanism = cb05tump_ae6_aq`: use the Euler Backward Iterative solver optimized for the Carbon Bond-05 mechanism with updated toluene chemistry and air toxics; this is the multipollutant mechanism in CMAQv5'' ''
    - `#set Mechanism = cb6r3_ae6_aq` : use ..
    - `#set Mechanism = cb6r3_ae6nvPOA_aq` : use ..
    - `#set Mechanism = racm2_ae6_aq` : use ..
    - `#set Mechanism = saprc07tb_ae6_aq`: use the Euler Backward Iterative solver optimized for the SAPRC-07 mechanism with updated toluene chemistry version B
    - `#set Mechanism = saprc07tc_ae6_aq`: use the Euler Backward Iterative solver optimized for the SAPRC-07 mechanism with updated toluene chemistry version C
    - `#set Mechanism = saprc07tc_ae6nvPOA_aq` : use ..
    - `#set Mechanism = saprc07tic_ae6i_aq` : use ..
    - `#set Mechanism = saprc07tic_ae6invPOA_aq` : use ..
    - `#set Mechanism = saprc07tic_ae6i_aqkmti` #> if using ebi solver, use ebi_saprc07tic_ae6i_aq

-   `Tracer [default trac0] `
    Specifies tracer species to use from the emissions, IC, or BC input files. Invoking inert tracer species in CMAQ requires defining the tracers using INCLUDE files and compiling CCTM with these files. The setting for this module corresponds to the directory name in the $CMAQ_HOME/include/release directory that contains the INCLUDE files for the tracer configuration to implement in CCTM. The default setting is to not use tracers in CCTM.
    - `set Tracer  = trac0`

-   `ModMech: MECHS/$Mechanism`
     Specifies the chemistry mechanism selected above.

-   `ModGas: [default: gas/ebi]`
     Gas-phase chemistry solver module.
     -  `#set ModGas = smvgear` : use the SMVGEAR chemistry solver
     -  `#set ModGas =  ros3`   : use the Rosenbrock chemistry solver
     -  `set ModGas =  ebi`: use the Euler Backward Iterative solver

-   `ModAero: [default: aero6]`
    CMAQ aerosol module.
    <!-- -   *aero5*: fifth-generation modal CMAQ aerosol model with extensions for sea salt emissions and thermodynamics; includes a new formulation for secondary organic aerosol yields -->
    -   set ModAero = aero6` : sixth-generation modal CMAQ aerosol model with extensions for sea salt emissions and thermodynamics; includes a new formulation for secondary organic aerosol yields
    <!-- -   *aero6_mp*: sixth-generation CMAQ aerosol model including air toxics; this is the multipollutant mechanism in CMAQv5 (prior to CMAQv5.2) -->

-   `ModCloud: [default: cloud/acm_ae6]`
    CMAQ cloud module for modeling the impacts of clouds on deposition, mixing, photolysis, and aqueous chemistry.
    <!-- -   *cloud_acm_ae5*: ACM cloud processor that uses the ACM methodology to compute convective mixing with heterogeneous chemistry for AERO5 -->
    -   `set ModCloud = cloud/acm_ae6`: ACM cloud processor that uses the ACM methodology to compute convective mixing with heterogeneous chemistry for AERO6
    -   `set ModCloud = cloud/acm_ae6_mp`: ACM cloud processor that uses the ACM methodology to compute convective mixing with heterogeneous chemistry for AERO6 and air toxics; this is the multipollutant mechanism in CMAQv5
    -   `set ModCloud = cloud/acm_ae6_kmt`: ACM cloud processor that uses the ACM methodology to compute convective mixing with heterogeneous chemistry for AERO6 and aqueous chemistry with kinetic mass transfer and Rosenbrock solver
    -   `set ModCloud = cloud/acm_ae6i_kmti`: ACM cloud processor that uses the ACM methodology to compute convective mixing with heterogeneous chemistry for AERO6 and aqueous chemistry with kinetic mass transfer and Rosenbrock solver with an extension to simulate the aqueous phase formation of SOA in cloud droplets (see: [CMAQv5.1 Aqueous Chemistry](https://www.airqualitymodeling.org/index.php/CMAQv5.1_Aqueous_Chemistry))

-   `ModUtil: [default: util]`
    CMAQ utility modules.
    -  `set ModUtil = util/util`: only configuration option at the module level

-   `ModPvO3: [default: pv_o3]`
    Potential vorticity
    - `set ModPvO3 = pv_o3`: parameterized free-troposphere ozone module.

<!-- -   `PABase = $GlobInc`
    Specifies the base directory location of the process analysis INCLUDE files to use when compiling CCTM. -->

-   `PAOpt: [default: pa_noop]`
    Specifies the process analysis configuration to use for CMAQ. The choices for the PAOpt variable are the available directories for process analysis INCLUDE files under the $M3MODEL/include/release directory.
     - `set PAOpt = pa_noop`

#### CCTM compilation

First, it is assumed that you have already installed and compiled the I/O API, netCDF, and MPICH libraries, or that these are already available from a previous CMAQ compilation.

[Chapter 5](CMAQ_OGD_ch05_sys_req.md) provides an overview of how to install and compile the CMAQ programs for the benchmark simulation. Follow the steps outlined in [Chapter 5](CMAQ_OGD_ch05_sys_req.md) (summarized below) to compile new versions of CCTM:

-   If you have not already done so, compile Bldmake, the CMAQ source code and compilation management program. This needs to be done only once—the first time CMAQ is installed.
-   If needed, configure the CCTM build script to use the available I/O API, netCDF, and MPICH libraries.
-   Configure the CCTM build script for your application (using the options discussed in [Chapter 5](CMAQ_OGD_ch05_sys_req.md)
-   Invoke the build script to create an executable:

`cd $CMAQ_HOME/CCTM/scripts/`
`./bldit.cctm`

#### CCTM execution options

The environment variables listed here are invoked during execution of the program and are set in the CCTM run script.

-   `PROC [default: mpi]` To specify if the run is to be run on 1 processor (serial) or to be run on multiple processors (mpi).

-   `EXEC [default: CCTM_$APPL_$EXECID]`

Executable to use for the simulation.

-   `NPCOL_NPROW [default: 1 1]` : if PROC is serial

-   `NPCOL_NPROW [default: 4 4]` : if PROC is mpi

Domain decomposition for parallel mode; recommended configuration is for the number of columns to be larger than the number of rows. For example, if running with 8 processors, the recommended setting is “4 2”.

-   `NPROCS [default: 1]` : if PROC is serial
-   `NPROCS [default: 16]` : if PROC is mpi and NPCOL_NPROW = "4 4"

Number of processors for parallel execution; equal to the product of NPCOL x NPROW.

Timestep run parameters

-   `STDATE` : Simulation start date in Julian format (YYYYDDD).
-   `STTIME` : Simulation start time (HHMMSS).
-   `NSTEPS [default: 240000]`:  Number of simulation time steps (HHMMSS).
-   `TSTEP [default: 010000]` : Simulation output time step interval (HHMMSS).
-   `YEAR ` : Simulation year specified using 4 digits (YYYY).
-   `YR` : Simulation year specified using 2 digits (YY) 
-   `MONTH` : Simulation Month specified using 2 digits (MM)
-   `DAY` : Simulation Day specified using 2 digits (DD)
-   `YMD` : Calculated using YMD   = ${YEAR}${MONTH}${DAY}

CCTM Configuration Options.
-   `LOGFILE [default: $BASE/$APPL.log]` : Uncomment to capture CCTM standard output to a log file; the LOGFILE variable sets the name and location of the log.

-   `GRID_NAME [default: CMAQ-BENCHMARK]` : Name of the grid definition contained in the GRIDDESC file that specifies the horizontal grid for the current application of the model.
-   `GRIDDESC [default: $CMAQ_HOME/scripts/GRIDDESC1]` : Grid description file for setting the horizontal grid definition.

-   `OUTDIR [default: $CMAQ_HOME/data/cctm]` :  CCTM output file directory location.
-   `CTM_APPL [default: $APPL]` : CCTM log file naming extension.
-   `CONC_SPCS [default: if not defined, all species]`:  Model species to be written to the CCTM CONC file.
-   `CONC_BLEV_ELEV [default: if not defined, all layers]` :  Vertical model layer range for the CONC-file concentrations; this variable sets the lower and upper layers over which to output the CONC file.
-   `AVG_CONC_SPCS [default: `if not defined, output all species] :  Model species for calculating integral average concentrations for each output time step. Options can be any of the standard output species that are written to the CCTM CONC file. The species in this list will be written to the ACONC output file.
-   `ACONC_BLEV_ELEV [default: if not defined, all layers]` :  Vertical model layer range for integral average concentrations; this variable sets the lower and upper layers over which to calculate integral average concentrations. For example, setting this variable to “1 5” will produce integral average concentrations for model layers 1 through 5.
-   `ACONC_END_TIME [default: N|F]`:  Change the time stamp of the ACONC file output time step from the default of the beginning of the hour to the end of the hour. Set to Y or T to set the time stamp to the end of each hour or set to N or F to set the time stamp to the beginning of the hour.

-   'EXECUTION_ID' Define the model execution id

Synchronization Time Step and Tolerance Options
-   `CTM_MAXSYNC [default: 720]`:  Maximum synchronization time step in seconds.
-   `CTM_MINSYNC [default: 60]` : Minimum synchronization time step in seconds.
-   `SIGMA_SYNC_TOP [default: .70]`: Top sigma level thru which sync step determined
-   `CTM_ADV_CFL [default: .75]`: maximum Courant–Friedrichs–Lewy (cfl) condition
-   `RB_ATOL [default: 1.0E-07]`: global ROS3 solver absolute tolerance

Science Options
-   `CTM_WB_DUST [default: Y|T]`:  Setting to calculate in-line windblown dust emissions in CCTM. Setting this variable to Y or T requires the availability of gridded land use input files that include the following BELD USGS land use classifications: shrubland, shrubgrass, and sprsbarren. See [Chapter 8](CMAQ_OGD_ch08_input_files.md#Table8-1) for a description of the DUST_LU_1 and DUST_LU_2 input files. Comment out variable or set to Y or T to turn on; set to N or F to turn off.
-   `CTM_ERODE_AGLAND [default: N|F]` Setting to use optional erodible agricultural land classifications for computing windblown dust emissions from agricultural land. Setting this variable to Y or T requires the availability of gridded crop timing data that describe planting start dates, planting end dates, and harvesting end dates for 18 crop types. See [Chapter 8](CMAQ_OGD_ch08_input_files.md#Table8-1) for a description of the CROPMAP01, CROPMAP04, and CROPMAP08 input files. If CTM_WB_DUST is set to N or F, this setting will be ignored. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-    `CTM_WBDUST_BELD [default: BELD3]:  Landuse database for identifying dust source regions;  ignore if CTM_WB_DUST = N
-   `CTM_LTNG_NO [default: Y|T]` : Setting to activate lightning NO emissions. Setting this variable to Y or T requires additional variables to define the configuration of the lightning NO emissions calculation. See the settings for LTNGNO, LTNGPARAM, and LTNGDIAG below. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-   `CTM_WVEL [default: N|F]`: Setting to output the CCTM-calculated vertical velocities to the CONC file. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-   `KZMIN [default: Y]`: If KZMIN is set to Y, CCTM will read the urban land use fraction variable (PURB) from the GRID_CRO_2D meteorology file and use this information to determine the minimum eddy diffusivity in each grid cell. In CMAQv5, grid cells that are predominantly urban use a KZMIN value of 1.0 m<sup>2</sup>/s and nonurban cells use a value of 0.01 m<sup>2</sup>/s. If this variable is set to N, the PURB variable will not be used and a uniform KZMIN value of 1.0 m<sup>2</sup>/s will be used throughout the modeling domain.
-   `CTM_ILDEPV [default: Y|T]`: Calculate in-line deposition velocities. Comment out variable or set to Y or T to turn on; set to N or F to turn off.
-   `CTM_MOSAIC [default N|F]`:  Setting to output land use specific deposition velocities and fluxes.
    -    `CTM_FST [devault: N|F]`:  Mosaic method to get land-use specific stomatal flux.
-   `CTM_ABFLUX [default: N|F]`:  Setting to activate fertilizer ammonia bidirectional flux for in-line emissions and deposition velocities. If CTM_ILDEPV is set to N or F this variable is ignored. Setting this variable to Y or T requires four additional input files that include gridded fractional crop distributions (B4LU_file), soil properties (E2C_Soilfile), fertilizer conditions (E2C_Fertfile), and an agricultural soil initial conditions file (INIT_MEDC_1). Activation of this setting will produce additional variables in the output dry deposition file. See [Chapter 8](CMAQ_OGD_ch08_input_files.md#Table8-1) for a description of the required input files. Set to Y or T to turn on; comment out or set to N or F to turn off.
-   `CTM_HGBIDI [default: N|F]`: Setting to activate mercury bidirectional flux for in-line emissions and deposition velocities. If CTM_ILDEPV is set to N or F this variable is ignored. Activation of this setting will produce additional variables in the output
 dry deposition file. Set to Y or T to turn on; comment out or set to N or F to turn off.
-   `CTM_SFC_HONO [default: Y|T]`:  Calculate surface HONO interactions. If CTM_ILDEPV is set to N or F this variable is ignored. Comment out or set to Y or T to turn on; set to N or F to turn off.
-   `CTM_GRAV_SETL [default N|F]`: Vertical diffusion aerosol gravitational sedimentation option.
-   `CTM_BIOGEMIS [default: Y|T]`:  Calculate biogenic emissions. Comment out or set to Y or T to turn on; set to N or F to turn off.  If the option to calculate in-line biogenic emissions is activated (i.e., CTM_BIOGEMIS is set to Y or T), the following variables must be set.     
    -   `GSPRO`: Directory path and file name for input speciation profiles.
    -   `B3GRD`: Directory path and file name for grid-normalized biogenic emissions input file.
    -   `BIOG_SPRO`:  Profile ID for speciating biogenic VOCs. This profile ID must be present in the GSPRO file.
    -   `BIOSW_YN [default: Y|T]`: Use the frost dates switch file to determine whether to use winter or summer biogenic emissions. Comment out or set to Y or T to turn on; set to N or F to turn off.
    -   `BIOSEASON`:  Directory path and file name for the frost date switch file.
    -   `SUMMER_YN [default: Y|T]`:  Use summer season normalized biogenic emissions. This variable is ignored if BIOSW_YN is set to Y. Comment out or set to Y or T to turn on; set to N or F to turn off.
    -   `PX_VERSION [Default: N|F]`:  Setting to indicate whether the Pleim-Xiu land-surface model was used for the input meteorology. If this setting is set to Y or T the input meteorology data must include soil moisture (SOILM), soil temperature (SOILT), and soil type (ISLTYP) variables for use in the calculation of soil NO emissions.
    -   `INITIAL_RUN`:  Set to Y or T if this is the first time that biogenic NO soil emissions will be calculated. If there is a previously created file, set to N or F.
    -   `SOILINP`:  Directory path and file name of biogenic NO soil emissions file. If INITIAL_RUN is set to N or F, the soil NO emissions file from the previous day’s simulation will be a required input file.
    -   `B3GTS_DIAG [default: N|F]`:  Setting to write the calculated biogenic emissions (mass units) to a diagnostic netCDF output file (B3GTS_S). Set to Y or T to turn on; comment out or set to N or F to turn off.
    -   `B3GTS_S`:  Directory path and file name for the diagnostic output biogenic emissions. This variable is ignored if B3GTS_DIAG is turned off.
-   `CTM_PT3DEMIS [default: N|F]`:  Calculate plume rise for elevated point sources. Set to Y or T to turn on; comment out or set N or F to turn off.
    If the option to calculate in-line plume rise is activated (i.e., CTM_PT3DEMIS is set to Y or T), the following variables must be set.
    -   `NPTGRPS [default: 1]`: The number of input point-source elevated emission sector file groups. A maximum of 9 sectors is allowed.
    -   `STK_GRPS_## ` : Directory path and file name of the stack groups file for sector ##, where ## = 01, 02,…,NPTGRPS. Each ## refers to one of the plume rise point-source sectors.
    -   `STK_EMIS_##` : Directory path and file name of the point emissions file for sector ##, where ## = 01, 02,…,NPTGRPS. Each ## refers to the one of the plume rise point-source sectors.
    -   `LAYP_STDATE [HHMMSS]` : Start date for calculating elevated-point-source emissions.
    -   `LAYP_STTIME [HHMMSS]` : Start time for calculating elevated-point-source emissions.
    -   `LAYP_NSTEPS [HHHHHH]` : Number of time steps for calculating elevated-point-source emissions.

<--
    -   `CTM_EMLAYS [##]` : Number of emissions layers for calculating elevated-point-source emissions.
-->
    -   `PT3DDIAG [default: N]` : Setting to write the in-line 3-D point-source emissions to a diagnostic netCDF output file (CTM_PT3D_DIAG). Set to Y to turn on; comment out or set to N to turn off.
    -   `PT3DFRAC [default: N]` : Setting to write the in-line 3-D point-source layer fractions to a diagnostic netCDF output file (PLAY_SRCID_NAME). Set to Y to turn on; comment out or set to N to turn off.
-   `REP_LAYER_MIN [default: -1]` : Minimum layer for reporting plume rise info

`[Add content]`


Lightning NOx configuration
-   `LTNGNO [default:InLine]`:  Setting to define whether the lightning emissions calculation will be in-line or off-line. This variable can be set to a gridded netCDF file of lightning NO emissions to use emissions calculated with a preprocessor outside of CCTM. Setting this variable to “inline” activates the in-line emissions calculation in CCTM and requires the LTNGPARM variable (see below) to define the configuration of the in-line emissions.
-    `USE_NLDN [default: Y|T]` : Uuse hourly NLDN strike file 
-   `LTNGPARAM [default: Y|T]` Setting to define the configuration of in-line lightning NO emissions. When the variable LTNGNO is set to “inline”, this setting is used to define how the in-line emissions will be calculated. Commenting out this variable or setting it to Y or T will compute lightning NO from input flash count observations. Setting this variable to N or F will compute lightning NO strictly from convective precipitation rates in the input meteorology data. When this variable is set to Y or T, an additional input lightning parameter file (LTNGPARM_FILE) will need to be available that includes gridded monthly flash counts, intercloud to cloud-to-ground flash ratios, scaling factors for calculating flashes using the convective precipitation rate, and the moles of NO per flash.
-  `NLDN_STRIKES` : ??
-  `LOG_START`    : 2.0   #> RC value to transit linear to log linear
-  `LTNGPARMS_FILE` : lightning parameter file; ignore if LTNGPARAM = N
-  `LTNGOUT` : lightning diagnostic file; ignore if LTNGDIAG = N

In-line biogenic emissions configuration
-  add content here

Windblown dust emissions configuration
-  add content here

In-line sea salt emisisions configuration
-   `OCEAN_1` : Directory path and file name for the ocean mask file to be used for calculating sea salt emissions.

Bidiretional ammonia configuration

I/O Controls
-   `IOAPI_LOG_WRITE [default: F]` :  `[T|F]; set to “T” to turn on excess WRITE3 logging by the I/O API.`
-   `FL_ERR_STOP [default: F]` : [T|F|Y|N]; set to “T” or “Y” to configure the program to exit if inconsistent headers are found in the input files.
-   `PROMPTFLAG [default: F]` : turn on I/O-API PROMPT*FILE interactive mode
-   `IOAPI_OFFSET_64: [default: NO]`:  I/O API setting for large time step records. If your output time step is going to produce data that are >2GB per time step, then this needs to be set to YES.

AeroSprint Controls
-   `CTM_AVISDIAG` : Diagnostic file
-   `CTM_PMDIAG`   : ??
-   `CTM_APMDIAG`  : ??
-   `APMDIAG_BLEV_ELEV` : layer range for average pmdiag
-   `APMDIAG_BLEV_ELEV` : layer range for average pmdiag = NLAYS
-   `AVG_FILE_ENDTIME` : ??

Diagnostic Output Flags
-   `CTM_CKSUM [default: Y|T]`
-   `CLD_DIAG [default: N|F]` : Setting to output an hourly wet deposition diagnostic file (CTM_WET_DEP_2) that includes convective wet deposition estimates. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-   `CTM_AERDIAG [default: N|F]` :  Setting to output an instantaneous hourly aerosol diagnostic file (CTM_DIAM_1) with the geometric mean diameters and the geometric standard deviations for the lognormal aerosol modes. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-   `CTM_PHOTDIAG [default: N|F]`: Setting to output the in-line photolysis rates and associated data to diagnostic netCDF output files. The file CTM_RJ_1 contains gridded photolysis rates for O3 (JO3O1D) and NO2 (JNO2) that include both clear-sky and cloud effects, total downward irradiance at the surface (ETOT_SFC_W), aerosol optical depth (TAU_AERO_W), total optical depth (TAU_TOT_W), optical depth of ozone above the model domain (TAUO3_TOP_W), Rayleigh optical depth above the model domain (TAU_RAY_W), and surface albedo (ALBEDO_W). The file CTM_RJ_2 contains gridded photolysis rates for all other photolysis reactions in the selected chemical mechanism. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-   `CTM_SSEMDIAG [default: N|F]` : Setting to output the calculated sea salt emissions to a diagnostic netCDF output file (CTM_SSEMIS_1). Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-   `CTM_DUSTEM_DIAG [default: N|F]`: Setting to output the in-line dust emissions to a diagnostic netCDF output file (CTM_DUST_EMIS_1). The diagnostic file includes not only the total dust emissions, but also dust emissions by land use category and dust model parameters, such as gridded erodible land use fractions. Set to Y or T to turn on; comment out variable or set to N or F to turn off. 
-   `CTM_DEPV_FILE [default: N|F]` : Generate an hourly diagnostic file (CTM_DEPV_DIAG) for the in-line deposition velocity calculations. If CTM_ILDEPV is set to N or F this variable is ignored. Set to Y or T to turn on; comment out or set to N or F to turn off.
-   `VDIFF_DIAG_FILE [default: N|F]`: Generate a diff & possibly aero grav. sedimentation diagnostic file
-   `LTNGDIAG [default: N|F]` : Setting to output the in-line lightning NO emissions to a diagnostic netCDF output file (LTNGOUT). The diagnostic file includes gridded, hourly average NO produced from lightning. Set to Y or T to turn on; comment out variable or set to N or F to turn off.
-    `CTM_AOD [default N|F]`:  AOD Disgnostic file.
     - `AODfile`:  File name containing aerosol optical depth values.
-   `B3GTS_DIAG [default: N|F]`: Setting to write the calculated biogenic emissions (mass units) to a diagnostic netCDF output file (B3GTS_S). Set to Y or T to turn on; comment out or set to N or F to turn off.k
-   `PT3DDIAG [default: N]`:  Setting to write the in-line 3-D point-source emissions to a diagnostic netCDF output file (CTM_PT3D_DIAG). Set to Y to turn on; comment out or set to N to turn off.
-   `PT3DFRAC [default: N]`:  Setting to write the in-line 3-D point-source layer fractions to a diagnostic netCDF output file (PLAY_SRCID_NAME). Set to Y to turn on; comment out or set to N to turn off.
-   `REP_LAYER_MIN [default: -1]` : Minimum layer for reporting plume rise info

MPI Optimization Flags
-   `MPI_SM_POOL` : Increase shared memory pool in case many MPI_SEND headers
-   `MP_EAGER_LIMIT` : Set MPI message passing buffer size to max
-   `MP_SINGLE_THREAD [default: N] : Optimizate for single threaded applications
-   `MP_STDOUTMODE` [default: ordered]: order output by the processor ID
-   `MP_LABELIO [default: N]` label output by processor ID
-   `MP_SHARED_MEMORY default: N]` : force use of shared memory for tasks on same node
-   `MP_ADAPTER_USE [default: shared]` : share the MP adapter with other jobs
-   `MP_CPU_USE [default: multiple]` : share the node with multiple users/jobs
-   `MP_CSS_INTERRUPT [default: N]` : specify whether arriving packets generate interrupts 

-   `DISP [default: keep]`
    Controls the maintenance of existing log files.
    -   *delete:* delete output log if it already exists
    -   *keep:* abort simulation if output log exists

Input/Output Directories 

add content here ??


Input Files
add content here ??
-   `ICFILE` : Initial conditions file
-   `BCFILE` : Boundary conditions file
-   `JVALfile` : Off-line photolysis rate file
-   `OMIfile` : Ozone column data
-   `OPTfile` : Optics file
Directory path and file names for the input meteorology files; generated by MCIP.
-   <tt>GRID_DOT_2D
-   GRID_CRO_2D
-   MET_CRO_2D
-   MET_CRO_3D
-   MET_DOT_3D
-   MET_BDY_3D</tt>
Offline Emission File
-   `EMISfile`
Inline Emissions Files
-   

-   `EMIS_1` : Directory path and file name for the input emissions file.
-   `TR_EMpath`
-   `TR_Emfile`

Directory path and file name for the input tracer emissions file.

-   `INIT_[GASC|AERO|NONR|TRAC]_1 `:  Directory path and file name for the input initial conditions file for gases, aerosols, nonreactive species, and tracers; generated by ICON.

-   `$INIT_MEDC_1` : Directory path and file name for the input initial conditions for the bidirectional surface exchange model. These variables are used only when CCTM is compiled to calculate the bidirectional surface exchange for mercury.

-   `BNDY_[GASC|AERO|NONR|TRAC]_1` : Directory path and file name for the input boundary conditions file for gases, aerosols, nonreactive species, and tracers; generated by BCON.

-   `XJDATA`

Directory path and file name of input clear-sky photolysis rate files; generated by JPROC.

-   `OMI`

Directory path and file name for ozone monitoring instrument look-up table.

#### CCTM output files

[Table 7-7](#Table7-7) lists the logical file names, formats, and descriptions of the output files that are produced by the base configuration of CCTM. Activating different science modules, in-line deposition, and in-line emissions processing produces additional output files from CCTM. [Table 7-8](#Table7-8) lists the logical file names, formats, and descriptions of the output files that are produced by optional configurations of CCTM.

<a id=Table7-7></a>

<center>**Table 7‑7. CCTM base output files**</center>

|File Name|Format|Description|
|---------|------|-----------|
|CTM_CONC_1|GRDDED3|Name and location of hourly 3-D instantaneous gas- and aerosol-phase pollutant estimates|
|CTM_CGRID_1|GRDDED3|Name and location of simulation-ending 3-D full CGRID (gas- and aerosol-phase pollutants) concentrations for use as a restart file|
|CTM_ACONC_1|GRDDED3|Name and location of hourly 2-D or 3-D integral average gas- and aerosol-phase pollutant estimates|
|CTM_DRY_DEP_1|GRDDED3|Name and location of hourly 3-D gas- and aerosol-phase dry deposition estimates|
|CTM_WET_DEP_1|GRDDED3|Name and location of hourly 3-D gas- and aerosol-phase wet deposition estimates|
|CTM_VIS_1|GRDDED3|Name and location of hourly 3-D visibility metrics|

<a id=Table7-8></a>

<center>**Table 7‑8. CCTM optional output files**</center>

|File Name|Format|Description|
|---------|------|-----------|
|CTM_SSEMIS_1|GRDDED3|Name and location of hourly 2-D sea salt emissions; set the variable CTM_SSEMDIAG to “T” or “Y” in the CCTM to run script to write this file|
|CTM_WET_DEP_2|GRDDED3|Name and location of hourly 2-D cloud diagnostics file; set the variable CLD_DIAG to “T” or “Y” in the CCTM run script to write this file|
|CTM_DEPV_DIAG|GRDDED3|Name and location of hourly 2-D in-line deposition diagnostics file; output when in-line deposition is activated by setting CTM_ILDEPV to “T” or “Y” and the variable CTM_DEPV_FILE is set to “T” or “Y” in the CCTM run script|
|CTM_DIAM_1|GRDDED3|Name and location of hourly 3-D aerosol diagnostics; dp and sigmas for Aitken and accumulation mode aerosol species; set the variable CTM_AERODIAG to “T” or “Y” in the CCTM run script to write this file|
|CTM_IPR_[1-3]|GRDDED3|Name and location of hourly 2-D or 3-D integrated process rate (IPR) files; multiple files written when CCTM is configured to run with IPR|
|CTM_IRR_[1-3]|GRDDED3|Name and location of hourly 2-D or 3-D integrated reaction rate (IRR) files; multiple files written when CCTM is configured to run with IRR|
|CTM_RJ_[1-2]|GRDDED3|Name and location of hourly photolysis diagnostic output file; multiple files written when there are a large number of photolytic reactions in a chemical mechanism; set the variable CTM_PHOTDIAG to “T” or “Y” in the CCTM run script to write this file|
|B3GTS_S|GRDDED3|Name and location of hourly biogenic emissions file; output when in-line biogenic emissions processing is activated by setting CTM_BIOGEMIS to “T” or “Y” and the variable B3GTS_DIAG is set to “T” or “Y” in the CCTM run script|
|SOILOUT|GRDDED3|Name and location of hourly soil NO emissions file; output when in-line biogenic emissions processing is activated by setting CTM_BIOGEMIS to “T” or “Y”|
|CTM_DUST_EMIS_1|GRDDED3|Name and location of hourly 2-D dust emissions file; output when the CCTM dust module is activated by setting CTM_WB_DUST to “T” or “Y” and the variable CTM_DUSTEM_DIAG is set to T” or “Y” in the CCTM run script|
|LTNGOUT|GRDDED3|Name and location of hourly 3-D lighting emissions file; output when the CCTM lightning module is activated by setting CTM_LTNG_NO to “T” or “Y” and the variable LTNGDIAG is set to “Y” in the CCTM run script|
|CTM_PT3D_DIAG|GRDDED3|Name and location of hourly 3-D point-source emissions file; output when in-line emissions processing is activated by setting CTM_PT3DEMIS to “T” or “Y” and the variable PT3DDIAG is set to “Y” in the CCTM run script|
|PLAY_SRCID_NAME|GRDDED3|Name and location of hourly 3-D layer fractions file; output when in-line emissions processing is activated by setting CTM_PT3DEMIS to “T” or “Y” and the variable PT3DFRAC is set to “Y” in the CCTM run script|
|INIT_MEDC_1|GRDDED3|Name and location of hourly mercury deposition output file; output when bidirectional mercury flux is activated by setting the variable CTM_HGBIDI to “T” or “Y” in the CCTM run script|

The default location of the CCTM output files is the `$CMAQ_HOME/data/cctm` directory, controlled by the `OUTDIR` variable in the run script. The default naming convention for all CCTM output files uses the `EXEC` and `APPL` environment variables in the file name. All of the variables for naming the CCTM outputs are set in the run script.

<a id="CHEMMECH"></a>
## CHEMMECH and CSV2NML

### Description

The program CHEMMECH generates mechanism INCLUDE files for all chemical mechanism-dependent CMAQ programs. Using an ASCII mechanism definition file as input, the Fortran program CHEMMECH creates all of the Fortran INCLUDE files that define the gas-phase chemical mechanisms for the CMAQ programs. The C-Shell script CSV2NML converts a comma-delimited text file that defines the processes (e.g., input as emissions, input through boundary conditions, transport, deposition) impacting the concentrations of each model species to a NAMELIST file for input to the CMAQ programs. In combination the INCLUDE and NAMELIST files define chemical mechanisms in the CMAQ programs.

Implementing new mechanisms created by CHEMMECH and CSV2NML in the CMAQ programs is a two-step process. CHEMMECH generates mechanism INCLUDE files that must be included in the compilation of CMAQ source code into an executable. CSV2NML generates species NAMELIST files that are input to the CMAQ programs during execution. Care must be taken to ensure that the INCLUDE and NAMELIST files are consistent with each other in order to correctly update or implement new mechanisms in CMAQ.

CHEMMECH reads in a mechanism definition (mech.def) text file that lists the stoichiometry and kinetics of a photochemical reaction mechanism. The program converts the mech.def file to two INCLUDE files, RXDT.EXT and RXCM.EXT, that get compiled with the CMAQ source code into a new executable. The INCLUDE files created by CHEMMECH must be manually moved to the correct location in the CMAQ source code directories to be available during compilation of the CMAQ programs.

CSV2NML reads in a series of CSV files that define the processes that impact the concentrations of all of the CMAQ species. The CSV files are converted to NAMELIST files that are invoked at execution of the various CMAQ programs. Environment variables in the run scripts for ICON, BCON, and CCTM must be set to point to the NAMELIST files for a particular mechanism.

See [Chapter 9](CMAQ_OGD_ch09_grid_defn.md)) for details on how to update existing mechanisms or create new mechanisms in CMAQ.

### Files, configuration, and environment variables

[Figure 7‑5](#Figure5-4) shows the input and output files and configuration options for CHEMMECH and CSV2NML. The full set of mechanism INCLUDE files required by the CMAQ programs is generated in two steps. In the first step, the program CHEMMECH is run with the mechanism definition file, mech.def, provided as input. The resulting RXDT.EXT and RXCM.EXT INCLUDE files are then input to the CMAQ build scripts to compile CMAQ with a new chemical mechanism configuration. CSV2NML is used to convert the species definition files from CSV format to NAMELIST files. The NAMELIST files are used as inputs to the CMAQ programs ICON, BCON, or CCTM to define the processes that will impact each model species. Three NAMELIST files define the processes for gas-phase species (GC.nml), aerosol species (AE.nml), and nonreactive species (NR.nml).

<a id=Figure7-5></a>

<center>![](./images/Figure7-5.png "Figure7-5.png")</center>
<center>**Figure 7‑5. CHEMMECH and CSV2NML input and output files**</center>

<br>
<p>To implement a new mechanism in CMAQ, start with a mechanism definition (mech.def) file and CSV species files from an existing mechanism in the model. Edit the mech.def file to include the new reactions, species, and reaction rates and provide this new mech.def file as input to CHEMMECH. Edit the CSV species files to include the new species and provide these files as input to CSV2NML. Detailed examples of updating an existing mechanism and adding a new mechanism to CMAQ are provided in [Chapter 9](CMAQ_OGD_ch09_grid_defn.md). Neither CHEMMECH nor CSV2NML requires horizontal grid, vertical layer, or temporal settings.</p>

#### CHEMMECH input files

<a id=Table7-9></a>

<center>**Table 7-9. CHEMMECH input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|MCFL (mech.def)|ASCII|CMAQ mechanism definition file; photochemical mechanism listing with both mechanistic and kinetic information about all reactions that compose a chemical mechanism|

#### CHEMMECH compilation

CHEMMECH is compiled with a Makefile. The configuration options in the Makefile include only the compiler and compiler flags to use for building the executable. The Makefile is located in the directory with the CHEMMECH source code. To compile CHEMMECH, invoke the Makefile at the command line:

`./make`

To port CHEMMECH to different compilers, change the compiler names, locations, and flags in the Makefile.

#### CHEMMECH execution options

The environment variables listed here are invoked during execution of the program and are set in the **CHEMMECH** run script. The default run script is called MP.saprc99.csh.

-   `BASE: [default: $cwd]`

Working directory path

-   `Xpath: [default: $BASE]`

Executable directory path

-   `Mpath: [default: ../mech]`

Directory path to the mech.def file

-   `Opath: [default: ../exts]`

Output file directory path

-   `APPL:`

Simulation identifier

-   `EXEC: [default: CHEMMECH]`

Executable name

-   `MCFL: [default: mech.def.saprc99]`

Mechanism definition file name

-   `SPCSDATX: [default: $Opath/SPECIES.ext]`

Name of output species INCLUDE file

-   `RXNSDATX: [default: $Opath/RXDT.EXT]`

Name of output mechanism data INCLUDE file

-   `RXNSCOMX: [default: $Opath/RXCM.EXT]`

Name of output mechanism common INCLUDE file

#### CHEMMECH output files

<a id=Table7-10></a>

<center>**Table 7‑10. CHEMMECH output files**</center>

|File Name|Format|Description|
|---------|------|-----------|
|RXCM.EXT|ASCII|Mechanism common INCLUDE file; lists all of the chemical mechanism variables and parameters|
|RXDT.EXT|ASCII|Mechanism data INCLUDE file; chemical mechanism definition formatted as DATA blocks to be read in as CMAQ source code|
|SPCS.EXT|ASCII|Species INCLUDE file; not used|

The location of the CHEMMECH output files is set in the run script by the variable Opath. To compile a version of the CMAQ programs that use the INCLUDE files created by CHEMMECH, these output INCLUDE files need to be moved to a new directory under the `$CMAQ_HOME/models/mechs/release` directory. Point the CMAQ build scripts to this new directory through the “Mechanism” variable.

#### CSV2NML input files

Detailed descriptions of the formats of the files shown in [Table 7-11](#Table5-9) are provided in [Chapter 8](CMAQ_OGD_ch08_input_files.md). <a id=Table7-11></a>

<center>**Table 7‑11. CSV2NML input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|GC.csv|ASCII|Gas-phase species process parameters. This file defines the source and sink processes that impact the concentrations of every gas-phase species in the chemical mechanism.|
|AE.csv|ASCII|Aerosol-phase species process parameters. This file defines the source and sink processes that impact the concentrations of every aerosol-phase species in the chemical mechanism.|
|NR.csv|ASCII|Nonreactive species process parameters. This file defines the source and sink processes that impact the concentrations of every nonreactive species in the chemical mechanism.|

#### CSV2NML usage

The CSV2NML script is configured to read in a CSV file from the command line and output a NAMELIST file that can be used with CMAQ. An example of how to use CSV2NML to create a gas-phase species NAMELIST file is include below:

`./csv2nml.csh GC.CSV`

#### CSV2NML output files

<a id=Table7-12></a>
<center>**Table 7‑12. CSV2NML output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|GC.nml|ASCII|Gas-phase species process parameters. This file defines the source and sink processes that impact the concentrations of every gas-phase species in the chemical mechanism|
|AE.nml|ASCII|Aerosol-phase species process parameters. This file defines the source and sink processes that impact the concentrations of every aerosol-phase species in the chemical mechanism|
|NR.nml|ASCII|Nonreactive species process parameters. This file defines the source and sink processes that impact the concentrations of every nonreactive species in the chemical mechanism|

ICON
----

### Description

The program ICON prepares chemical initial conditions (ICs) for CCTM from either ASCII vertical profiles or from an existing CCTM output concentration (CONC) file. ICON creates an output file with a single time step that represents the chemical conditions in each grid cell at the beginning of a CCTM simulation. The ICs can be either spatially uniform or variable across the model grid, depending on the source of the initial chemical concentration data. If deriving ICs from the ASCII vertical profiles, ICON can create only spatially uniform ICs *within* each model layer; it can create different ICs *across* model layers. From CONC files, ICON can extract spatially varying ICs, either on the same grid cell resolution, as a windowed modeling domain, or for a finer-resolution model grid (as for a nested simulation).

There are two distinct modes of operation for ICON, depending on the nature of the input data. When creating ICON executables, the user must specify whether the input data will be ASCII vertical profiles or a CONC file by selecting either “profile” or “m3conc”, respectively, for the setting of the *ModInpt* variable. This variable determines the input module to use when creating an ICON executable.

CCTM can also be forced with initial conditions downscaled from global chemistry models (GCMs), such as GEOS-Chem and MOZART. ICON does not support the processing of data from GCMs. ICs derived from GCMs must be calculated with custom codes or scripts that are not available in the CMAQ distribution package.

### Files, configuration, and environment variables

[Figure 7‑6](#Figure7-6) shows the input and output files and configuration options for ICON. A distinction is made between the options that are invoked at compilation versus those invoked at execution of the program. When compiling ICON, the user specifies a chemical mechanism to determine the gas-phase chemistry and aerosol mechanism for which to calculate chemical ICs. Setting the *ModMech* and Mechanism variables in the ICON compile script configures the program to use a specific set of mechanism INCLUDE files to build an executable. Separate ICON executables are required for each mechanism configuration. Setting the *ModType* variable in the ICON compile script configures the program to input either a text file of static concentrations or a binary netCDF file of time-dependent concentrations for estimating ICs for CCTM. Separate ICON executables must be prepared for different mechanism and input file configurations.

<a id=Figure7-6></a>

<center>
![](./images/Figure7-6.png "Figure7-6.png")

</center>
<center>**Figure 7‑6. ICON input and output files**</center>

At execution the user provides a data file of chemical conditions that ICON converts to ICs on a predefined model grid. Through the specification of the *ModInpt* variable in the ICON run script, ICON will input either an ASCII vertical profile file (IC_PROFILE) or an existing CCTM concentration file (CTM_CONC_1); the choice depends on how the user compiled the model. The IC input file provided by the user must have chemical speciation that is consistent with the mechanism configuration of the ICON executable. For example, if ICON was compiled to create ICs using the CB05 mechanism, the input IC profile data must be in terms of the CB05 mechanism. CMAQv5 is distributed with ASCII vertical profiles representing clean continental ICs for North America for the following chemical mechanisms: cb05_ae6_aq, saprc07t_ae6_aq, and saprc99_ae6_aq. It is the user’s responsibility to generate IC inputs for other mechanism configurations.

The horizontal grid and vertical layer structures for ICON are defined at execution through the input of a grid description (GRIDDESC) file and a meteorology cross-point 3‑D (MET_CRO_3D) file, respectively. ICON interpolates between the input vertical layer structure and output layer structure if they are different.

#### ICON input files

<a id=Table7-13></a>

<center>**Table 7‑13. ICON input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|IC_PROFILE|ASCII|Vertical chemical profiles from which to derive initial conditions; this file is created by the user; used only when the IC environment variable is set to “profile”|
|CTM_CONC_1|GRDDED3|Name and location of the CMAQ concentration file from which to derive initial conditions; this file is output from CCTM; used only when the BC environment variable is set to “m3conc”|
|MET_CRO_3D_CRS|GRDDED3|Name and location of the coarse-grid MET_CRO_3D file that is required for creating the vertical grid structure if this structure changes between nested simulations; this file is output by MCIP|
|MET_CRO_3D_FIN|GRDDED3|Name and location of the fine grid MET_CRO_3D file that is required if the vertical grid structure changes between nested simulations; this file is output by MCIP|
|GRIDDESC|ASCII|Horizontal grid description file for defining the model grid; this file is output by MCIP or can be created by the user|
|LAYER_FILE|GRDDED3|3-D cross-point meteorology file for defining the vertical layer structure of the model grid; this file is output by MCIP|
|gc_matrix.nml|ASCII|Namelist file for defining the gas-phase species that are input to the model through the boundary|
|ae_matrix.nml|ASCII|Namelist file for defining the aerosol species that are input to the model through the boundary|
|nr_matrix.nml|ASCII|Namelist file for defining the nonreactive species that are input to the model through the boundary|
|tr_matrix.nml|ASCII|Namelist file for defining the tracer species that are input to the model through the boundary|

#### ICON compilation options

The configuration options listed here are set during compilation of the ICON executable. When these options are invoked they create a binary executable that is fixed to the specified configuration. To change these options you must recompile ICON and create a new executable.

-   `Opt: [default: verbose]`
    Defines the action to be taken by the program Bldmake when extracting source code from CVS and compiling an executable.
    -   *compile_all:* force compile, even if all the object files are current
    -   *clean_up:* remove all source files upon successful compilation
    -   *no_compile:* do everything except compile
    -   *no_link:* do everything except link
    -   *one_step:* compile and link in one step
    -   *parse_only:* check configuration file syntax
    -   *show_only:* show requested commands but do not execute them
    -   *verbose:* show requested commands as they are executed
-   `MakeOpt`
    Uncomment to build a Makefile to compile the executable.

-   `ModType: [default: profile]`
    Defines the format of the initial conditions input files to be used by ICON.
    -   *m3conc*: input a CCTM CONC file; used for nested simulations or windows of a parent domain
    -   *profile*: input an ASCII vertical profiles file
    -   *tracer*:
-   `ModMech: [default: cb05]`
    Defines the base gas-phase mechanism to use for creating boundary conditions.
    -   *cb05*: Carbon Bond version 05
    -   *saprc99*: SAPRC version 1999
    -   *saprc07t*: SAPRC version 2007 with updated toluene mechanism
-   `Mechanism: [default: cb05tucl_ae6_aq]`
    Specifies the gas-phase, aerosol, and aqueous-phase chemical mechanisms for which to create initial conditions. The choices for the *Mechanism* variable are the mechanism directory names under the $M3MODEL/include/release directory. Examples include:
    -   *cb05cl_ae5_aq*: CB05 gas-phase mechanism with active chlorine chemistry, fifth-generation CMAQ aerosol mechanism with sea salt, aqueous/cloud chemistry
    -   *cb05tucl_ae5_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, fifth-generation CMAQ aerosol mechanism with sea salt, aqueous/cloud chemistry
    -   *cb05tucl_ae6_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM Other, aqueous/cloud chemistry
    -   *cb05tump_ae6_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, mercury, and air toxics, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM, aqueous/cloud chemistry; this is the CMAQv5 multipollutant mechanism
    -   *saprc99_ae5_aq*: SAPRC-99 gas-phase mechanism, fifth-generation CMAQ aerosol mechanism with sea salt, aqueous/cloud chemistry
    -   *saprc99_ae6_aq*: SAPRC-99 gas-phase mechanism, sixth-generation CMAQ aerosol mechanism with sea salt, aqueous/cloud chemistry
    -   *saprc07tb_ae6_aq*: SAPRC-07 gas-phase mechanism with toluene updates and sixth-generation CMAQ aerosol mechanism

#### ICON compilation

First, it is assumed that you have already installed and compiled the I/O API and netCDF libraries (see [Chapter 5](CMAQ_OGD_ch05_sys_req.md#Table5-3), or that these are already available from a previous CMAQ compilation.

[Chapter 5](CMAQ_OGD_ch05_sys_req.md) provides an overview of how to install and compile the CMAQ programs for the tutorial simulation. Follow the steps outlined in Chapter 5 (summarized below) to compile new versions of ICON:

-   If you have not already done so, compile Bldmake, the CMAQ source code and compilation management program. This needs to be done only once—the first time CMAQ is installed.
-   If needed, configure the ICON build script to use the available I/O API and netCDF libraries.

-   Configure the ICON build script for your application (using the options discussed in [Chapter 5(CMAQ_OGD_ch05_sys_req.md])
-   Invoke the build script to create an executable:

`cd $CMAQ_HOME/PREP/icon/scripts`
`./bldit.icon`

#### ICON execution options

The environment variables listed here are invoked during execution of the program and are set in the ICON run script.

-   `EXEC: [default: ICON_${APPL}_${EXECID}]`

Executable to use for the simulation. The variable CFG is set in the ICON run script. The variable EXECID is set in the config.cmaq configuration file.

-   `GRIDDESC: [default: $CMAQ_HOME/scripts/GRIDDESC1]`
    Grid description file for setting the horizontal grid definition.

-   `GRID_NAME: [default: [CMAQ-BENCHMARK]`
    Name of the grid definition contained in the GRIDDESC file that specifies the horizontal grid for the current application of the model.

-   `IOAPI_ISPH: [default: 20]`
    I/O API setting for spheroid type. See I/O API documentation for [setsphere](https://www.cmascenter.org/ioapi/documentation/3.1/html/SETSPHERE.html) for more information.

-   `LAYER_FILE:[default: `none]
    Name and location of a MET_CRO_3D file for specifying the vertical layer structure for the current application of the model

-   `gc_matrix.nml: [default: none]`
    Gas-phase species namelist file. This file is used to configure the gas-phase species that will be output by ICON.

-   `ae_matrix.nml: [default: none]`
    Aerosol-phase species namelist file. This file is used to configure the aerosol-phase species that will be output by ICON

-   `nr_matrix.nml: [default: none]`
    Nonreactive species namelist file. This file is used to configure the nonreactive species that will be output by ICON

-   `tr_matrix.nml: [default: none]`
    Tracer species namelist file. This file is used to configure the tracer species that will be output by ICON

-   `OUTDIR: [default: $CMAQ_HOME/data/icon]`
    Output data directory

-   `IC:`
    Sets the input file type. The setting of this variable determines how the run script sets the input and output environment variables.
    -   *profile*: sets the output file name to include the tag “profile” in the name; uses the variable IC_PROFILE to point to an ASCII vertical profile file for input to **ICON**. Also optionally looks for the variable MECH_CONV_FILE to point to a user-defined mechanism conversion file.
    -   *m3conc*: used for nested simulations; sets the output file name to include a start date in the name; uses the variable CTM_CONC_1 to point to a CCTM CONC file for input to **ICON**.
-   `DATE:`
    Sets the Julian date to use in naming the **ICON** output file for nested runs.

-   `SDATE: [default: ${DATE}]`
    Julian start date for extracting initial conditions from a CCTM CONC file for a nested simulation. If SDATE is not set, ICON will use the first hour of the CTM_CONC_1 file.

-   `STIME: [default: 000000]`
    Start time for extracting initial conditions from a CCTM CONC file for a nested simulation. If SDATE is not set, ICON will use the first hour of the CTM_CONC_1 file.

#### ICON output files

<a id=Table7-14></a>

<center>**Table 7‑14. ICON output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|INIT_CONC_1|`GRDDED3`|Name and location of the gridded initial conditions data output on the model grid defined by `GRID_NAME`|

The default location of the ICON output files is the $CMAQ_HOME/data/icon directory, controlled by the `OUTDIR` variable in the run script. The default naming convention for all ICON output files uses the `APPL` and `GRID_NAME` environment variables in the file name. For initial conditions created from existing CCTM CONC files, the Julian date is also used in the file name through the `DATE` environment variable. All of the file-naming variables for ICON outputs are set in the run script.

<a id="JPROC"></a>
## JPROC

### Description

The program JPROC calculates daily clear-sky photolysis rates from look-up tables of molecular absorption cross-section and quantum yield (CSQY) data, and climatologically derived ozone-column and optical depth data. The outputs from JPROC are ASCII look-up tables of daily clear-sky photolysis rates for photochemical reactions in a selected gas-phase photochemical mechanism at different altitudes, latitudes, and hours from noon. The photochemical mechanism from which these rates are derived is selected during compilation of JPROC. The altitudes (meters), latitudes (degrees), and hour angles (from noon) for which the rates are derived are hardwired in the JPROC source code.

CCTM includes an in-line photolysis option that calculates photolysis rates using predicted ozone and aerosols. JPROC is not needed if the in-line photolysis option is selected in CCTM (*ModPhot* set to phot_inline). JPROC is required to produce daily photolysis rate look-up tables if CCTM is compiled with *ModPhot* set to phot_table.

### Files, configuration, and environment variables

[Figure 7‑7](#Figure7-7) shows the input and output files for JPROC. Some options are invoked at compilation, while others are invoked with execution of the program. When compiling JPROC, the user specifies a chemical mechanism to indicate the gas-phase chemistry for which to calculate photolysis rates. Setting the *Mechanism* variable in the JPROC compile script configures the program to use a specific set of mechanism INCLUDE files to build an executable. JPROC executables are hard-wired to a specific mechanism configuration.

<a id=Figure7-7></a>

<center>
![](./images/Figure7-7.png "Figure7-7.png")

</center>
<center>
**Figure 7‑7. JPROC input and output files**

</center>
While JPROC does not require any technical configuration at execution, such as domain specifications,, there are several required and optional input files that the user must provide to the program. For the selected photochemical mechanism, the user must provide a set of molecular absorption CSQY data files that are consistent with the photolysis reactions in the mechanism. CMAQ is distributed with a full set of CSQY files for the CB05, SAPRC-99, and SAPRC-07 mechanism versions supported by the model. If new mechanisms are added to CMAQ, the user must produce the appropriate CSQY data files for the added mechanism. The user also has the option of using the default atmospheric profiles contained in the PROFILES input file or using Total Ozone Mapping Spectrometer (TOMS) data to replace the climatologically derived ozone column data in the PROFILES file.

#### JPROC input

<a id=Table7-15></a>
<center>**Table 7‑15. JPROC input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|ET|ASCII|Extraterrestrial radiation as a function of wavelength|
|PROFILES|ASCII|Seasonal vertical profiles of ozone concentrations, aerosol attenuation, temperature, air density and Dobson values|
|TOMS|ASCII|Total ozone column measurements from the Total Ozone Mapping Spectrometer instrument aboard the sun-synchronous polar orbiting Nimbus satellite|
|O2ABS|ASCII|Absorption CSQY data for molecular oxygen as a function of wavelength|
|O3ABS|ASCII|Absorption CSQY data for ozone as a function of wavelength|
|CSQY|ASCII (directory path)|Directory path containing absorption CSQY data for gas-phase photolysis reactions as a function of wavelength|

#### JPROC compilation options

The configuration options listed here are set during compilation of the JPROC executable. When these options are invoked they create a binary executable that is fixed to the specified configuration. To change these options it is necessary to recompile JPROC and create a new executable.

-   `Opt: [default: verbose]`
    Defines the action to be taken by the program Bldmake when extracting source code from CVS and compiling an executable.
    -   *compile_all:* force compile, even if all the object files are current
    -   *clean_up:* remove all source files upon successful compilation
    -   *no_compile:* do everything except compile
    -   *no_link:* do everything except link
    -   *one_step:* compile and link in one step
    -   *parse_only:* check configuration file syntax
    -   *show_only:* show requested commands but do not execute them
    -   *verbose:* show requested commands as they are executed
-   `MakeOpt`
    Uncomment to build a Makefile to compile the executable.

-   `Mechanism: [default: cb05tucl_ae6_aq]`
    Specifies the gas-phase, aerosol, and aqueous-phase chemical mechanisms for which to create photolysis rates. The choices for the *Mechanism* variable are the mechanism directory names under the $M3MODEL/include/release directory. Examples include:
    -   *cb05cl_ae5_aq*: CB05 gas-phase mechanism with active chlorine chemistry, fifth-generation CMAQ aerosol mechanism with sea salt, aqueous/cloud chemistry
    -   *cb05tucl_ae5_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, fifth-generation CMAQ aerosol mechanism with sea salt, aqueous/cloud chemistry
    -   *cb05tucl_ae6_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM Other, aqueous/cloud chemistry
    -   *cb05tump_ae6_aq*: CB05 gas-phase mechanism with active chlorine chemistry, updated toluene mechanism, mercury, and air toxics, sixth-generation CMAQ aerosol mechanism with sea salt and speciated PM, aqueous/cloud chemistry; this is the CMAQv5 multi-pollutant mechanism
    -   *saprc07tb_ae6_aq*: SAPRC-07 gas-phase mechanism with toluene updates and sixth-generation CMAQ aerosol mechanism

#### JPROC compilation

First, it is assumed that you have already installed and compiled the I/O API and netCDF libraries ([Chapter 5](CMAQ_OGD_ch05_sys_req.md#Table5-3)), or that these are already available from a previous CMAQ compilation.

[Chapter 5](CMAQ_OGD_ch05_sys_req.md) provides an overview of how to install and compile the CMAQ programs for the tutorial simulation. Follow the steps outlined in Chapter 5 (summarized below) to compile new versions of JPROC:

-   If you have not already done so, compile Bldmake, the CMAQ source code and compilation management program. This needs to be done only once—the first time CMAQ is installed.
-   If needed, configure the JPROC build script to use the available I/O API and netCDF libraries.
-   Configure the JPROC build script for your application (using the options discussed in [Chapter 7](CMAQ_OGD_ch07_programs_libraries.md))
-   Invoke the build script to create an executable:

`cd $CMAQ_HOME/PREP/jproc/scripts`
`./bldit.jproc`

#### JPROC execution options

The environment variables listed here are invoked during execution of the program and are set in the JPROC run script.

-   EXEC: [default: JPROC_${CFG}]

Executable to use for the simulation

#### JPROC output files

<a id=Table7-16></a>

<center>**Table 7‑16. JPROC output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|`JTABLE_$Date`|`ASCII`|Daily clear-sky photolysis rates file|

The default location of the JPROC output files is the `$CMAQ_HOME/data/jproc` directory, controlled by the `OUTDIR` variable in the run script. The default naming convention for all JPROC output files uses the Date environment variable in the file name, which is aliased to the `STDATE` environment variable in the run script.

<a id="LTNG_2D_DATA"></a>
LTNG_2D_DATA

### Description

Lightning is the largest source of nitrogen in the upper troposphere. NO<sub>x</sub> produced by lightning influences ambient nitrogen concentrations and wet deposition of total nitrates within the boundary layer. To simulate the impacts of lightning NO<sub>x</sub> emissions on ambient gas and particle concentrations and on deposition, CMAQv5 is instrumented with a lightning NO<sub>x</sub> module. CCTM can either read in an input file of lightning NO<sub>x</sub> emissions that were calculated off-line or it can compute lightning NO emissions in-line during a simulation. The in-line emissions can be calculated either directly from lightning flash counts estimated from the convective precipitation field in the input meteorology or by using a combination of observed flash counts and simulated convective precipitation. The former approach is recommended for hemispheric-scale simulations or when no flash count data are available, while the latter approach is recommended for all simulations at less than 50-km horizontal grid resolutions. The [National Lightning Detection Network](http://gcmd.nasa.gov/records/GCMD_NLDN.html) (NLDN) is a source of flash count data for North America.

This section describes how to prepare input parameter files for calculating lightning NO<sub>x</sub> emissions in-line in CCTM. If off-line lightning NO<sub>x</sub> emissions are preferred, it is the responsibility of the user to prepare a 4-D (row x col x layer x hour) I/O API netCDF file of lightning NO emissions with units of moles/s.

The program LTNG_2D_DATA prepares the input parameter file for estimating lightning NO emissions in-line with CCTM. The statistical package [R](http://en.wikipedia.org/wiki/R_%28software%29) is used as a preprocessor to LTNG_2D_DATA to prepare two input files to the program:

-   a land/ocean mask file
-   a file with ratios of intercloud to cloud-to-ground flashes

LTNG_2D_DATA reads those input files, plus

-   A [MET_CRO_2D](#MET_CRO_2D:_Two-dimensional_meteorological_cross-point_fields) file from MCIP
-   An I/O API netCDF file of monthly flash densities gridded to your modeling domain. 12-km continental U.S. monthly flash densities are available from the [CMAS Data Clearinghouse](https://www.cmascenter.org/download/data/nldn.cfm).
-   If those are not available, the program NLDN_2D (packaged with LTNG_2D_DATA) can produce them from raw hourly flash counts from the [NLDN](http://gcmd.nasa.gov/records/GCMD_NLDN.html).

LTNG_2D_DATA outputs [a parameters file](#LTNG_2D_DATA_output_files).

[Figure 7-8](#Figure7-8) shows the input and output files for LTNG_2D_DATA. The preprocessor Fortran program NLDN_2D inputs a text file of NLDN flash count data and a MET_CRO_2D file and outputs an I/O API netCDF file for input to LTNG_2D_DATA. The R batch script ocean_mask.R reads a MET_CRO_2D file to create an ocean mask CSV file on the modeling grid. The R batch script iccg.R reads a MET_CRO_2D file and a flash ratio text file and outputs a CSV-formatted flash ratio file on the modeling grid. The outputs of these processes are used by LTNG_2D_DATA, along with a MET_CRO_2D file, to produce a lightning NO<sub>x</sub> parameters file for input to CCTM. The MET_CRO_2D file must be consistent across all of the programs associated with LTNG_2D_DATA.

<a id=Figure7-8></a>

<center>
![](./images/Figure7-8.png "Figure7-8.png")

</center>
<center>
**Figure 7‑8. LTNG_2D_DATA input and output files**

</center>
### LTNG_2D_DATA input files

<a id=Table7-17></a>

<center>**Table 7‑17. LTNG_2D_DATA input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|NLDNFILE|GRDDED3|NLDN monthly flash totals, aka monthly average strike density; contains the variable, NLDNstrk, that is the number of flashes per month in each grid cell|
|OCEANMASK|CSV|Ocean mask file; 1 = land, 0 = open ocean. Can be built by `ocean_mask.R` from [MET_CRO_2D](#MET_CRO_2D:_Two-dimensional_meteorological_cross-point_fields)|
|ICCG|CSV|Intercloud to cloud-to-ground ratios by model grid cell. Example ICCG files are in the [CMAQ distribution](#Installing_CMAQ_on_your_system) @ `$CMAQ_HOME/data/lightning/input/`|
|METFILE|GRDDED3|MET_CRO_2D file from MCIP; must include convective precipitation (RC) and cloud-top variables (CLDT)|

Example LTNG_2D_DATA inputs for North America can be found at the [CMAS Data Clearinghouse](https://www.cmascenter.org/download/data/nldn.cfm): the NLDN monthly flash total files there are named `NLDN.$YYYY.$MM.ioapi`. To rebuild them,

1.  Obtain raw hourly flash counts from the [NLDN](http://gcmd.nasa.gov/records/GCMD_NLDN.html)
2.  [Install CMAQ](#Installing_CMAQ_on_your_system)
3.  Follow the directions in `$CMAQ_HOME/PREP/ltng/docs/README`
    1.  Build `NLDN_2D` using `$CMAQ_HOME/PREP/ltng/src/Makefile.NLDN_2D`
    2.  Run `$CMAQ_HOME/PREP/ltng/scripts/run.NLDN_2D.csh`
    3.  Copy the resulting output to `$CMAQ_HOME/data/lightning/flash_data/monthly_flash_density/`

### LTNG_2D_DATA compilation options

All model configuration options for LTNG_2D_DATA and NLDN_2D are set during execution. System compiler options must be set in the provided Linux Makefile to build the program for different operating system/compiler combinations. Example compiler paths, flags, and library locations are provided in the default. The R scripts ocean_mask.R and iccg.R require that the netCDF and RGDAL packages be compiled and linked in with your R installation.

### LTNG_2D_DATA execution options

The environment variables listed here are invoked during execution of the program and are set in the LTNG_2D_DATA run script.

-   `CREATE_OCEANMASK: [default: Y]`
    Setting to run the R script that generates the ocean mask file for the modeling domain. This time-independent file needs to be generated only once per modeling domain.

-   `CREATE_FLASHRATIOS: [default: Y]`
    Setting to run the R script that generates the flash ratios file for the modeling domain. This time-independent file needs to be generated only once per modeling domain.

-   `CREATE_NLDN2DDATA: [default: Y]`
    Setting to run the LTNG_2D_DATA program to prepare a monthly lightning NO<sub>x</sub> file for input to CCTM.

-   `CREATE_PLOTS: [default: Y]`
    Setting to run the R script that generates plots of the lightning NO<sub>x</sub> variables in the LTNG_2D_DATA output file.

-   `BASE: [default: $cwd]`
    Base directory for the LTNG_2D_DATA installation.

-   `RScript: [default: $BASE/R-scripts]`
    Base directory for the installation of the R scripts used to support LTNG_2D_DATA.

-   `OUTDIR: [default: $CMAQ_HOME/data/lightning]`
    Directory for the LTNG_2D_DATA output files.

-   `months: [example: 06]`
    Array of months (##) for which to create flash count data.

-   `year: [example: 2006]`
    Year (####) for which to create flash count data.

-   `METFILE: [default: none]`
    Meteorology data file containing hourly convective precipitation (RC) and cloud-top (CLDT) variables. This file can be either the MCIP output file MET_CRO_2D or an extraction of this file that contains only those two variables.

-   `OCEANMASKIMG: [default: $BASE/R-out/ocean_mask.png]`
    Ocean mask image file. Produced only if CREATE_OCEANMASK = Y.

-   `OCEANMASKFILE: [default: $BASE/R-out/ocean_mask.csv]`
    Ocean mask file for input to LTNG_2D_DATA. Produced only if CREATE_OCEANMASK = Y.

-   `ICCGIN: [default: $BASE/input/iccg.Boccippio.summer.txt]`
    Seasonal (summer/winter) intercloud to cloud-to-ground ratio file. Needed only if CREATE_FLASHRATIOS = Y.

-   `ICCG: [default: $BASE/R-out/iccg.interpolate.csv]`
    Intercloud to cloud-to-ground ratio file for input to LTNG_2D_DATA. Produced only if CREATE_FLASHRATIOS = Y.

-   `ICCGIMG: [default: $BASE/R-out/iccg.png]`
    Intercloud to cloud-ground ratios image file. Produced only if CREATE_FLASHRATIOS = Y.

-   `NLDNFILE: [default: none]`
    Monthly flash density file output from the program NLDN_2D gridded to the same modeling domain as METFILE. Needed only if CREATE_NLDN2DDDATA = Y.

-   `STRIKE_FACTOR: [default: 147.0]`
    Needed only if CREATE_NLDN2DDDATA = Y

-   `MOLES_N_CG: [default: 500.0]`
    Moles of nitrogen per cloud-to-ground flash. Needed only if CREATE_NLDN2DDDATA = Y

-   `MOLES_N_IC: [default: 500.0]`
    Moles of nitrogen per intercloud flash. Needed only if CREATE_NLDN2DDDATA = Y

-   `OUTFILE: [default: $OUTDIR/LTNG_RATIO.$year.$month.ioapi]`
    Lightning NO<sub>x</sub> parameters output from the program LTNG_2D_DATA. Produced only if CREATE_NLDN2DDATA = Y.

-   `PLOTNOx: [default: $BASE/R-out/plot_LNOx_params.pdf]`
    Lightning NO<sub>x</sub> parameters image file. Produced only if CREATE_NLDN2DDATA = Y and CREATE_PLOTS = Y.

### LTNG_2D_DATA output files

<a id=Table7-18></a>
<center>**Table 7‑18. LTNG_2D_DATA output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|`OUTFILE`|`GRDDED3`|Monthly lightning NO<sub>x</sub> parameters file for input to CCTM simulations with in-line lightning emissions; the default naming convention of this file is LTNG_RATIO.$YEAR.$MM.ioapi.|

The output files have the following fields (with units):

1.  STRKCNT (unitless): Lightning flash count
2.  NLDNstrk (km<sup>2</sup>/day): Monthly flash totals (normalized to day<sup>-1</sup>) from NLDN input data
3.  LTstrk (km<sup>2</sup>/day): Monthly flash totals (normalized to day<sup>-1</sup>) in each CMAQ grid cell calculated as the convective precipitation rate (RC) x STRKCNT
4.  NLDNstrk (km<sup>2</sup>/day): Monthly flash totals from the input NLDN data
5.  LTratio (unitless): Grid-cell scaling factors for calculating flashes using the convective precipitation rate. These scaling factors ensure that the calculated flash count matches the monthly total.
6.  ICCG (unitless): Ratio of intercloud to cloud-to-ground flashes
7.  OCNMASK (unitless): Land/water mask to remove spurious flashes over the ocean.
8.  MOLSN (molN): moles of N per cloud-to-ground flash
9.  MOLSNIC (molN): moles of N per intercloud flash

The default location of the LTNG_2D_DATA output files is the `$CMAQ_HOME/data/lightning` directory, controlled by the `OUTDIR` variable in the run script. To build these CCTM parameter files,

1.  [Install CMAQ](#Installing_CMAQ_on_your_system)
2.  Obtain or build [NLDN gridded monthly flash counts](#LTNG_2D_DATA_input_files)
3.  Follow the directions in `$CMAQ_HOME/scripts/lnox/README`
    1.  Build `LTNG_2D_DATA` using `$CMAQ_HOME/scripts/lnox/src/Makefile.LTNG_2D_DATA`
    2.  Run `$CMAQ_HOME/scripts/lnox/run.LTNG_2D.csh`
    3.  Compare your output to reference results in `$CMAQ_HOME/data/ref/lnox`

<a id="MCIP"></a>
## MCIP

### Description

The Meteorology-Chemistry Interface Processor (MCIP) processes meteorological model output from either MM5 or WRF‑ARW model into I/O API-formatted files that are compatible with CMAQ and SMOKE. MCIP automatically determines whether an input file is generated by MM5 or WRF‑ARW by trying to open the file as a netCDF file. If the file can be read as netCDF, MCIP assumes the input is a WRF‑ARW dataset; otherwise, MM5 is assumed.

Many of the fields that are simulated by the meteorological model are not modified by MCIP for the emissions model and CCTM, and they are written to I/O API files. Fields that are required for the transformation to CMAQ’s generalized coordinate system are calculated within MCIP. The dry deposition velocities are no longer calculated by the current version of MCIP. CMAQv5 can now calculate all deposition velocities, MCIPv3.4 will be the last version of MCIP to calculate those velocities internally.

MCIP can extract both temporal and spatial subsets of the input meteorology files. The run script allows the user to specify the beginning and end dates/times of the MCIP simulation; these dates/times can fall anywhere within the range of the input meteorological time period, but must be consistent with the time granularity of the meteorological files. MCIP cannot perform temporal interpolations to artificially increase the temporal resolution of the meteorology fields. Two types of horizontal domain windowing are allowed with MCIP. The boundary trim option (“BTRIM”) uniformly trims grid cells off each of the four lateral boundaries of the input meteorology grid. The nonuniform trim option specifies an offset from the lower left corner of the input meteorology domain and the number of cells in the X and Y directions from the revised origin to extract from the input domain. More information about how to invoke these options is provided in the next section: [MCIP execution options](#MCIP execution options). MCIP also provides the capability to reconfigure the vertical layer structure in the input meteorology through interpolation from the input structure to an output structure defined through sigma coordinates in the run script. Commonly referred to as “layer collapsing,” this option should be exercised with caution as it can significantly impact the conservation of energy assumption inherent in the meteorology through its effects on the predicted wind fields.

### Files, configuration, and environment variables

[Figure 7‑9](#Figure7-9) shows the input and output files and configuration options for MCIP. All MCIP configurations are accomplished at execution (rather than at compile time) and via Fortran namelist variables, a distinction from the rest of the CMAQ programs. The user does not need to directly edit the MCIP namelist file. All configuration settings are contained in the MCIP run script, which automatically creates a new namelist file each time the script is executed.

<a id=Figure7-9></a>

<center>
![](./images/Figure7-9.png "Figure7-9.png")

</center>
<center>
**Figure 7‑9. MCIP input and output files**

</center>
#### MCIP input files

<a id=Table7-19></a>

<center>**Table 7‑19. MCIP input files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|InMetFiles|binary (MM5) or netCDF (WRF‑ARW)|List of MM5 or WRF‑ARW output files for input to MCIP|
|InTerFile|binary|MM5 Terrain file with fractional land use categories; used for calculating land-use-dependent vertical diffusivity. Not necessary with WRF‑ARW; this information is included in the WRF-ARW met file.|
|InSatFiles||GOES satellite cloud data|

#### MCIP compilation options

All model configuration options for MCIP are set during execution. System compiler options must be set in the provided Linux Makefile to build the program for different operating system/compiler combinations. Example compiler paths, flags, and library locations are provided in the default Makefile.

#### MCIP compilation

First, it is assumed that you have already installed and compiled the I/O API and netCDF libraries ([Chapter 5](CMAQ_OGD_ch05_sys_req.md)), or that these are already available from a previous CMAQ compilation.

[Chapter 5](CMAQ_OGD_ch05_sys_req.md) provides an overview of how to install and compile the CMAQ programs for the tutorial simulation. Follow the steps outlined in Chapter 5 (summarized below) to compile new versions of **MCIP**:

-   Configure the Makefile for the current operating system/compiler combination. Comment out the configuration that does not apply to the current system. Uncomment the configuration that is closest to that of the current system and make the necessary changes to point to the compiler path, I/O API location, and netCDF locations on the current system.
-   Invoke the Makefile to create an executable by typing the following command in the directory that contains the Makefile and **MCIP** source code:

./make

#### MCIP execution options

The environment variables listed here are invoked during execution of the program and are set in the MCIP run script.

-   APPL:

Application name; scenario ID for file naming

-   CoordName:

Coordinate system name of the MCIP output grid that is written to the GRIDDESC file

-   GridName:

Model grid name of the MCIP output grid that is written to the GRIDDESC file

-   DataPath:

Input/output data directory path

-   InMetDir:

Path of the input data directory containing the MM5 or WRF‑ARW output data files

-   InTerDir:

Path of the input data directory containing the MM5 TERRAIN file; not compatible with WRF‑ARW.

-   InSatDir:

Path of the input data directory containing the GOES satellite files.

-   OutDir: [default: $CMAQ_HOME/data/mcip]

Path of the **MCIP** output data directory

-   ProgDir:[default: $cwd]

Working directory containing the **MCIP** executable

-   WorkDir:

Temporary working directory for Fortran links and the namelist file

-   InMetFiles:

List of input meteorology files, including the directory path for each file; without modifying MCIP, up to 300 meteorological model output files are allowed as input to a single **MCIP** execution

-   InSatFiles:

List of input GOES satellite cloud data files.

-   IfTer:[default: T]

Binary flag indicating the availability of an input MM5 TERRAIN file; options include T (true) or F (false)

-   InTerFile:

Name and location of input MM5 TERRAIN file

-   LPV: [default: 0]

Determines whether MCIP will compute and output potential vorticity.

-   -   *0*: Do not compute and output potential vorticity
    -   *1*: Compute and output potential vorticity
-   LWOUT:[default: 0]

Determines whether MCIP will output vertical velocity.

-   -   *0*: Do not output vertical velocity
    -   *1*: Output vertical velocity

-   LUVCOUT [default: 0]:

Determines whether MCIP will output u- and v-component winds on C-grid.

-   -   *0*: Do not output u- and v-component winds on C-grid
    -   *1*: Output u- and v-component winds on C-grid

-   LSAT [default: 0]:

Determines whether GOES satellite cloud observations will replace model-derived input on clouds

-   -   *0*: No satellite data are available
    -   *1*: Use GOES observed cloud information to replace model-derived input
-   MCIP_START:[format: YYYY-MM-DD-HH:MM:SS.SSSS]

Beginning date and time (UTC) of data to output from **MCIP**. The start date and time must be contained within the input data from MM5 or WRF‑ARW.

-   MCIP_END:[format: YYYY-MM-DD-HH:MM:SS.SSSS]

End date and time (UTC) of data to output from **MCIP**. The end date and time must be contained within the input data from MM5 or WRF‑ARW.

-   INTVL: [default: 60]

Output interval in minutes. This setting determines the amount of model time contained in each output time step. The output interval for MCIP can be less frequent than the incoming meteorological model output (e.g., process 30-minute data for CCTM from 15-minute WRF‑ARW output).

-   CTMLAYS: [default: “-1.0” ]

Sigma values of the vertical layers in the 3-D **MCIP** output. Comma-delimited values for each sigma value must be in descending order starting at 1 and ending with 0. There are a maximum of 100 layers allowed. To use the all of the layers from the input meteorology without collapsing (or explicitly specifying), set CTMLAYS = ‑1.0.

-   MKGRID: [default: T]

Determines whether to output static (GRID) meteorology files

-   BTRIM: [default: 5]

The number of boundary points to remove on each of the four horizontal sides of the MCIP domain. Setting BTRIM = 0 will specify the maximum extent of the input meteorology domain. To remove the MM5 or WRF‑ARW lateral boundaries, set BTRIM = 5 (recommended).

This setting affects the output MCIP horizontal domain by reducing the input meteorology domain by 2*BTRIM + 2*NTHIK + 1, where NTHIK is the lateral boundary thickness (from the BDY files). The extra point reflects the conversion from the grid points (dot points) to grid cells (cross points).

For windowing a subset domain of the input meteorology, set BTRIM = -1; this setting causes BTRIM to be replaced by the information provided by X0, Y0, NCOLS, and NROWS (see below).

-   X0: [used only if BTRIM = -1]

The *x*-coordinate of the lower-left corner of the full MCIP cross-point domain (including the MCIP lateral boundary) based on the input MM5 or WRF‑ARW domain. X0 refers to the east-west direction.

-   Y0: [used only if BTRIM = -1]

The *y*-coordinate of the lower-left corner of the full MCIP cross-point domain (including the MCIP lateral boundary) based on the input MM5 or WRF‑ARW domain. Y0 refers to the north-south direction.

-   NCOLS: [used only if BTRIM = -1]

Number of columns in the output MCIP domain (excluding MCIP lateral boundaries)

-   NROWS: [used only if BTRIM = -1]

Number of rows in the output MCIP domain (excluding MCIP lateral boundaries)

-   LPRT_COL: [default: 0]

Column cell coordinate for diagnostic outputs on the MCIP modeling domain

-   LPRT_ROW: [default: 0]

Row cell coordinate for diagnostic outputs on the MCIP modeling domain

-   WRF_LC_REF_LAT: [default: -999.0]

WRF Lambert Conformal reference latitude. Use this setting to force the reference latitude in the output MCIP data. If not set, MCIP will use the average of the two true latitudes. This setting is useful for matching WRF grids to existing MM5 grids.

#### MCIP output files

<a id=Table7-20></a>

<center>**Table 7‑20. MCIP output files**</center>

|**File Name**|**Format**|**Description**|
|---|---|---|
|GRIDDESC|ASCII|Grid description file with coordinate and grid definition information|
|GRID_BDY_2D|BNDARY3|Time-independent 2-D boundary meteorology file|
|GRID_CRO_2D|GRDDED3|Time-independent 2-D cross-point meteorology file|
|GRID_CRO_3D|GRDDED3|Time-independent 3-D cross-point meteorology file|
|GRID_DOT_2D|GRDDED3|Time-independent 2-D dot-point meteorology file|
|MET_BDY_3D|BNDARY3|Time-dependent 3-D boundary meteorology file|
|MET_CRO_2D|GRDDED3|Time-dependent 2-D cross-point meteorology file|
|MET_CRO_3D|GRDDED3|Time-dependent 3-D cross-point meteorology file|
|MET_DOT_3D|GRDDED3|Time-dependent 3-D dot-point meteorology file|
|mmheader|ASCII|Content of MM5 header including configuration information; not generated for WRF‑ARW input|

The default location of the MCIP output files is the `$CMAQ_HOME/data/mcip3/$GridName directory`. Since the default file names do not have any information about the model grid that they are simulating, the name of the grid is set in the output directory path. The default naming convention for all MCIP output files uses only the APPL environment variable in the file name. All of the file-naming variables for the MCIP outputs are set in the run script.

<a id="References_Chapter7"></a>
## References for Chapter 7: CMAQ Programs and Libraries
---------------------------------------------------

Arya, P., 1984: Parametric relations for the atmospheric boundary layer. Bound.-Layer Meteor., 30, 57–73.

Byun, D. W., and J. K. S. Ching, 1999: Science Algorithms of the EPA Models-3 Community Multiscale Air Quality (CMAQ) Modeling System. U. S. Environmental Protection Agency Rep. EPA‑600/R‑99/030, 727 pp. [Available from Office of Research and Development, EPA, Washington, DC 20460.]

Hanna, S. R., G. A. Briggs, and R. P. Hosker, 1982: Handbook on atmospheric diffusion, U.S. DOE, DOE/TIC-11223, DE82002045, National Technical Info. Center, Springfield, VA.

Hicks, B.B., 1985: Behavior of turbulence statistics in the convective boundary layer. J. Clim. Appl. Meteor., 24, 607–614.

Irwin, J. S., 1979: Scheme for estimating dispersion parameters as a function of release height, EPA-600/4-79-062, Research Triangle Park, NC.

Niewstadt, F. T. M., 1984: Some aspects of the turbulent stable boundary layer. Bound.-Layer Meteor., 30, 31–55.

Pleim, J. E., A. Xiu, P. L. Finkelstein, and T. L. Otte, 2001: A coupled land-surface and dry deposition model and comparison to field measurements of surface heat, moisture, and ozone fluxes. Water Air Soil Pollut. Focus, 1, 243–252.

Venkatram, A., 1988: Dispersion in the stable boundary layer. Chapter 5, in Lectures on Air Pollution Modeling, A. Venkatram and J. Wyngaard, Eds., American Meteorology Society, Boston, MA.

Weil. J. C., 1988: Dispersion in the convective boundary layer. Chapter 4, in Lectures on Air Pollution Modeling, A. Venkatram and J. Wyngaard, Eds., American Meteorology Society, Boston, MA.

Wesely, M. L., 1989: Parameterization of surface resistances to gaseous dry deposition in regional-scale numerical models. *Atmos. Environ.,* **23**, 1293–1304.

***
[<< Previous Chapter](CMAQ_OGD_ch06_req_lib.md) - [Home](README.md) - [Next Chapter >>](CMAQ_OGD_ch08_input_files.md)<br>
CMAQ Operational Guidance Document (c) 2016<br>