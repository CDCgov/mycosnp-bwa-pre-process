# MycoSNP: BWA Pre-Process

## Overview

This repository contains the MycoSNP BWA Pre-Process workflow, which consists of ten steps:

1. Combine FASTQ file lanes if they were provided with multiple lanes.
2. Down sample FASTQ files to a desired coverage or sampling rate using SeqTK 1.3.
3. Trim reads and assess quality using FaQCs 2.09.
4. Align FASTQ reads to a reference using BWA 0.7.17.
5. Sort BAM files using SAMTools 1.10.
6. Mark and remove duplicates in the BAM file using Picard 2.22.7.
7. Clean the BAM file using Picard 2.22.7 "CleanSam".
8. Fix mate information in the BAM file using Picard 2.22.7 "FixMateInformation".
9. Add read groups to the BAM file using Picard 2.22.7 "AddOrReplaceReadGroups".
10. Index the BAM file using SAMTools 1.10.

## Requirements

Before installing and running this workflow, follow the instructions in the main repository to install the requirements: https://github.com/CDCgov/mycosnp

## Installation

First install GeneFlow and its dependencies as follows:

1. Activate the [previously installed](https://github.com/CDCgov/mycosnp) Python virtual environment.

    ```bash
    cd ~/mycosnp
    source gfpy/bin/activate
    ```

2. Clone and install the mycosnp-bwa-pre-process workflow.

    ```bash
    gf install-workflow --make-apps -g https://github.com/CDCgov/mycosnp-bwa-pre-process mycosnp-bwa-pre-process
    ```

    The workflow should now be installed in the `~/mycosnp/mycosnp-bwa-pre-process` directory.

## Execution

View the workflow parameter requirements using GeneFlow's `help` command:

```
cd ~/mycosnp
gf help mycosnp-bwa-pre-process
```

Execute the workflow with a command similar to the following. Be sure to replace the `/path/to/reference/..` with your reference sequence, and replace the `/path/to/bwa/index` with the path to the index created by the MycoSNP BWA Reference workflow:

```
gf --log-level debug run mycosnp-bwa-pre-process \
    -o ./output \
    -n test-mycosnp-bwa-pre-process \
    --in.input_folder ./test-data1 ./test-data2 ./test-data3 \
    --in.reference_index /path/to/bwa/index \
    --in.reference_sequence /path/to/reference/candida-auris_clade-i_B8441_GCA_002759435.2.fasta \
        --param.rate 1.0 \
        --param.threads 4
```

Alternatively, to execute the workflow on an HPC system, you must first set the DRMAA library path environment variable. For example:

```
export DRMAA_LIBRARY_PATH=/opt/sge/lib/lx-amd64/libdrmaa2.so
```

Note that the DRMAA library for your specific scheduler (either UGE/SGE or SLURM) must be installed, and the installed library path may be different in your environment. Once the environment has been configured, execute the workflow as follows:

```
gf --log-level debug run mycosnp-bwa-pre-process \
    -o ./output \
    -n test-mycosnp-bwa-pre-process \
    --in.input_folder ./test-data1 ./test-data2 ./test-data3 \
    --in.reference_index /path/to/bwa/index \
    --in.reference_sequence /path/to/reference/candida-auris_clade-i_B8441_GCA_002759435.2.fasta \
        --param.rate 1.0 \
        --param.threads 4 \
        --ec default:gridengine \
        --ep \
            default.slots:4 \
            'default.init:echo `hostname` && mkdir -p $HOME/tmp && export TMPDIR=$HOME/tmp && export _JAVA_OPTIONS=-Djava.io.tmpdir=$HOME/tmp && export XDG_RUNTIME_DIR=' 
```

Arguments are explained below:

1. ``-o ./output``: The workflow's output will be placed in the ``./output`` folder. This folder will be created if it doesn't already exist. 
2. ``-n test-mycosnp-bwa-pre-process``: This is the name of the workflow job. A sub-folder with the name ``test-mycosnp-bwa-pre-process`` will be created in ``./output`` for the workflow output. 
3. ``--in.input_folder``: This must point to one or more folders, separated by spaces, that contain a sub-folder for each sample. Each sub-folder must contain FASTQ paired-end reads named according to standard Illumina file naming convention. Samples may be split into multiple lanes. 
4. ``--in.reference_index ./output/test-mycosnp-bwa-reference-2e2116de/bwa_index/bwa_index``: This must point to the output folder of a MycoSNP BWA Reference workflow run that contains the BWA index. 
5. ``--in.reference_sequence``: This is the reference FASTA file, which is needed to calculate the length of the reference in order to estimate coverage. 
6. ``--param.rate`` and ``--param.coverage``: If ``param.rate`` is specified, then ``param.coverage`` is ignored. ``param.rate`` specifies the rate for downsampling FASTQ files. A rate of 1.0 indicates that 100% of reads in the FASTQ files are retained, which effectively "skips" downsampling. If ``param.coverage`` is specified and ``param.rate`` is not specified, ``param.coverage`` is used to calculate a downsampling rate that results in the specified coverage. For example if ``param.coverage 70``, then FASTQ files are downsampled such that, when aligned to the reference, the result is approximately 70x coverage. 
7. ``--param.threads``: Number of CPU threads used for sequence alignment. This should ideally match the ``default.slots`` parameter.
8. ``--ec default:gridengine``: This is the workflow "execution context", which specifies where the workflow will be executed. "gridengine" is recommended, as this will execute the workflow on the HPC. However, "local" may also be used. 
9. ``--ep``: This specifies one or more workflow "execution parameters".
   a. ``default.slots:4``: This specifies the number of CPUs or "slots" to request from the gridengine HPC when executing the workflow.
   b. ``'default.init:echo `hostname` && mkdir -p $HOME/tmp && export TMPDIR=$HOME/tmp && export _JAVA_OPTIONS=-Djava.io.tmpdir=$HOME/tmp && export XDG_RUNTIME_DIR='``: This specifies a number of commands to execute on each HPC node to prepare that node for execution. This commands ensure that a local "tmp" directory is used (rather than /tmp), and also resets an environment variable that may interfere with correct execution of singularity containers.

After successful execution, the output directory should contain two sub-directories: ``bam_index`` and ``qc_trim``. ``bam_index`` should contain a sub-directory for each sample, and this sub-directory should contain a BAM file and a BAM index. ``qc_trim`` should contain a sub-directory for each sample, and this sub-directory should contain trimmed FASTQ files and QC reports. Note: the output of the "No-QC" version of the workflow should not contain a ``qc_trim`` sub-directory. 

## Public Domain Standard Notice
This repository constitutes a work of the United States Government and is not
subject to domestic copyright protection under 17 USC § 105. This repository is in
the public domain within the United States, and copyright and related rights in
the work worldwide are waived through the [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).
All contributions to this repository will be released under the CC0 dedication. By
submitting a pull request you are agreeing to comply with this waiver of
copyright interest.

## License Standard Notice
The repository utilizes code licensed under the terms of the Apache Software
License and therefore is licensed under ASL v2 or later.

This source code in this repository is free: you can redistribute it and/or modify it under
the terms of the Apache Software License version 2, or (at your option) any
later version.

This source code in this repository is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the Apache Software License for more details.

You should have received a copy of the Apache Software License along with this
program. If not, see http://www.apache.org/licenses/LICENSE-2.0.html

The source code forked from other open source projects will inherit its license.

## Privacy Standard Notice
This repository contains only non-sensitive, publicly available data and
information. All material and community participation is covered by the
[Disclaimer](https://github.com/CDCgov/template/blob/master/DISCLAIMER.md)
and [Code of Conduct](https://github.com/CDCgov/template/blob/master/code-of-conduct.md).
For more information about CDC's privacy policy, please visit [http://www.cdc.gov/other/privacy.html](https://www.cdc.gov/other/privacy.html).

## Contributing Standard Notice
Anyone is encouraged to contribute to the repository by [forking](https://help.github.com/articles/fork-a-repo)
and submitting a pull request. (If you are new to GitHub, you might start with a
[basic tutorial](https://help.github.com/articles/set-up-git).) By contributing
to this project, you grant a world-wide, royalty-free, perpetual, irrevocable,
non-exclusive, transferable license to all users under the terms of the
[Apache Software License v2](http://www.apache.org/licenses/LICENSE-2.0.html) or
later.

All comments, messages, pull requests, and other submissions received through
CDC including this GitHub page may be subject to applicable federal law, including but not limited to the Federal Records Act, and may be archived. Learn more at [http://www.cdc.gov/other/privacy.html](http://www.cdc.gov/other/privacy.html).

## Records Management Standard Notice
This repository is not a source of government records, but is a copy to increase
collaboration and collaborative potential. All government records will be
published through the [CDC web site](http://www.cdc.gov).

## Additional Standard Notices
Please refer to [CDC's Template Repository](https://github.com/CDCgov/template)
for more information about [contributing to this repository](https://github.com/CDCgov/template/blob/master/CONTRIBUTING.md),
[public domain notices and disclaimers](https://github.com/CDCgov/template/blob/master/DISCLAIMER.md),
and [code of conduct](https://github.com/CDCgov/template/blob/master/code-of-conduct.md).
