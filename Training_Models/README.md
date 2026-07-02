# Cell-Level Classification — Training Notebooks

This folder contains the Jupyter notebooks used to train and evaluate the four convolutional neural network (CNN) baselines reported in our AICAS paper for **IRIS functional cell-level classification** into three classes:

| Label index | Class | Description |
|:---:|:---|:---|
| `0` | `filler` | Filler / decap cells (non-functional area fill) |
| `1` | `logic` | Combinational logic gates (AND, OR, INV, MUX, XOR, buffers, etc.) |
| `2` | `ff` | Sequential elements — flip-flops and latches |

Each notebook trains one model architecture on the balanced cell-image dataset produced by the data-curation stage (see `Bunnie_Huang_IRIS_Processing_Scripts/extract_dataset_cell.py`) and reports precision / recall / F1 / accuracy on train, validation, and test splits.

---

## Folder Contents

```
Training_Models/
├── CNN.ipynb          # Custom shallow baseline CNN (three convolutional layers)
├── VGG.ipynb          # VGG-16, pretrained on ImageNet
├── GoogLeNet.ipynb    # GoogLeNet (Inception V1), pretrained on ImageNet
└── ResNet_Model.ipynb # ResNet-18, pretrained on ImageNet — best-performing baseline
```

| Notebook | Architecture | Pretrained Weights | Reported Test Accuracy (paper) |
|---|---|:---:|:---:|
| `CNN.ipynb` | Custom 3-layer CNN | — | 88.42% |
| `VGG.ipynb` | VGG-16 | ImageNet1K-V1 | 88.12% |
| `GoogLeNet.ipynb` | GoogLeNet (Inception V1) | ImageNet1K-V1 | 87.58% |
| `ResNet_Model.ipynb` | ResNet-18 | ImageNet1K-V1 | **88.99%** |

All four notebooks share the same data-loading, augmentation, sampling, training, and evaluation code. Only the model definition cell and the checkpoint filename differ.

---

## Shared Pipeline (Common to All Four Notebooks)

Every notebook follows the same structure. If you want to add a new architecture, copy any existing notebook and change only the *Model Definition* cell.

**1. Reproducibility.** Random seed fixed to `SEED = 56`; NumPy, PyTorch, and CUDA RNGs are seeded, `cudnn.deterministic = True`, `cudnn.benchmark = False`.

**2. Data loading.** All `*.pkl` files produced by `extract_dataset_cell.py` are read from `imaging/dataset_extraction/cell_dataset/`. Each pickle contains `{"data": [...], "labels": [...], "cell_details": [...]}` for one functional block.

**3. Class balancing.** The raw dataset contains **246,410 images** (164,228 filler / 75,560 logic / 6,622 ff). Random undersampling to the minority-class count yields a balanced set of **19,866 images** (6,622 per class).

**4. Splits.** Stratified 70 / 15 / 15 train / validation / test split, seeded, per class.

**5. Preprocessing & augmentation.** All images are resized to **224 × 224** and normalized with ImageNet mean/std. Random horizontal and vertical flips are applied to the training set only.

**6. Sampler.** A `WeightedRandomSampler` is used on the training loader as a second layer of imbalance handling on top of the undersampling.

**7. Training.**
- Optimizer: `AdamW`, `lr = 1e-4`, `weight_decay = 1e-4`
- Loss: `nn.CrossEntropyLoss` with per-class weights (inverse class frequency, clamped)
- Batch size: 64
- Max epochs: 50
- Early stopping: patience = 5, monitors validation accuracy (`mode='max'`), `min_delta = 1e-4`, restores best weights
- Best model checkpoint saved as `<ModelName>_best_model.pth`

**8. Evaluation.** After training, the best checkpoint is loaded and evaluated on train, validation, and test loaders. Reported metrics: per-class and macro precision / recall / F1-score, overall accuracy, and confusion matrix (via `sklearn.metrics`).

---

## Requirements

Tested with Python 3.10+ on a single CUDA-capable GPU.

```bash
pip install torch torchvision                    \
            numpy opencv-python matplotlib       \
            scikit-learn                         \
            jupyter notebook
```

Recommended (used during development):
- PyTorch ≥ 2.0 with CUDA
- ~8 GB of GPU memory is sufficient at `BATCH_SIZE = 64` and `224 × 224` inputs.

---

## Expected Input

Before running any notebook, either:

