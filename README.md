# StreetCrafter: Street View Synthesis with Controllable Video Diffusion Models

### [Project Page](https://zju3dv.github.io/street_crafter) | [Paper](https://arxiv.org/abs/2412.13188)

> StreetCrafter: Street View Synthesis with Controllable Video Diffusion Models  
> [Yunzhi Yan*](https://yunzhiy.github.io/), [Zhen Xu*](https://zhenx.me/), [Haotong Lin](https://haotongl.github.io/), [Haian Jin](https://haian-jin.github.io/), [Haoyu Guo](https://github.com/ghy0324), [Yida Wang](https://wangyida.github.io/), Kun Zhan, Xianpeng Lang, [Hujun Bao](http://www.cad.zju.edu.cn/home/bao/), [Xiaowei Zhou](https://www.xzhou.me/), [Sida Peng](https://pengsida.net/)<br>
> CVPR 2025

https://github.com/user-attachments/assets/1f5fafb4-bf91-480b-be78-2183d1f347b6



### Installation

#### Clone this repository
```
git clone https://github.com/zju3dv/street_crafter.git --recursive
```

#### Set up the environment

Our model is tested on one A100/A800 80GB GPU.

```
conda create -n streetcrafter python=3.9
conda activate streetcrafter

# Install dependencies.
pip install torch==2.4.0 torchvision==0.19.0 torchaudio==2.4.0 --index-url https://download.pytorch.org/whl/cu121

# Install requirements
pip install -r requirements.txt 

# Install gsplat
pip install "git+https://github.com/dendenxu/gsplat.git" 
# This issue might help when installation fails: https://github.com/nerfstudio-project/gsplat/issues/226

# Install submodules
pip install ./submodules/sdata
pip install ./submodules/simple-knn
```


### Data Processing
Please go to `data_processor` and refer to [README.md](data_processor/README.md) for processing details.
We provide some example scenes on this [link](https://drive.google.com/drive/folders/1a9RirdkWONZ6DUNXEo_wUk-yefM5ryEd?usp=drive_link). You can skip the processing steps and download the data to `data/waymo` directory.


### Model Weights
The pretrained model weights can be downloaded from this [link](https://drive.google.com/file/d/1Qtdkm0wvIUSMWQMVldd-d16rHZsNFFt1/view?usp=drive_link) to `video_diffusion/ckpts` directory. We also provide the model weights trained using multi-cameras of Waymo under this [link](https://drive.google.com/file/d/1GUZw4s2-B9KmUWYNduHa-ur5kVciOyTI/view?usp=drive_link).


### Inference

Inference video diffusion model
```
python render.py --config {config_path} mode diffusion
```

We also provide another option for inference by setting the meta info file path.
```
# run the command under video diffusion directory
python sample_condition.py  
```

### Distillation

We distill the video diffusion model into dynamic 3D representation based on the codebase of [Street Gaussians](https://zju3dv.github.io/street_gaussians/). Please refer to `street_gaussian/config/config.py` for details of parameters.

<details> 
<summary>Training tips</summary>

  
#### Breakpoint
In the EasyVolCap framework, if you need to add breakpoints to the code, please use `breakpoint()`.

#### Sky Model

The initially released code contains a bug where the background color is set to black for all scenes.  
The default sky representation has been changed to cubemap to align with the paper,.  
However, the cubemap representation may cause artifacts in novel view synthesis, particularly in regions where the sky mask is inaccurate, such as cables.  
For these scenes, there are two alternative options:  
- **Option 1**: Set `model.nsg.include_sky` to `True` for scenes with large sky areas.  
  In this case, the sky will be represented by a separate Gaussian model located outside a predefined sphere.  
- **Option 2**: Set `model.nsg.include_cube_map` to `False` for scenes with small sky areas.  
  In this case, the sky will be integrated into the background Gaussian model.
  
Remember that you need to set the `data.white_background` according to whether each scene is daytime or nighttime.

</details>


Train street gaussian
```
python train.py --config {config_path} 
```

Render input trajectory
```
python render.py --config {config_path} mode trajectory
```

Render novel trajectory
```
python render.py --config {config_path} mode novel_view
```


### Training 
First download the model weights of Vista from this [link](https://drive.google.com/file/d/1bCM7XLDquRqnnpauQAK5j1jP-n0y1ama/view) to `video_diffusion/ckpts` directory. 
We finetune the video diffusion model based on the codebase of [Vista](https://opendrivelab.com/Vista/). Please refer to their official [Documents](video_diffusion/docs/) for environment setup and training details.
```
# run the command under video diffusion directory
sh training.sh
```



### Overview
![pipeline](assets/pipeline.png)
(a) We process the LiDAR using calibrated images and object tracklets to obtain a colorized point cloud, which can be rendered to image space as pixel-level conditions. 
(b) Given observed images and reference image embedding $\mathbf{c}_\text{ref}$, we optimize the video diffusion model conditioned on the LiDAR renderings to perform controllable video generation. 
(c) Starting from the rendered images and LiDAR conditions under novel trajectory, we use the pretrained controllable video diffusion model to guide the optimization of the dynamic 3DGS representation by generating novel views as extra supervision signals. 

### Citation

If you find this code useful for your research, please use the following BibTeX entry.

```
@inproceedings{yan2024streetcrafter,
  title={StreetCrafter: Street View Synthesis with Controllable Video Diffusion Models},
  author={Yan, Yunzhi and Xu, Zhen and Lin, Haotong and Jin, Haian and Guo, Haoyu and Wang, Yida and Zhan, Kun and Lang, Xianpeng and Bao, Hujun and Zhou, Xiaowei and Peng, Sida},
  booktitle={Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
  year={2025},
}
```
