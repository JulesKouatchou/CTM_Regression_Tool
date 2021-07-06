# GEOS CTM Regression Testing

<center>
Jules Kouatchou (Jules.Kouatchou@nasa.gov)
<br>
NASA Goddard Space Flight Center, Code 606, Greenbelt, Maryland, USA
</center>
<br>
<br>
#### Abstract:
We describe procedures to perform GEOS CTM regression test.
<br>

## Introduction
The GEOS CTM regression tool is a set of python scripts and configuration
files designed to automatically obtain, compile and run GEOS CTM.
The tool functionality is driven by the settings specified in a single
configuration file.

The configuration file is a text file with a particular structure readable by
Python scripts and easily understood.
The file is organized into sections, and each section contains name-value pairs
for configuration data.
The file can have any name but must have the `.yaml` extension.
The entries in the YAML file are described in the next section.

<br>
<br>
The tool does three basic operations:

- **Initialize:** Read the configuration file, obtain the code(s), compile the code(s), register the experiments to be performed and create a directory tree.
- **Run:** Carry out the experiments and do the necessary comparisons.
- **Finalize:** Write a summary report and electronically send it to the user.

It can perform three types of experiments:

- **Verification Test:** We only want make sure that the code (under any Chemistry option) can run.
- **'1+1=2' Test:** We select a Chemistry option (not including GMI) and do the following:
     (1) run for one day to create a restart file; (2) run for another day using the
      restart file from (1); (3) do a two-day run with the same initial condition as (1).
     We want to verify that the outputs at the end of (2) is the same as that of (3).
- **Baseline Test:** Compare the results (from any Chemistry option) of the new tag
     against that of a baseline tag.
     The user has the option of either providing a baseline tag name (the tool will then obtain,
     compile and run with the baseline tag) or an existing directory containing previously
     generated results.
     <br>
     There two options for comparing the ressults of the two runs (after one day of integration):
     - Option 1: All the tracers in a user provided colection are compared
           (should be identical up to a tolerance).
     - Option 2: The user provide a collection and a list of tracers of interest.
           The tool will verify if each tracer (on the list) from the new tag does not differ
           by more tnan $10\%$ (this treshold can be modified) from the corresponding baseline tracer.

Note that users can choose (through the configuration file) as many experiments they want to carry out.



## Description of the Configuration File

The regression testing configuration contains at least two sections:

```
     USERCONFIG:

     <experiment-type>:
```

#### USERCONFIG]
The section `USERCONFIG:` contains various settings that are commonly changed by a user.  The appears in name-value pairs as:

```
    var_name: value
```

The list of variables are:
```
repo_type:       Type of repository (cvs, git)
repo_url:        url to the repository
new_tag:         Name of the code tag we want to test
baseline_tag:    Name of the baseline tag we want to compare against
repo_module:     Module name you want to check out (GEOSctm, GEOSagcm, etc.)
model_type:      Type of model (CTM, AGCM, etc.)
repo_user_id:    User ID for the repository

baseline_dir:    Location of the GEOS CTM output files.
                 This is only necessary if you do a baseline test
                 experiment and baseline_tag is not provided.

parallel_build:  Do you want a parallel build? (yes or no)

sponsor_id:      Sponsor ID required by SLURM
mail_to:         Email address where to mail the regression test report
use_html:        Set to yes/no to determine if the report should an HTML format

scratch_dir:     Directory where all the work will be done
clean_scratch:   Do you want to clean scratch_dir? (yes or no)

ref_year:        Starting year of the experiment (default is 2010)
ref_month:       Starting month of the experiment (default is 1)
ref_day:         Starting day of the experiment (default is 1)
ref_hour:        Starting hour of the experiment (default is 0)
ref_min:         Starting minute of the experiment (default is 0)
ref_sec:         Starting second of the experiment (default is 0)
```

#### experiment_type
The following is a the list of variables needed for setting an experiment:

