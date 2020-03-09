# Introduction and setup

This document is a guide to using the RH code during AST5210. We will be using the [RH 1.5D](https://github.com/ITA-Solar/rh) version of the code, which also has [online documentation](https://rh15d.readthedocs.io). You are expected to run the code yourself, either on your personal computer or using the Institute's Linux machines. The documentation describes how to [download, compile and install](https://rh15d.readthedocs.io/en/latest/installation.html) the code, but here are some quick instructions. 

RH 1.5D needs two external libraries: HDF5 (for the input/output) and MPI (for parallel processing). If you are using the Institute's Linux machines, they are already installed. If using your laptop, they are probably not installed. See instructions below for compiling in both cases.

## Installing in Linux machines at Institute

1. SSH into a local machine (e.g. beehive) and load the intel compilers and hdf5 module
2. Clone the RH repository
3. Edit `Makefile.config`, change `CC` to `mpiicc`, `F90C` to `ifort`, `LD` to `mpiicc`, and `HDF5_DIR` to `/astro/local/hdf5/1.10.1/intel`
4. Compile

Here's a summary of the commands to do the above:

```
$ ssh beehive
beehive$ module load Intel_parallel_studio/2018 hdf5/Intel/1.10.1
beehive$ git clone https://github.com/ita-solar/rh.git
beehive$ cd rh
beehive$ nano Makefile.config
beehive$ make -j8
beehive$ cd rh15d
beehive$ make -j8
```

## Installing in your personal computer

*This procedure may not work in Windows; it was only tested for Linux and MacOS*. 

RH needs C and Fortran compilers with the OpenMPI and HDF5 (parallel build) libraries. Manual installation is possible, but not recommended for those not experienced in building Unix packages. The easiest way to install all the necessary compilers and libraries is through the [miniconda](https://conda.io/miniconda.html) or [Anaconda](https://www.anaconda.com/download/) Python distributions. If you don't have conda installed, download and install it first (**get the latest python 3.x versions**). 

The first step is making sure you have C and Fortran compilers installed. Most Linux machines already have this installed (`gcc` and `gfortran`); if not you can install them via conda:

```
$ conda install -c conda-forge gfortran_linux-64 gcc
```

If you are on a Mac, you can also install the compilers with conda:

```
$ conda install -c conda-forge gfortran_osx-64 clang_osx-64
```

Once you have the compilers working, you need to the following:

1. Install MPI and hdf5 from Anaconda
2. Clone the RH repository
3. Edit `Makefile.config`, change `CC` to `mpicc`, `F90C` to `gfortran` (or other, if you are using another Fortran compiler), `LD` to `mpicc`, and `HDF5_DIR` to `$HOME/anaconda/` (edit if you have miniconda or Anaconda in a different location)
4. Compile! (Make sure mpicc is the conda version)

Here's a summary of the commands to do the above:

```
$ conda install -c conda-forge openmpi hdf5=1.10.4=mpi_openmpi_hd93f08e_1005
$ git clone https://github.com/ita-solar/rh.git
$ cd rh
$ nano Makefile.config
$ make -j8
$ cd rh15d
$ make -j8
```

If you don't have git, you can [download a zip file](https://github.com/ITA-Solar/rh/archive/master.zip) with RH instead.

## Installing Jupyter and ``helita`` for analysis of output

The [Jupyter notebook](https://jupyter.org/) (and associated packages) will be necessary to follow the exercise notebooks and visualise the output of RH 1.5D. If you are using the Linux machines, Jupyter and all necessary packages are already installed. If you don't have it installed in your personal computer, you can again use Anaconda:

```
$ conda install -c conda-forge jupyterlab ipywidgets ipympl widgetsnbextension xarray nodejs
```

You will also need [helita](https://github.com/ITA-Solar/helita/), a Python package that contains modules to load and visualise the output of RH 1.5D. To install it, you do:

```
$ git clone https://github.com/ITA-solar/helita.git
$ cd helita
$ python setup.py install --user
```

There is also a [zip file](https://github.com/ITA-Solar/rh/archive/master.zip) if you don't have git. If installing in your personal computer, you need C and Fortran compilers (see above).

For both the Institute machines and your own computer, you will need to activate the Jupyter widgets with:

```
$ jupyter labextension install @jupyter-widgets/jupyterlab-manager jupyter-matplotlib
```

After that, you are ready to start jupyter lab:

```
$ jupyter lab
```

To test your installation and try out the RH 1.5D widgets, you can run the sample notebooks inside the ``rh/doc/notebooks`` directory. In the terminal, got to that directory and enter ``jupyter lab``. 

You can also test the widget output of Jupyter by running the ``slab`` and ``transp`` radiative transfer visualisation widgets. These do not require any files from RH, and are based on the IDL `xslab.pro` and `xtransp.pro` routines. You can start them in Jupyter in the following way:


```python
%matplotlib widget
from helita.sim import rh15d_vis
```


```python
rh15d_vis.slab()
```


```python
rh15d_vis.transp()
```


### Running Jupyter lab remotely from the Linux machines

*This step is optional.*

It is possible to not only run Jupyter in your own machine, but also having it running on a remote computer at set to display the notebook in your own web browser. This way you can combine the best of both worlds: use a powerful computer with large memory and disk space, but keep a responsive graphical interface in your own computer. (You don't even need to install jupyter or all the programs in your computer.)

To to this using the Institute computers, you need to set up an ssh tunnel and configure your browser to connect to that address. Here's an example using the ``beehive`` machine:

```
$ ssh beehive
beehive$ module load python
beehive$ jupyter lab --no-browser
(...)
    http://localhost:8888/?token=5089680adec1040
```


You see above that when you start jupyter, you'll get a lot of output but most importantly a URL. **Save that URL.** It is unique for this session and you'll need it later. The above gets jupyter started, then you need to connect to it. For that, we will use an SSH tunnel. First, we need to be able to connect to ``beehive`` directly, so you need to change the ``~/.ssh/config`` file in your own computer. Add the following to the file (if it doesn't exist, create it):

```
Host beehive*
    ForwardX11 yes
    User username
    ProxyCommand ssh -XY username@login.astro.uio.no -W %h:%p
```

And replace ``username`` by your own username (two locations). Then you run the following command in your computer to start the SSH tunnel:

```
$ ssh -NL 9999:localhost:8888 beehive
```

This will map port 8888 on beehive to port 9999 on your computer. You then use the URL you got from jupyter, change 8888 into 9999, and enter it in your browser. Then you should see jupyter running on ``beehive``.


## Using Jupyter for these exercises

We will be using the Jupyter notebook to run Python code to work with RH. If you haven't used it, there are [several](https://www.datacamp.com/community/tutorials/tutorial-jupyter-notebook) [tutorials](https://medium.com/codingthesmartway-com-blog/getting-started-with-jupyter-notebook-for-python-4e7082bd5d46) you can read to familiarise yourself with it.

RH has a few Jupyter notebooks that illustrate some visualisation routines. You can try them out interactively with your own runs of RH, but for most of the work here is is recommended that you start your own notebooks and type code as you go.

In the first cell of *all your notebooks* it is recommended to keep all the necessary imports and other boilerplate code that you don't want to type often. The following cell should cover all your basic imports for these exercises:


```python
%matplotlib widget
import numpy as np
import matplotlib.pyplot as plt
from helita.sim import rh15d, rh15d_vis
```

In all the examples from now on, it is expected you have already ran the code above in your notebooks. This code uses the `ipympl` backend for `matplotlib`, which allows interactive figures inside notebooks but has some drawbacks. If you redo plots without clearing old figures or creating new figures, you may run into problems with disappearing plots  or plots updating in a new cell. The recommended approach for creating plots in a new cell is the following:


```python
fig, ax = plt.subplots()
ax.plot(...)  # or ax.imshow(...), or any other matplotib command
```

In the above you need to replace `"..."` with your plot command. In subsequent cells, you should name figures and axes differently, e.g.:


```python
fig1, ax1 = plt.subplots()
```

Otherwise, you will be updating the plots in the previous cell. If you get stuck with a disappearing figure, or have too many open figures (`matplotlib` will complain if you have more than 20), you can always close all of them:


```python
plt.close("all")
```
