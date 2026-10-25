# 🫀 ECG Signal Denoising Using Pole-Zero Placement (IIR Filter Design)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![MIT-BIH Database](https://img.shields.io/badge/Dataset-MIT--BIH%20Arrhythmia-red.svg)](https://physionet.org/content/mitdb/1.0.0/)
[![Course](https://img.shields.io/badge/HUST-Signals%20%26%20Systems-red)](https://hust.edu.vn/)

A comprehensive project for the **Signals and Systems** course at Hanoi University of Science and Technology (HUST). This repository implements a discrete-time Linear Time-Invariant (LTI) cascaded IIR filter system designed via geometric **Pole-Zero Placement on the Z-plane** to remove biological and environmental artifacts from Electrocardiogram (ECG) recordings.

---

## 📌 Project Overview & Objectives

Clinical ECG signals typically possess low amplitude ($< 2 \text{ mV}$) with key spectral energy residing between **0.5 Hz and 40 Hz**. Raw acquisitions are heavily distorted by:
1. **Powerline Interference (50 Hz):** Electromagnetic induction from the AC power grid, obscuring P and T waves.
2. **Baseline Wander ($< 0.5 \text{ Hz}$):** Low-frequency respiration and electrode-motion artifacts causing undulating isoelectric baselines.

This project designs and evaluates a cascaded IIR filtering pipeline ($H(z) = H_{\text{notch}}(z) \cdot H_{\text{hp}}(z)$) to restore the morphological integrity of the P-QRS-T complexes.

---

## 📐 Mathematical Formulation & Z-Domain Design

All filters are designed using a sampling frequency of **$F_s = 360 \text{ Hz}$** from the MIT-BIH Arrhythmia Database.

### 1. Notch Filter (50 Hz Suppression)
- **Target Digital Frequency:** $\omega_0 = 2\pi \frac{50}{360} = \frac{5\pi}{18} \approx 0.8726 \text{ rad/sample}$
- **Zeros ($z_{1,2}$):** Placed directly on the unit circle at $z_{1,2} = e^{\pm j\omega_0}$ to achieve infinite attenuation at 50 Hz.
- **Poles ($p_{1,2}$):** Placed inside the unit circle at $p_{1,2} = r e^{\pm j\omega_0}$ with radius $r = 0.95$ for a sharp transition bandwidth.
- **Transfer Function:**
  $$H_{\text{notch}}(z) = \frac{1 - 2\cos(\omega_0)z^{-1} + z^{-2}}{1 - 2r\cos(\omega_0)z^{-1} + r^2 z^{-2}} = \frac{1 - 1.2856z^{-1} + z^{-2}}{1 - 1.2213z^{-1} + 0.9025z^{-2}}$$

### 2. High-Pass Filter ($< 0.5 \text{ Hz}$ Baseline Wander Removal)
- **Zero ($z_3$):** Placed at $z = 1$ ($\omega = 0$, DC) to eliminate zero-frequency baseline drift.
- **Pole ($p_3$):** Placed on the positive real axis at $p_3 = \alpha = 0.99$ inside the unit circle ($f_c \approx 0.57 \text{ Hz}$).
- **Transfer Function:**
  $$H_{\text{hp}}(z) = \frac{1 - z^{-1}}{1 - 0.99z^{-1}}$$

### 3. Cascaded System & BIBO Stability (Integration)
The overall system transfer function is obtained by polynomial multiplication:
$$H_{\text{total}}(z) = H_{\text{notch}}(z) \cdot H_{\text{hp}}(z) = \frac{1 - 2.2856z^{-1} + 2.2856z^{-2} - z^{-3}}{1 - 2.2113z^{-1} + 2.1116z^{-2} - 0.8935z^{-3}}$$

- **Region of Convergence (ROC):** For a causal LTI system, $\text{ROC} = \{z \in \mathbb{C} : \vert{}z\vert{} > \max(r, \alpha) = 0.99\}$.
- **BIBO Stability:** Because the largest pole magnitude is $0.99 < 1$, the ROC strictly encloses the unit circle ($\vert{}z\vert{} = 1$), ensuring **strict causal BIBO stability**.
- **Real-Time Linear Difference Equation:**
  $$y[n] = x[n] - 2.2856x[n-1] + 2.2856x[n-2] - x[n-3] + 2.2113y[n-1] - 2.1116y[n-2] + 0.8935y[n-3]$$

---

## 🛠️ Project Structure

```text
├── data/
│   └── ecg_mitbih_sample.csv       # Standardized MIT-BIH Record 100 sample (Fs = 360 Hz)
├── docs/
│   ├── Tong_Hop_Cong_Thuc_Tuan_1_ECG-v2.docx   # Detailed mathematical derivations
│   └── math.pdf                                 # Initial analytical notes
├── scripts/
│   ├── generate_ecg_mitbih.py      # Data extraction & noise simulation script
│   └── calculation.py              # Filter simulation, Pole-Zero and Bode plotting
├── figures/
│   └── notch_filter_results.png    # Time & frequency domain verification plots
├── .gitignore
└── README.md
