# LTE and non-LTE

## Populations and intensities

In your run directory, change the `atoms.input` file so that the Ca is `PASSIVE` and the H atom is `ACTIVE`. In `keyword.input` set `15D_WRITE_POPS=TRUE` so that the level populations are written. Leave everything else the same, run `rh15d_ray` go through the following tasks:

* Examine the output. Look at intensities of the H$\alpha$ line
* Look at the level populations, and departure coefficients. Which level has the strongest departures?

Now rename the `output_ray.hdf5` file, e.g.:

    $ mv output/output_ray.hdf5 output/output_ray_NLTE.hdf5
    
Run RH again, but this time in LTE with `rh15d_lteray`. This is a special binary that only runs the problem in LTE. While normal `rh15d_ray` can write out both LTE and non-LTE populations, to obtain the LTE *intensities* you must run `rh15d_lteray` (which only writes `output_ray.hdf5`, and no populations or any other output). 

Compare the H$\alpha$ intensities in LTE and NLTE. What are the differences?

Note that by default, `rh15d.Rh15dout()` will read the file named `output_ray.hdf5` only. You can call it twice to make two objects, but make sure the second object does not load the same files by switching off the automatic read:


```python
fig, ax = plt.subplots()
data1 = rh15d.Rh15dout()
data2 = rh15d.Rh15dout(autoread=False)
data2.read_ray('output_ray_NLTE.hdf5')
data1.ray.intensity.plot()
data2.ray.intensity.plot()
```

Or you can just read the files directly with xarray:


```python
import xarray as xr
data1 = xr.open_dataset("output_ray.hdf5")
data2 = xr.open_dataset("output_ray_NLTE.hdf5")
data1.intensity.plot()
data2.intensity.plot()
```

## Initial solution

In this part we will only do runs in NLTE. RH solves the radiative transfer equation in an iterative procedure, using the accelerated $\Lambda$ iteration to arrive at the final solution. How quickly it arrives at the final result depends on how good our initial estimate is. RH has different methods for this, and different atoms work better with different initial solutions.

Let's experiment with the initial solution in a hydrogen atom. In `atoms.input`, you'll see that hydrogen has an initial solution of `ZERO_RADIATION`. Run `rh15d_ray` again and check how many iterations were necessary to achieve convergence. Plot the array `data.mpi.delta_max_history` (available when you load the output with `rh15d.Rh15dout()`), which shows the convergence history:


```python
fig, ax = plt.subplots()
data.mpi.delta_max_history.plot()
```

Now change the H initial solution to `LTE_POPULATIONS`. How many iterations did it take? Compare the convergence plots with both initial solutions. Can you see the accelerated iterations? What is their period? Try running with no acceleration to see the result (set `NG_ORDER=0` in `keyword.input`).

!!! info
    When you run RH 1.5D multiple times, the output files will be **overwritten**. If you want to save the results, it is recommended you copy the `output*hdf5` files to a different directory. With the `helita` interface, you can load RH output from different directories by passing the directory name as argument to the `Rh15dout` call, e.g.: `data1 = rh15d.Rh15dout()` reads from current directory, while `data2 = rh15d.Rh15dout('mydir/')` reads the output from `mydir/`.
