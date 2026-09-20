# LGG-MRI-UNet-segmentation
U-Net segmentation of lower-grade gliomas in brain MRI: patient-level split, ImageNet-pretrained vs from-scratch encoders, and failure analysis by tumour size.
# Brain Tumour Segmentation in MRI with U-Net

Segmentation of lower-grade gliomas (LGG) in brain MRI, with a study of whether
ImageNet pretraining helps on medical images.

## Data
LGG MRI Segmentation dataset (Buda et al., 2019): 110 patients, 3,929 slices of
256x256 pixels with expert masks of the FLAIR abnormality. 1,373 slices contain
a tumour, 2,556 do not.
Split BY PATIENT (70% / 15% / 15%) so that no patient appears in two sets.

## Method
U-Net (segmentation_models_pytorch) with a ResNet-34 encoder, Dice + BCE loss,
Adam (lr 1e-4), 25 epochs, horizontal flip augmentation.
Inputs normalised with ImageNet channel statistics.
Evaluated with the Dice score, reported separately on tumour slices, because two
thirds of the slices are empty and inflate the overall average.

## How to run
Open `LGG_UNet_Segmentation.ipynb` in Google Colab with a GPU runtime.
The dataset downloads automatically via `kagglehub` (a free Kaggle account is
required).

## Results

| Model | Dice (all) | Dice (tumour) | Small | Medium | Large |
|---|---|---|---|---|---|
| Pretrained, no normalisation | 0.846 | 0.676 | 0.448 | 0.705 | 0.870 |
| Pretrained + normalisation | 0.902 | 0.736 | 0.557 | 0.789 | 0.858 |
| From scratch + normalisation | 0.886 | 0.700 | 0.609 | 0.717 | 0.770 |

Slices missed entirely (Dice < 0.01), by tumour size:

| Model | Small | Medium | Large |
|---|---|---|---|
| Pretrained + normalisation | 28% | 6% | 0% |
| From scratch + normalisation | 20% | 15% | 7% |

## Findings
Input normalisation with ImageNet statistics improved Dice on tumour slices from
0.676 to 0.736 with the same architecture and pretraining, the largest single gain
in this project. With normalisation applied to both, the ImageNet-pretrained
encoder outperformed the same architecture trained from scratch (0.736 vs 0.700).
The pretrained model was clearly better on medium and large tumours and never
missed a large one, against 7% missed by the scratch model, while the scratch
model was slightly better on small tumours (0.609 vs 0.557).

Across all models, small tumours remain the main failure mode, with 20-28% missed
entirely. Lowering the decision threshold from 0.5 to 0.2 did not recover them,
which suggests the model assigns uniformly low probabilities on those slices
rather than values just below the threshold.

These results come from a single patient-level split with 16 test patients, so
differences of this size should be confirmed with cross-validation before drawing
firm conclusions.

## Limitations
2D slices only, without context from neighbouring slices; a single public dataset
of 110 patients; one random split with 16 test patients, so small differences are
not statistically confirmed.

## Possible next steps
Cross-validation or several random seeds; a loss that penalises missed tumours
more strongly (e.g. Tversky) to address the small-tumour failures; a 3D model.

## Reference
M. Buda, A. Saha, M. A. Mazurowski, "Association of genomic subtypes of
lower-grade gliomas with shape features automatically extracted by a deep
learning algorithm", *Computers in Biology and Medicine*, 2019.

Dataset: https://www.kaggle.com/datasets/mateuszbuda/lgg-mri-segmentation
