# LUNG-RP-XAI

### Cross-Population Generalizability of Lung Cancer Radiomics and Dosiomics: Implications for AI Deployment in a South Asian Cohort

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Preprint on Zenodo](https://img.shields.io/badge/Preprint-Zenodo-blue.svg)](https://doi.org/10.5281/zenodo.XXXXXXX)
![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)

Code and reproducibility assets for a forensic audit of **radiomic and dosiomic domain shift** between a public Western lung cancer cohort and a multi-center Bangladeshi clinical cohort. The work was completed as a BSc thesis in Medical Physics at the Department of Medical Physics and Biomedical Engineering (MPBME), Gono Bishwabidyalay (Gono University), Savar, Dhaka, Bangladesh, and is available as a preprint on Zenodo.

**Preprint:** https://doi.org/10.5281/zenodo.XXXXXXX

---

## Overview

AI models for predicting radiation pneumonitis in lung cancer radiotherapy are almost always trained on standardized Western datasets. Before they can be used in Low- and Middle-Income Countries, a basic question must be answered: does the input data look the same? Differences in scanners, acquisition protocols, patient demographics and background lung pathology can create a *domain shift* that makes imported models unreliable.

This project does not train a toxicity-prediction model. It is an **audit of covariate shift**. It measures how far local South Asian radiomic and dosiomic feature distributions are from a public Western reference, and uses Explainable AI (SHAP) to identify which features drive the difference.

## Study Cohorts

| Cohort | Role | Patients | Acquisition / planning |
|---|---|---|---|
| NSCLC-Radiomics (TCIA, Maastro Clinic, Netherlands) | Source domain | 422 | Siemens Somatom, 3 mm slices |
| Square Hospital, Dhaka | Target domain | 121 | GE LightSpeed RT16, Varian Eclipse |
| Ahsania Mission Cancer & General Hospital, Dhaka | Target domain | 54 | Legacy/mixed CT sources, Elekta Monaco |
| **Total** | | **597** | |

The dosiometric audit covers both local centers (n = 175). The radiomic domain-shift analysis compares the Public cohort with Square Hospital, because coordinate-system inconsistencies in the Ahsania exports prevented reliable radiomic extraction for the shift analysis.

## Methods

1. **Forensic DICOM curation:** a header-based *Universal Organizer* sorts unstructured exports by DICOM modality; fuzzy matching harmonizes ROI names (GTV, PTV, lungs); a *Force-Match* algorithm aligns contours to CT slices by Z-coordinate (±3 mm tolerance) when Frame of Reference links are broken, recovering 50 of 54 Ahsania cases (92.5%). All data were de-identified.
2. **Preprocessing:** isotropic 1 × 1 × 1 mm resampling (B-spline), fixed 25 HU bin width, +1000 HU voxel shift.
3. **Feature extraction:** 107 IBSI-compliant radiomic features per patient (PyRadiomics) and dosiomic metrics (D95, Dmax, Dmean, V20, V5, Homogeneity Index, dose texture) from RT-DOSE (dicompyler-core).
4. **Statistics:** Shapiro-Wilk, Mann-Whitney U and Welch's t-test, FDR correction, fold change, Cohen's d, and PCA on z-scored features.
5. **XAI audit:** a Random Forest domain discriminator (Public vs. Local) explained with SHAP (global importance, beeswarm, patient-level waterfall plots).

## Key Results

- **The physics transfers.** Both local centers achieved consistent target coverage despite different planning systems: mean D95 of 60.5 ± 2.1 Gy (Square) and 59.8 ± 3.4 Gy (Ahsania), with no significant difference (p > 0.05).
- **The images do not.** A majority of the 107 radiomic features differed significantly between cohorts, and PCA showed clear separation (PC1 + PC2 = 39.2% of variance). Examples include Sphericity (fold change 1.81×), Maximum3DDiameter (0.45×) and LowGrayLevelEmphasis (0.23×, p < 0.001).
- **The cohort is identifiable from features alone.** The domain discriminator reached **AUC = 1.00** on the held-out test set. SHAP identified texture (GLDM, GLCM) and shape features as the main drivers.

> **Note on AUC = 1.00.** This is a *domain classifier*, not a toxicity model. It shows that the two cohorts are fully separable in radiomic feature space, which is evidence of strong domain shift, not of clinical predictive performance. Scanner vendor, slice thickness, tumor stage, age and contouring practice are all confounded with cohort, so the shift cannot be attributed to a single cause.

**Implication:** Western-trained radiomic models should not be imported directly into Bangladeshi workflows without local calibration, harmonization or retraining.

## Repository Structure

> Update this section to match the folders and notebook names in the repository.

```
LUNG-RP-XAI/
├── notebooks/
│   ├── 01_data_organization.ipynb
│   ├── 02_feature_extraction.ipynb
│   ├── 03_domain_shift_statistics.ipynb
│   └── 04_xai_shap_audit.ipynb
├── sample_data/          # Anonymized sample data for testing the pipeline
├── requirements.txt
├── LICENSE
└── README.md
```

## Installation and Usage

Requires Python 3.9 or later.

```bash
git clone https://github.com/iamSHShUvo/LUNG-RP-XAI.git
cd LUNG-RP-XAI
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Main libraries: `pyradiomics`, `SimpleITK`, `pydicom`, `rt-utils`, `dicompyler-core`, `scikit-learn`, `shap`, `scipy`, `pandas`.

Run the notebooks in numerical order (`01` to `04`), setting the data path in the first cell of each. To test the pipeline without the real data, use the files in `sample_data/`.

## Data Availability and Ethics

- **Public cohort:** NSCLC-Radiomics is available from [The Cancer Imaging Archive](https://www.cancerimagingarchive.net/) and is not redistributed here.
- **Local cohort:** Patient data from Square Hospital and Ahsania Mission Cancer & General Hospital cannot be shared because of privacy and institutional agreements. Only anonymized sample data is included.
- **Ethics:** The protocol was approved by the Institutional Review Board of Gono Bishwabidyalay and the ethics committees of the participating centers. Informed consent was waived for this retrospective study, with all data irreversibly de-identified before analysis.

## Limitations

- No outcome labels (survival, CTCAE toxicity) were available for the local cohort, so the study audits covariate shift only and cannot measure degradation of a trained model's performance.
- Radiomic shift analysis is limited to one local center (Square Hospital); at Ahsania, PTV was used as a surrogate where GTV was missing.
- Scanner, slice thickness, stage and contouring practice are confounded with cohort, and the local cohort (n = 175) is smaller than the public one (n = 422).


## Citation

```bibtex
@misc{shuvo2026lungrpxai,
  author    = {Shuvo, Md Shahadat Hossen},
  title     = {Cross-Population Generalizability of Lung Cancer Radiomics and Dosiomics: Implications for AI Deployment in a South Asian Cohort},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.XXXXXXX},
  url       = {https://doi.org/10.5281/zenodo.XXXXXXX}
}
```

## Supervision and Acknowledgements

- **Supervisor:** Hozzatul Islam Sayed, Lecturer, MPBME, Gono Bishwabidyalay
- **Joint supervisor:** Dr. Md Anwarul Islam, Coordinator Medical Physicist, Department of Radiation Oncology, Square Hospitals Ltd.
- **Clinical supervisor:** Dr. M. Jahangir Alam, Chief Medical Physicist and Head of the Radiotherapy Department, Ahsania Mission Cancer & General Hospital

Thanks to the radiotherapy teams at both hospitals for data access and technical support, and to The Cancer Imaging Archive for the NSCLC-Radiomics collection.

## License

Released under the [MIT License](LICENSE). Please credit the work by citing the preprint and this repository.

## Contact

**Md Shahadat Hossen Shuvo**
BSc in Medical Physics, Department of Medical Physics and Biomedical Engineering (MPBME), Gono Bishwabidyalay, Savar, Dhaka, Bangladesh

| | |
|---|---|
| **Email** | shuvo.mpbme@gmail.com |
| **Phone / WhatsApp** | +8801571361658 |
| **LinkedIn** | https://www.linkedin.com/in/s-h-shuvo |
| **GitHub** | [@iamSHShUvo](https://github.com/iamSHShUvo) |
| **ORCID** | [https://orcid.org/0000-0000-0000-0000](https://orcid.org/my-orcid?orcid=0009-0004-5557-077X) |
| **Google Scholar** |[ https://scholar.google.com/citations?user=XXXXXXXX](https://scholar.google.com/citations?view_op=list_works&hl=en&hl=en&user=xZ41z-IAAAAJ) |


**Research interests:** medical physics, radiotherapy, radiomics and dosiomics, healthcare AI, AI generalizability in global health, explainable AI.

For code questions, please open a [GitHub Issue](https://github.com/iamSHShUvo/LUNG-RP-XAI/issues). For collaboration or other enquiries, please email.
