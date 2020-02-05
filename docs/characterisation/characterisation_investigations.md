# Investigations into Characterisation of Data sets

## Overview

The initial part of the investigation into characterisation of datasets involved looking at the problems captured by the ESMValTool. The idea of this being that we could capture all the characterisitcs in datasets in CMIP5, CMIP6 and CORDEX that are responsible for causing errors. The final aim of the investigation was to have a list of all characteristics that should be stored in our characterisation store.

These characteristics were found by recording the problem, the fix and then which part of the dataset the problem corrsponded to e.g. coordinate, global attribute, variable attribute or the data.

## Scope of investigation

This particular investigation focused only on the problematic characteristics that had been captured by the ESMValTool and so was limited to CMIP5 and CMIP6 datasets. These files were organised by the model the problem had occurred in and the fix was applied to individual variables in a particular model. 

## Results

The results from this investigation showed that none of the recorded problems were in relation to the global attributes. 
However, there were some common problems of note in the CMIP5 and CMIP6 datasets. 

In the CMIP5 datasets a common problem was that the name of the unit of the main variable was incorrect and/or the data was in the incorrect unit, so the data had to be scaled and the name of the unit changed. As well as this, for some variables in some models the coordinate points varied between files in the dataset i.e. between time ranges. This was fixed by  

A common issue was that a fill value had been used for missing data and this had been corrected to a mask on these values by the ESMValTool. Finally, a noticeable issue was that the incorrect standard name of the main variable was incorrect and had to be renamed. 

In the CMIP6 datasets the main problem that was captured was a missing coordinate. While the coordinate was not always the same one, for example some variables were misisng a height coordinate and others a typesea coordinate, the majority were missing a height coordinate (either 2m or 10m).

There were other issues that were captured in the ESMValTool, all the issues captured are summarised in the table below along with the proposed approach. 

## Analysis and proposed approach

More detail is provided in the table below, however, the characteristics to extract as found from this investigation are as follows:

- Max and min of variable data
- Name of units of main variable
- Standard name of main variable
- Long name of main variable
- List of the variables in the dataset 
- List of coordinates in the dataset
- Values of coordinates (latitude, longitude, time and others)
- Fill value of the main variable
- Long name of time variable

Further suggestions following on from this are:

- Long name / standard name of all variables
- Rank of main variable
- Shape of main variable


The table below summaries the problems that were captured. It then follows on by suggesting what characeristic to extract and explaining how this would be used to test for the problem. Finally, the final column shows how to extract this characteristic using xarray, where 'ds' corresponds to the dataset under investigation and 'var' corresponds to the mian variable of the dataset. 

| Problem  | Characteristic to extract | How to check for problem | How to extract using xarray |
|---|---|---|---|
| Calendar/time units incorrect  |   |   |   |
| Data in incorrect units | Max and min of data | Check if data lies within acceptable range as specified in MIP tables | ds.var.values |
| Name of units incorrect | Name of units of main variable | Check name of units of main variable against that described in MIP tables | ds.var.units |
| Bad standard name | Standard name of main variable | Check standard name against that described in MIP tables | ds.var.standard_name |
| Bad long name | Long name of main variable | Check long name against that described in MIP tables | ds.var.long_name |
| Extra variable preventing cube concatenation | List of variables in dataset | Check for existence of variable in list | ds.variables |
| Air pressure coordinate has incorrect name (AR5PL35) | List of coordinates in dataset | Check for existence of coordinate in list | ds.coords |
| Missing scalar dimension | List of coordinates in dataset | Check for existence of coordinate in list  | ds.coords |
| Missing depth coordinate | List of coordinates in dataset | Check for existence of coordinate in list  | ds.coords |
| Missing height coordinate | List of coordinates in dataset | Check for existence of coordinate in list  | ds.coords |
| Missing latitude coordinate | List of coordinates in dataset | Check for existence of coordinate in list  | ds.coords |
| Missing typleland coordiante | List of coordinates in dataset | Check for existence of coordinate in list  | ds.coords |
| Missing typesea coordinate | List of coordinates in dataset | Check for existence of coordinate in list  | ds.coords |
| Coordinate points vary between files in dataset | Values of coordinates |   | ds.coord.values |
| Plev has wrong name (called air_pressure) | List of coordinates in dataset | Check for existence of coordinates in list | ds.coords |
| Fill value of 0 | Fill value of dataset | Search for values equal to the fill value of the dataset |   |
| Fill value of 1.0e33 | Fill value of dataset | Search for values equal to the fill value of the dataset |   |
| Fill value of 1.0e36 | Fill value of dataset | Search for values equal to the fill value of the dataset |   |
| Fill value of 273.15 | Fill value of dataset | Search for values equal to the fill value of the dataset |   |
| Latitude values greater than 90 and less than -90  | Values of latitude | check the values are in the correct range  | ds.lat.values |
| Incorrect variable name for lat and lon | List of variables in dataset | Check for existence of lat/lon and latitude/longitude in list | ds.variables |
| Missing lat bounds and lon bounds | List of variables in dataset | Check for existence of variables in the list | ds.variables |
| Missing long name for time coordinate | Long name of time variable | Check for the existence of the name | ds.time.long_name |
| Incorrect units for parent time |  |  |  |
  
   
