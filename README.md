# Adversarial Attacks on Steganalyzers: A Comparative Study of GAN and Diffusion Models with Quantum Enhancement

## 📌 Project Overview
This repository contains the source code, reports, and presentation slides for the Final Project of the CSC14101 Course.

* **Track 1:** Strengthening GAN-based Steganography with Quantum State Locking.
* **Track 2:** Quantum Diffusion Steganography Pipeline.

## 📂 Repository Structure
* `Track 1/`: Source code and Jupyter Notebooks for GAN-based attacker and Quantum Payload Mask.
* `Track 2/`: Source code for Diffusion Inversion, Quantum Sync Lock, and Quantum Seal.
* `Report/`: LaTeX source code and the final PDF report.
* `Slides/`: Presentation slides used for the final defense.

## ⚠️ Datasets & Pre-trained Models
Due to GitHub's file size limits, the large datasets (BOSSBase) and model checkpoints (SRNet, GAN) for **Track 1** are not included in this repository. 

Please download them from our Google Drive link below:
👉 [Download Datasets & Checkpoints](https://drive.google.com/drive/folders/1siVE-WqbtNGUQSDUebamOO46dLnov3SO?usp=sharing)

**Instructions:** After downloading, extract and place the `Datasets/` and `checkpoints/` folders directly inside the `Track 1/` directory before running the Jupyter notebook.

## ⚛️ Track 2: Quantum Diffusion Steganography Pipeline

Track 2 implements a **Quantum-Enhanced Diffusion Steganography** framework that leverages **Stable Diffusion inversion** and **DPM-Solver++** to hide a secret image inside a visually natural cover image while preserving high reconstruction fidelity.

The proposed pipeline integrates diffusion-based image inversion with **quantum-inspired security mechanisms**, including:

* **Quantum Sync Lock** for key-based latent synchronization.
* **Latent Space Scrambling** to protect hidden information from unauthorized recovery.
* **Quantum Seal** for tamper-evident packaging and integrity verification.
* **Diffusion Inversion & Reconstruction** using DPM-Solver++ to achieve high-quality secret image recovery.

### 🔬 Main Pipeline

```
Secret Image
      │
      ▼
VAE Encoding → Latent Representation
      │
      ▼
DPM-Solver++ Inversion
      │
      ▼
Quantum Sync Lock + Latent Scrambling
      │
      ▼
Public Prompt Guided Diffusion
      │
      ▼
Stego / Hide Image
      │
      ▼
Quantum Seal Packaging
      │
      ▼
Transmission
      │
      ▼
Quantum Seal Verification
      │
      ▼
Reverse Diffusion + Key Unlock
      │
      ▼
Recovered Secret Image
```

### 📓 Kaggle Notebook

The complete implementation and experiments for Track 2 are available on Kaggle:

👉 https://www.kaggle.com/code/yeshenhi/quantumstega-q-series-dpm-solver/edit/run/325932786

### 🚀 Features

* Stable Diffusion v1.5 latent inversion.
* DPM-Solver++ based forward and reverse diffusion.
* Quantum-inspired synchronization and latent locking.
* Tamper-evident Quantum Seal packaging mechanism.
* High-fidelity secret image reconstruction.
* Evaluation using image quality metrics such as **PSNR** and **SSIM**.

### 📦 Running the Notebook

1. Open the Kaggle notebook linked above.
2. Enable **GPU acceleration** in the notebook settings.
3. Execute all cells in order.
4. The pipeline will generate:

   * `gt.png` – original secret image.
   * `hide.png` – generated stego image.
   * `reverse.png` – recovered secret image.
   * Quantum package files and evaluation results.

### 📊 Expected Results

The generated stego image should appear visually unrelated to the original secret image, while the recovered image should preserve most semantic and structural information. Experimental results demonstrate strong reconstruction quality with high similarity between the secret and recovered images, while maintaining effective concealment in the stego image.
