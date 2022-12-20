# Facial Landmark Detection With Synthetic Data

# Acknowledgement

This project makes use of the following repositories:

https://github.com/braindotai/Facial-Landmarks-Detection-Pytorch

https://microsoft.github.io/FaceSynthetics/

# Requirements
`Python3`
```
opencv-python
dlib
numpy
matplotlib
imageio
scikit-image
torchsummary
tqdm
torch
imutils
```

# File Configuration
Two directories should be set up:

## Data Directory
```bash
/Landmark Dataset <-- custom name
    /ibug_300W_large_face_landmark_dataset <-- ibug 300W dataset
    /dataset_5000 <-- MS synthesis dataset
        00000.png
        00000_ldmks.txt
        00001.png
        00001_ldmks.txt
    labels_ibug_300W_train.xml
    labels_ms_synthesis_mix.xml
    labels_ms_synthesis_landmark.xml
    labels_ms_synthesis_dlib.xml
```

The 300W dataset can be downloaded directly from [here](http://dlib.net/files/data/ibug_300W_large_face_landmark_dataset.tar.gz), or take a look at [here](https://github.com/braindotai/Facial-Landmarks-Detection-Pytorch/blob/master/Face%20Landmark%20Detection.ipynb).

The MS synthesis dataset should be direclty downloaded from [here](https://github.com/microsoft/FaceSynthetics). Due to limit on training system, we choose 5000 image sets randomly. You can choose any subset of your choice.

The XML files contains file names for each dataset and the corresponding landmark labels and face boundary box (bbox). The 300W xml is directly taken from the provided dataset; the synthesis xmls are generated by script `/helper/build_synthesis_xml.py`. The `_mix, _landmark, _dlib` represents the bbox generation method used. `dlib` means bbox are created using the *dlib frontal face detector*, which has only about 60% of face detection rate on the synthesis dataset. `landmark` means that we use provided landmark labels to infer face boundaries. We find the min and max values for both axes, and add a little offset (15 pixels). The `_mix` label means the bbox is created with *dlib* package whenever possible, then with *landmarks* if dlib fails.

## Training Directory

For training, just use the `/src` directory and the `train.py` script.

Run the command `train.py --folder $train_subdir$` to start training. Make sure:

```
/src
    train.py
    /train_set1
        train.json
```

Look at the `exmaple_train_folder` for reference.

The training will run automatically and add 3 files under the `train_subdir`: `model.epoch`, `model.pt`, and `datalist.pickle`. These are model checkpoint files and used for evaluation and resumption. Read the `train.py` comments for more information.

To resume training, run: `train.py --folder $train_subdir$ --resume`.

## Testing Directory

Testing directory is the same as training direcoty. You should use `Testing.ipynb` notebook file.

# Training Configuration
```JSON
{
    "data_dir": "/Users/dongxuhuang/Downloads/Landmark",
    "bbox_option": "dlib",  # dlib, landmark, mix
    "size": "2000",
    "ft_ratio": "0.7",
    "train_test_split": "1.0",
    "image_dim": "128",
    "brightness": "0.24",
    "saturation": "0.3",
    "contrast": "0.15",
    "hue": "0.14",
    "angle": "14",
    "face_offset": "32",
    "crop_offset": "16",
    "epochs": "100"
}
```

The training configration file `train.json` in each training set subdir is used to initialize the training. The `data_dir` is the *absolute* path to the data directory you defined. The `bbox_option` tells the training script which bbox method to use (which xml file to pull, see previous section for explanation). The `size` tells how many images should be contained the training dataset. Please note only a limited number of synthesis images can be processed by *dlib* for bbox; if the requested `size` exceed this limit, `mix` option will be used. The `ft_ratio` is the ratio of synthesis-to-real faces in training data. Use `1.0` if you want to train with fully synthetic data. Otherwise the script will randomly pick some real-faces from the 300W dataset. `epochs` defines number of epochs used for training.

All other parameters are preprocessing parameters defined in model. Please read the model to understand.