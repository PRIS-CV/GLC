# GLC: Unsupervised Domain Adaptation for Semantic Segmentation with Global and Local Consistency

## Paper
![](https://github.com/samsxx/GLC/blob/master/Figure1.png)

[GLC: Unsupervised Domain Adaptation for Semantic Segmentation with Global and Local Consistency](https://link.springer.com/chapter/10.1007/978-3-031-20497-5_13)  
Xiangxuan Shan, Zijin Yin, Jiayi Gao, Kongming Liang, Zhanyu Ma, and Jun Guo  
School of Artificial Intelligence, Beijing University of Posts and Telecommunications  
CAAI International Conference on Artificial Intelligence (CICAI), 2022

If you find this code useful for your research, please cite our paper:

```
@inproceedings{shan2022unsupervised,
  title={Unsupervised Domain Adaptation for Semantic Segmentation with Global and Local Consistency},
  author={Shan, Xiangxuan and Yin, Zijin and Gao, Jiayi and Liang, Kongming and Ma, Zhanyu and Guo, Jun},
  booktitle={CAAI International Conference on Artificial Intelligence},
  pages={154--165},
  year={2022},
  organization={Springer}
}
```
## Overview
As illustrated in the paper, our code is divided into two parts to achieve global and local consistency separately. Thanks to the outstanding works of [PCEDA](https://github.com/donglao/PCEDA) and [ADVENT](https://github.com/valeoai/ADVENT), we can achieve it easily. To achieve the same results as we did, you need to first follow PCEDA to ensure global consistency and then follow ADVENT to ensure local consistency. The details are as follows.

## Global Consistency
### Environments
First of all, you need to follow the guidance in [CyCADA](https://github.com/jhoffman/cycada_release) just as PCEDA says. You need to install the packages in `requirements.txt` from CyCADA.

### Datasets
Please put [GTA5](https://download.visinf.tu-darmstadt.de/data/from_games/) to the folders `trainA` and put [Cityscapes](https://www.cityscapes-dataset.com/) to the folders `trainB`. You can put them in `PCEDA_Phase/datasets/data_semseg/` or other folders if you are willing to change the training command. 

### Train
Please go to the folder named `PCEDA_Phase` and execute the following command for training the image transformation just as PCEDA says.
```
python3 train.py --dataroot='./datasets/data_semseg' --gpu_ids='0' --model='cycle_gan' --display_freq=100 --display_ncols=2 --save_latest_freq=2000 --save_epoch_freq=1 --save_by_iter --niter=5 --niter_decay=15 --batch_size=1 --soft_phase=True --norm='instance' --normGNET='instance' --netG='resnet_9blocks' --display_port=18099 --name='gta2city' --lambda_A=10.0 --lambda_B=10.0 --lambda_identity=5.0 --lambda_P=50000.0 --lr=1e-5 --lrG=5e-5
```

### Exporting transformed images
First comment out Line 41 in the dataloader `PCEDA_Phase/data/unaligned_dataset.py` to disable cropping and then execute the following command:
```
python3 export_images.py --dataroot='./datasets/data_semseg' --phase='train' --soft_phase=True --model='cycle_gan' --netG='resnet_9blocks' --norm='instance' --normGNET='instance' --name='gta2city' --load_iter=10000 --num_test=20 --results_dir='./results/gta2city'
```

Now you have the transformed images in `./results/gta2city` that have the similar style information with target real images and have the same semantic information with source synthetic images. So the global consistency is ensured.

## Local Consistency
### Environments
Please follow the 'Pre-requisites', 'Installation' and 'Pre-trained models' in [ADVENT](https://github.com/valeoai/ADVENT) to install the new environment.

### Datasets
You need to put the transformed images and the labels of GTA5 in `./data/GTA5/images/` and `./data/GTA5/labels/` and put the images and labels of Cityscapes in `./data/Cityscapes/leftImg8bit/` and `./data/Cityscapes/gtFine`. You can also change the data dir in `./advent/domain_adaptation/config.py` to avoid the trouble of moving data.

### Train
Please go to the folder named `./advent/scripts/` and execute the following command for training the semantic segmentation.
```
python train.py --cfg ./configs/advent.yml --tensorboard
```  
The logs and snapshots will be stored in `./experiments/`.

### Testing
To test ADVENT:
```
python test.py --cfg ./configs/advent.yml
```

## Acknowledgements
This codebase is heavily borrowed from [PCEDA](https://github.com/donglao/PCEDA) and [ADVENT](https://github.com/valeoai/ADVENT).
