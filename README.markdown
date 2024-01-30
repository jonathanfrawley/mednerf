# MedNeRF: Medical Neural Radiance Fields for Reconstructing 3D-aware CT-Projections from a Single X-ray

[Paper](https://arxiv.org/abs/2202.01020)
Accepted at IEEE EMBC 2022


## Get the Data
You can find all DRR in the following [link](https://drive.google.com/file/d/1_EJX3LnRMG5uXEhZ63C2eYoY4hjwmipP/view?usp=sharing). Here is a description of the folders:

An <em>instance</em> comprehends 72 DRRs (each at 5 degrees) from a 360 degree rotation of a real CT scan.


`chest_xrays` all images of the 20 chest instances (.png, res. 128x128).

`knee_xrays` all images of the 5 knee instances (.png, res. 128x128)

## Get the weights
You can download our models' weights through the following links:

[Knee model](https://drive.google.com/file/d/17cznfl6OSV4p2rzjKJXRJ_cmOM7z2tiP/view?usp=sharing)

[Chest model](https://drive.google.com/file/d/13NpwA3TiogPHYBeuvI5QapfghdPio3SF/view?usp=sharing)

## Train a model
Refer to graf-main folder and execute, replacing CONFIG.yaml with knee.yaml or chest.yaml
```
python train.py configs/CONFIG.yaml
```

## Reconstruction given an X-ray
After training a model, you can test its capacity to reconstruct 3D-aware CT projections given a single X-ray. 

To execute the reconstruction, please refer to graf-main folder and execute:
```
python render_xray_G.py configs/experiment.yaml /
    --xray_img_path path_to_xray_folder /
    --save_dir ./renderings /
    --model path_to_trained_model/model_best.pt /
    --save_every 25 /
    --psnr_stop 25 
```
Update: This step requires more memory as rays are now tracked during the optimization process, so the higher the resolution of the image, more memory it requires. For this, we add an additional argument (img_size) with a default value of 64 if GPU size <=10GB.

**Optimizing both G and z** (`render_xray_G_Z.py`): By setting the model to generator.train() when calling reconstruction(), we can optimize z as well and NLL will change over time. Finetuning like this, optimizes faster when considering a single X-ray projection, however, G would need to figure out 3D consistency for this new instance almost from scratch.

**Optimizing G only** (`render_xray_G.py`): By setting generator.eval(), only G and part of its parameters will be optimized. NLL doesn't play a role anymore in the reconstruction process. 3D consistency from G is preserved.

We consider optimizing G only the way to go (at least for now). This open new future experiments. Both rendering scripts if you'd like to try them. 

## PixelNeRF instructions
First, download the camera poses from [this link](https://drive.google.com/file/d/1xasjb7Q2nROiwDDeUVHrC6GjIEAm0HRs/view?usp=sharing). We referred to Plastimatch's [projection geometry](http://plastimatch.org/proj_geometry_15.html) and [Projection matrix](http://plastimatch.org/proj_mat_file_format.html) documentation to generated the camera matrices. To use pixelNeRF model download the following configuration files [here](https://drive.google.com/drive/folders/19AXTan3GAttwL-BLMwsnSC-AaXaminC4?usp=sharing) and place them in the following folders:

```
pixel-nerf/conf/exp/ct_single.conf
pixel-nerf/conf/exp/drr.conf
```

Also refer to the following script to render CT images:
```
pixel-nerf/eval/render_ct.py
```

## Generate DRR images from CT scans
To generate xrays images (.png) at different angles from CT scans use the script `generate_drr.py` under the folder `data/`. To run it you need to install the [Plastimatch's build](http://plastimatch.org/). Version 1.9.3 was used.

An updated version of the script has been added (generate_drr_multiple_dirs.py). Use this script to automatically generate the set of DRRs when you have a global folder with multiple folders containing CT scans. The files do not necessarily need to be in the immediate subfolder. You only need to assign the path location of the global folder and the global folder to save the set of DRRs.

### Overview of input arguments
Replace the following variables within the file:

- `input_path`: path to the .dcm files or .mha file of the CT.
- `save_root_path`: path where you want the xrays images to be saved. 
- `plasti_path`: path of the build. 
- `multiple_view_mode <True | False>`: generate single xrays from lateral or frontal views or multiple images from a circular rotation around the z axis.
    If False you need to specify the view with the argument `frontal_dir <True | False>` (false for lateral view).
    If True you need to specify `num_xrays` to generate equally spaced number of views and `angles` to input the difference between neighboring angles (in  degrees).
- `preprocessing <True | False>`: set this to True if files are .dcm for Hounsfield Units conversion. Set to False if given file is raw (.mha), for which       you need to provide its path under the variable `raw_input_file`.
- `detector_size`: pair of values in mm
- `bg_color`: choose either black or white background.
- `resolution`: size of the output xrays images.

## Acknowledgments

This codebase is heavily based on the [GRAF](https://github.com/autonomousvision/graf) code base. We also use the code from [pixel-nerf](https://github.com/sxyu/pixel-nerf) for baseline experiments.

We thank all authors for the wonderful code!

## Citation
If you use our model for your research, please cite the following work.

```bash
@misc{coronafigueroa2022mednerf,
      title={MedNeRF: Medical Neural Radiance Fields for Reconstructing 3D-aware CT-Projections from a Single X-ray}, 
      author={Abril Corona-Figueroa and Jonathan Frawley and Sam Bond-Taylor and Sarath Bethapudi and Hubert P. H. Shum and Chris G. Willcocks},
      year={2022},
      eprint={2202.01020},
      archivePrefix={arXiv},
      primaryClass={eess.IV}
}
```
