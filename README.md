
# Comparing C+ and NH Maps

This project loads two FITS files containing C+ 158 micron moment 0 and N_H maps, aligns the NH map to the C+ map using WCS, and creates an overlay plot of the two maps.

## Steps

### 1. Import Required Libraries
```python
import numpy as np
import matplotlib.pyplot as plt
from astropy.io import fits
from astropy.wcs import WCS
from reproject import reproject_interp
```
Import necessary libraries for data handling, visualization, and reprojecting images.

### 2. Load the FITS Files
```python
cii_fits = fits.open('NGC7538_CII_moment0.fits')
nh_fits = fits.open('NGC7538_Tdust_tau160_NH_220px.fits')
```
Open the FITS files containing the C+ and NH data.

### 3. Set Global Plot Settings
```python
plt.rcParams['font.family'] = 'Times'
plt.rcParams['font.size'] = 25
```
Configure global settings for plots, such as font family and size.

### 4. Extract Data and WCS Information
```python
cii_data = cii_fits[0].data
cii_wcs = WCS(cii_fits[0].header)

nh_data = nh_fits[0].data
nh_wcs = WCS(nh_fits[0].header)

print("CII WCS info:", cii_wcs)
print("NH WCS info:", nh_wcs)
```
Extract the data and WCS information from the FITS files and print WCS details for debugging.

### 5. Handle Potential Dimension Mismatch
```python
if nh_data.ndim > 2:
    nh_data = nh_data[0]  # Take the first slice if it's a 3D array
    nh_wcs = nh_wcs.dropaxis(2)  # Drop the third axis from WCS

if cii_data.ndim > 2:
    cii_data = cii_data[0]  # Take the first slice if it's a 3D array
    cii_wcs = cii_wcs.dropaxis(2)  # Drop the third axis from WCS
```
Check and handle cases where the data arrays have more than two dimensions by taking the first slice and updating WCS accordingly.

### 6. Reproject NH Data to Match C+ Data
```python
nh_reprojected, footprint = reproject_interp((nh_data, nh_wcs), cii_wcs, shape_out=cii_data.shape)
```
Reproject the NH data to match the C+ data using WCS and the shape of the C+ data.

### 7. Create the Overlay Plot
```python
fig, ax = plt.subplots(figsize=(14, 10))

cii_plot = ax.imshow(cii_data, cmap='Greens', origin='lower', aspect='equal', vmin=0, vmax=400)
nh_plot = ax.imshow(nh_reprojected, cmap='Blues', origin='lower', alpha=0.5, aspect='equal', vmin=18, vmax=28)

cbar_nh = fig.colorbar(nh_plot, ax=ax, orientation='vertical', pad=0.00, aspect=28)
cbar_nh.set_label(r'N$_\mathrm{H}$ (cm$^{-2}$)')

cbar_cii = fig.colorbar(cii_plot, ax=ax, orientation='vertical', pad=0.00, aspect=28)
cbar_cii.set_label(r'C$^+$ (K km s$^{-1}$)')

ax.set_xlabel('RA (J2000)')
ax.set_ylabel('Dec (J2000)')
ax.set_title('Overlay of N$_H$ (Blue) on C$^+$ Emission (Green)')

plt.tight_layout()
plt.savefig("Cplus_NHmaps_overlay_corrected.png", dpi=300, bbox_inches='tight')
#plt.show()
```
Create a figure and plot the C+ map in green and the reprojected NH map in blue, with appropriate labels and title. Save the plot as a PNG file.

### 8. Close the FITS Files
```python
cii_fits.close()
nh_fits.close()
```
Close the FITS files to free up resources.


### Result

![Cplus_NHmaps_overlay_corrected](https://github.com/umitkavak/TwoColorMap/assets/26542534/ceb95a8a-55d9-455f-92d7-32591fcb2ceb)


The figure is from Kavak et al. to be submitted to ApJ.

