# Tutorial 2 - Add icebergs to WED025

In this tutorial, it is shown how to add icebergs to the WED025 configuration. This Tutorial covers:
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
```
!-----------------------------------------------------------------------  
&namberg       !   iceberg parameters                                   (default: OFF)  
!-----------------------------------------------------------------------  
   ln_icebergs = .true.      ! activate iceberg floats (force =F with "key_agrif")  
   nn_test_icebergs        =  10     ! Create test icebergs of this class (-1 = no)  
   !                                 ! Put a test iceberg at each gridpoint in box (lon1,lon2,lat1,lat2)  
   rn_test_box             = -80,  2, -80.0, -61.0  
/  
```
with `ln_icebergs = .true.` icebergs are activated; by unsing `nn_test_icebergs =10` we prescirbe icebergs at all ocean points in the box given by `rn_test_box`.

To see the effect of this change we need to add iceberg output to our `file_def_nemo-oce.xml` by:  
```
  <!-- ice berg fields  -->
          <field field_ref="berg_melt"      name="berg_melt" />   
          <field field_ref="berg_virtual_area"    name="berg_virtual_area"  />
``` 
Now the experiment is ready to be run.  

### 2.2 Iceberg using calving files

To run WED025 with an iceberg calving file, we need to edit the `namelist_cfg`, provide a calving file and edit `file_def_nemo-oce.xml`

To set up this experiment, we make a copy of EXP01 from Tutorial 1 by:          
`cp -r EXP01 EXP02_icbcalving`

Then we edit the `namelist_cfg`: 
- add in the `namberg` section, which should be empty, the following lines:    
```
!-----------------------------------------------------------------------  
&namberg       !   iceberg parameters                                   (default: OFF)  
!-----------------------------------------------------------------------
   ln_icebergs = .true.      ! activate iceberg floats (force =F with "key_agrif")  
   nn_test_icebergs=-1
   cn_dir      = './'      !  root directory for the calving data location  
   !___________!_________________________!___________________!___________!_____________!________!___________!__________________!__________!_______________!    
   !           !  file name              ! frequency (hours) ! variable  ! time interp.!  clim  ! 'yearly'/ ! weights filename ! rotation ! land/sea mask !    
   !           !                         !  (if <0  months)  !   name    !   (logical) !  (T/F) ! 'monthly' !                  ! pairing  !    filename   !    
   sn_icb     =  'calving_example'              ,         -12.        ,'calvingmask',  .true.   , .true. , 'yearly'  , ''               , ''       , ''
/
``` 
with `ln_icebergs = .true.` icebergs are activated; by unsing `ln_use_calving          = .true.`we activte the use of a calving file; which is prescribed in `sn_icb`. How to create `calving_example.nc` is shown in Section 3 of this tutorial.


To see the effect of this change we need to add iceberg output to our `file_def_nemo-oce.xml` by:  
```
  <!-- ice berg fields  -->   
          <field field_ref="berg_melt"      name="berg_melt" />  
          <field field_ref="berg_virtual_area"    name="berg_virtual_area"  />
``` 

Now the experiment is ready to be run. 


### 2.3 Quick check of the results

When running a regional configuration with icebergs NEMO 5.0.1 they accumulate at the open boundaries. To fix this, one routine needs to be modified. This is shown in Tutorial 3. 

NOTE: This section is still under development  - more to come soon
 
### 3. Prepare a calving file

NOTE:  This section is still under development - more to come soon 

add jupyter script


