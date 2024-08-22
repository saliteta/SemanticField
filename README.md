# Semantic Field
- This is an implementation of Semantic Field
- We use a backend called feature splat which is a modified version for gsplat
- Currently, there is still no training result, but training is on the go


## Semantic Field Method Introduction
- We obtained a reliable 3D Gaussian Splat Representation (Means, Covariance Matrix, Alpha, Color) according to splatfacto
- We then add features (up to 1024 dimension) for each Gaussian
- Currently the feature is obtained according to these two repositories: [Semantic Consistancy Latent SAN](https://github.com/saliteta/SC-latent-SAM), and [Open Vocabulary Mask Query](https://github.com/saliteta/OpenVocabMaskQuery)

## Install
- Install NeRF studio and [feature_splat](https://github.com/saliteta/feature_splat) according to the feature_splat requirment
- cd to SemanticField directory and run:
```
pip install .
```

## Data Set Preparation
### Get Colmap dataset
If one has no idea what is COLMAP, please follow [Gaussian Splatting](https://github.com/graphdeco-inria/gaussian-splatting) to install and run sparse reconstruction
- We need to construt the dataset in the following way. At the first stage it is exactly formated like what is displayed in the colmap data structure for NeRF-Stduio Processing
```
--DATASET_FOLDER
|    |--colmap
|       |--sparse
|           |----0 
|              |--camera.bin
|              |--images.bin
|              |--camears.bon
|              |--cfg_args
|    |--images
|      |--image1
|      |--image2
|        ...
```

### Get the Gaussian Splat
Our method only serves as a maping from feature to Gaussians, not optimizing Gaussian's other attributes, Therefore, we need to first obtain a usable Gaussian Splat as follow
```
ns-train splat-facto --data $DATASET_FOLDER --output-dir $output_directory colmap
```
Then we need to use an export function to convert Gaussians Splats checkpoints to usable .ply file
```
ns-export gaussian-splat $output_directory/$DATASET_FOLDER/{some time you might need to find by yourselves}/config.yml --output-dir $DATASET_FOLDER 
```
Then your model should looks like the following: 
```
|--DATASET_FOLDER
|    |--colmap
|       |--sparse
|           |----0 
|              |--camera.bin
|              |--images.bin
|              |--camears.bon
|              |--cfg_args
|    |--images
|      |--image1
|      |--image2
|        ...
|    |--splat.ply
```

### Get Semantic Information from Refined Mask and CLIP
- Strictly follow what SC-latent-SAM and OpenVocab Mask Query has done. In the final stage, there is a script called prepare_data.py, it will convert data to the format suitable for Gaussian:
```
CUDA_VISIBLE_DEVICES=0 python -W ignore prepare_data.py\
    --mask_location ${REFINED_OUTPUT_DIR}refined_mask.npz \
    --label_location ${REFINED_OUTPUT_DIR}refined_label.npz \
    --features_location ${REFINED_OUTPUT_DIR}semantic_features.npz\
    -o $DATASET_FOLDER
```

At the end of current stage, one will have the following Dataset Structure:
```
|--DATASET_FOLDER
|    |--colmap
|       |--sparse
|           |----0 
|              |--camera.bin
|              |--images.bin
|              |--camears.bon
|              |--cfg_args
|    |--features
|      |--image1.npz
|      |--image1.npz
|        ...
|    |--images
|      |--image1.png
|      |--image2.png
|        ...
|    |--masks
|      |--image1.npz
|      |--image1.npz
|        ...
|    |--colors
|      |--image1.npz
|      |--image1.npz
|        ...
|    |--splat.ply
```

## Training 
Training is simple
```
ns-train --data $DATASET_FOLDER --output-dir $output
```

