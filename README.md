## 1. Problem to Be Tackled

Arctic sea ice plays a crucial role in Earth’s climate system, with **First-Year Ice (FYI)** and **Multi-Year Ice (MYI)** differing in structure, reflectivity, and vulnerability to climate change. Accurate mapping of these ice types is essential for climate modelling, shipping safety, and environmental monitoring.

This project explores whether **unsupervised machine learning** — specifically **K-Means** and **Gaussian Mixture Models (GMM)** — can distinguish FYI from MYI using satellite-derived features. The approach leverages two EO products:

- **Sentinel-2 MSI optical imagery**  
  `S2B_MSIL1C_20240501T203839_N0510_R014_T09XWA_20240501T235028.SAFE`
- **Sentinel-3 SRAL radar altimetry**  
  `S3A_SR_2_LAN_SI_20240426T200940_20240426T202745_20240523T020002_1084_111_342______PS1_O_NT_005.SEN3`

While supervised learning approaches require labelled datasets — often scarce or costly in polar regions — this project adopts a **label-free clustering approach** to investigate spectral and structural distinctions between FYI and MYI. Previous works using GMM for separating **sea ice from leads** (e.g., [Zhang et al., 2024](https://www.mdpi.com/2072-4292/16/5/842)) inform this adaptation to a finer-grained FYI/MYI distinction.

> This project contributes to scalable, low-resource Arctic ice monitoring by integrating multi-sensor EO data into a probabilistic unsupervised classification framework.

---

## 2. Background

Arctic sea ice exists in two main forms:

- **First-Year Ice (FYI)**: Newly formed, smoother, snow-covered, and highly reflective
- **Multi-Year Ice (MYI)**: Older, thicker, rougher, with lower albedo due to melt ponds or surface scattering

Distinguishing between FYI and MYI is vital due to their differing roles in ocean-atmosphere feedback, melt dynamics, and long-term climate stability. Remote sensing offers a non-invasive means to monitor these forms, particularly through:

- **Optical reflectance (Sentinel-2 MSI Band 4)** — sensitive to surface brightness and snow cover
- **Radar waveform features (Sentinel-3 SRAL)** — including peakiness, SSD (stacked standard deviation), and backscatter

While previous studies have successfully applied GMM to detect sea ice and leads (e.g., [Huang et al., 2024](https://www.researchgate.net/publication/380542977)), few have addressed the more subtle spectral and geometric differences between FYI and MYI using an unsupervised approach.

This project builds on prior applications of unsupervised clustering to sea ice (e.g., [Bishop, 2006](https://www.springer.com/gp/book/9780387310732); [Chellappa et al., 2020](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2020GL091285)), and applies these models to a multi-sensor dataset collected over the **Beaufort Sea (72.48°N, -127.35°W)** on **1 May 2024**, a time and location likely to contain both FYI and MYI.

## 3. The SENTINEL-2 and SENTINEL-3 Satellites

This project uses data from two Copernicus Sentinel missions, each contributing different sensing capabilities:

### 🛰 Sentinel-2 MSI (Multispectral Instrument)

- Provides high-resolution optical imagery in 13 spectral bands.
- Band 4 (Red) at 10m spatial resolution is used in this project.
- Useful for detecting surface reflectance differences between FYI and MYI, especially under clear-sky conditions.

### 🛰 Sentinel-3 SRAL (Synthetic Aperture Radar Altimeter)

- Provides radar altimetry data with 20 Hz Ku-band waveforms.
- Enables extraction of sea ice surface properties through:
  - **Peakiness** (waveform sharpness)
  - **Stacked Standard Deviation (SSD)** (waveform width)
  - **Backscatter coefficient (σ₀)** (surface reflectivity)

Together, these sensors offer complementary views: **Sentinel-2 captures brightness and texture**, while **Sentinel-3 captures surface structure** through radar returns.

---

## 4. Machine Learning Methodology: K-Means & GMM Clustering

This project implements two unsupervised clustering algorithms:

### 🔹 K-Means Clustering
- Partitions data into _k_ groups by minimizing intra-cluster variance.
- Assumes spherical, equally sized clusters.
- Efficient, but may struggle with overlapping or complex boundaries.

### 🔹 Gaussian Mixture Model (GMM)
- A probabilistic clustering method using multiple Gaussian distributions.
- Provides **soft labels** — each point has a probability of belonging to each cluster.
- More flexible for real-world remote sensing data where FYI and MYI may not be clearly separable.

#### ✳ Workflow Summary:
1. **Sentinel-2 Band 4 reflectance** is flattened and downsampled.
2. A **GMM with 2 components** is fitted to the valid reflectance values.
3. The resulting clusters are reshaped back into a 2D image and interpreted as FYI vs MYI based on spatial structure and brightness.

```{r echo=FALSE, out.width="100%", fig.align='center'}
knitr::include_graphics("figure_gmm.png")



