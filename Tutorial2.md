# NEMO Demonstrator for WED025 in NEMO Version 5.0.1

# Tutorial 2 - Add icebergs to WED025

In this Tutorial, we add icebergs to the WED025 configuration. This Tutorial covers:
- how to add icebergs   
  - in a test case  
  - using a calving file  
- how to prepare a calving file 

DISCLAIMER: The iceberg tests shown here are just examples and not fully realistic. The WED025 is set up using a iceberg melt climatology for icebergs, but when adding icebers, as shown in this tutorial, freshwater fluxes from icebergs will be double counted (climatology and icebergs). For more realistic results, the climatological forcing used in 'icb_melt' in 'WED025_icb*.nc' needs to be corrected for icebergs calved in the regional configuration.

## 1. Prerequisities
- have a running WED025 configuration (see Tutorial 1)


## 2. Run WED025 with icebergs

### 2.1 Iceberg test case

To run the iceberg test in WED025, we need to modify the `namelist_cfg` and `file_def_nemo-oce.xml`.

To set up this experiment, we make a copy of EXP01 from Tutorial 1 by:  
`cp -r EXP01 EXP02_icbtest`

Then we edit the `namelist_cfg`: 
- add in the `namberg` section, which should be empty, the following lines: 
!-----------------------------------------------------------------------  
&namberg       !   iceberg parameters                                   (default: OFF)  
!-----------------------------------------------------------------------  
   ln_icebergs = .true.      ! activate iceberg floats (force =F with "key_agrif")  
   nn_test_icebergs        =  10     ! Create test icebergs of this class (-1 = no)  
   !                                 ! Put a test iceberg at each gridpoint in box (lon1,lon2,lat1,lat2)  
   rn_test_box             = -80,  2, -80.0, -61.0  
/  

with `ln_icebergs = .true.` icebergs are activated; by unsing `nn_test_icebergs =10` we prescirbe icebergs at all ocean points in the box given by `rn_test_box`.

To see the effect of this change we need to add iceberg output to our `file_def_nemo-oce.xml` by:
`  <!-- ice berg fields  -->  
          <field field_ref="berg_melt"      name="berg_melt" />  
          <field field_ref="berg_virtual_area"    name="berg_virtual_area"  />`  


### 2.2 Iceberg using calving files

To run WED025 with an iceberg calving file, we need to edit the `namelist_cfg`, provide a calving file and edit `file_def_nemo-oce.xml`

To set up this experiment, we make a copy of EXP01 from Tutorial 1 by:          
`cp -r EXP01 EXP02_icbcalving`

