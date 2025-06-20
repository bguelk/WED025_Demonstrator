# NEMO Demonstrator for WED025 in NEMO Version 5.0.1

This demonstrator includes in a first section an update of the existing demonstrator for the regional NEMO configuration WED025 (https://zenodo.org/records/6817000) now using NEMO 5.0.1,  which is aimed at Beginners. In a second part, this demonstrator shows examples for advanced user, e.g.,  how to modify the code and how to create a new forcing field. 

# 1.Section: Update of WED025 in NEMO version 5.0.1 (Beginners)

Now you can submit your job to run the simulation.

Once terminated, you now have run your first WED025 simulation.

# 2.Section: Update the NEMO code (Advanced User)

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
