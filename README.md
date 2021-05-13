# Vectorization historical map

## Pipeline
<p align="center">
<img src="./images/pipeline_v1.svg" width="100%">
</p>

## Abstract
Maps have been a unique source of knowledge for centuries. 
Such historical documents provide invaluable information for analyzing the complex spatial transformation of landscapes over important time frames. 
This is particularly true for urban areas that encompass multiple interleaved research domains (social sciences, economy, etc.). 
The large amount and significant diversity of map sources call for automatic image processing techniques in order to extract the relevant objects under a vectorial shape. 
The complexity of maps (text, noise, digitization artifacts, etc.) has hindered the capacity of proposing a versatile and efficient raster-to-vector approaches for decades. 
We propose a learnable, reproducible, and reusable solution for the automatic transformation of raster maps into vector objects (building blocks, streets, rivers). 
It is built upon the complementary strength of mathematical morphology and convolutional neural networks through efficient edge filtering.
Evenmore, we modify ConnNet and combine with deep edge filtering architecture to make use of pixel connectivity information and built an end-to-end system without requiring any post-processing techniques.
In this paper, we focus on the comprehensive benchmark on various architectures on multiple datasets coupled with a novel vectorization step.
Our experimental results on a new public dataset using COCO Panoptic metric exhibit very encouraging results confirmed by a qualitative analysis of the success and failure cases of our approach.
Code, dataset, results and extra illustrations are freely available at this github repository.

## Table of Content