```
one_plus_one:    Are you doing the 1+1=2 test? (yes or no)
baseline_test:   Are you doing the basline test? (yes or no)
                 Note that one_plus_one and baseline_test cannot
                 be both yes
collection:      Name of the target collection in HISTORY.rc
                 if either one_plus_one or baseline_test is yes.
field_names:     List of tracers of interest if doing a baseline test

exp_duration:    Duration (in days) of the experiment in case
                 you are doing a verification run only.
                 The default value is 1.

---> Setting for ctm_setup
exp_desc:        Experiment description
exp_klone:       Are you cloning the experiment? (FALSE or TRUE)
exp_hor_res:     Model resolution (c48, c90, etc.)
exp_lev:         Number of vertical levels (72)
exp_chem:        Chemistry type (1, 2, 3, 4, 5)
                 1:TR, 2:GOCART, 3:GMI,
                 4:GEOS-Chem, 5:Idealized Passive Tracers
exp_emission:    Emission files needed for GOCART
                 (MERRA2, PIESA, etc.)
exp_aero:        AEROsols provider for GMI (1: GOCART, 2: GMICHEM)
exp_met:         Driving datasets (MERRA2, MERRA1, FPIT)
exp_hst:         HISTORY.rc file template (HISTORY.GEOSCTM.rc.tmpl)

```

The appendix contains sample configuration settings.

## Desecription of Python Scripts

All the scripts are written in Python and use the following main modules:
- logging
- subprocess
- yaml
- Numpy: needed to compare two netCDF files.
- netCDF4: needed to compare two netCDF files.

The regression tool contains the files:

```
geosCTM_class.py           : A class with functionalities to obtain,
                             compile and run the GEOS CTM code
main_regression_geosCTM.py : Regression script driver
shared/utils_general.py    : Utility functions to perform general tasks
shared/utils_getcode.py    : Utilily functions to get the code from repository
shared/utils_compare.py    : Utility functions for comparing files
shared/utils_config.py     : Utility functions for configuration operation
shared/utils.py            : Utility functions for GEOS specific tasks
```

## Performing Regression Test

To run the scripts, specify the configuration file name as an argument to `main_regression_geosCTM.py`:

```
     python main_regression_geosCTM.py yaml_config_file
```

Note that the configuration file extension is not required.
It is expected that the file `yaml_config_file.yaml` exists.
Upon execution the user will see some messages echoed to STDOUT but
all the output will be logged to a file named `yaml_config_file.LOG`.

When all the tasks are completed, an email test report will be emailed to the recipient
specified in the `mail_to` field in `USERCONFIG`.

## Appendix


### Sample Setting for USERCONFIG

```
---
USERCONFIG:

   # ---> Test report message (One sentence, no quotes)
   message: Regression testing of GEOS CTM code base

   #-----------------
   # User information
   #-----------------
   # ---> sponsor ID required by SLURM
   sponsor_id: s1043
   # ---> Where to mail tests report
   mail_to: Jules.Kouatchou@nasa.gov
   use_html: yes

   #-----------------------
   # Repository information
   #-----------------------
   repo_module: GEOSctm
   model_type: CTM

   # ---> Git repository
   EXTERN: null

   # ---> User ID
   repo_user_id: jkouatch

   #-----------------
   # Compilation type
   #-----------------
   parallel_build: yes
   serial_build: no
   debug_build: no

   #---------------------------------
   # New tag and Baseline information
   #--------------------------------
   # ---> New tag name we want to test
   new_tag_repo_type: git
   new_tag_repo_url: git@github.com:GEOS-ESM/GEOSctm.git
   new_tag: v10.19.0-CTM
   new_tag_using_mepo: yes


   # ---> Baseline tag name that served as refence
   baseline_tag_repo_type: cvs
   baseline_tag_repo_url: progressdirect:/cvsroot/esma
   baseline_tag_using_mepo: no
   baseline_tag: Icarus-3_2_p9_CTM_MEM_16-r3-SLES12

   # Location of GEOS CTM output baseline files if you do not provide a baseline tag.
   # This setting is only important if you are doing a baseline test.
   # Do not change unless you have baseline files in that location.
   #baseline_dir :  /discover/nobackup/jkouatch/regressionScripts

   #---------------------
   # Where to do the work
   #---------------------
   # ---> Filesystem where we are doing all the work.
   #      If it does not exist, it will be created.
   scratch_dir: /discover/nobackup/jkouatch/regressionScripts/testDIR
   # ---> Clean the regression testing scratch space (under scratchdir)
   clean_scratch: yes
   #
   # Update baseline_dir with new model answers (change to yes when your code is ready)
   #update_base: no

   #----------------
   # Start date/time
   #----------------
   ref_year: 2011
   ref_month: 1
   ref_day: 1
   ref_hour: 0
   ref_min: 0
   ref_sec: 0

   # Tolerance for comparing two fields
   tolerance_field: 1.0e-01       # used to compare one variable  in two files
   tolerance_file: 1.0e-05        # used to compare all variables in two files
```

