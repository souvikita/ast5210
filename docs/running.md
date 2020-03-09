# Running RH 1.5D

## Auxiliary files

The RH 1.5D distribution that you obtained from Github does not contain all the files necessary for this course. Please download an [additional archive](rh_ast5210.tar.bz2) and unpack it in the RH main folder:

```
$ tar jxvf rh_ast5210.tar.bz2
```

This will place all files in their correct directories.

## Quickstart: running and looking at output

You should run the code in a `run` directory. When you get the source, there should be a directory under `rh/rh15d/run_example`. You can copy this directory to your own so you can make your changes:

```
$ cp -rp run_example run
$ cd run
```

Once inside `run`, there are already some input files and directories. You can do a test run of RH by doing:

    ../rh15d_ray
    
By default this will run with only one processor, and an example calculation with a Ca atom. You can read a lot more detail into the [input files](https://rh15d.readthedocs.io/en/latest/input.html) and [running options](https://rh15d.readthedocs.io/en/latest/running.html) in the documentation.

Inside your `run` directory go through the different tasks:

* Run the different binaries: `rh15d_ray`, `rh15d_ray_pool`, `rh15d_lteray`. 
* What happens if you run `rh15d_ray_pool` without calling `mpiexec` or `mpirun`?
* Explore the output files in the command line with `ncdump -h` or `h5dump -H`.

### Using Jupyter notebooks

You can have a look at the RH 1.5D sample notebooks under ``rh/doc/notebooks/``. For the work in these exercises, it is recommend it that you run jupyter from a directory of your chosing, ideally where you have output files from RH (e.g. the directory ``output/`` inside your run directory). The notebook source files (``*.ipynb``) will be saved there.

Once in the jupyter starting page, select "New" and then "Python 3". In the first cell, add the boilerplate code mentioned earlier:


```python
%matplotlib widget
import numpy as np
import matplotlib.pyplot as plt
from helita.sim import rh15d, rh15d_vis
```

And now you are ready to start playing with RH.

### Exploring input atmospheres

Your default `keyword.input` file uses the `FALC_82_5x5.hdf5` atmosphere file, which is a FAL C atmosphere converted to HDF5 format and replicated to 5x5 columns in 3D. All columns have the same information. Under the directory `rh/Atmos` you will also find a file called `bifrost_cb24bih_s385_cut.hdf5`, which is a cut from a 3D simulation from Bifrost. You can explore both with the `rh15d_vis.InputAtmosphere` procedure. You need to pass the filename of an atmosphere file as argument:


```python
rh15d_vis.InputAtmosphere('MY_ATMOS_DIR/Atmos/FALC_82_5x5.hdf5');
```

Replace ``MY_ATMOS_DIR`` with the directory where you have the RH atmospheres (typically in ``rh/Atmos``). You can give relative or absolute paths.

### Exploring the output

If you are in a directory with the output files of RH, you can simply enter the following to load the data:


```python
data = rh15d.Rh15dout()
```

If your output files are in a different directory, pass that directory as an argument to ``Rh15dout()``. Once you have the output loaded, a simple inspection of the intensity can be made with:


```python
fig, ax = plt.subplots()
data.ray.intensity.plot()
```

By default this will show all calculated wavelengths, but you can zoom in to a line of interest with matplotlib's interactive figure.
