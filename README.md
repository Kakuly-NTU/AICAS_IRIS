# IRIS Functional Cell-Level Classification for Hardware Assurance

## Overview
This repository presents an end-to-end pipeline for automated chip inspection using Infra-Red In-Situ (IRIS) imaging and Computer Vision (CV). The work introduces the first labeled IRIS functional cell-level dataset and demonstrates the feasibility of cell-level classification for hardware verification.

## Key Contributions
- End-to-End Pipeline: IRIS → Alignment → GDS-guided Cell Extraction → Classification → Aggregation
- Dataset: 246,410 labeled cell images (logic, flip-flop, filler)
- Models: CNN, VGG-16, GoogLeNet, ResNet
- Best Accuracy: ResNet achieves 88.99%

## Pipeline
IRIS Imaging → Chip Extraction → Chip Alignment with GDS Files → Cell-level Cropping → Dataset → Training → Classification → Counting

## Dataset
- Balanced dataset: 19,866 images (6,622 per class)
- Preprocessing: Resize (224×224), normalization, augmentation

## Training
- Framework: PyTorch
- Epochs: 50
- Loss: Weighted Cross-Entropy
- LR: 1e-4

## Results
Consistent ~88% accuracy across models.

## Hardware Assurance
Cell classification enables cell-count estimation and detection of structural anomalies or tampering.

## Usage
```bash
git clone https://github.com/Kakuly-NTU/AICAS_IRIS.git
pip install -r requirements.txt
```

## Dataset Release
Dataset and code will be released upon publication.

## Future Work
- Transformer models
- Contrastive learning
- Higher resolution IRIS imaging
