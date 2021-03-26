---
theme: solarized
title: "Numerical Reading Group: Pipeline Background"
author: Richard Shaw
institute: UBC
date: Mar 24th 2021
toc: false
slide-level: 2
---

## Overview

- Pipeline Concepts
  - Containers
  - Tasks
  - Parameters
  - Job script/manager
- Parallelisation
- Cluster usage


## Key Pipeline Concepts

Pipeline infrastructure is designed for processing of *large* datasets, held
*in-memory* in a more traditional HPC style. This is where it differs from things
like Spark/Hadoop etc.

- Containers hold the data being processed
- Tasks are the parts that do operations to the data
- Parameters are the way the tasks are configured
- The job script specifies how all the pipeline tasks are connected together, their parameters, which containers they should write to disk, etc.


## Containers

- Containers hold data in a typed and structured way
  - Examples are: `Map`, `RingMap`, `SiderealStream`, `CorrData`, `SpectroscopicCatalog`...
  - Each is a conceptually distinct type of data
- Store both datasets (arrays) and attributes (metadata, i.e. `dict`)
- When running the containers are held entirely *in-memory*
- Containers are easily read/written to disk
- Lowest level containers are `caput.memh5.MemDiskGroup`, but to be most useful they should derive from `draco.core.containers.ContainerBase`
  - Provides an easy way to define a container, and easy methods to derive them from input containers

## Minimal Container Example

- An `_axes` attribute gives the set of axes which must be specified at initialisation.
- A `_dataset_spec` dict attribute describes the datasets that can exist.
  - Each key gives the dataset name
  - Each value is a `dict` describing the dataset structure. `"axes"` names the axis
  for each dimension of the array, `"dtype"` gives a `numpy` datatype for each
  element of the array
```python
class FooContainer(ContainerBase):
    _axes = ("foo",)

    _dataset_spec = {
        "fooset": {"axes": ["foo"], "dtype": np.float64}
        }
    }
```

## Container Inheritance

- Containers inherit both axes and datasets
```python
class FooContainer(ContainerBase):
    _axes = ("foo",)

    _dataset_spec = {
        "fooset": {"axes": ["foo"], "dtype": np.float64}
        }
    }

# Bar implicitly gets a foo axis will also have a `fooset` dataset
class BarContainer(FooContainer):
    _axes = ("bar",)

    _dataset_spec = {
        "barset": {"axes": ["foo", "bar"], "dtype": np.float64}
    }
```


## Actual Container Example