### Sample Settings for Experiments
#### Idealized Passive Tracer Configuration

```
idealPT:
   # ---> Do we do the one plus one equal two experiment?
   one_plus_one: no
   baseline_test: no
   collection: idealPT
   exp_duration: 10
   #
   #-------------------------------------------------
   # Settings needed for running the ctm_setup script
   #-------------------------------------------------
   exp_id: idealPT
   exp_desc: Test Idealized Passive Tracers
   exp_klone: FALSE
   exp_hor_res: c90
   exp_lev: 72
   exp_chem: 5
   exp_emission: MERRA2
   exp_aero: 1
   exp_met: MERRA2
   exp_hst: HISTORY.GEOSCTM.rc.tmpl
```

#### TR Configuration

```
pTracerTR:
   # ---> Do we do the one plus one equal two experiment?
   one_plus_one: yes
   baseline_test: no
   collection: pTracerTR

   #-------------------------------------------------
   # Settings needed for running the ctm_setup script
   #-------------------------------------------------
   exp_id: pTracerTR
   exp_desc: Test Passive Tracers
   exp_klone: FALSE
   exp_hor_res: c90
   exp_lev: 72
   exp_chem: 1
   exp_emission: MERRA2
   exp_aero: 1
   exp_met: MERRA2
   exp_hst: HISTORY.GEOSCTM.rc.tmpl
```

#### GOCART Configuration

```
GOCART_base:
   # ---> Do we do the one plus one equal two experiment?
   one_plus_one: no
   baseline_test: yes
   collection: tavg2d_aer_x
   field_names: ["NIEXTTAU", "BRCEXTTAU", "DUEXTTAU", "SSEXTTAU", "SUEXTTAU", "BCEXTTAU", "OCEXTTAU"]
   exp_duration: 2
   #
   #-------------------------------------------------
   # Settings needed for running the ctm_setup script
   #-------------------------------------------------
   exp_id: testGOCART
   exp_desc: Test GOCART Tracers
   exp_klone: FALSE
   exp_hor_res: c90
   exp_lev: 72
   exp_chem: 2
   exp_emission: MERRA2
   exp_aero: 1
   exp_met: MERRA2
   exp_hst: HISTORY.GEOSCTM.rc.tmpl
```

#### GMI Configuration

```
GMI:
   ## ---> Do we do the one plus one equal two experiment?
   one_plus_one: no
   baseline_test: yes
   collection: gmi_inst
   field_names: ["O3", "CO", "NO"]
   exp_duration: 2
   #
   #-------------------------------------------------
   # Settings needed for running the ctm_setup script
   #-------------------------------------------------
   exp_id: testGMI
   exp_desc: Test GMI
   exp_klone: FALSE
   exp_hor_res: c90
   exp_lev: 72
   exp_hydro: TRUE
   exp_ioserver: FALSE
   exp_proc_type: hasw
   exp_chem: 3
   exp_emission: MERRA2
   exp_aero: 1
   exp_met: MERRA2
   exp_hst: HISTORY.GEOSCTM.rc.tmpl
```

#### Sample Regression Test Report

The report below was generated by doing the three experiments presented above.

```
--------------------------------------------------------------------------------
                    Regression testing of GEOS CTM code base
--------------------------------------------------------------------------------
---> New Tag:      Icarus-3_2_p9_CTM_MEM_16-r4-SLES12
---> Baseline Tag: Icarus-3_2_p9_CTM_MEM_16-r3-SLES12
--------------------------------------------------------------------------------
Experiment Name         Experiment Type    Run Completed?      Passed Test?
--------------------------------------------------------------------------------
idealPT                        V                 +                  -
--------------------------------------------------------------------------------
pTracerTR                      O                 +                  +
--------------------------------------------------------------------------------
GOCART_base                    B                 +
                                                               DUEXTTAU(+)
                                                               SSEXTTAU(+)
                                                               SUEXTTAU(+)
                                                               BCEXTTAU(+)
                                                               OCEXTTAU(+)
--------------------------------------------------------------------------------
Time taken = 03:01:14
--------------------------------------------------------------------------------
Legend:
-------------------------
+   : experiment success
F   : experiment failure
-   : Not available
Experiment Type
    B   : Baseline
    O   : 1+1=2
    V   : Verification
Notes:
------
```
