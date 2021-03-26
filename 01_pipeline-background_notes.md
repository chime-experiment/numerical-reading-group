# Numerical Reading Group: Pipeline Background

author: Richard Shaw

## Overview

- [Pipeline Concepts](#key-pipeline-concepts)
    - [Containers](#containers)
    - [Tasks](#tasks)
    - [Parameters](#parameters)
    - [Job script/manager](#pipeline-scripts)
- [Parallelisation](#parallelisation)

## Key pipeline concepts

Pipeline infrastructure is designed for processing of **large** datasets, held
**in-memory** in a more traditional HPC style. This is where it differs from things
like Spark/Hadoop etc. We avoid writing to disk when possible.

## Containers

**Containers** hold specific sorts of data in a way that is typed or structured.
Each type of container is a conceptually distinct type of data.

For example, **CorrData** holds raw visibilities. We have a container for **Map**s and **RingMap**s as well.
**SiderealStream** holds visibilities transformed to a specific sidereal plane(?).
**SpectroscopicCatalog** has a catalogue of galaxies, transits, etc obtained from other sources.

Containers store both datasets (arrays), and attributes (metadata, i.e. `dict`).
When running containers are held entirely **in-memory**, though they are also easily read/written to disk.
Their format on disk is hdf5 (and their structure internally is similar to h5py's model for hdf5).

Lowest level containers are `caput.memh5.MemDiskGroup`, but to be most useful they should derive from `draco.core.containers.ContainerBase`.
These provide an easy way to define a container, and easy methods to derive them from input containers.

### Minimal Container Example

```
class FooContainer(ContainerBase):
    _axes = ("foo",)

    _dataset_spec = {
        "fooset": {
            "axes": ["foo"],
            "dtype": np.float64
        }
    }
```

The key attributes of a container are `_axes` and `_dataset_spec`.
An `_axes` attribute gives the set of axes which must be specified at installation.
All of the possible `axes` that the datasets may have should be defined there.

A `_dataset_spec` dict attribute describes the datasets that can exist.
Each key gives the dataset name. Each value is a `dict` describing the dataset structure.
`"axes"` names the axis for each dimension of that datset's array. An individual dataset does not need
to have all of the axes defined in `_axes`.
`"dtype"` gives a `numpy` datatype for each element of the array.

### Container Inheritance

Containers inherit both `_axes` and datasets.

```
# Bar implicitly gets a foo axis, and will also have a `fooset` dataset
class BarContainer(FooContainer):
    _axes = ("bar",)

    _dataset_spec = {
        "barset": {
            "axes": ["foo", "bar"],
            "dtype": np.float64
        }
    }
```

### Actual Container Example

```
class DelaySpectrum(ContainerBase):
    """Container for a delay spectrum."""

    _axes = ("baseline", "delay")

    _dataset_spec = {
        "spectrum": {
            "axes": ["baseline", "delay"],
            "dtype": np.float64,
            "initialise": True,
            "distributed": True,
            "distributed_axis": "baseline",
        }
    }

    @property
    def spectrum(self):
        return self.datasets["spectrum"]
```

We create properties for frequently accessed dataset or metadata indexing.

### Container usage

```
# Taken from the regridder code
sdata = containers.SiderealStream(axes_from=timestream_data, ra=self.samples)
sdata.redistribute("freq") # sets the distributed axis for the container
sdata.vis[:] = sts         # you can assign arrays directly to containers
sdata.weight[:] = ni
sdata.attrs["lsd"] = self.start
sdata.attrs["tag"] = "lsd_%i" % self.start
```

## Tasks

Containers get passed around to **Tasks**. Containers hold data, there is no computation within them.

**Tasks** are the Python classes designed to actually operate on the data.
A low level interface for them is in `caput.pipeline.TaskBase`, but it is recommended you start with `draco.core.task.SingleTask`.
Tasks provide additional facilities like:
    - Writing out their output
    - Logging (that is also MPI aware)

### Task methods

Tasks have 3.5 key methods.

- `setup(self, *args)`
    - accepts (optional) input containers and is run **once** as the task is setup.
    - useful for operations that only need to happen once.
- `process(self, *args)`
    - the meat
    - allows you to take inputs and runs every time there is input available to run on
    - may return a **single** output container
- `process(self)`
    - you can either have a `process(self, *args)` or a `process(self)`
    - (i.e. no input)
    - it is run repeatedly until the `PipelineSopIteration` exception is raised
    - Tasks like this start the pipeline, read files, generate random data, etc.
- `process_finish(self)`
    - called when there is no more input for the `process` method
    - this allows any cleanup or final tasks to be done
    - it can do things like calculate the final average after you finish collecting all the input data

## Parameters

**Parameters** define typed class attributes that are easily read from configration files.
They are used to configure Tasks.
All are subclasses of `caput.config.Property`.

Examples:

    - `config.Property(proptype=float)`
    - `config.utc_time()`
    - `config.float_in_range(0, 2 * np.pi)`

## Task example

```
class ReceiverTemperature(task.SingleTask):
    """Add a basic receiver temperature term into the data.

    Attributes
    ----------
    recv_temp : float
        The receiver temperature in kelvin.
    """
    recv_temp = config.Property(proptype=float, default=0.0)

    def process(self, data):

        # Iterate over the products to find the auto-correlations and add the noise into them
        for pi, prod in enumerate(data.prodstack):

                # Great an auto!
                if prod[0] == prod[1]:
                    data.vis[:, pi] += self.recv_temp

        return data
```

## Pipeline Scripts

Pipeline jobs can be run in two ways, via a YAML file, or they can be constructed in pure Python.
For most uses, a YAML pipeline script is strongly recommended. Pure Python is mostly for prototyping.

Pipeline scripts can also embed information about how to run on a cluster: how many nodes? how much memory? which queue?

Pipeline jobs are run with `caput-pipeline run jobscript.yaml`. (Addendum: you usually want to do that wihin an mpirun)

On **cedar**, `caput-pipeline queue jobscript.yaml` will submit a pipeline job to the batch queue.
It will automatically configure it to be run under MPI. (Editors notes: Do not run it within an mpirun.)

### Pipeline script example

```
pipeline:
    tasks:
        - type: draco.core.io.LoadProductManager # what the task should be; corresponds to a class in Python
        out: manager # name of output
        params: # configured parameters being passed to config.Property
            product_directory: "/scratch/jrs65/bt_empty/chime_4cyl_allfreq/"

        - type: draco.core.io.LoadBasicCont # this is an example of a process with no inputs
        out: sstream # it will output one sstream per file that it finds
        params: # it needs to know what to do from config alone
            files: "/project/rpp-krs/chime/chime_processed/daily/rev_00/*/sstream_col_lsd_*.h5"

        - type: ch_pipeline.analysis.flagging.DayMask
        in: sstream # same name as previous output, that data has gotten passed on
        out: sstream_mask

        - type: draco.analysis.flagging.RFIMask
        in: sstream_mask
        out: sstream_rfi2
        params:
            stack_ind: 66

        - type: draco.analysis.delay.DelaySpectrumEstimator
        requires: amanger # this task requires one input in its setup
        in: sstream_rfi2
        params:
            nsamp: 40
            save: Yes # Write to disk
            output_file: "delayspectrum_{tag}.h5" # name of file on disk
```

The pathway through the job should form a directed acyclic graph. If there is a cycle, it will likely go on forever, until it runs out of time.

### Cluster config example

```
cluster:
    name: delay_spec # Jobname
    directory: /scratch/jrs65/reprocess_dspec_short/ # where to place output files, and job config files

    venv: /home/jrs65/chime_pipeline_py3/venv/ # python venv to use

    ## Cluster configuration (ignored unless on a cluster)
    nodes: 16 # Number of nodes to run on
    time: 60 # Time to run for (in minutes); if time runs out, the job will end
    ompnum: 6 # Number of OpenMP threads
    pernode: 8 # number of processes per node
    mem: 192000M # Memory required per node
    system: 'cedar' # Which system we are running on (only cedar works)
    account: rpp-krs # Allocation to charge

pipeline

    # Setup MPI aware log levels
    tasks:
        - type: draco.core.task.SetMPILogging
        params:
            level_rank0: DEBUG # have only rank 0 output debug logs
            level_all: INFO # have them all output info logs
```

## Parallelisation

There are two problems we face:

- our data is **large**, e.g. 1 TB+ per day, but nodes have 192 GB of memory
- nodes have 48 cores, but most Python code can only use one at a time

These are two distinct problems, but the solution is to do parallelisation of various sorts.

There are two types of parallelisation in CHIME pipeline:

- OpenMP: thread-based, ensure more CPU usage on a single machine (parallelisation on a single machine)
- MPI: process-based, helps with spreading of data across multiple machines (parallelisation across machines)

### OpenMP

Allows a single process to use more of the computing resources on a single machine in parallel.
Uses threads to do that. Threads are parallel streams of computation that *share* memory.

**Note**: threads are typically problematic in Python due to the Global Interpreter Lock (GIL).
Essentially, only one thread can do something at a time; Python is only ever running one stream of computation.
This is to prevent all the bad things that can happen when multiple threads interact with the same bit of memory.
You can use threads, but not written in Python.

OpenMP parallelisation happens in roughly two places (which avoid the GIL issues):

The first: Hidden in third party numerical routines. These are usually written in C.
Examples are `BLAS` (linear algebra, used in `scipy.linalg`, `np.dot`), `libsharp` (Spherical Harmonic Transforms).
In NumPy, only `np.dot` is parallelised using OpenMP, so bias towards `scipy.linalg`, if that is important to you.

The second: explicitly in **Cython** modules written by us (e.g., `draco.util._fast_tools.pyx`).

### MPI

Splits computations between processes, which cannot share memory, but occupy different cores.
MPI allows multiple processes to coordinate computation, and split up large datasets.
This is network aware, processes can share data over the network.
We use `OpenMPI` and the python wrapper `mpi4py`. It is pretty good.

MPI uses communicator objects (in our code `comm`), to exchange information. They are essentially shared across all processes.
Each process has a `rank` per communicator (essentially an index within that communicator).
Each communicator has a shared `size`, i.e. the total number of processes that are members.
In practice, we use a single communicator `MPI_COMM_WORLD`, which has *all* of the processes.

### MPI aware containers

MPI can be very complicated, but most problems we have are *trivially parallelisable*.
If the data is split up in the right way, the computation does not need to exchange information to run.
e.g., a regridder needs all time samples, but does not need to see all frequencies or baselines.
Most containers can be distributed; they can be split across a specified axis.
Different computations do have different requirements, and so we do need to be able to transform how the data is distributed.
Within our code, the axis across which paralellisation occurs can be changed dynamically with `.redistribute(axis_name)`.
