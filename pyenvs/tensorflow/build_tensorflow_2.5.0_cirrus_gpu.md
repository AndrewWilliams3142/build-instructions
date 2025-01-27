Instructions for building a TensorFlow Miniconda3 environment on Cirrus (GPU)
=============================================================================

These instructions show how to build a TensorFlow Miniconda3 environment for the Cirrus GPU nodes
(Cascade Lake, NVIDIA Tesla V100-SXM2-16GB).

The TensorFlow build starts with [the instructions for building a base python environment](/pyenvs/gpu/build_miniconda3_cirrus_gpu.md);
these are provided by the `create.sh` script indicated below.

The `miniconda3/4.9.2-gpu-tensorflow` environment also includes the [Horovod](https://horovod.readthedocs.io/en/stable/index.html) package
required for running TensorFlow over multiple GPUs distributed across multiple compute nodes.


Setup initial environment
-------------------------

```bash
PRFX=/path/to/work  # e.g., PRFX=/lustre/sw
cd ${PRFX}

PYTHON_LABEL=py38
MINICONDA_LABEL=miniconda3
MINICONDA_VERSION=4.9.2
MINICONDA_ROOT=/lustre/sw/${MINICONDA_LABEL}/${MINICONDA_VERSION}-gpu
MINICONDA_ML_ROOT=${PRFX}/${MINICONDA_LABEL}/${MINICONDA_VERSION}-gpu-tensorflow

${MINICONDA_ROOT}/create.sh ${PRFX} tensorflow
. ${MINICONDA_ML_ROOT}/activate.sh

export PS1="(tensorflow) [\u@\h \W]\$ "

CUDA_VERSION=11.2
OPENMPI_VERSION=4.1.0
BOOST_VERSION=1.73.0
CMAKE_VERSION=3.17.3

module load nvidia/cuda-${CUDA_VERSION}
module load nvidia/mathlibs-${CUDA_VERSION}
module load openmpi/${OPENMPI_VERSION}-cuda-${CUDA_VERSION}
module load boost/${BOOST_VERSION}
module load cmake/${CMAKE_VERSION}
```

Remember to change the setting for `PRFX` to a path appropriate for your Cirrus project.


Install the machine learning packages
-------------------------------------

```bash
pip install wandb
pip install gym
pip install scikit-learn
pip install scikit-image

pip install tensorflow
pip install tensorflow-gpu
```


Install Horovod linking with the Nvidia Collective Communications Library (NCCL)
--------------------------------------------------------------------------------

```bash
NVIDIA_HPCSDK_ROOT=/lustre/sw/nvidia/hpcsdk-212/Linux_x86_64/21.2
CUDA_ROOT=${NVIDIA_HPCSDK_ROOT}/cuda/${CUDA_VERSION}
NCCL_ROOT=${NVIDIA_HPCSDK_ROOT}/comm_libs/${CUDA_VERSION}/nccl
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:${NCCL_ROOT}/lib

HOROVOD_CUDA_HOME=${CUDA_ROOT} HOROVOD_NCCL_HOME=${NCCL_ROOT} \
HOROVOD_GPU=CUDA HOROVOD_BUILD_CUDA_CC_LIST=70 \
HOROVOD_CPU_OPERATIONS=MPI HOROVOD_GPU_OPERATIONS=NCCL \
HOROVOD_WITH_MPI=1 HOROVOD_WITH_TENSORFLOW=1 \
HOROVOD_WITH_PYTORCH=0 HOROVOD_WITH_MXNET=0 \
    pip install --no-cache-dir horovod[tensorflow]
```

Now run `horovodrun --check-build` to confirm that [Horovod](https://horovod.readthedocs.io/en/stable/index.html) has been installed
correctly. That command should return something like the following output

```
2021-08-13 14:53:51.732625: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Successfully opened dynamic library libcudart.so.11.0
Horovod v0.22.1:

Available Frameworks:
    [X] TensorFlow
    [X] PyTorch
    [ ] MXNet

Available Controllers:
    [X] MPI
    [X] Gloo

Available Tensor Operations:
    [X] NCCL
    [ ] DDL
    [ ] CCL
    [X] MPI
    [X] Gloo 
```

The first line output by the build check may appear several times. Further, note that PyTorch is marked as an available framework; this looks
to be due to an error within `horovodrun` as none of the PyTorch packages are present in the TensorFlow environment.


Finish by deactivating the virtual environment
----------------------------------------------

```bash
conda deactivate
export PS1="[\u@\h \W]\$ "
```