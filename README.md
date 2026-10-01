# Liver Lesion Segmentation XNAT Container

This repository holds the runtime code that gets embedded into a custom XNAT bootstrap image built with `pymipl/build_deploy_custom_image.sh`.

The workflow mirrors the design of the existing `Colon-Mets-Liver-Segmentation-Toolbox-20260430` project:

1. detect structural CT scans in the mounted XNAT session
2. prepare a temporary nnUNet input directory for each scan
3. create a symlink named `liver_<scan_no>_0000`
4. run `nnUNetv2_predict -i I -o O ...` against `/data/nnunet/results/Dataset006_Liver`
5. rename the output from `liver_<scan_no>.nii.gz` to `liver_tumor_mask_<scan_no>.nii.gz`
6. convert the mask to DICOM RTSTRUCT and upload it back to XNAT
7. upload the session output bundle to the resource named `LIVER-LESION_SEG_OUT`

## Build model

This repo is not a standalone Docker build project. It is the runtime source that is copied into the image built by `pymipl/build_deploy_custom_image.sh` using the standard `xnat_env_bootstrap/Dockerfile.custom` pattern.

A build config file like the following should be used:

```bash
REMOTE_IMAGE=mmilch01/liver-lesion-seg:latest
LOCAL_IMAGE="$REMOTE_IMAGE"
DOCKER_REPO=docker.io
LOCAL_ENV_FOLDER="/opt/packages/user/env_repo"
LOCAL_ALG_FOLDER="/home/mmilchenko/src/liver-lesion-seg"
LOCAL_RUNTIME_FOLDER="/home/mmilchenko/src/liver-lesion-seg"
ENV_VARS="USER=xnat LOGNAME=xnat nnUNet_raw=/opt/nnunet/raw nnUNet_preprocessed=/opt/nnunet/preprocessed nnUNet_results=/opt/nnunet/results"
```

## Runtime assumptions

- CT only
- output resource name: `LIVER-LESION_SEG_OUT`
- RTSTRUCT series number: `structural_scan_number + 101`
- labels: `LIVER = 1`, `LESION = 2`
