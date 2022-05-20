# Automated Brain Extraction

### [3D U-Net](https://github.com/neuronets/nobrainer/blob/master/nobrainer/models/unet.py)

This model achieved a median Dice score of 0.97, mean of 0.96, minimum of 0.91, and maximum of 0.98 on a validation dataset of 99 T1-weighted brain scans and their corresponding binarized FreeSurfer segmentations (public and private sources). This model should be agnostic to orientation and can predict the brainmask for a volume of size 256x256x256 in approximately three seconds.

This model was trained for five epochs on a dataset of 10,000 T1-weighted brain scans comprised of both public and private data. The ground truth labels were binarized FreeSurfer segmentations (i.e., binarized `aparc+aseg.mgz`). All volumes were size 256x256x256 and had 1mm isotropic voxel sizes. Because the full volumes could not fit into memory during training, each volume was separated into non-overlapping blocks of size 128x128x128, and training was performed on blocks in batches of two. The Jaccard loss function was used. During training, approximately 50% of the volumes were augmented with random rigid transformations. That is to say, 50% of the data was rotated and translated randomly in three dimensions (the transformation for pairs of features and labels was the same). The augmented features were interpolated linearly, and the augmented labels were interpolated with nearest neighbor. Each T1-weighted volume was standard scored (i.e., Z-scored) prior to training.

![Predicted brain mask on T1-weighted brain scan](/images/unet-best-prediction.png)

![Predicted brain mask on T1-weighted brain scan with motion](/images/unet-worst-prediction.png)

```diff
! CAUTION: this tool is not a medical product and is only intended for research purposes. !
```

## Requirements for use

- Input data should be T1-weighted magnetic resonance images. The model was trained and evaluated on this type of data.
- CPU or, optionally, GPU
  - Runtime will be much faster when a GPU is available. Please see the Singularity usage example below for running the model on a GPU.

## Singularity usage example

To run using singularity, first pull the image:

```
singularity pull docker://neuronets/brainy:latest-gpu
```

You have a few options when running the image. To see them call help.

```
singularity run --cleanenv --bind $(pwd):/data -W /data --nv brainy_latest-gpu.sif --help
```

Here is an example.

```
singularity run --cleanenv --bind $(pwd):/data -W /data --nv \
    brainy_latest-gpu.sif T1_001.nii.gz output
```

This will generate two files `output.nii.gz` and `output_orig.nii.gz`. The first is the mask in conformed FreeSurfer space. The second is the mask in the original input space.

## Docker usage example

Instead of singularity with GPU, once can also use Docker directly. This is an example with a CPU. Note that the CPU-based run is significantly slower.

```
docker run -it --rm -v $(pwd):/data --user 1001:1000 neuronets/brainy:latest-cpu T1_001.nii.gz output
```

The above examples assume there is a file named `T1_001.nii.gz` in the working directory. The option `--user 1001:1000` runs the container with user ID 1001 and group ID 1000. This will prevent the created files from being owned by root. Replace the IDs with your own IDs. On Unix-like systems, one can use `id -u` and `id -g` to get user ID and group ID, respectively.

### DockerHub tags

The DockerHub tags follow the following naming scheme:

- `master-gpu`: gpu version of current GitHub master
- `latest-gpu`: gpu version of latest release
- `SEMVER-gpu`: gpu version of semantically versioned release

for `cpu` versions replace `gpu` with `cpu`

# nobrainer

This model is based on the nobrainer framework. The original model is publicly available and can be found on [neuronets/trained-models](https://github.com/neuronets/trained-models#3d-u-net).

The pre-trained model can be downloaded via `datalad`.
