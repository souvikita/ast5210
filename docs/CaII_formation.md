# Formation of Ca II lines

In this exercise we will explore the formation of Ca II lines. First, we will work with different model atoms, and adjust the `atoms.input` file accordingly.

## Working with different atom files

We've seen before that atoms can be either `PASSIVE` or `ACTIVE`. `ACTIVE` means they will be treated in full non-LTE, while `PASSIVE` are only included in background opacity calculations if their transitions are covered by a wavelength used in the calculations (determined by the `ACTIVE` atoms). 

For completeness, we will include a few large atoms as `PASSIVE`. Create a new `atoms.input` file with the following atom files:

* MgII-IRIS.atom
* CaII_PRD.atom
* Si.atom
* Al.atom
* Fe.atom
* He.atom
* N.atom
* Na.atom
* S.atom

These are available in the `../../Atoms` directory, so adjust the paths. Use `LTE_POPULATIONS` for all but the H, Ca II and Mg II atoms (use `ZERO_RADIATION` for these). The population file column can be left empty, but adjust the `Nmetal` value at the start of `atoms.input`. Set only `CaII_PRD.atom` as `ACTIVE` and run RH.

## Selecting wavelengths for detailed output

Look at the output from the previous run. Chose a line of Ca II to work with (e.g. Ca II H or Ca II 854.2 nm). To examine the output in detail not all wavelengths are written to disk (to save space), so we must decide which ones to include. You will need to create a file `ray.input` that contains the indices of wavelengths to be saved. 

The indices of wavelengths are dependent on the model atoms and other input options, so it is handy to examine the previous run so you can identify the indices. For example, after loading the output into `data`, you can find the indices of 392.8 $< \lambda <$ 394.0 by doing:


```python
data = rh15d.Rh15dout()
wave = data.ray.wavelength
indices = np.arange(len(wave))[(wave > 392.8) & (wave < 394.0)]
```

We want to save also the wavelength of 500 nm. To make sure it is actually calculated, you can do:


```python
wave.sel(wavelength=500, method='nearest')
```

and get its index with:


```python
index500 = np.argmin(np.abs(wave.data - 500))
```

To save this into a file `ray.input` we do:


```python
f = open('ray.input', 'w')  # this will overwrite any existing file!
f.write('1.00\n')
output = str(len(indices) + 1)
for ind in indices:
    output += ' %i' % ind
output += ' %i\n' % index500 
f.write(output)
f.close()
```

And now we run RH again! If you examine the `output_ray.hdf5` with `ncdump -h` or `h5dump -H` you'll see that it contains several other arrays, such as `chi` and `source function`. We will work with these next.

## Calculating optical depths

For the detailed output wavelengths, RH saves the monochromatic linear extinction coefficient (per length unit, variable `chi`) but not the optical depth. We can obtain the optical depths by integrating `chi` over height. *After* you have run RH with detailed output and loaded the arrays, you can do:


```python
from scipy.integrate.quadrature import cumtrapz

height = data.atmos.height_scale[0, 0].dropna('height')  # first column
index500 = np.argmin(np.abs(data.ray.wavelength_selected - 500))  # index of 500 nm
tau500 = cumtrapz(data.ray.chi[0, 0, :, index500].dropna('height'), x=-height)
tau500 = np.concatenate([[1e-20], tau500])  # ensure tau500 and height have same size
```

!!! info
    In the above we make use of `.dropna('height')`, a method in `xarray`. This is only needed when you ran RH with the option `15D_DEPTH_ZCUT = TRUE`. When `TRUE`, RH 1.5D will not include the top of the atmosphere, defined as where the temperature first reaches a temperature equal to `15D_TMAX_CUT`, another option in `keyword.input`. The reasoning for this is to save computer time by removing depth points from the calculation, and prevent any numerical instabilities from large gradients when calculating lines formed in the photosphere or chromosphere. When `15D_DEPTH_ZCUT = TRUE`, the output files will still have arrays that have all the depth points of the input atmosphere. RH 1.5D will write the missing points with a special fill value (typically 9.96921e+36), which `xarray` will interpret as `NaN`. Using `.dropna('height')` will cause `xarray` to exclude points with `NaN` from the calculations.



Now you can plot $\tau_{500}$ vs height:


```python
fig, ax = plt.subplots()
ax.plot(height / 1e6, tau500)  # height in Mm
```

Now you can answer the following:

* At what height does $\tau$ reach unity at 500 nm? What about in the core of your Ca II line?
* Plot the departure coefficients for the ground Ca II level on a height scale and on a $\tau_{500}$ scale
* Plot the $\tau$=1 height as function of wavelength for the Ca II line

## Source function widget

Now that your output files have the detailed output, you can use the `SourceFunction` widget from `rh15d_vis`:


```python
rh15d_vis.SourceFunction(data);
```

And you can answer the following:

* At what height does the source function depart from the Planck function for the wings of the Ca II line? And at the line core?
