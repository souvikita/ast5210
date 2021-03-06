# Formation of Mg II lines

In this exercise we will explore the formation of Mg II lines. You should have completed (or understood) the previous Ca II example to work on this one. The objective of this exercise is to synthesise the spectral region corresponding to the NUV window of the IRIS spectrograph, including the Mg II lines but also the many blended lines.

## Setup

In your run directory, we'll use almost the same setup as for the Ca II exercise. First, you need to modify `atoms.input` and set the `MgII-IRIS.atom` as `ACTIVE` (and all other atoms as `PASSIVE`). Now, because we changed the active atom, the wavelengths written to file will be different. This means that the wavelength indices we previously wrote in `ray.input` no longer match what we expected. For simplicity, in this exercise we can use a simple `ray.input` that writes no extra output. You can create a `ray.input` file with only the following:

```
1.00
0
```

The Mg II h & k lines are formed under partial redistribution (PRD). PRD is a complex topic and is not taught in this introductory course. But for this exercise you only need to activate it in RH. In your `keyword.input` file, find the option `PRD_N_MAX_ITER`. This should be set to zero. Now change it to `PRD_N_MAX_ITER = 3`, and you will activate PRD for atoms that require it. 

Now run `rh15d_ray` with that setup and look at the intensity in the region of 279 - 280.5 nm, where the Mg II lines are located. How many wavelength points does the intensity have?

## Wavelength files

The wavelengths that RH 1.5D uses depend on the *active* model atoms. In the atom file you can configure the number of   wavelength points that each transition has, as well as how they are distributed. For bound-bound transitions, the parameter `Nlambda` defines how many wavelength points they have. This is the most straightforward way to change the wavelength grid.

In addition to changing the atom files, there is another mechanism to add more wavelengths: the wavelength files. These files contain a list of additional wavelengths for RH to calculate. They are useful when you want to add lines from lists (see below) that are not covered by the active atoms, when you want to add more wavelengths to a specific line or region, or when you want better control over the wavelength grid.

### Using wavelength files

Only one wavelength file (or wavelength table) is allowed in RH. To use a wavelength file, you need to set the option `WAVETABLE` equal to the wavelength file. Edit your `keyword.input` file so that `WAVETABLE = ../../Atoms/wave_files/IRIS_NUV_full.wave`. This file contains a list of the IRIS NUV wavelengths, and if you've previously unpacked the archive `rh_ast5210.tar.bz2`, it should be in that directory.

Run `rh15d_ray` again with that new option. How many wavelength files were used now? You should also have noticed that the calculation took much longer now because it scales with the number of wavelengths.

### Reading and writing wavelength files

