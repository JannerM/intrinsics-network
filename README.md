# Rendered Intrinsics Network
Code and data to reproduce the experiments in [Self-Supervised Intrinsic Image Decomposition](http://people.csail.mit.edu/janner/papers/intrinsic_nips_2017.pdf).

<p align="center">
    <img src='git/figures/intrinsic.png' width='500'/>
</p>

## Installation
Get [PyTorch](http://pytorch.org/) and `pip install -r requirements`

You will need [Blender](https://www.blender.org/) (2.76+) and the [ShapeNet](https://www.shapenet.org/) repository. In `config.py`, replace these lines:
```
blender = '/om/user/janner/blender-2.76b/blender'
shapenet = '/om/data/public/ShapeNetCore.v1'
```
with the absolute paths to the Blender app and the ShapeNet library on your machine. The Blender-supplied Python might not come with numpy and scipy. You can either fulfill the same requirements with the Blender Python or replace `include` with a directory containing those libraries. 

## Data

All of the code to render the training images is in `dataset`. 
1. Make an array of lighting conditions with `python make_array.py --size 20000`. See the parser arguments for lighting options. An array with the defaults is already at `dataset/arrays/shader.npy`.
2. `python run.py --category motorbike --output output/motorbike --low 0 --high 100 --repeat 5` will render 100 composite motorbike images (numbered 0 through 100) along with the corresponding images. It will reuse a given motorbike model 5 times before loading a new one. (Images of the same model will differ in orientation and lighting.) 

The saved images in `dataset/output/motorbike/` should look something like this:

<p align="center">
    <img src='git/shapenet/96_composite.png' width='140'/>
    <img src='git/shapenet/96_albedo.png' width='140'/>
    <img src='git/shapenet/96_shading.png' width='140'/>
    <img src='git/shapenet/96_normals.png' width='140'/>
    <img src='git/shapenet/96_depth.png' width='140'/>
    <img src='git/shapenet/96_lights.png' width='140'/> 
</p>
<p align="center">
    <em> A motorbike with its reflectance, shading, and normals map. The lighting conditions are visualized on a sphere.</em>
</p>

The available ShapeNet categories are given in `config.py`. There are also a few geometric primitives (`cube`, `sphere`, `cylinder`, `cone`, `torus`) and standard test shapes (Utah `teapot`, Stanford `bunny`, Blender's `suzanne`). If you want to render other categories from ShapeNet, just add its name and ID to the dictionary in `config.py` and put the location, orientation, and size parameters in `dataset/utils.py`.

#### Batching
Since rendering can be slow, you might want to render many images in parallel. If you are on a system with SLURM, you can use `divide.py`, which works like `run.py` but also has a `--divide` argument to launch a large rendering job as many smaller jobs running concurrently.

#### Download
We also provide a few of the datasets for download if you do not have Blender or ShapeNet. 
```
./download_data.sh { motorbike | airplane | bottle | car | suzanne | teapot | bunny }
``` 
will download train, val, and test sets for the specified category. The ShapeNet categories have about 2GB of data each, and the test shapes have about 1 GB. 

## Shader

<p align="center">
    <img src='git/figures/shader.png' width='750'/>
</p>
<p align="center">
    <em> Example input shapes and lighting conditions alongside the model's predicted shading image. After training only on synthetic cars like those on the left, the model can generalize to images like the real Beethoven bust on the right.</em>
</p>

To train a differentiable shader:
```
python shader.py --data_path dataset/output --save_path saved/shader --num_train 10000 --num_val 20 \
		 --train_sets motorbike_train,airplane_train,bottle_train \
		 --val_set motorbike_val,airplane_val,bottle_val
```
where the train and val sets are located in `data_path` and were rendered in the previous step. The script will save visualizations of the model's predictions on the validation images every epoch and save them to `save_path` along with the model itself. Note that `num_train` and `num_val` denote the number of images <i>per dataset</i>, so in the above example there will be 30000 total training images and 60 validation images.

## Intrinsic image prediction

```
python decomposer.py --data_path dataset/output --save_path saved/decomposer --array shader --num_train 20000 \
		     --num_val 20 --train_sets motorbike_train --val_set motorbike_val
```

will train a model on just motorbikes, although you can specify more datasets with a comma-separated list (as shown for the `shader.py command`). The rest of the options are analogous as well except for `array`, which is the lighting parameter array used to generate the data. The script will save the model, visualizations, and error plots to `save_path`.

## Transfer
Coming soon. If you are comfortable with Lua, check out `lua/composer.lua`. 

