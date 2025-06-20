---
title: Tutorial 3
---
# Tutorial 3 - Modify a NEMO routine
 
In this Tutorial, it is shown how to overcome the iceberg accumulation at the boundaries, which was shown in Section 2.3 in Tutorial 3. 

## 1. Modify the code
When modifying the code, we need to recompile it. As we do not want to have several experiments with different source codes, we create a new configuration.
Therefore, we first compile a NEMO configuration basedon the reference configuration WED025 by:
 
`./makenemo -m ifort_SPIRIT -r WED025 -n WED025_dem2 -j 8 --add_key key_xios3`

The routine, which needs to be modified for removing the icebergs at the open boundary is **icbthm.F90**. This routine is representing the thermodynamics of icebergs. Within this file the icebergs are deleted when they are melted. We add that if icebergs are reaching the boundary of the domain they are deleted.  This modification will be included in further NEMO releases.

To do this modification, we go into the newly configured WED025_dem2 folder:  
`cd <YOURNEMODIRECTORY>/cfgs/WED025_dem2`

next we copy the routine from WORK into MY_SRC:  
`cp WORK/icbthm.F90 MY_SRC/`  
**It is important that, if you want to modify a routine, it is copied into the MY_SRC! Otherwise you might accidently modify the original code, which can impact all configurations.**


To do this, the routine needs to be modified in two places, therefore we do the following steps:  
- Open the file icbthm.F90 in the WORK 
- Next, add in line 27:  
`USE bdy_oce, ONLY : bdytmask,ln_bdy`  
it is always good to make a annotation where you added things yourself in the code, e.g., use: ! your initials
- add in line 275 ( above the ELSE) :
```
ELSE IF(ln_bdy .AND. bdytmask(ii,ij)==0.) THEN ! Delete the berg if at bdy
            CALL icb_utl_delete( first_berg, this )
```
- save the file

Now the configuration needs to be recompiled.  
Go to:  
`cd <YOURNEMODIRECTORY>`  
and compile NEMO by:   
`./makenemo -m ifort_SPIRIT -r WED025_dem2 -n WED025_dem2 -j 8`

To validate if the modification is now in the code go to the WORK in WED025_dem2  
`cd cfgs/WED025_dem2/WORK`  
and open the icbthm.f90 and find the added lines.

If the compilation failed, have a look at the routine provided along with the demonstrator.

## 2. Run NEMO

### 2.1 Test the changes

Now as the code is modified and compiled we can run a test experiment. Go to your new configuration:    
`cd <YOURNEMODIRECTORY>/cfgs/WED025_dem2/`

As we just want to do a reproduction of `WED025_dem/EXP02_icbtest` (see Tutorial 2, Section 2.1) and see how the changes of the code impact the model output, we can make a copy of that experiment:  
`cp -r ../WED025_dem/EXP02_icbtest ./EXP02_icbtest `  

As no namelist parameter has to be modified this experiment can be run straight away.

