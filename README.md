## How to run it?

### Run the container

```bash
$ docker run -ti -v $PWD:/tmp -w /tmp/  --entrypoint=/bin/bash mintproject/mint_dt
```

### Run the MIC run file

Then, you must run the `run` file.

The run file is a bash file that expects the name of the inputs, parameters and name of the outputs 

*Note: the run file extracts any zip file*

#### Example 

For topoflow_climate, the inputs, parameters, and outputs are:

- INPUTS1: A nc4 or zip file
- PARAMS1: var_name
- PARAMS2: bounding_box
- PARAMS3: xres_arcsecs
- PARAMS4: yres_arcsecs
- OUTPUTS1: output file


```bash
./run -i1 demo-gpm.nc4 -p1 "precipitation" -p2 "23.995416666666, 6.532916666667, 28.020416666666, 9.566250000000" -p3 30 -p4 30 -o1 outputs1.zip
```

### Editing the number of parameters, inputs and outputs

If you change the number of inputs, parameters or outputs. You must edit the line number 3:

For example, the topoflow 

```bash
. $BASEDIR/io.sh ${NUMBER_INPUTS} ${NUMBER_PARAMETERS} ${NUMBER_OUTPUTS} "$@"` 
```

#### Example 

In the previous example, the result is:

```bash
#!/bin/bash
BASEDIR=`dirname $0`
. $BASEDIR/io.sh 1 4 1 "$@"
CURDIR=`pwd`
set -e
```

### Replacing the template file.

To replace the variables from the run in the configuration file, you must export the variables in run file.

#### Example

```
#!/bin/bash
BASEDIR=`dirname $0`
. $BASEDIR/io.sh 1 4 1 "$@"
CURDIR=`pwd`
set -e

export PARAMS1
export PARAMS2
export PARAMS3
export PARAMS4
export OUTPUTS1
```

#### Editing the configuration file with the varibles. 

Replace each value with the variable.

**Important**: 
Please, dont use absolute paths in the configuration file. It can be fragile to changes.

```
  input_dir: "/tmp/files"
```
Use the ${PWD} variable
```
  input_dir: "${PWD}/files"
```

#### Example
```
version: "1"
description: Data transformation to generate TopoFlow-ready precipitation files (RTS) from Global Precipitation Measurement (GPM) data sources
inputs:
  input_dir: "${PWD}/files"
  temp_dir: "${PWD}/temp"
  output_file: ${OUTPUTS1}
  var_name: ${PARAMS1}
  bounding_box: "${PARAMS2}"
  xres_arcsecs: "${PARAMS3}"
  yres_arcsecs: "${PARAMS4}"
adapters:
  tf_climate:
    comment: My topoflow climate write adapter
    adapter: funcs.Topoflow4ClimateWriteFunc
    inputs:
      input_dir: $$.input_dir
      temp_dir: $$.temp_dir
      output_file: $$.output_file
      var_name: $$.var_name
      DEM_bounds: $$.bounding_box
      DEM_xres_arcsecs: $$.xres_arcsecs
      DEM_yres_arcsecs: $$.yres_arcsecs
```




## Example of a execution

This a example of the execution

```
./run -i1 demo-gpm.nc4 -p1 "precipitation" -p2 "23.995416666666, 6.532916666667, 28.020416666666, 9.566250000000" -p3 30 -p4 30 -o1 outputs1.zip
Archive:  ./outputs1.zip
 extracting: outputs1.rts
 extracting: outputs1.rti
++ readlink -f topoflow_climate.yml
+ config_file=/tmp/topoflow_climate.yml
++ readlink -f .env
+ env_file=/tmp/.env
++ readlink -f outputs1.zip
+ output=/tmp/outputs1.zip
+ mkdir -p files
+ find . -type f -name '*.nc4' -exec cp '{}' files ';'
cp: './files/demo-gpm.nc4' and 'files/demo-gpm.nc4' are the same file
+ envsubst
+ pushd /ws
/ws /tmp
+ dotenv -f /tmp/.env run python -m dtran.main exec_pipeline --config /tmp/topoflow_climate.yml
Importing TopoFlow 3.6 packages:
   topoflow.utils
   topoflow.utils.tests
   topoflow.components
   topoflow.components.tests
   topoflow.framework
   topoflow.framework.tests

Importing TopoFlow 3.6 packages:
   topoflow.utils
   topoflow.utils.tests
   topoflow.components
   topoflow.components.tests
   topoflow.framework
   topoflow.framework.tests

Paths for this package:
framework_dir = /ws/funcs/topoflow/topoflow/framework/
parent_dir    = /ws/funcs/topoflow/topoflow/
examples_dir  = /ws/funcs/topoflow/topoflow/examples/
__file__      = /ws/funcs/topoflow/topoflow/framework/emeli.py
__name__      = funcs.topoflow.topoflow.framework.emeli

Warning 1: No UNIDATA NC_GLOBAL:Conventions attribute
>>> convert gpm file to geotiff: demo-gpm at 21:32:10|**|finish read metadata data at 21:32:10|**|finish read array data at 21:32:10|**|finish rotating and transforming netcdf data at 21:32:10|**|finish write geotiff data at 21:32:11
0it [00:00, ?it/s]
>>> write to files
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1/1 [00:00<00:00, 17.38it/s]
Finished creating info structure.

Finished writing new RTI file:
   /tmp/temp/outputs1.rti


Max precip rate = -1
bad_count = 0
n_grids   = 0
Finished saving data to rts file and generating a matching rti file.

Zipping /tmp/temp/outputs1.rts and /tmp/temp/outputs1.rti...
Zipping is complete. Please check outputs1.zip
+ mv outputs1.zip /tmp/outputs1.zip
+ popd
/tmp
+ set -e
+ cd .
+ . ./output.sh
```