The wavelength files are written in [binary XDR format](https://rh15d.readthedocs.io/en/latest/input.html#line-lists-and-wavelength-files). In `helita` there are functions to read and write these files. 

!!! info
    The contents of the wavelength files should be in *vacuum* wavelengths, and in nm. RH uses vacuum wavelengths internally, although by default all the output for $\lambda$ > 200 nm will be given in *air* wavelengths.

To read an existing an existing wavelength file, use the function `helita.sim.rh15d.read_wave_file`:


```python
IRIS_wave = rh15d.read_wave_file('/Users/tiago/codes/rh/Atoms/wave_files/IRIS_NUV_full.wave')
```

Now check the content of that IRIS wavelength file. What is the wavelength separation?

To write a wavelength file, you use the function `helita.sim.rh15d.make_wave_file`:


```python
# this will write wavelenghts from 650 to 650 nm, 0.01 nm spacing
rh15d.make_wave_file('my.wave', 650, 660, 0.01)
# this will write an existing array "my_waves", if it exists
rh15d.make_wave_file('my.wave', new_wave=my_waves)
```

By default, `make_wave_file` will assume your wavelengths are in air and convert to vacuum wavelengths. If you don't want this, use the option `air=False`.

Create a new wavelength file with the same range of wavelengths as in the IRIS file, but with a spacing of 0.001 nm. We'll use this file in the exercises below.

When you run with a large wavelength file, RH can take a long time to finish. How to check if RH is indeed running properly? In the run directory there is a subdirectory called `scratch`. Inside, you can find a lot of temporary scratch files (`*.dat`), and also log files (`rh_p*.log`). Each process writes its own log file. Because you've been running `rh15d_ray` with only one process, there should only be one file called `rh_p0.log`. This file is updated as RH is running. If RH is running and you want to check if it's actually doing anything, you can follow that file with tail:

```
$ tail -f output/scratch/rh_p0.log
```

## Line lists

Now that you've calculated the spectrum, you'll see that it only contains the lines of Mg II (the active atom). Adding extra wavelengths from the file does not change that. We'd like to make this as realistic as possible and include as many lines as possible. The most correct way to do this is to include atoms of relevant elements with all the necessary transitions and calculate all spectra in NLTE. Unfortunately, this would require a very large number of atoms and at least hundreds of thousands of transitions, requiring many wavelength points and RH 1.5D would probably not finish before you are done with AST5210. A simpler and much faster option is to calculate those lines in LTE and use a line list. Again, LTE is probably not a good approximation for many of the strong spectral lines, but this a tradeoff that we'll have to live with.

A line list is a file with a list of bound-bound transitions and their parameters. RH supports line lists with the format of the [Kurucz line lists](http://kurucz.harvard.edu/linelists.html). In this format, each line in the file is a different spectral line. The various columns have different atomic data, starting with wavelength, log gf, and element code (see [Kurucz page](http://kurucz.harvard.edu/linelists.html) for more details).

To use a line list with RH, you will need to uncomment the setting `KURUCZ_DATA = kurucz.input` in `keyword.input`. This merely points to a text file (in this case we use `kurucz.input`) that has a list of line list files. Uncomment that line, and edit the file `kurucz.input`. You can start by using only a single entry: 

    ../../Atoms/Kurucz/gfMgIIhk_IRIS_full
    
If you've previously unpacked the archive [rh_ast5210.tar.bz2](https://folk.uio.no/tiago/teaching/ast5210/rh_ast5210.tar.bz2), the file above should be in your RH directory. It contains 3714 lines that are present in the wavelength region 278.1745 - 283.4151 nm.

Run `rh15d_ray` again with the line list, the wavelength file `IRIS_NUV_full.wave` and inspect the results. Run timings with the shell command `time` and see how much more time it takes to include the wavelength file, and both the line lists.

Included in your `rh/Atoms/Kurucz/` is another line list called `gfMgIIhk_IRIS_full.trim_85`. This file containts a subset of the `gfMgIIhk_IRIS_full`. It was made to approximate the full line list but only keeping the strongest lines (around the 85th percentile of line strength). Run with this new file and see if you can see a difference in the output intensity.

You may notice that the extra lines do not appear very well resolved (e.g. triangular shapes). This is because the file `IRIS_NUV_full.wave` only has enough wavelength points to cover the spatial resolution of IRIS, and was not meant for detailed analysis of individual lines. Now run RH again but with the wavelength file you created (separation 0.001 nm), and see if the lines are better resolved now. How many wavelengths were calculated?


## $\tau$=1 heights

In the exercises above we are not saving detailed output, so we don't have the total absorption (chi) that is necessary to calculate the optical depth. However, even without detailed output it is possible to save an additional quantity that helps us diagnose approximately where the line is formed: the height where the optical depth reaches unity. In `keyword.input` you need to set `15D_WRITE_TAU1 = TRUE` (it should not be enabled by default). Re-run RH and you'll find that the `output_ray.hdf5` has an additional output variable called `tau_one_height`. 

The variable `tau_one_height`, just like `intensity`, is a function of wavelength. Plot it as a function of wavelength. You can also plot the intensity separately. Which line is formed higher? What is the difference in $\tau$=1 height between the Mg II h & k lines with your model atmosphere? Aside from the Mg II lines, which other lines are formed higher?

## Putting it all together

Now we'll put the above concepts together by applying them to the Ca II H line (air wavelength 396.847 nm):

* Run `rh15d_ray` using the CaII_PRD.atom as active atom (Mg II can be passive)
* Create a new wavelength file to adequately sample all the blending lines on the wings of Ca II H. Choose an appropriate spacing.
* Use the file `rh/Atoms/Kurucz/gf0400.10` as a starting point to create a line list to cover the wavelength range of the Ca II H line. Just delete all the lines whose wavelengths are outside the range.
* Run RH with and without the new wavelengths/lines in the wings of Ca II H. Do you find any regions in the line wings that are not covered by blends?