```python
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

## Container usage

```python
# Taken from the regridder code
sdata = containers.SiderealStream(axes_from=timestream_data, ra=self.samples)
sdata.redistribute("freq")
sdata.vis[:] = sts
sdata.weight[:] = ni
sdata.attrs["lsd"] = self.start
sdata.attrs["tag"] = "lsd_%i" % self.start
```

## Tasks

- Tasks are Python classes designed to operate on data
- Low level interface in `caput.pipeline.TaskBase` but I recommend starting with `draco.core.task.SingleTask`
- Tasks provide additional facilities:
  - Writing out their output
  - Logging (that is also MPI aware)


## Task methods

- Tasks have 3.5 key methods:
  - `setup(self, *args)` -- accepts (optional) input containers and is run *once* as the task is setup.
  - `process(self, *args)` -- runs every time there is input available to run on, may return a *single* output container ... OR ...
  - ... `process(self)` -- (i.e. no input) is run repeatedly until the `PipelineStopIteration` exception is raised. Tasks like this start the pipeline, reading files; generating random data etc.
  - `process_finish(self)` -- called when there is no more input for the `process` method. This allows any cleanup or final tasks to be done, e.g. averaging input data together


## Parameters

- Parameters define typed class attributes that are easily read from files
- All are subclasses of `caput.config.Property`.
- Used to configure tasks
- Examples: `config.Property(proptype=float)`, `config.utc_time()`, `config.float_in_range(0, 2 * np.pi)`


## Task example

```python
class ReceiverTemperature(task.SingleTask):
    """Add a basic receiver temperature term into the data.

    Attributes
    ----------
    recv_temp : float
        The receiver temperature in Kelvin.
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

## Pipeline Scripts/Manager

- Pipeline jobs can be run in two ways, via a YAML file, or can be constructed in pure Python
- For most uses a YAML pipeline script in strongly recommended
- Pipeline scripts can also embed information about how to run on a cluster: how many nodes? how much memory? which queue?
- Pipeline jobs are run with `caput-pipeline run jobscript.yaml`
- On *cedar* you can submit directly with with `caput-pipeline queue jobscript.yaml`

## Pipeline script example

```yaml
pipeline:
  tasks:
    - type: draco.core.io.LoadProductManager
      out: manager
      params:
        product_directory: "/scratch/jrs65/bt_empty/chime_4cyl_allfreq/"

    - type: draco.core.io.LoadBasicCont
      out: sstream
      params:
        files: "/project/rpp-krs/chime/chime_processed/daily/rev_00/*/sstream_col_lsd_*.h5"

    - type: ch_pipeline.analysis.flagging.DayMask
      in: sstream
      out: sstream_mask

    - type: draco.analysis.flagging.RFIMask
      in: sstream_mask
      out: sstream_rfi2
      params:
        stack_ind: 66

    - type: draco.analysis.delay.DelaySpectrumEstimator
      requires: manager
      in: sstream_rfi2
      params:
        nsamp: 40
        save: Yes
        output_file: "delayspectrum_{tag}.h5"
```

## Cluster config example

```yaml
cluster:
    name: delay_spec  # Jobname
    directory: /scratch/jrs65/reprocess_dspec_short/

    venv: /home/jrs65/chime_pipeline_py3/venv/

    ## Cluster configuration (ignored unless on a cluster)
    nodes: 16  # Number of nodes to run on
    time: 60  # Time to run for (in minutes)
    ompnum: 6  # Number of OpenMP threads
    pernode: 8  # Number of processes per node
    mem: 192000M  # Memory required per node
    system: 'cedar'  # Which system we are running on (only cedar works)
    account: rpp-krs  # Allocation to charge


pipeline:

  # Setup MPI aware log levels
  tasks:
    - type: draco.core.task.SetMPILogging
      params:
        level_rank0: DEBUG
        level_all: INFO

  # continued ...
```


## Parallelisation

- Our data is **large**, e.g. 1 TB+ per day in memory, but nodes have 192 GB of memory
- Nodes have 48 cores, but most Python code can only use one at a time
- These are two distinct problems, but the solution is to parallelisation of various sorts:
- Two types of parallelisation in CHIME pipeline
  - OpenMP: thread based, parallelisation on a single machine
  - MPI: process based, parallelisation across machines


## OpenMP

- Allows a single process to use more of the compute resources on a single machine in parallel
- Uses threads, which are parallel streams of computation that *share* memory
- **NOTE**: threads are typically problematic in Python due to the Global Interpreter
  Lock (GIL), big issue you can read about elsewhere
- OpenMP parallelisation happens in roughly two places (which avoid the GIL issues)
  - Hidden in third party numerical routines: `BLAS` (linear algebra, used in `scipy.linalg`, `np.dot`), `libsharp` (Spherical Harmonic Transforms)
  - Explicitly in `Cython` modules written by us (e.g. `draco.util._fast_tools.pyx`)

## MPI

- Allows multiple processes to coordinate computation and split up large datasets
- This is network aware, processes can exchange data over the network
- We use `OpenMPI` and `mpi4py` for this
- MPI uses communicator objects to exchange information. They are essentially shared across all processes.
- Each process has a `rank` per communicator (essentially the index within a communicator).
- Each communicator has a shared `size`, i.e. the total number of processes that are members
- In almost all cases we use a single communicator `MPI_COMM_WORLD` which is *all* processes


## MPI aware containers

- MPI can be very complicated, but we've tried to make most things very easy
- Most problems we have a *trivially parallelisable*, i.e. if the data is split up in
  the right way the computation does not need to exchange information to run, e.g. a
  regridder needs all time samples, but does not need to see all frequencies or
  baselines
- Most containers can be distributed, i.e. they can be split across a specified axis
  and this can be changed dynamically with `.redistribute(axis_name)`
