# Foreground Removal of 21 cm Cosmological Signal using Gaussian Process Regression (GPR)

This repository contains Python code and analysis for removing astrophysical foregrounds from synthetic 21 cm intensity-mapping data using **Gaussian Process Regression (GPR)**. The workflow follows the methodology used at the STARC Lab, IIT Indore.

## Project Summary

The redshifted **21 cm line** from neutral hydrogen is a key probe of the Epoch of Reionization (EoR). However, the cosmological signal is extremely faint and dominated by bright foregrounds such as Galactic synchrotron emission, free-free emission, extragalactic radio sources, and instrumental noise. This project uses **Gaussian Process Regression (GPR)** to model the smooth spectral foregrounds and subtract them from the data, recovering the underlying HI fluctuations.

## Data Description

The synthetic dataset used has shape:

`(128, 128, 142)`

Where:

- `128 × 128` → sky pixels (RA × DEC)
- `142` → frequency channels

The dataset includes:

- `FGnopol_HI_noise` → foreground + HI + noise
- `HI_noise` → HI + noise (reference)
- `noise` → noise-only cube
- `freqs` → frequency array

## Method Overview

### 1. Load Data

    data = pd.read_pickle("example_data.pkl")
    FGnopol_HI_noise_data = data.beam.FGnopol_HI_noise
    HI_noise_data = data.beam.HI_noise
    freqs = data.freqs

### 2. Visualize Sky Patches

The notebook plots:

- Foreground-contaminated sky maps
- HI + noise reference maps
- Frequency spectra for selected pixels

### 3. Define GPR Kernels

Foreground (smooth):

    kern_fg = GPy.kern.RBF(1)

HI signal (oscillatory):

    kern_21 = GPy.kern.Exponential(1)

Constraints are applied to variance and lengthscale for stability.

### 4. Prepare Data for GPR

Convert the cube into line-of-sight spectra:

    Input = obs.LoSpixels(FGnopol_HI_noise_data)

### 5. Fit GP Model

    kern = kern_fg + kern_21
    model = GPy.models.GPRegression(freqs[:, None], Input, kern)
    model.optimize_restarts(num_restarts=10)

### 6. Predict Foreground

    fg_fit, fg_cov = model.predict(freqs[:, None], full_cov=True, kern=kern_fg)

### 7. Subtract Foreground

    gpr_res = Input - fg_fit

Residuals are reshaped back to `(128, 128, 142)`.

### 8. Compare With Reference HI Data

The notebook computes:

- Pixel-wise differences
- Frequency-wise differences
- Noise variance
- Pure HI extraction using noise subtraction
- Diagnostic plots for all comparisons

## Outputs

The notebook generates:

- Foreground-removed data cube
- Foreground prediction cube
- Residual HI + noise maps
- Difference maps between:
  - Raw vs. cleaned data
  - Reference HI vs. reconstructed HI
  - Noise-subtracted HI vs. GPR-derived HI
- Frequency-domain diagnostics
- Noise variance vs. frequency plots

## Dependencies

- `numpy`
- `pandas`
- `matplotlib`
- `GPy`
- `gpr4im`

## How to Use

1. Place `example_data.pkl` in the correct directory.
2. Install dependencies.
3. Run the notebook cell-by-cell.
4. Ensure `gpr4im` is installed and importable.
5. Visualize outputs and compare with reference HI data.

## Notes

- `gpr4im` must be installed for the notebook to run.
- GPR is computationally expensive for large datasets.
- Kernel choice strongly affects foreground modeling.
- Zero-noise `GPRegression` is used unless heteroscedastic noise is enabled.

## Acknowledgement

This work was carried out at the Space Technology and Radio Cosmology (STARC) Lab, IIT Indore, under the supervision of Prof. Abhirup Datta.