* [1. Visual Results](#1-visual-results)
* [2. Installation](#2-installation)
* [3. Prepare training data](#4-preparetrainingdata)
* [4. Train](#4-train)
* [5. Evaluation](#5-evaluation)

## 1. Visual Results

## 2. Installation

### 2.1. Get the code and install Dependencies

Install Pytorch adapted to your CUDA version : 

* [Pytorch 1.8.1+cu111](https://pytorch.org/get-started/previous-versions) 
* [Torchvision 0.9.1+cu111](https://pytorch.org/get-started/previous-versions)

```shell script
git clone https://github.com/soduco/ICDAR-2021-Vectorization.git
cd ICDAR-2021-Vectorization
```
You now have the code in directory ICDAR-2021-Vectorization.
At this point, you should probably create a virtual environment. For instance:
```shell script
python -m venv vec-env
source ./vec-env/bin/activate
```
Now, install the dependencies (with pip).
```shell script
pip install -r requirements.txt
```

### 2.2. Pre-trained models 

Download pre-trained models:

Please check the [Releases](https://github.com/soduco/ICDAR-2021-Vectorization/releases) to download it.

### 2.3. Datasets

Dataset used to train, validate and test our approach:

Please check the [Releases](https://github.com/soduco/ICDAR-2021-Vectorization/releases) to download it.

## 3. Prepare training data

### 3.1. Download the dataset

To ease the usage of training data, we provide two versions of dataset:

1. Full images (~7000px * 11000px)

2. Patch version (already cutting into small patches to ease training, every patch have size 500px 500px), and file index .lst file: *test_pari.lst*, *train_pair.lst*, *val_pair.lst*
Please check the [Releases](https://github.com/soduco/ICDAR-2021-Vectorization/releases) to download it.

### 3.2. Reproducing the training on historical maps 

The training patch data need to be downloaded from [here](https://github.com/soduco/ICDAR-2021-Vectorization/releases), zipped and saved into `./data_generator`. 
The file index .lst file can be also download in the same Release.

The file structure is:

```
./ICDAR-2021-Vectorization/data_generator
├── train_data_tiles/
├── val_data_tiles/
├── test_data_tiles/
├── train_gt_tiles/
├── val_gt_tiles/
├── test_gt_tiles/
├── train_pari.lst
├── val_pair.lst
└── test_pair.lst
```

The configuration file *cfg.py*, 'data_lst' should link with the path of the .lst file, *mean_bgr* is the average value of bgr channel for the whole *train/val/test* dataset, but you can also configure *cfg.py* and more options are available:

```python
config = {
    'HistoricalMap2020': {'data_root': '../../data_generator/',
                'data_lst': 'train_pair.lst',
                'mean_bgr': [168.1872554, 195.97301654, 204.64264255]},
}

config_test = {
    'HistoricalMap2020': {'data_root': '../../data_generator/',
                'data_lst': 'test_pair.lst',
                'mean_bgr': [158.0, 191.0, 210.0]},
}

config_val = {
    'HistoricalMap2020': {'data_root': '../../data_generator/',
                'data_lst': 'val_pair.lst',
                'mean_bgr': [168.1872554, 195.97301654, 204.64264255]},
}

```

### 3.3. Create image patch from full map images
To create image patches for training, it consist two seperate step:
1. Create image patches
2. Generate .lst file index from image patches

#### 3.3.1. Create Image patches

Since the size of whole image is too big as the input of the network, we require to divide the whole map image into batches.

The batch images will save into folder *output_directory/image* and *output_directory/gt*

```shell script
python ./data_generator/create_image_batch.py <map_image_input> <ground_truth_image_input> <output_directory>
```

#### 3.3.2. Generate .lst file index from image patches
The prepare training text can create .lst file with image file indexing in order to train the network.

```shell script
python ./data_generator/prepare_training_text.py
```

The file structure should be the same as in section 3.2.


## 4. Train
we can now train the models with the prepared data. Remeber the data should be patched and .lst file should be correctly generated and put into the right place. If you want to get more control for your training, you can list all the arguments:

``` Bash
cd ./train/<model_name>
python train.py --help
```

```
usage: train.py [-h] [-d {HistoricalMap2020}] [--param-dir PARAM_DIR] [--lr BASE_LR] [-m MOMENTUM] [--model MODEL] [-c] [-g GPU] [--weight-decay WEIGHT_DECAY] [-r RESUME] [-p PRETRAIN] [--epochs EPOCHS] [--max-iter MAX_ITER] [--iter-size ITER_SIZE][--average-loss AVERAGE_LOSS] [-s SNAPSHOTS] [--step-size STEP_SIZE] [-b BALANCE] [-l LOG] [-k K] [--batch-size BATCH_SIZE] [--crop-size CROP_SIZE] [--complete-pretrain COMPLETE_PRETRAIN] [--side-weight SIDE_WEIGHT] [--fuse-weight FUSE_WEIGHT] [--gamma GAMMA]
```

If you want to train HED and BDCN with pre-train model (The folder *./train/HED_pretrain* and *./train/BDCN_pretrain*) please download the .pth pretrain model file (vgg16-397923af.pth) from [here](https://github.com/soduco/ICDAR-2021-Vectorization/releases) and put it into *./pretrain_weight/*.

## 5. Inference and restruction EPM to full size image

Once models are trained and best models are selected, the function pred_full_map.py can predict and reconstruct the predicted EPM patches into full-size image. The main goal for the EPM recosntruction is used furthur for watershed process. 

``` Bash
cd ./train/<model_name>
python pred_full_map.py --cuda --gpu <gpu_index> --model <trained_save_model> --original_image_path <original_image_before_patch> --EPM_border <border_of_frame> --batch_image_size <size_of_patches>
```

## 6. Watershed process

A linux build is provided but, if for some reason it doesn't work for you, you can rebuild it yourself. Here is how.

```shell script
pip install conan
conan remote add lrde-public https://artifactory.lrde.epita.fr/artifactory/api/conan/lrde-public

cd ./watershed/histmapseg/
mkdir newbuild && cd newbuild
conan install .. --build missing -s compiler.libcxx=libstdc++11 -s compiler.cppstd=20 -g cmake
cmake .. -DCMAKE_POSITION_INDEPENDENT_CODE=OFF -DCMAKE_BUILD_TYPE=Release
cmake --build . --config Release
```
Here you go: you now have your executable in *newbuild/bin/histmapseg*.

## Run the watershed segmentation executable

```shell script
./watershed/histmapseg/build/bin/histmapseg <input.png> <dynamic> <area_closing> <ws.tiff> <out.png>
```

*input.png*: this should be the file of EPM

*dynamic*: the parameter of dynamic

*area_closing*: the size for closing area

*ws.tiff*: the resulting watershed tiff file

*out.png*: the colorized watershed file

For instance, to use one of the parameter sets from the paper:

```shell script
./watershed/histmapseg/build/bin/histmapseg result_epm_mask 7 400 ws.tiff out.png
```

## 7. Evaluation



