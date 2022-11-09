# PhaseDancer

##  Workflow of the algorithm

![Diagram showing PhaseDancer algorithm workflow](/images/PHASEDANCER.png?raw=true "PhaseDancer algorithm workflow")


## Configuration


### Step 1: Creating directory with sample data

Dictionary for storing the sequencing data of a certain sample should be created in the **data** dictionary.
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

### Step 2: Creating sub-directories for start contigs to extend

For every contig that will be extended by the algorithm one sub-directory should be created.
Inside each sub-directory a start contig sequence in fasta format should be placed.
The name of the start contig should be the same as the name of the corresponding sub-directory (ie. if sub-dictionary name is *telomere2Bp*, the fasta file inside it should have name *telomere2Bp.fasta*).
Name of the sub-dictionary should be descriptive and **should start with a letter and contain letters, numbers and hyphens only**.
One directory for data associated with a certain sample can contain several sub-directories for start contigs.

For example, the following commands:

```
mkdir -p data/chaos-pb-sequel/telomere2Aq
mv ~/tel2Aq.fasta data/chaos-pb-sequel/telomere2Aq/telomere2Aq.fasta
```
```
mkdir -p data/chaos-pb-sequel/telomere2Bp
mv ~/tel2Bp.fasta data/chaos-pb-sequel/telomere2Bp/telomere2Bp.fasta
```

can be executed to place start contig tel2Aq.fasta and tel2Bp.fasta from the home directory in the appropriate phaseDancer data sub-directory.

### Step 3: Creating configuration file

Configuration file **config.yaml** list inputs for the PhaseDancer algorithm.
Tree structure of the file enables to list all samples and contigs to extend together with the parameters of the algorithm.

Root element is *samples* dictionary that contains configuration for every sample by its name.
For every sample *technology* and *contigs* property should be specified.
For each sample all start contigs should be listed as a dictionary (where keys are start contig names and values are properties overriding those declared at the sample level) or a list of start contig names.
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
Parameters *contig-extension-size* and *iterations* are overridden in the case of contig *telomere2Aq*.
Parameters *igv-screenshots: true* and *iterations* are overridden in the case of contig *telomere2Bp*.

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
                iterations: 40
                contig-extension-size: 6000
            telomere2Bp:
                iterations: 40
```

For example, the following *yaml* file can be created for sample *chaos-pb-sequel*, and two contigs to extend (*telomere2Aq*, *telomere2Bp*).
All parameters are set at the sample level and property *contigs* contain list of start contigs.

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

### Step 6: Creating indices of sequenced sample

To create *fai* and *minimap2* indices of sequenced sample following line should be executed:

```
snakemake --snakefile=GenerateIndices.smk  --config sample=sample_name  --cores 5
```

To create indices of the chaos-pb-sequel sample, the following line of code should be executed:

```
snakemake --snakefile=GenerateIndices.smk  --config sample=chaos-pb-sequel  --cores 5
```

### Step 7: Loading index into RAM memory

To load the minimap index into RAM memory following line should be executed:

```
$PHASEDANCER_DIR/load_index.sh sample_name
```

For example, to load the minimap index into RAM memory of chaos-pb-sequel sample, the following line of code should be executed:

```
$PHASEDANCER_DIR/load_index.sh chaos-pb-sequel
```

### Step 8: Starting the main algorithm

To start the main algorithm, the following line of code should be executed with the *number_of_threads* replaced by the maximum number of threads that algorithm can use and *sample_name* with the name of the sample.

```
$PHASEDANCER_DIR/snakemake --config sample=sample_name --cores number_of_threads
```

For example, following command will execute assembly of all contigs listed in **config.yaml** file for the sample *chaos-pb-sequel*, i.e . *telomere2Aq*, *telomere2Bp* using 10 processors.

```
$PHASEDANCER_DIR/snakemake --config sample=chaos-pb-sequel --cores 10
```

### Step 9: Unloading index from RAM memory

To unload the minimap index from RAM memory following line should be executed:

```
$PHASEDANCER_DIR/unload_index.sh sample_name
```

For example, to unload the minimap index from RAM memory of chaos-pb-sequel sample, the following line of code should be executed:

```
$PHASEDANCER_DIR/unload_index.sh chaos-pb-sequel
```

##  PhaseDancerViewer

PhaseDancerViewer is an application for visualizing the results of every iteration of the PhaseDancer algorithm.

More information about the PhaseDancerViewer application can be found on [https://github.com/bposzewiecka/phaseDancerViewer].

![Image PhaseDancerViewer application](/images/phaseDancerViewer.png?raw=true "PhaseDancerViewer application")

Author: Barbara Poszewiecka
