# PhaseDancer


![Diagram showing PhaseDancer algorithm workflow](/images/PHASEDANCER.png?raw=true "PhaseDancer algorithm workflow")

## Quick start

You can try PhaseDancer assembler and accompanying [PhaseDancerViewer](https://github.com/bposzewiecka/phaseDancerViewer) application on the example data generated by the [PhaseDancerSimulator](https://github.com/bposzewiecka/phaseDancerSimulaator) of segmental duplications using docker container.
If you do not have docker installed, please check [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/).

For Linux and Macs download script [example.sh](example.sh) and run it using the following command:

```
./example.sh
```

You can view the results of every iteration of the assembly using [PhaseDancerViewer](https://github.com/bposzewiecka/phaseDancerViewer)  application on [http://127.0.0.1:8000/](http://127.0.0.1:8000/).  When you finish execute following commands:

```
docker stop phasedancer_viewer
docker rm phasedancer_viewer
```

## PhaseDancer configuration


### Step 1: Creating directory with sequenced data

Dictionary for storing the sequencing data of a certain sample should be created in the dictionary named  **data**.
Name of the dictionary should be descriptive and **should start with a letter and contain letters, numbers and hyphens only**.

For example, the following commands:

```
mkdir -p data/chaos-pb-sequel
```

can be used to create the dictionary for the sequencing data of the sample from the chimpanzee Chaos sequenced using Pacbio Sequel technology.

In this newly created directory fastq or fasta file of sequenced sample should be placed.
This file name should have the same name as the directory (plus extension).

For example, the following command:

```
mv ~/SAMPLE1234.fastq data/chaos-pb-sequel/chaos-pb-sequel.fastq
```

can be executed to place input file *SAMPLE1234.fastq* with sequenced genome from home directory in the appropriate phaseDancer sequencing data directory.
Alternatively, symbolic link to such file can be used.

```
ln -s ~/SAMPLE1234.fastq data/chaos-pb-sequel/chaos-pb-sequel.fastq
```

### Step 2: Creating sub-directories for initial contigs to extend

For every contig that will be extended by the algorithm one sub-directory should be created.
Inside each sub-directory a initial contig sequence in fasta format should be placed.
The name of the initial contig should be the same as the name of the corresponding sub-directory (ie. if sub-dictionary name is *telomere2Bp*, the fasta file inside it should have name *telomere2Bp.fasta*).
Name of the sub-dictionary should be descriptive and **should start with a letter and contain letters, numbers and hyphens only**.
One directory for data associated with a certain sample can contain several sub-directories for initial contigs.

For example, the following commands:

```
mkdir -p data/chaos-pb-sequel/telomere2Aq
mv ~/tel2Aq.fasta data/chaos-pb-sequel/telomere2Aq/telomere2Aq.fasta
```
```
mkdir -p data/chaos-pb-sequel/telomere2Bp
mv ~/tel2Bp.fasta data/chaos-pb-sequel/telomere2Bp/telomere2Bp.fasta
```

can be executed to place initial contig tel2Aq.fasta and tel2Bp.fasta from the home directory in the appropriate phaseDancer data sub-directory.

### Step 3: Creating configuration file

Configuration file **config.yaml** list inputs for the PhaseDancer algorithm.
Tree structure of the file enables to list all samples and contigs to extend together with the parameters of the algorithm.

Root element is *samples* dictionary that contains configuration for every sample by its name.
For every sample *technology* and *contigs* property should be specified.
For each sample all initial contigs should be listed as a dictionary (where keys are initial contig names and values are properties overriding those declared at the sample level) or a list of initial contig names.
All properties except  *technology* and *contigs* can be specified at the level of sample and contig.
When property is specified on both levels, property value specified at the contig level overrides property value from the sample level.

```yaml
samples:
    sample1-name:
        technology: pb|hifi|ont
        contig-size: number
        contig-extension-size: number
        browser: true|false
        iterations: number
        mappings: [ reference-name1, reference-name2 ]
        contigs:
            contig-name1:
                key: value
                key: value
                ...
            contig-name2:
                key: value
                key: value
                ...
            contig-name_n:
                key: value
                key: value
                ...
    sample2-name:
        technology: pb|hifi|ont
        contig-size: number
        contig-extension-size: number
        browser: true|false
        iterations: number
        contigs: [ contig-name1, contig-name2, ..., contig-name_n ]
```

| Property | Description | Value | Required | Default | Level |
|---|---|---|---|---|---|
| technology | Technology used for sequencing the sample. Possible values are: <ul><li>**pb** - for PacBio  CLR reads</li><li>**hifi** - for PacBio HiFi/CCS reads</li><li>**ont** - for Oxford Nanopore reads</li></ul> | pb, hifi or ont |  yes  | -  | sample |
| contig-size | Size of the contig. Should be the same as the size of the fasta file in the contig sub-directory. | number  | yes | - | sample, contig |
| contig-extension-size | Maximum size by which the newly created contig will be extended. | number  | yes  | - | sample, contig |
| browser |  If true, the browser will be shown in the PhaseDancerViewer application. | true or false | no | true | sample, contig |
| mappings | List of reference genomes that every contig will be mapped on. Mapping will generate files in *.paf* format. If the name *genome_name* is on the list, file or symbolic link *genome_name.fasta*  should be placed in *phaseDancer/data/refs/* directory.  | list  | no  | -  | sample, contig |
| iterations | Number of iteration of the algorithm extending conitgs to the right. | number | yes  | 0  | sample, contig |
| contigs  | List or dictionary of contigs to extend. | dictionary or list | yes  | - | sample |

For example, the following *yaml* file can be created for sample *chaos-pb-sequel*, and two contigs to extend (*telomere2Aq*, *telomere2Bp*).
In this case technology used to sequence the data is Pacbio Sequel (attribute *technology* is set to *pb*).
Parameters *technology*, *contig-size*, *contig-extension-size*, *browser*, *iterations* are set for all contigs at the level of sample.
Parameters *contig-extension-size* is overridden in the case of contig *telomere2Aq*.
Parameters *iterations" is overridden in the case of contig *telomere2Bp*.

```yaml
samples:
    chaos-pb-sequel:
        technology: 'pb'
        contig-size: 10000
        contig-extension-size: 5000
        browser: true
        iterations: 10
        mappings: [ hg38, panTro6 ]
        contigs:
            telomere2Aq:
                contig-extension-size: 6000
            telomere2Bp:
                iterations: 40
```

For example, the following *yaml* file can be created for sample *chaos-pb-sequel*, and two contigs to extend (*telomere2Aq*, *telomere2Bp*).
All parameters are set at the sample level and property *contigs* contain list of initial contigs.

```yaml
samples:
    chaos-pb-sequel:
        technology: 'pb'
        contig-size: 10000
        contig-extension-size: 5000
        browser: true
        iterations: 10
        mappings: [ hg38, panTro6 ]
        contigs: [ telomere2Aq, telomere2Bp ]
```

The input data for the algorithm should have following structure:

```
│  config.yaml
└─── sample1-name
|   ├───  contig-name1
|   |   └───  contig-name1.fasta
|   ├─── contig-name2
|       └─── contig-name2.fasta 
|   ...    
|   └─── contig-name2_n
|       └─── contig-name_n.fasta        
└─── sample2-name
   ├───  contig-name1
   |   └───  contig-name1.fasta
   ├─── contig-name2
       └─── contig-name2.fasta 
   ...    
   └─── contig-name2_n
       └─── contig-name_n.fasta       
```

In case of the example input data structure should have following structure:
   
```
│  config.yaml
└─── chaos-pb-sequel
   ├─── telomere2Aq
   |   └─── telomere2Aq.fasta
   └─── telomere2Bp
       └─── telomere2Bp.fasta
```
### Step 5: Runninh the docker container

Befere starting the main algorithm  docker container shoul be run using following command:

```
docker run -d -v  /path/to/data:/phaseDancerData -it --name phasedancer bposzewiecka/phasedancer:1.0
```

with ' /path/to/data' replacded by the absolute path to the dictionary containing data.

### Step 4: : Starting the PhaseDancer algorithm

To start the main algorithm, the following line of code should be executed with the *number_of_threads* replaced by the maximum number of threads that algorithm can use and *sample_name* with the name of the sample, contig_name with the name of the contig or string “all”.

```
docker exec phase_dancer ./run_phaseDancer.sh sample_name cotig_name|all number_of_indices number_of_threads
```
For example, following command will execute concurrent assembly of all contigs listed in config.yaml file for the sample chaos-pb-sequel, i.e . telomere2Aq, telomere2Bp using 10 processors and 1 index.

```
 docker exec phase_dancer ./run_phaseDancer.sh chaos-pb-sequel all 1 10
```

### Step 5: Unloading indices from RAM memory

To unload the minimap index from RAM memory following line should be executed:

```
docker exec phase_dancer ./unload_index.sh sample_name number_of_indices
```

For example, to unload the minimap index from RAM memory of chaos-pb-sequel sample, the following line of code should be executed:

```
docker exec phase_dancer ./unload_index.sh chaos-pb-sequel 1
```

### Step 6: Stopping the docker container

After completion of the algorithm the docker container should be stopped

```
docker stop phase_dancer
```

##  PhaseDancerViewer

PhaseDancerViewer is an application for visualizing the results of every iteration of the PhaseDancer algorithm.

More information about the PhaseDancerViewer application can be found on [https://github.com/bposzewiecka/phaseDancerViewer].

![Image PhaseDancerViewer application](/images/phaseDancerViewer.png?raw=true "PhaseDancerViewer application")

Author: Barbara Poszewiecka




