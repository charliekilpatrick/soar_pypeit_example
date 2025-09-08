# SOAR Pypeit Example

An example reduction for reducing SOAR/Goodman data with pypeit

# References

pypeit documentation: https://pypeit.readthedocs.io/en/stable/
anaconda: 

# Setup and Installation

Install pypeit via a conda installation

```
conda create -n pypeit python=3.11
conda activate pypeit
pip install pypeit
```

Ancillary atmospheric grid files can be found under: https://drive.google.com/drive/folders/1FFRWjUZ58HiDuDD33MYqBzMWDQanBRRy?usp=drive_link

These need to be installed under: `$CONDA_PREFIX/lib/python3.11/site-packages/pypeit/data/telluric/atm_grids`.  The files themselves are large (up to several GB), which is why they're not packaged with `pypeit`.

# Create the initial *.pyepit file and run pypeit

All data needed for this example are found in the `Test_Data.zip` file.  Unzip this file and enter the directory:

```
unzip Test_Data.zip
cd Test_Data
```

From here, you can start the reduction procedure with: 

```
pypeit_setup -s soar_goodman_red -c all
```

which should create the file `soar_goodman_red_A/soar_goodman_red_A.pypeit`.  This file controls all of the settings for our main reduction and contains a table with all of the input files.  Inspect the file and then proceed to run the pypeit reduction with:

```
run_pypeit soar_goodman_red_A/soar_goodman_red_A.pypeit
```

# Flux calibrate, coadd, and telluric correction

The most relevant files are located under `Science`, including `spec1d*.fits` files that contain extracted 1D spectra, `spec1d*.txt` files that contain tables describing the location, signal-to-noise, FWHM, and wavelength precision of each extracted spectrum, and `spec*2d.fits` files containing the calibrated 2D spectra and extraction residuals.

You can inspect these files with, e.g.,

```
pypeit_show_1dspec Science/spec1d_0206_SN-2024abbv_07-08-2025-SN2024abbv_red_20250807T061716.460.fits --obj SPAT0519-SLIT0497-DET01
```

This should show the correct trace for the supernova spectrum.  The trace is at the same spatial position (`SPATXXXX`) in the other file.

To start the calibration procedure run:

```
pypeit_flux_setup Science
```

which will automatically generate a `*.flux` and `*.coadd` file that we need to edit manually.  Edit your `soar_goodman_red.flux` file so the data block looks like:

```
                                                                  filename | sensfile
spec1d_0205_SN-2024abbv_07-08-2025-SN2024abbv_red_20250807T055705.668.fits |         ../HR7596_soar_ghts_red_400m2_20250807_sensfunc.fits
spec1d_0206_SN-2024abbv_07-08-2025-SN2024abbv_red_20250807T061716.460.fits |         ../HR7596_soar_ghts_red_400m2_20250807_sensfunc.fits
flux end
```

then run `pypeit_flux_calib soar_goodman_red.flux`.  This should flux calibrate your spectra and add the appropriate columns to the `spec1d*.fits` files.

Next edit the `soar_goodman_red.coadd1d` data block so it looks like:

```
                                                                  filename |                  obj_id
spec1d_0205_SN-2024abbv_07-08-2025-SN2024abbv_red_20250807T055705.668.fits | SPAT0519-SLIT0497-DET01
spec1d_0206_SN-2024abbv_07-08-2025-SN2024abbv_red_20250807T061716.460.fits | SPAT0519-SLIT0497-DET01
coadd1d end
```

You will also need to edit: `coaddfile = SN2024abbv_red_20250807.fits` to the output file name.  Finally, run `pypeit_coadd_1dspec soar_goodman_red.coadd1d`.

The final step is the telluric correction, which will fit the atmospheric model grid file `TellPCA_3000_26000_R15000.fits` to the molecular oxygen bands in your spectrum (if they cover the A and B bands at 7620 and 6880 angstroms).  Simply run:

```
pypeit_tellfit SN2024abbv_red_20250807.fits --objmodel poly
```

and pypeit will output a `SN2024abbv_red_20250807_tellcorr.fits` file, which is the final reduced 1D spectrum file.  You can inspect this file with: `lt_xspec SN2024abbv_red_20250807_tellcorr.fits`.
