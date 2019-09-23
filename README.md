# carol-fi_cuda-parallel 


CAROL-FI CUDA is an cuda-gdb based fault injector. 
CAROL-FI GDB Fault Injector should work on any recent machine with cuda-gdb.

# Requeriments

- NVIDIA GPU, tested with Kepler, Maxwell, Pascal, and Volta architectures 
- CUDA GDB (installed with CUDA drivers)
- Python 2.7
- Libpython2.7 sudo apt install libpython2.7


# How to run a simple test

To test simple_add run the make into the simple_add folder '/codes/simple_add'.

<ol>
<li>First step is the profiler</li>

```{r, engine='bash', code_block_name} 
$ ./app_profiler.py -c codes/simple_add.conf
```

</ol>


 A folder into /tmp directory will be created with the binaries and files needed.
 
 

Then, to run the fault injector use the following command:

```{r, engine='bash', code_block_name} 
$ ./fault_injector.py -c ./codes/simple_add/simple_add.conf -i 10
```

The fault injector will run quicksort 10 times and, the logs will be stored in the *logs* folder

## Configuration files

A configuration file can have several sections, each section (except the DEFAULT section) will specify one fault injection to be performed by CAROL-FI. The DEFAULT section contains information that will be used by every other section.

### DEFAULT section

The DEFAULT section contains the following keys:

* debug - Print additional information in the log files generated by CAROL-FI

* gdbExecName - How to run gdb, you can use the absolute path or execute a specific version of gdb. Usually, you can set the value to 'gdb'

* faultModel - Which fault model to use: 0 -> Single; 1 -> Double; 2 -> Random; 3 -> Zero; 4 -> Least Significant Bits (LSB)

* injectionSite - Where fault will be injected, only RF (Regsiter File) currently working

* maxWaitTimes - This value will be multiplied by the average program execution time to determine if app hanged or not.

* benchmarkBinary - Where the binary file is located, generally at the same folder of the sources

* benchmarkArgs - The args of your program must be  in one line only

* csvFile - CSV output file. It will be overwrite at each injection

* goldenCheckScript - Compare script path. You should create a script to verify the gold output vs. injection output.

* seqSignals - How many SIGINT signals to send to the application, your application cannot use this signal

* initSleep - Wait time to start sending signals. Generally, wait time for memory allocation and cudaMemcpy (in seconds).

You can see an example of a DEFAULT section below:

```

[DEFAULT]

debug = False

gdbExecName = /usr/local/cuda/bin/cuda-gdb

faultModel = 0

injectionSite = RF

maxWaitTimes = 5

benchmarkBinary = /home/carol/carol-fi/codes/micro_mp_hard/cuda_micro_mp_hardening

benchmarkArgs =  --verbose --iterations 10 --precision double --redundancy dmrmixed --inst compose

csvFile = ./fi_micro_compose_dmrmixed_double_single_bit.csv

goldenCheckScript = codes/micro_mp_hard/sdc_check.sh

seqSignals = 20

initSleep = 0

```

### SDC and DUE check script (goldenCheckScript)
To make CAROL-FI able to compare the outputs of your benchmark, you have to create a shell script that compares the output files within the files generated in the profile process. 
The following environment variables will be available at the moment of the execution of the shell script. The environment variables are previously set in the common_parameters.py. I strongly suggest using the default parameters.

* GOLD_OUTPUT_PATH - The Gold file that contains the golden stdout output from the profiler process;

* INJ_OUTPUT_PATH - The injection file that contains the current injection stdout output;

* GOLD_ERR_PATH - The Gold file that contains the golden stderr output from the profiler process;

* INJ_ERR_PATH - The injection file that contains the current injection stderr output;

* DIFF_LOG - The diff output between GOLD_OUTPUT_PATH and INJ_OUTPUT_PATH. 
CAROL-FI will use this file to check if the app returned SDC or not. 
It is good to keep in mind that things that vary in each execution like execution time must be removed from DIFF_LOG
 since it can lead to a false SDC;

* DIFF_ERR_LOG - The diff output between GOLD_ERR_PATH and INJ_ERR_PATH. 
CAROL-FI will use this file to check if the app returned DUE or not. 
It is good to keep in mind that things that vary in each execution like execution time must be removed from DIFF_ERR_LOG 
since it can lead to a false DUE;

#### Golden Check script example:

```
TODO
```

## Fault Models
Currently, we have 5 fault models implemented:

* (0) Single - flips only one bit from all the bits that compose the data
* (1) Double - flips two bits from all the bits that compose the data
* (2) Random - replaces all the bits from the data by a random bit
* (3) Zero - replaces all the bits by zero
* (4) Least Significant Bits (LSB) - flips one bit from the first byte of the data

## Injection sites
Currently, we have only RF injection site implemented:

* RF - Register File
* INST_OUT - Instruction Output (NOT IMPLEMENTED YET)
* INST_composed -> Instruction Adress (NOT IMPLEMENTED YET)
