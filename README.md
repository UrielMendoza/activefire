# Active Fire Detection in Landsat-8 Imagery: a Large-Scale Dataset and a Deep-Learning Study


## Authors

[Gabriel Henrique de Almeida Pereira](https://github.com/pereira-gha)

[Andre Minoro Fusioka](https://github.com/Minoro)

[Bogdan Tomoyuki Nassu](https://github.com/btnassu)

Rodrigo Minetto


## Downloading the data and creating the masks

If you want to download the original data and process by yourself, the code inside the folder `src/landsat` gives you an easy way to get data from Landsat-8 satellite available on AWS. The script `src/landsat/downloader.py` will download the full images from Landsat-8 (excluding band 8) and the BQA information. The images will be downloaded from a CSV file,  this repository include a file `dataset/images202009.csv`, but you can easily change the file in the `csv` variable. To download the data go to `src/landsat` diretory and run:

```
python downloader.py
```

The images will be saved in the `dataset/images/tif_images/` folder.

After the download of the images you can create the masks with the script `src/landsat/create_masks.py`. This will process the Landsat-8 images and create active fire masks based on the literature algorithm. The masks will be saved in the `dataset/masks/patches/` folder and will be named after the algorithm used to create them. This means that masks created by the [Kumar and Roy (2018)](https://doi.org/10.1080/17538947.2017.1391341) algorithm will be named with the sufix `*_GOLI_v2.TIF` (the v2 is used by organization only). The masks generated by [Murphy et al. (2016)](https://doi.org/10.1016/j.rse.2016.02.027) conditions will be named `*_Murphy.TIF`. And the images created by [Schroeder  et  al.  (2016)](https://doi.org/10.1016/j.rse.2015.08.032) will be named `*_Schroeder.TIF`.


Alternatively, this repository contains some samples that can be used in your experiments. The images patches are in the `dataset/images/pathches.zip` file and the masks are in the `dataset/masks/patches_*.zip` files. To unpack this samples you can use the script `src/utils/unzip_patches.py`:

```
python unzip_patches.py
```

**Atention:** The zip files in this repository are versioned in the Git Large File Storage (LFS), because of that, when you clone this repository the zip files may be corrupted (see [issue #2](https://github.com/pereira-gha/activefire/issues/2)). To fix that you can download manually the zip files and replace your local ones:

To download the zip files directly from the GitHub repository:
[patches.zip](https://github.com/pereira-gha/activefire/blob/main/dataset/images/patches.zip) or the [direct link](https://github.com/pereira-gha/activefire/raw/main/dataset/images/patches.zip)

The `patches.zip` should be placed at `<your-local-code>/dataset/images/patches.zip`

To download the Kumar and Roy (2018) masks: [patches_goli.zip](https://github.com/pereira-gha/activefire/blob/main/dataset/masks/patches_goli.zip) or [direct download](https://github.com/pereira-gha/activefire/raw/main/dataset/masks/patches_goli.zip)

To download the Murphy et al. (2016) masks: [patches_murphy.zip](https://github.com/pereira-gha/activefire/blob/main/dataset/masks/patches_murphy.zip) or [direct download](https://github.com/pereira-gha/activefire/raw/main/dataset/masks/patches_murphy.zip)

To download the Schroeder  et  al.  (2016) masks: [patches_schoreder.zip](https://github.com/pereira-gha/activefire/blob/main/dataset/masks/patches_schroeder.zip) or [direct download](https://github.com/pereira-gha/activefire/raw/main/dataset/masks/patches_schroeder.zip)

This zip files should be placed at `<your-local-code>/dataset/masks/patches_*.zip`

Besides this masks we used artificial masks created by the combination of the previous one. You will find a script `src/utils/make_masks.py` that can help you to create this artificial masks. It will generate a mask based on the intersection of the previous masks and the voting of occurence of fire, where at least two masks must agree where fire occurs. These masks will be saved at `dataset/masks/intersection` and `dataset/masks/voting` folders.

## Models

Each folder inside the `src/train` directory contains the CNNs approachs trained with different active fire detection algorithms. The code inside these folders are almost the same, the only existing changes are in the constants that configure the CNN (number of filters, number of channes, masks used, etc).

The masks used to train the model are obtained through some well-known active fire detection algorithms and the combination of them. The models inside `goli` folder were trained with the masks generated by [Kumar and Roy (2018)](https://doi.org/10.1080/17538947.2017.1391341) conditions. The `murphy` folder use the [Murphy et al. (2016)](https://doi.org/10.1016/j.rse.2016.02.027) conditions. And the `schroeder` folder use the [Schroeder  et  al.  (2016)](https://doi.org/10.1016/j.rse.2015.08.032) conditions. The `intersection` and `voting`folder use the masks generated by the combination of the three previous masks, while the first use the masks generated by the intersection of them (all masks must agree where fire occurs), the second use the masks generated by the agreement of at least two of them.

The folder named `unet_16f_2conv_762` (aka. U-Net-Light (3c)) means that the model used is a `U-net` starting with 16 convolutional filters, with 2 convolutions per block, using the channels 7, 6 and 2 from the source image. The folder named `unet_64f_2conv_762` (aka. U-Net (3c)) starts with 64 convolutional filters and use the channels 7, 6 and 2 from the source image. And the folder named `unet_64f_2conv_10c` (aka. U-Net (10c) ) is pretty much the same, starting with 64 convolutional filters in the first convolutional block and using the 10 channels.

## Sampling the data

The samples used for training, test and validation are defined by CSV files inside the model folder named `dataset` (e.g `src/train/goli/unet_16f_2conv_762/dataset`). The file `images_masks.csv` list all images and the corresponding mask for the approach. The `images_train.csv` and `masks_train.csv` list the files used to train the model, the `*_val.csv` hold the files for the validation and the `*_test.csv` has the files used in the test phase.

This repository include the CSV files used in our experiments, this means that not all images and masks may be found in this repository (due to the size limit) and if you don't download all the dataset you will need to generate new CSV files. You can create new CSV files to use, for this purpose you will find the script in `src/utils/split_dataset.py` that will split your data in three different set.

You may change the constans `IMAGES_PATH` and `MASKS_PATH` with the folder that hold your images and masks. You also need to change the `MASK_ALGORITHM` with the name of the algorithm that created the masks (Schroeder, Murphy or GOLI_v2), this constant will define with image and mask will be included in the CSV files. If you are tranining a `src/train/goli/unet_*` model set it to `GOLI_v2`, if you are traning a `src/train/murphy/unet_*` set it to `Murphy`, etc.

Inside the `src/utils` directory run:
```
python split_dataset.py
```

This will create the `images_masks.csv`, the`images_*.csv` and the `masks_*.csv`. For consistent experiments you need to copy this files inside all models of an approach. So, if you set the `MASK_ALGORITHM` with `GOLI_v2`, for example, all `src/train/goli/unet_*/dataset` must have the same files.

By default the data will be divided in a proportion of 40% for training, 50% for testing and 10% for validation. If you want to change this proportion you need to change the `TRAIN_RATIO`, `TEST_RATIO` and `VALIDATION_RATIO`.

## Training

If you want to train the model from scratch you need to navigate to the desired folder (e.g `src/train/goli/unet_16f_2conv_762`) and simply run:

```
python train.py
```

This will execute all steps needed to train a new model. You may change the constant `CUDA_DEVICE` to the number of the GPU you want to use. This code expectes that the samples are in a sibling folder of `src` named `dataset`, the images must be in the `dataset/images` and the masks in `dataset/masks`. The artifical masks use a different folder: `dataset/intersection` for intersection masks and `dataset/voting` for voting masks. If you are using other directory to hold your samples you may change the `IMAGES_PATH` and `MASKS_PATH` constants.

The output produced by the training script will be placed at `train_output` folder inside the model folder. This repository already include trained weights inside this folder for the U-Net-Light (3c) models, so if you retrain the model **this weights will be overwrited**.

Besides the final weights, this script will save checkpoints every 5 epochs, if you need to resume from a checkpoint you just need to set the constant `INITIAL_EPOCH` with the epoch corresponding to the checkpoint.

## Testing the trained models

**Attention:** this process will fit all data in your RAM, so this can freeze your machine. If you are running this code in a low memory environment you can reduce the rows in the `images_test.csv` and `masks_test.csv` before these steps.

The testing phase are divided in two main steps. The first one is to apply the trained model over the `images_test.csv` images and save the output as a txt file, where 0 represents background and 1 represents fire. The masks in the `masks_test.csv` will also be converted in a txt file. These files will be written in the `log` folder inside the model folder. The output prediction produced by the CNN will be saved with the name `det_<image-name>.txt` while the corresponding mask will be saved with the name `grd_<mask-name>.txt`. To execute this process run:

```
python inference.py
```

You may change the constant `CUDA_DEVICE` to the number of the GPU you want to use. If your samples are placed in other diretory than the default you need to change the constant `IMAGES_PATH` and `MASKS_PATH`. The output produced by the CNN are converted to interger through a thresholding process, the default threshold are `0.25` you can change this value in the `TH_FIRE` constant.

After this processes you can start the second step to evaluate your trained model. You can simply run:

```
python evaluate_v1.py
```

This will show the results to your model.

If you face the message `[ERROR] Dont match X - Y` this means that a mask or a prediction is missing. Make sure all predictions produced by your model have a corresponding mask.


## Compare against the Groundtruth

The code inside the folder `src/groundtruth` gives you a way to compare both the masks of the algorithms and the output produced by CNN agaiinst the Groundtruth. To compare the algorithms (Schroeder, Murphy and GOLI) against the groundtruth run:

```
python evaluate.py

```

This will give you the same metrics available in the test phase.

In the same way, you may test the output produced by the CNN against the groundtruth. The code inside `src/groundtruth/cnn_compare` gives you a way to compare each model. It is important to copy the trained weights inside a folder named `weights` inside each model folder. Alternatively you can change the constant `WEIGHTS_FILE` with the desired weights.

You can compare each model individually, for this access the desired model folder and run:

```
python evaluate.py
```

Or you can run all tests running the `src/groundtruth/cnn_compare/run_all.sh` script. This will create a `txt` file inside each model folder with the results.


## Show the CNN output

You can also generate images from trained models. For this purpose you can access `src/utils/cnn` folder and you will find the `generate_inference.py` script. This script will apply the selected model over and image patch and generate a PNG image with the prediction. You need to define the constant `IMAGE_NAME` with the desired image, and if set the constant `IMAGE_PATH` with the path where this image can be found. It is important to define the constant `MASK_ALGORITHM` with the approach you want to use, the `N_CHANNELS` and `N_FILTERS` with the number of channels used and number of filters of the model. Make sure that the trained weights defined in `WEIGHTS_FILE` is consistent with the parameters defined. After that you can run:

```
python generate_inference.py
```

You may want to compare the output produced by the different architectures and also the masks. The script `generate_figures.py` applies the different networks trained on all patches from the image defined in the constant `IMAGE_NAME`. The output folder will have the output produced from the CNNs, the masks available and an image path with the combination of channels 7, 6 and 2 in a PNG format. This script can be run using:
```
python generate_figures.py
```

## Useful stuff

In the `src/utils` you will find some scripts that may help you understant the images and masks you are working with. For example, you may want to know how many fire pixels are in a mask, for this you can use the `count_fire_pixels.py` script. You just define the `IMAGE_NAME` with the desired mask name and run:

```
python count_fire_pixels.py
```
Alternatively you can define a partial name in the constant `IMAGE_NAME` and the `PATCHES_PATTERN` with a pattern to be found and count fire pixels from many patches.

The masks has the value 1 where fire occurs and 0 otherwise, because of that the masks will not display "white" and "black" if oppened. To help you see the mask you can use the script `transform_mask.py`, this script will convert the image to a PNG with white where fire occurs with a black background. You just need to define the mask you want to convert in the constant `MASK_PATH` and run:

```
python transform_mask.py
```

The images available in the dataset are also difficult to view, as they are images with 10 channels. You can convert them into a visible format, with the combination of bands 7, 6 and 2, using the `convert_patch_to_3channels_image.py` script. You just need to set the path to the desired image in the constant `IMAGE` and run:

```
python convert_patch_to_3channels_image.py
```

## Citation

If you find our work useful in your research, please cite our paper:

```
@misc{pereira2021active,
      title={Active Fire Detection in Landsat-8 Imagery: a Large-Scale Dataset and a Deep-Learning Study}, 
      author={Gabriel Henrique de Almeida Pereira and André Minoro Fusioka and Bogdan Tomoyuki Nassu and Rodrigo Minetto},
      year={2021},
      eprint={2101.03409},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```
