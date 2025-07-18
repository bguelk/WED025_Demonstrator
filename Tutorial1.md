# Tutorial 1 - Run WED025 in version 5.0.1

## 1. Prerequisities
- Access to an HPC with at least 32 cores available
- Netcdf package installed on your HPC
- XIOS package installed on your HPC
- An arch file for your HPC (this just sets paths to XIOS, netcdf and the compiling option which depends on your compiler)
- Python installed on your HPC or personal computer (for analysis and preparation of iceberg calving file)
- Knowledge to submit a script to the batch scheduler of your HPC
- Knowledge on how to run a programme on multicores (at least 32)

## 2. Provided:
- Topography, coordinates, boundary conditions, ocean and sea ice initial conditions, runoff and atmospheric forcing.
- Namelist to build the domain_cfg.nc file from the topography and coordinates file adapted to the demonstrator.
 
All the needed files can be found [here](https://zenodo.org/records/6817000) in the file: `WED025_demonstrator_forcings.tar.zip`

The file can be dowloaded with
`wget https://zenodo.org/records/6817000/files/WED025_demonstrator_forcings.tar.zip`

## 3. Set up the configuration

### 3.1 Install and compile XIOS3

Download the XIOS3 version with

`svn co http://forge.ipsl.jussieu.fr/ioserver/svn/XIOS3/trunk <YOURXIOSDIRECTORY>`

Setup your arch files (one for the environment (.env), another for the compiler (.fcm) and a last one for the path (.path) ):
- cd arch to see examples of what this file looks like e.g. arch/arch-ifort_MESOIPSL.*.
- setup your arch files (`arch/arch-MY_COMPUTER.*`) depending of your environement, compiler and computer.
- Then you compile XIOS (cd .. to `<YOURXIOSDIRECTORY>`) referring to your set of arch files:

`./make_xios --arch MY_COMPUTER --full --prod --job 8`

- Now cd .. back to your workdir

XIOS3 is now compiled

### 3.2 Install and compile NEMO version 5.0.1

Download NEMO version 5.0.1 by:

`git clone --branch 5.0.1 https://forge.nemo-ocean.eu/nemo/nemo.git <YOURNEMODIRECTORY>`

After successfully downloading, add your arch file (`arch/arch-MY_COMPUTER.fcm`) under `<YOURNEMODIRECTORY>/arch` and set up the correct path for netcdf, HDF5 and XIOS (%NCDF_HOME, %HDF5_HOME and %XIOS_HOME). Examples are available in the directory `arch`

Now, you can start compiling the configuration based on the reference configuration WED025, as we use XIOS3 the keys in the compilation need to be changed. The new configuration is called ‘WED025_dem’.
To compile WED025_dem run the following line (ifort_SPIRIT is the used arch file):

`./makenemo -m MY_COMPUTER -r WED025 -n WED025_dem -j 8 --add_key key_xios3`

Now the configuration is compiled.

### 3.3 Create domain_cfg file

The domain_cfg.nc file describes the domain by providing information of the horizontal and vertical mesh, the bathymetry and ice shelf draft. To create the `domain_cfg.nc` file, the bathymetry, iceshelf draft and coordinates for the horizontal grid as well as the domain geometry needs to be known.

To create a `domain_cfg.nc` file, the DOMAINcfg tool can be used. Therefore, this tool needs to be compiled.
First, go into the NEMO tool directory:
`cd <YOURNEMODIRECTORY>/tools` 
Then compile the DOMAINcfg tool using the same arch file as in the NEMO compilation (here: ifort_SPIRIT):
`./maketools -m MY_COMPUTER -n DOMAINcfg`

The DOMAINcfg tool is now compiled.

The next step is to build the `domain_cfg.nc` file. To create this file from the provided information:
- Go back to your forcing directory, WED025_demonstrator_forcings, and create your own DOMAIN Folder, e.g. mkdir DOMAIN_WED025
- Then from the DOMAINcfg tools folder copy or link the following into your local DOMAIN_WED025 directory:  
`ln -s <YOURWORKDIRECTORY>/<YOURNEMODIRECTORY>/tools/DOMAINcfg/BLD/bin/make_domain_cfg.exe .`  
`ln -s <YOURWORKDIRECTORY>/<YOURNEMODIRECTORY>/tools/DOMAINcfg/BLD/bin/dom_doc.exe .`  
`cp <YOURWORKDIRECTORY>/<YOURNEMODIRECTORY>/tools/DOMAINcfg/namelist_ref .`

- From your forcing folder, copy namelist_cfg_dom, bathy_meter_WED025.nc and
coordinates_WED025.nc from your forcings folder into your DOMAIN_WED025
subfolder
- Then in your subfolder (DOMAIN_WED025) rename namelist_cfg_dom namelist_cfg (`mv namelist_cfg_dom namelist_cfg`)
Instead of providing a ready to use namelist, the namelist provided needs to be personalized to
your configuration:

- Open namelist_cfg and change the experience name:  
```

!-----------------------------------------------------------------------  
&namrun ! parameters of the run  
!-----------------------------------------------------------------------  
cn_exp = "WED025" ! experience name
```

- Rename to read in our files:  
```
!-----------------------------------------------------------------------  
&namdom ! space and time domain (bathymetry, mesh, timestep)  
!-----------------------------------------------------------------------  
cn_fcoord = 'coordinates_WED025.nc' ! external coordinates file (jphgr_msh = 0)  
cn_topo = 'bathy_meter_WED025.nc' ! external topo file (nn_bathy =1/2)  
cn_fisfd = 'bathy_meter_WED025.nc' ! external isf draft (nn_bathy =1 and ln_isfcav =.true.)  
```
- Change the dimensions to match the bathymetry file and change the name of the
configuration:  
```
!-----------------------------------------------------------------------  
&namcfg ! parameters of the configuration  
!-----------------------------------------------------------------------  
cp_cfg = "WED" ! name of the configuration  
jpidta = 322 ! 1st lateral dimension ( >= jpi )  
jpjdta = 328 ! 2nd " " ( >= jpj )  
jpkdta = 75 ! number of levels ( >= jpk )  
Ni0glo = 322 ! 1st dimension of global domain --> i =jpidta  
Nj0glo = 328 ! 2nd - - --> j =jpjdta  
jpkglo = 75  
```
- Also change the minimum thickness of the ice shelf draft and water column thickness  
```
!-----------------------------------------------------------------------  
&namzgr_isf ! isf cavity geometry definition (default: OFF)  
!-----------------------------------------------------------------------  
rn_isfdep_min = 20. ! minimum isf draft tickness (if lower, isf draft set to this value)  
rn_glhw_min = 0.01 ! minimum water column thickness to define the grounding line  
```
Finally, we can create the `domain_cfg.nc` file:
- Execute domain_cfg.exe:  
`./make_domain_cfg.exe`

Note: If you do not have enough computing resources, you can create and submit a script that
executes make_domain_cfg.exe on 1 or several cpus. this submission script is very dependent on
your HPC, so a general recommendation is to look at your cluster documentation or ask
your friends or colleagues.

Domain_cfg.nc file is now created.

### 3.4 Using REBUILD_NEMO tool

If the DOMAINcfg tool is run on one cpu, this step can be skipped, else the DOMAINcfg tool provides output fields for each used cpu, named `domain_cfg_xxxx.nc`. These files can be combined to one file using the REBUILD_NEMO tool.

First, go to your NEMO tool directory  
`cd <YOURNEMODIRECTORY>/tools`

Then compile the tool in a similar way as the DOMAINcfg tool has been compiled and use the same arch file as before:  
`./maketools -m MY_COMPUTER -n REBUILD_NEMO`

After compilation of the REBUILD_NEMO tool, the `domain_cfg_xxxx.nc` can be combined.
cd to the DOMAIN_WED025 folder, where you have the `domain_cfg_xxxx.nc` files and combine them to one file by:  
` <YOURWORKDIRECTORY>/<YOURNEMODIRECTORY>/tools/REBUILD_NEMO/rebuild_nemo domain_cfg <NO_CPUs> `

where `<NO_CPUs>` is the number of cpus used in the domain file. This will generate one filed called `domain_cfg.nc`.

NOTE: This tool can also be used for other files which are generatedon several cpus, like mesh_mask, restart files. Those files can be identified by *name_xxxx.nc*, where *xxxx* is starting from 0000 and increasing up to the numbers of cpu used.

### 3.5 Run NEMO

Now, everything is ready and the configuration can be run.
Go to your demonstrator folder:  
`cd <YOURNEMODIRECTORY>/cfgs/WED025_dem/`

copy the experiment folder which is already there, here EXP00, to keep the initial files safe:  
` cp -r EXP00 EXP01`

Go into EXP01, where we will run the configuration.
Before the configuration is ready to run, the forcing files and the `domain_cfg.nc` needs to be copied or linked in the folder.

- Link domain_cfg.nc here:  
`ln -s <PATHTO_DOMAIN_WED025>/domain_cfg.nc .`

- Link all the WED025 demonstrator forcings netcdfs to this experiment folder:  
`ln -s <YOURWORKDIR>/WED025_demonstrator_forcings/*nc .`

- check in the `namelist_cfg` the section for the following lines:  
```
!-----------------------------------------------------------------------  
&namsbc_rnf    !   runoffs                                              (ln_rnf =T)  
!-----------------------------------------------------------------------  
   ln_rnf_icb   = .true.   !  read iceberg flux  
   cn_dir      = './'      !  root directory for the runoff data location  
   !___________!_________________________!___________________!___________!_____________!________!___________!__________________!__________!_______________!  
   !           !  file name              ! frequency (hours) ! variable  ! time interp.!  clim  ! 'yearly'/ ! weights filename ! rotation ! land/sea mask !  
   !           !                         !  (if <0  months)  !   name    !   (logical) !  (T/F) ! 'monthly' !                  ! pairing  !    filename   !  
   sn_rnf      = 'WED025_runoff'         ,  -1               , 'runoff'  ,   .true.    , .true., 'yearly'  , ''               , ''       , ''  
   sn_i_rnf    = 'WED025_icb'            ,  -1               , 'icb_melt',   .true.    ,.false., 'yearly'  , ''               , ''       , ''  
/
```
Before you launch the job, you need to change the last timestep of the simulation (nn_itend) to
match the length of the simulation you want to do. In this case we will do a one month

Example how to calculate your end timestep:
In this configuration we use a timestep of 2400 seconds, given in the `namelist_cfg` by:  
`rn_Dt = 2400.`

So we want 31 days x (24 x 60 x 60) = 2678400 seconds.
Now divide by the timestep 2678400 / 2400 = 1116

Edit namelist_cfg:  
`nn_itend = 1116 ! last time step`

For now, we use 5-daily output, if other output frequencies are wished this can be modified in the `file_def_nemo-oce.xml` and `file_def_nemo-ice.xml`. Note that `field_def_*` is where all the output
variables available are listed and which frequency those are outputted.
For example to change the ouput to monthly output, replace each mention of 5d with 1mo in `file_def_nemo-oce.xml` and `file_def_nemo-ice.xml` and at the end of the script, remove this line:  
`<file_group id="1m" output_freq="1mo" output_level="10" enabled=".TRUE."/> <!-- real monthly files →`


For the use of XIOS3, NEMO and XIOS3 needs to be run in detached mode.
This means, in the file` iodef.xml` the following line needs to be:  
` <variable id="using_server"              type="bool">true</variable>`  
and the following line needs to be removed or commented:  
` <variable id="oasis_codes_id"            type="string" >oceanx</variable>`  


Last thing to do, is to copy the `xios_server.exe` into the folder of the configuration:    
` cp <YOURXIOSDIRECTORY>/bin/xios_server.exe ./ `

Now you have everything you need to run your regional configuration.
For this you need to build a script to run on HPC. We suggest you use 32 MPI.
Suggestion: look at the supercomputer documentation or copy from a friend.

As XIOS is in detached mode, when submitting the run, cpus for NEMO and XIOS3 need to be assigned in a bash-script.
Here is an exemple for a simple bash-script for running NEMO with 4 cpus and XIOS with 1 cpu: 
```
#!/bin/sh
#OPENMPI JOB
#SBATCH --ntasks=5
#SBATCH --time 4:00:00

mpirun -np 4 ./nemo -np 1 ./xios_server.exe
```
The command `mpirun` is HPC dependent and can vary between machines.

Now you can submit your job to run the simulation.

Once terminated, you now have run your first WED025 simulation.



---

