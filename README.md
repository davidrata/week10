🎥 [Watch project walkthrough video](https://youtu.be/h9gUut5GQoE?si=EL9QQHcCR2fCsQRI)

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

- **First-Year Ice (FYI)**: Newly formed, smoother, and often covered with fresh snow, leading to higher surface reflectance. FYI typically has a more uniform albedo due to limited melt pond development and relatively flat surface morphology ([Polashenski et al., 2012](https://doi.org/10.1029/2011JC007231)).

- **Multi-Year Ice (MYI)**: Older, thicker, and more structurally deformed due to repeated freeze–thaw cycles. MYI generally exhibits lower albedo, appearing darker in optical imagery due to deeper melt ponds and increased surface roughness ([Landy et al., 2023](https://tc.copernicus.org/preprints/tc-2023-75/)).

These differences are detectable in satellite data and form the basis for remote sensing-based classification approaches.

Distinguishing between FYI and MYI is vital due to their differing roles in ocean-atmosphere feedback, melt dynamics, and long-term climate stability. Remote sensing offers a non-invasive means to monitor these forms, particularly through:

- **Optical reflectance (Sentinel-2 MSI Band 4)** — sensitive to surface brightness and snow cover
- **Radar waveform features (Sentinel-3 SRAL)** — including peakiness, SSD (stacked standard deviation), and backscatter

While previous studies have successfully applied GMM to detect sea ice and leads (e.g., [Huang et al., 2024](https://www.researchgate.net/publication/380542977)), few have addressed the more subtle spectral and geometric differences between FYI and MYI using an unsupervised approach.

This project builds on prior applications of unsupervised clustering to sea ice (e.g., [Bishop, 2006](https://www.springer.com/gp/book/9780387310732); [Chellappa et al., 2020](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2020GL091285)), and applies these models to a multi-sensor dataset collected over the **Beaufort Sea (72.48°N, -127.35°W)** on **1 May 2024**.

## 3. The SENTINEL-2 and SENTINEL-3 Satellites

### Figure 1 : Multi-Sensor Workflow for Arctic Sea Ice Type Classification

![multisensor_2](https://github.com/user-attachments/assets/61cb5301-9c8d-4f2d-9c0b-0d53fdef93c4)

This figure outlines the end-to-end workflow integrating **Sentinel-3 radar altimetry** and **Sentinel-2 optical imagery** for sea ice classification.

This project uses data from two Copernicus Sentinel missions, each contributing different sensing capabilities:

###  Sentinel-2 MSI (Multispectral Instrument)

- Provides high-resolution optical imagery in 13 spectral bands.
- Band 4 (Red) at 10m spatial resolution is used in this project.
- Useful for detecting surface reflectance differences between FYI and MYI, especially under clear-sky conditions.

###  Sentinel-3 SRAL (Synthetic Aperture Radar Altimeter)

- Provides radar altimetry data with 20 Hz Ku-band waveforms.
- Enables extraction of sea ice surface properties through:
  - **Peakiness** (waveform sharpness)
  - **Stacked Standard Deviation (SSD)** (waveform width)
  - **Backscatter coefficient (σ₀)** (surface reflectivity)

Together, these sensors offer complementary views: **Sentinel-2 captures brightness and texture**, while **Sentinel-3 captures surface structure** through radar returns.

---

## 4. Machine Learning Methodology: K-Means & GMM Clustering
###  Figure 2 : GMM Algorithm & Implementation on Sentinel-2 Reflectance Data

<div align="center">
  <img width="600" alt="figure_gmm" src="https://github.com/user-attachments/assets/527dd8bd-69d6-4a2f-a985-eb2736bdfdf1" />
</div>

This figure illustrates the application of **Gaussian Mixture Model (GMM) workflow**  to Sentinel-2 Band 4 reflectance.

This project implements two unsupervised clustering algorithms:

###  K-Means Clustering
- Partitions data into _k_ groups by minimizing intra-cluster variance.
- Assumes spherical, equally sized clusters.
- Efficient, but may struggle with overlapping or complex boundaries.

### Gaussian Mixture Model (GMM)
- A probabilistic clustering method using multiple Gaussian distributions.
- Provides **soft labels** — each point has a probability of belonging to each cluster.
- More flexible for real-world remote sensing data where FYI and MYI may not be clearly separable.

#### Workflow Summary:

1. **Sentinel-2 Band 4 reflectance data** is loaded and preprocessed. Non-zero valid pixels are selected and optionally downsampled to manage memory usage.

2. The cleaned 1D reflectance array is used to fit a **Gaussian Mixture Model (GMM)** with 2 components, representing a hypothesis of FYI vs MYI based on surface brightness differences.

3. The GMM cluster labels are projected back into 2D to generate a classified image. Visual interpretation of spatial patterns (e.g., textured vs. smoother areas) helps link the clusters to sea ice types.

4. Separately, **Sentinel-3 SRAL altimetry data** is read and processed to extract waveform-based features:
   - **Peakiness** (sharpness of waveform return)
   - **Stack Standard Deviation (SSD)** (spread of returned power)
   - **Backscatter coefficient (σ₀)**

5. These features are combined and standardized to form a clustering input matrix. A GMM with 2 components is applied to classify radar waveforms, under the FYI vs MYI hypothesis.

6. The resulting clusters are visualized in multiple ways:
   - Feature scatter plots (e.g., σ₀ vs peakiness)
   - Average waveform plots with standard deviation shading
   - Full waveform traces for each cluster to assess consistency and distinctiveness

7. **Waveform alignment** is performed for each cluster (using cross-correlation) to reduce noise and highlight structural waveform differences between the ice types.

8. Observations across both sensors are qualitatively compared to support the interpretation of FYI vs MYI classification.





## 5. Environmental Cost Assessment

This project was designed with computational efficiency and environmental awareness in mind. Here, we assess the carbon footprint of the work by evaluating:

1. The energy required to **process and cluster satellite data** using cloud resources.
2. The upstream energy cost of **acquiring the Sentinel-2 and Sentinel-3 datasets**, including satellite operations.

###  Local Computational Footprint

All data preprocessing, clustering, and visualisation were performed using **Google Colab’s standard CPU runtime**, without GPU acceleration. The notebook runtime involved:

- One CPU-only session
- Approximately 30–40 minutes of active compute time
- Lightweight processing due to downsampling and small clustering models

According to [Lacoste et al. (2019)](https://arxiv.org/abs/1906.02243), an average cloud CPU session for light machine learning work (no deep neural networks) consumes roughly **0.05–0.10 kWh per hour**. Based on this:

> **Estimated compute energy use: ~0.05 kWh**  
> **Equivalent CO₂ footprint: ~25–30 g CO₂e**

This is less than the emissions from charging a smartphone twice or watching ~30 minutes of HD video streaming.

###  Upstream Cost: Satellite Data Acquisition

Although Sentinel data is **freely available**, its acquisition requires substantial infrastructure, including:

- **Satellite launches**, using ESA’s Vega or Soyuz rockets
- **On-orbit satellite operations**, including ground stations and data downlinks

The **Copernicus Sentinel satellites** are designed to last 7–12 years, and their **operational footprint is amortised** across thousands of users. According to [ESA's Life Cycle Assessment of Copernicus](https://www.mdpi.com/2072-4292/12/24/4055), the total carbon footprint per satellite per year is estimated at **50–70 tons CO₂e**, which includes launch, ground stations, and data processing centers.

> When averaged over all users and data requests, the **per-user impact is negligible** — estimated to be less than **0.01 kg CO₂e per image access**.

###  Design Considerations for Low-Carbon Research

This project minimized environmental cost through:

- **Unsupervised learning**: no costly model training or inference
- **Single-band optical analysis**: no multi-spectral fusion or GPU-heavy workflows
- **Cloud-based execution** on Colab: shared infrastructure with relatively efficient hardware utilization
- **Publicly available data**: no new acquisitions or custom campaigns required

---

###  Summary

The environmental cost of this project is minimal by design. The use of shared infrastructure, open-access satellite data, and computationally efficient methods ensures that the total carbon footprint remains low. No custom data collection was required, and the analysis relied solely on short-duration CPU-based execution. As such, this project exemplifies how satellite remote sensing and machine learning can be combined in an environmentally responsible way for Earth system monitoring.

---
## 6. Getting Started

This project was created and executed using **Google Colab**, a cloud-based environment for running Python code.

To replicate or adapt this project, follow these steps:

1. Download the notebook file from this repository (`Unsupervised_Classification_of_First_Year_and_Multi_Year_Sea_Ice.ipynb`) or open it directly in Google Colab.
2. Download the required satellite datasets. See the section below for download instructions.
3. Upload the datasets to your Google Drive, or place them in your local filesystem if working offline.
4. Modify the file paths in the notebook (`band_path`, `waveform_file`, etc.) to point to the location of the data you downloaded.
5. If using a different dataset, be sure to update the filenames and timestamps used in the notebook accordingly.
6. Run the notebook cells sequentially. Additional comments and instructions are embedded in the notebook itself.

---

## 7. Datasets Used

The datasets used for this project are Level-1C Sentinel-2 MSI optical imagery and Level-2 Sentinel-3 SRAL radar altimetry data. Both were obtained from the **Copernicus Open Access Hub** and selected for their coverage of the **Beaufort Sea region**, which contains both First-Year Ice (FYI) and Multi-Year Ice (MYI) during the early melt season.

<img width="658" alt="1" src="https://github.com/user-attachments/assets/d8fd78fa-553e-46e8-b84a-bca5bfdc5f67" />

This map shows the area where the data was taken by Sentinel 2 And 3 Satellites

The specific datasets used are:

- Sentinel-2 MSI Band 4 (Red):  
  `S2B_MSIL1C_20240501T203839_N0510_R014_T09XWA_20240501T235028.SAFE`
  
- Sentinel-3 SRAL Level-2 Sea Ice Product:  
  `S3A_SR_2_LAN_SI_20240426T200940_20240426T202745_20240523T020002_1084_111_342______PS1_O_NT_005.SEN3`

These datasets provide complementary information: surface reflectance and brightness from Sentinel-2, and radar waveform-based structural characteristics from Sentinel-3.

---

## 8. Downloading Datasets

The datasets used in this project have not been uploaded to this repository due to their size. However, anyone interested in replicating the analysis can access the exact same files through the following Google Drive link:

**[Download Project Datasets (Google Drive)](https://drive.google.com/drive/folders/1M0AF5AOtN4xZplQnSBAaw1VbnUu3kCuK?usp=sharing)**

Alternatively, to retrieve the original data from ESA Copernicus services:

1. Visit the [Copernicus Open Access Hub](https://scihub.copernicus.eu/).
2. Sign in with a free Copernicus account.
3. Use the product IDs listed above to search and download the data directly.
4. Unzip the `.SAFE` and `.SEN3` folders into your working directory or upload them to Google Drive for use in Colab.

Make sure to adjust notebook file paths to reflect the location where you save your data.

## 9. License

This project is distributed under the MIT License. You are free to reuse, modify, and distribute the code and documentation, provided that proper credit is given.

See the `LICENSE` file for full terms.

---

## 10. Contact

For questions, feedback, or collaboration inquiries, please contact:

**Student ID**: (22107927)
**Email**: (zcfbbab@ucl.ac.uk)
**GitHub**: (https://github.com/davidrata)

---

## 11. Acknowledgments

This project was developed as part of the AI4EO module (GEOL0069) at University College London.

Special thanks to:

- Dr Michel Tsamados, Weibin Chen and Connor Nelson for guidance on Earth Observation machine learning
- The Copernicus Programme and European Space Agency (ESA), for making satellite data openly accessible
- Colab and scikit-learn teams, for enabling fast prototyping of clustering models

---

## 12. References

- Lacoste, A., Luccioni, A., Schmidt, V., & Dandres, T. (2019). Quantifying the carbon emissions of machine learning. *arXiv preprint* [arXiv:1906.02243](https://arxiv.org/abs/1906.02243)

- Belward, A., et al. (2020). Life cycle assessment of Earth observation satellite missions. *Remote Sensing, 12*(24), 4055.

- Zhang, Y., et al. (2024). Sea Ice Extraction via Remote Sensing Imagery. *Remote Sensing*, 16(5), 842. [https://www.mdpi.com/2072-4292/16/5/842](https://www.mdpi.com/2072-4292/16/5/842)

- Chellappa, R., et al. (2020). Bayesian Unsupervised Machine Learning Approach to Segment Arctic Sea Ice. *Geophysical Research Letters*, 47(21). [https://doi.org/10.1029/2020GL091285](https://doi.org/10.1029/2020GL091285)

- Huang, X., et al. (2024). Enhanced sea ice classification using GMM. *Remote Sensing Letters*. [DOI or link] (replace with correct one if known)

- Bishop, C. (2006). *Pattern Recognition and Machine Learning*. Springer.
- Polashenski, C., Perovich, D., & Courville, Z. (2012). Melt pond formation on Arctic sea ice: Dynamics and seasonal evolution. *Journal of Geophysical Research: Oceans*, 117(C1). [https://doi.org/10.1029/2011JC007231](https://doi.org/10.1029/2011JC007231)

- Landy, J. C., et al. (2023). Melt pond evolution and its impact on Arctic ice–albedo feedback. *The Cryosphere Discussions*. [https://tc.copernicus.org/preprints/tc-2023-75/](https://tc.copernicus.org/preprints/tc-2023-75/)





