# DEM_segmentation

In this repo, we implement a Unet model with data preprocessing for segmenting geographical feature such as buildings or erosion features on Ditial Elevation Maps (DEM).

## Setup

### Prerequisites
- Tensorflow
- keras
- gdal (optional, matters when you need to retile from a large DEM)
  + Anaconda: `conda install -c conda-forge gdal`
  + pip : `pip install GDAL`

- Clone this repo:
```bash
https://github.com/Chen-Yifan/DEM_building_segmentation.git
cd DEM_building_segmentation/
```

## Dataset in TIFF

Download the dataset from IEEE dataport at <https://dx.doi.org/10.21227/p2fp-mn16>

To cite: 
```
@data{p2fp-mn16-22,
doi = {10.21227/p2fp-mn16},
url = {https://dx.doi.org/10.21227/p2fp-mn16},
author = {Chen, YIfan and Soliman, Aiman and Kindratenko, Volodymyr and Luo, Shirui and Makharov, Rauf},
publisher = {IEEE Dataport},
title = {DEM_building_Illinois},
year = {2022} }
```

The dataset details:
  ```
  DEM:
    DEM_clip.tif

  Labels:
    label_clip.tif: 1 - building, 0 - background
    label_clip_012.tif: 1 - Microsoft (MS) building in MS labelling area, 2 - MS building in manual labelled area, 0 - background
    manual_label_raster_lzw.tif: manually labelled buildings. 2 - background, 1 - mannual labelled building footprints
  ```


## Sampling Datasets from TIFF

### 1. (Optional) Create derivatives of DEM

For example, to generate aspect map of the original DEM map, you may run:
```bash
gdaldem aspect DEM_clip.tif $OUTPUT.tif -of GTiff -b 1
```
To generate different derivatives, the second argument `aspect` can be changed to `slope`, `roughness`, `TPI`, `TRI`.

### 2. (Optional) Merge multiple maps into one multi-band map 
Combine the separate bands in a single image;  all bands will be initialized using 0 

```bash
gdal_merge.py -separate -init 0 -o $MERGED.tif DEM_slope.tif DEM_aspect.tif DEM_rough.tif DEM_tpi.tif DEM_tri.tif DEM_clip.tif
```

### 3. Generate Tiled Images (Ex: Size 128x128 with overlap 64)

Generate tiled feature images

```bash
gdal_retile.py -v -r near -ps 128 128 -co “TILED=YES”  -targetDir frames_128_overlap  -tileIndex  tiles_frames  -overlap 64   -csv frames.csv  -csvDelim ,  $FEATURE_MAP.tif 
```

Generate tiled label images

```bash
gdal_retile.py -v -r near -ps 128 128 -co “TILED=YES”  -targetDir annotations_128_overlap  -tileIndex  tiles_annotations  -overlap 64     -csv annotations.csv  -csvDelim ,  $LABEL_MAP.tif 
```

## Model Setup
Based on Unet, we experiment with Convolution2D regularization parameter, Dropout layer, and we also modify the number of filters. We tried both Adam and Adadelta as the optimizer.

## Train and Test 
To train the model:  ```./scripts/train_building.sh```

  ```main.py```: the main function
  
  ```data_loader.py```: load data as arrays, and do preprocess such as normalization. In this part, we only experiment with single-band DEM input or its gradient. To include their derivatives, please change the shape of the arrays also the size of input, output of the model.
  
  ```dataGenerator.py```: data batch augmentation for training.
  
  ```build_model.py```: the actual model structure and compile the model.
  
  ```define_model.py```: fit the model and callback specifications for train and test.
  
  ```losses.py```: loss functions that might be helpful.
  
  ```metrics.py```: metrics to measure the performance.
 
 
 

