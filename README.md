# AIMI Annotations Initiative - Prostate MRI Segmentation

## Overview

This AIMI Annotations initiative task was to provide automated segmentation of the prostate from T2 MRI scans for the [ProstateX](https://wiki.cancerimagingarchive.net/pages/viewpage.action?pageId=23691656) collection in the [NCI Imaging Data Commons](https://portal.imaging.datacommons.cancer.gov/) (IDC). ProstateX was originally an AI challenge, this project's goal was to use an [nnUNet](https://github.com/MIC-DKFZ/nnUNet/tree/nnunetv1) model where the input and output of the model are dicom to support the [IDC's](https://portal.imaging.datacommons.cancer.gov/) goal of standardizing and searchable public radiology datasets.

The [model_performance](model_performance.ipynb) notebook contains the code to evaluate the model performance on ProstateX and other datasets.

## Training Dataset

Four different prostate datasets were used to train a prostate segmentation model. These datasets originated from ProstateX (N = 347), [Prostate158](https://github.com/kbressem/prostate158) (N = 139), [NCI-ISBI](https://wiki.cancerimagingarchive.net/pages/viewpage.action?pageId=21267207) (N = 70)15, and [PI-CAI](https://pi-cai.grand-challenge.org/PI-CAI/) (N = 1172)16. For each dataset, annotations were either found through various Grand Challenges or within the IDC. The ProstateX dataset alone contained annotations from 3 different sources. 66 annotations were [high-resolution annotations](https://wiki.cancerimagingarchive.net/pages/viewpage.action?pageId=61080779) acquired from the IDC. 32 were [zonal annotations](https://wiki.cancerimagingarchive.net/pages/viewpage.action?pageId=70230177) (combined to form 1 single annotation) and those were also acquired from the IDC. 134 annotations were obtained through the original ProstateX Grand Challenge. A test/validation split of 81/34 was created from the remaining 115 prostates without annotations. Prostate158 and the NCI-ISBI prostates provided their own annotations with the dataset.

To ensure no cross-over in the PI-CAI annotations a preliminary prostate model was trained on part of ProstateX (N = 232) and all Prostate158 and NCI-ISBI prostates. The preliminary model used an ensemble of fivefold cross-validation within the nnUNet framework for automatic prostate segmentation on T2 MRI images. This model was then used to predict the 81 test prostates of ProstateX and the 1172 prostates of PI-CAI. A portion of the PI-CAI dataset contained a much larger field of view than the field of view used in the other datasets. To combat the increased risk of additional off-targeting regions in the predictions the centermost segmentation (in all directions) was assumed to be the prostate and all additional regions were removed. A secondary model was then trained using these predictions in addition to the data used in the preliminary model. This model also used an ensemble of fivefold cross-validation within the nnUNet framework for automatic prostate segmentation on T2 MRI images. After completing training this model provided annotations for all 347 T2 scans of ProstateX in DICOM format.

## Running the model

The pretrained model weights are [available at zenodo](https://doi.org/10.5281/zenodo.8290092). The simpliest way to use the model is to build and run the docker container.

### Build container from pretrained weights

```bash
cd {REPO_DIR}/container
docker build -t bamf_prostate_mr:latest .
```

### Running inference

By default the container takes an input directory that contains DICOM files of CT scans, and an output directory where DICOM-SEG files will be placed. To run on multiple scans, place DICOM files for each scan in a separate folder within the input directory. The output directory will have a folder for each input scan, with the DICOM-SEG file inside.

example:

```bash
docker run --gpus all -v /path/to/input/dicoms:/data/input -v /path/for/output/dicoms:/data/output bamf_prostate_mr:latest
```

There is an optional `--nifti` flag that will take nifti files as input and output.

#### Run inference on IDC Collections

This model was run on CT scans from the ProstateX collection. The AI segmentations are available in the prostate-mr.zip file on the [zenodo record](https://doi.org/10.5281/zenodo.8345959). A radiologist reviewed 10% of the segmentations and found that all of them were acceptable without changes.

- [ ] TODO: You can reproduce the results with the [run_on_idc_data](run_on_idc_data.ipynb) notebook on google colab.

#### Run inference on other datasets

It is useful to run the model on other datasets. The model_performance notebook uses predictions for several other datasets to evaluate the model. First run `prostate_mr_qa_datasets.ipynb` to prepare and download the datasets. THen run `model_performance.ipynb` to evaluate the model.

### Training your own weights

- [ ] TODO: Refer to the [training instructions](training.md) for more details.