Then we edit the `namelist_cfg`: 
- add in the `namberg` section, which should be empty, the following lines:  
!-----------------------------------------------------------------------
&namberg       !   iceberg parameters                                   (default: OFF)
!-----------------------------------------------------------------------
   ln_icebergs = .true.      ! activate iceberg floats (force =F with "key_agrif")  
   ln_use_calving          = .true. ! Use calving data even when nn_test_icebergs > 0  
   rn_speed_limit          = 0.      ! CFL speed limit for a berg (safe value is 0.4, see #2581)    
   cn_dir      = './'      !  root directory for the calving data location
   !___________!_________________________!___________________!___________!_____________!________!___________!__________________!__________!_______________!  
   !           !  file name              ! frequency (hours) ! variable  ! time interp.!  clim  ! 'yearly'/ ! weights filename ! rotation ! land/sea mask !  
   !           !                         !  (if <0  months)  !   name    !   (logical) !  (T/F) ! 'monthly' !                  ! pairing  !    filename   !  
   sn_icb     =  'calving_example'              ,         -1.        ,'calvingmask',  .true.   , .true. , 'yearly'  , ''               , ''       , ''
/

with `ln_icebergs = .true.` icebergs are activated; by unsing `ln_use_calving          = .true.`we activte the use of a calving file; which is prescribed in `sn_icb`. How to create `calving_example.nc` is shown in Section 3 of this tutorial.


To see the effect of this change we need to add iceberg output to our `file_def_nemo-oce.xml` by:
`  <!-- ice berg fields  -->   
          <field field_ref="berg_melt"      name="berg_melt" />   
          <field field_ref="berg_virtual_area"    name="berg_virtual_area"  />`  

### 2.3 Quick check of the results

When running a regional configuration with icebergs NEMO 5.0.1 they accumulate at the open boundaries. To fix this, one routine needs to be modified. This is shown in Tutorial 3. 

NOTE: This section is still under development  - more to come soon
 
### 3. Prepare a calving file

NOTE:  This section is still under development - more to come soon 

add jupyter script

## 1. Modify the code
In this section, it is demonstrated how the code can be modified and recompiled. 

To create a WED025 configuration with a modified code, first we compile a NEMO configuration basedon the reference configuration WED025 by:
 
`./makenemo -m ifort_SPIRIT -r WED025 -n WED025_dem2 -j 8 --add_key key_xios3`

In this demonstrator, we want to modify the routine ==icbthm.F90==, this routine is representing the thermodynamics of icebergs. Within this file the icebergs are deleted when they are melted. We add that if icebergs are reaching the boundary of the domain are deleted. == This modification will be included in further NEMO versions ==

To do this modification, we go into the newly configured WED025_dem2 folder
`cd *yournemodirectory*/cfgs/WED025_dem2`

next we copy from WORK the routine into MY_SRC:
`cp WORK/icbthm.F90 MY_SRC/`
== It is important that, if you want to modify a routine, it is copied into the MY_SRC! Otherwis you might accidently modify the original code, which can impact all configurations.==

To do this, the routine needs to be modified in two places, therefore we do the following steps:
-Open the file icbthm.F90 in the WORK 
-Next, add in line 27
`USE bdy_oce, ONLY : bdytmask,ln_bdy`
it is always good to make a annotation where you added things yourself in the code, e.g., use: ! your initials
- add in line 275 ( above the ELSE) :
`ELSE IF(ln_bdy .AND. bdytmask(ii,ij)==0.) THEN ! Delete the berg if at bdy
            CALL icb_utl_delete( first_berg, this )`
- save the file

Now the configuration needs to be recompiled.
Go to:
`cd *yournemodirectory*`
and compile NEMO by: 
`./makenemo -m ifort_SPIRIT -r WED025_dem2 -n WED025_dem2 -j 8`

To validate if the modification is now in the code go to the WORK in WED025_dem2
`cd cfgs/WED025_dem2/WORK`
and open the icbthm.f90 and find the added lines.

If the compilation failed, have a look at the routine provided along with the demonstrator.

## 2. Run NEMO

### 2.1 Test the changes

Now as the code is modified and compiled we can run a test.

Therefore, we set up a configuration as explained in Section 1;

As the reference WED025 configuration does not include icebergs, we need to edit the namelist_cfg.
- open the namlist_cfg
- add in the namberg section, which should be empty the following lines:
!-----------------------------------------------------------------------
&namberg       !   iceberg parameters                                   (default: OFF)
!-----------------------------------------------------------------------
   ln_icebergs = .true.      ! activate iceberg floats (force =F with "key_agrif")
   nn_test_icebergs        =  10     ! Create test icebergs of this class (-1 = no)
   !                                 ! Put a test iceberg at each gridpoint in box (lon1,lon2,lat1,lat2)
   rn_test_box             = -80,  2, -80.0, -61.0
/

with  ln_icebergs = .true. icebergs are activated in the configuration by unsing nn_test_icebergs =10 we prescirbe icebergs at all ocean points in the box given by rn_test_box.

To see the effect of this change we need to add iceberg output to our file_def_nemo-oce.xml by:
 
  <!-- ice berg fields  --> 
          <field field_ref="berg_melt"      name="berg_melt" />
          <field field_ref="berg_virtual_area"    name="berg_virtual_area"  />


If a comparison to the unmodified code from Section 1, is wished, one can setup an experiment in the WED025_dem by copying EXP01 to a new folder, cleaning the output and modifing the namelist_cfg and file_def_nemo-oce.xml as done above.


### 2.2 Create a new calving file
Now as the code is modified and compiled we can run a test.

Therefore, we set up a configuration as explained in Section 1;

As the reference WED025 configuration does not include icebergs, we need to edit the namelist_cfg.
- open the namlist_cfg
- add in the namberg section, which should be empty the following lines:
!-----------------------------------------------------------------------
&namberg       !   iceberg parameters                                   (default: OFF)
!-----------------------------------------------------------------------
   ln_icebergs = .true.      ! activate iceberg floats (force =F with "key_agrif")
   ln_use_calving          = .false. ! Use calving data even when nn_test_icebergs > 0
   rn_speed_limit          = 0.      ! CFL speed limit for a berg (safe value is 0.4, see #2581)
   !___________!_________________________!___________________!___________!_____________!________!___________!__________________!__________!_______________!
   !           !  file name              ! frequency (hours) ! variable  ! time interp.!  clim  ! 'yearly'/ ! weights filename ! rotation ! land/sea mask !
   !           !                         !  (if <0  months)  !   name    !   (logical) !  (T/F) ! 'monthly' !                  ! pairing  !    filename   !
   sn_icb     =  'calving'              ,         -1.        ,'calvingmask',  .true.   , .true. , 'yearly'  , ''               , ''       , ''
/

/




# NOTES TO MYSELF:
/NEMO_Hackathon/nemo_5.0.1/cfgs/WED025_dem/EXP01
original code and no icebergs
/NEMO_Hackathon/nemo_5.0.1/cfgs/WED025_dem/EXP02icb
original code and icebergs
~/NEMO_Hackathon/nemo_5.0.1/cfgs/WED025_dem2/EXP01$
modified code and added icebergs
