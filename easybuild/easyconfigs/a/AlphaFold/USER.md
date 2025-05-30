# AlphaFold container user instructions

**BETA VERSION, problems may occur and may not be solved quickly, 
and the documentation needs further development.**

The AlphaFold container is developed by AMD specifically for LUMI and contains the
necessary parts to run AlphaFold on LUMI, including a suitable version of ROCm for the version of AlphaFold.
The container includes the dependencies of AlphaFold, including TensorFlow,
JAX, aria2 and OpenMM with HIP support.

The EasyBuild installation with the EasyConfigs mentioned below will do three things:

1.  It will copy the container to your own file space. We realise containers can be
    big, but it ensures that you have complete control over when a container is
    removed again.
    
    We will remove a container from the system when it is not sufficiently functional
    anymore, but the container may still work for you.

    If you prefer to use the centrally provided container, you can remove your copy 
    after loading of the module with `rm $SIF` followed by reloading the module. This
    is however at your own risk. 

2.  It will create a module file. 
    When loading the module, a number of environment variables will
    be set to help you use the module and to make it easy to swap the module with a
    different version in your job scripts.
    
    -   `SIF` and `SIFALPHAFOLD` both contain the name and full path of the singularity
        container file.
        
    -   `SINGULARITY_BIND` will mount all necessary directories from the system,
        including everything that is needed to access the project, scratch and flash
        file systems.

    -   `RUNSCRIPTS` and `RUNSCRIPTSALPHAFOLD` contain the full path of the directory
        containing some sample run scripts that can be used to run software in the 
        container, or as inspiration for your own variants.
        
3.  It creates currently 1 script in the $RUNSCRIPTS directory:

    -   `conda-python-simple`: This initialises Python in the container and then calls Python
        with the arguments of `conda-python-simple`. It can be used, e.g., to run commands
        through Python that utilise a single task but all GPUs.
        
The container uses a miniconda environment in which Python and its packages are installed.
That environment needs to be activated in the container when running, which can be done
with the command that is available in the container as the environment variable
`WITH_CONDA` (which for this container is
`source /opt/miniconda3/bin/activate alphafold`).

The container (when used with `SINGULARITY_BIND` of the module) also provides
the wrapper script `/runscripts/conda-python-simple` to start the Python command from the
conda environment in the container. That script is also available outside the 
container for inspection after loading the module as
`$RUNSCRIPTS/conda-python-simple` and you can use that script as a source
of inspiration to develop a script that more directly executes your commands or
does additional initialisations.

Example (in an interactive session):

```
salloc -N1 -pstandard-g -t 30:00
module load LUMI AlphaFold/69afc4d-rocm-5.5.1-python-3.9-singularity-20231108
srun -N1 -n1 --gpus 8 singularity exec $SIF /runscripts/python-conda-simple \
    -c 'import numpy'
```

The LUMI User Support Team cannot know all programs that users might use and none
of the team members is an expert in AlphaFold. This container is 
originally developed for a specific project and provided as-is until a user 
contributes to its documentation.

After loading the module, the docker definition file used when building the container
is available in the `$EBROOTALPHAFOLD/share/docker-defs` subdirectory. As it requires some
licensed components from LUMI and some other files that are not included, it currently
cannot be used to reconstruct the container and extend its definition.


## Installation

To install the container with EasyBuild, follow the instructions in the
[EasyBuild section of the LUMI documentation, section "Software"](https://docs.lumi-supercomputer.eu/software/installing/easybuild/),
and use the dummy partition `container`, e.g.:

```
module load LUMI partition/container EasyBuild-user
eb AlphaFold-69afc4d-rocm-5.5.1-python-3.9-singularity-20231108.eb
```

To use the container after installation, the `EasyBuild-user` module is not needed nor
is the `container` partition. The module will be available in all versions of the LUMI stack
and in [the `CrayEnv` stack](https://docs.lumi-supercomputer.eu/runjobs/lumi_env/softwarestacks/#crayenv)
(provided the environment variable `EBU_USER_PREFIX` points to the right location).