- **Generate the cell-level PKL dataset yourself** by running `Bunnie_Huang_IRIS_Processing_Scripts/extract_dataset_cell.py` on the aligned IRIS chip images and their GDS blueprints, **or**
- **Download the ready-made dataset** from our IEEE DataPort release: [IRIS Functional Cell-Level Classification Dataset for Hardware Assurance](https://ieee-dataport.org/documents/iris-functional-cell-level-classification-dataset-hardware-assurance).

The notebooks expect the PKL files at:

```
imaging/dataset_extraction/cell_dataset/*.pkl
```

Each PKL file contains one functional block's extracted cell crops with the schema:

```python
{
    "labels":       [int, ...],          # 0 = filler, 1 = logic, 2 = ff
    "data":         [np.ndarray, ...],   # BGR/RGB cell-image crops
    "cell_details": [str, ...],          # "x0_y0_x1_y1_<cellname>_<label_index>"
}
```

If your dataset lives elsewhere, edit the `pkl_directory` variable in the *Configuration* cell near the top of each notebook.

---

## How to Run

1. Launch Jupyter:
   ```bash
   jupyter notebook
   ```
2. Open the notebook for the architecture you want to train (e.g. `ResNet_Model.ipynb`).
3. Verify the *Configuration* cell — in particular `pkl_directory`, `BATCH_SIZE`, `NUM_WORKERS`, and the CUDA device index.
4. **Run all cells.** Training will run for up to 50 epochs, with early stopping on validation accuracy. The best checkpoint is saved next to the notebook.
5. The final cells print classification reports and confusion matrices for the train, validation, and test splits.

---

## Configuration Knobs

The most useful knobs are grouped in the *Configuration* cell near the top of each notebook:

| Variable | Default | Meaning |
|---|:---:|---|
| `pkl_directory` | `imaging/dataset_extraction/cell_dataset/` | Location of the cell-level PKL files |
| `TARGET_H, TARGET_W` | `224, 224` | Model input resolution |
| `USE_GRAYSCALE` | `False` | If `True`, converts inputs to single-channel |
| `BATCH_SIZE` | `64` | Training / eval batch size |
| `NUM_WORKERS` | `4` | DataLoader worker count |
| `SEED` | `56` | Global random seed |
| `EPOCHS` | `50` | Maximum training epochs (early stopping typically triggers earlier) |

---

## Outputs

Running a notebook produces:

- `<ModelName>_best_model.pth` — PyTorch state-dict of the best (highest val-accuracy) epoch.
- Inline plots and printed classification reports for train / validation / test.
- A confusion matrix per split.

---

## Reproducing Table I of the Paper

To reproduce our reported numbers:

1. Regenerate the cell-level PKL dataset from the same aligned IRIS images and GDS blueprints (using `Bunnie_Huang_IRIS_Processing_Scripts/extract_dataset_cell.py`) — or download it directly from our [IEEE DataPort release](https://ieee-dataport.org/documents/iris-functional-cell-level-classification-dataset-hardware-assurance).
2. Run each notebook end-to-end **without changing** `SEED`, split ratios, optimizer settings, augmentation, or the sampler.
3. Read off the final train / validation / test classification reports; they should match Table I within stochastic noise on non-deterministic CUDA kernels.

> **Note.** Full bit-exact reproducibility across different GPUs is not guaranteed — some cuDNN kernels remain non-deterministic even with `cudnn.deterministic = True`. Reported accuracies should nonetheless be within a fraction of a percent.

---

## Citation

If you use these notebooks or the associated dataset, please cite our AICAS paper:

> *(Citation will be provided once the proceedings are published at the conference!)*


And, where relevant, the model architectures themselves:

- **VGG:** Simonyan & Zisserman, *"Very Deep Convolutional Networks for Large-Scale Image Recognition,"* arXiv:1409.1556, 2015.
- **GoogLeNet:** Szegedy et al., *"Going Deeper with Convolutions,"* arXiv:1409.4842, 2014.
- **ResNet:** He, Zhang, Ren, & Sun, *"Deep Residual Learning for Image Recognition,"* arXiv:1512.03385, 2015.

---

## Future Work

As noted in Section III.B of the paper, the reported CNN accuracies (~88%) should be viewed as **strong baselines rather than upper bounds**. Promising directions for extending this folder:

- **Vision Transformers (ViT)** and hybrid CNN–attention models to capture longer-range structural dependencies across cell layouts.
- **Domain-adaptive pretraining** on IRIS imagery.
- **Contrastive / self-supervised representation learning** on the unlabeled portions of the IRIS dataset.
- **Custom architectures** designed for the periodic, grid-aligned nature of standard-cell layouts.
