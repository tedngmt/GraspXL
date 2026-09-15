# GraspXL on this computer

Installed in WSL distribution `Ubuntu` (Ubuntu 22.04), Conda environment
`graspxl`, using Python 3.8.10 and PyTorch 2.3.0 with CUDA 11.8.
The repository remains at `/mnt/c/Linux/GraspXL`.

## Run the demo

Start the Windows viewer by opening
`C:\Linux\GraspXL\raisimUnity\win32\RaiSimUnity.exe`.
This uses the NVIDIA GPU through Direct3D. The default Linux viewer crashes
on this WSL installation's Vulkan renderer; the bundled Linux OpenGL viewer
also has graphics/mesh-loading errors here.

The Windows viewer's resource directories now contain only GraspXL's MANO
meshes and `mixed_train` object folders; obsolete `artigrasp` entries were
removed. A backup of the previous resource settings is in
`build/viewer-resource-settings-backup.json`.

Open Ubuntu, then run:

```bash
conda activate graspxl
cd /mnt/c/Linux/GraspXL/raisimGymTorch
```

Keep the default localhost server address (`127.0.0.1`) and port `8080`.
Enable **Auto-connect** in the viewer, then run:

```bash
python raisimGymTorch/env/envs/ours_demo/demo.py
```

## Rebuild after changing an environment.hpp

```bash
conda activate graspxl
cd /mnt/c/Linux/GraspXL/raisimGymTorch
python setup.py develop --no-deps --CMAKE_PREFIX_PATH "$CONDA_PREFIX"
```

Dependencies were installed with pip. `--no-deps` avoids the legacy
`easy_install` dependency resolver in `setup.py develop`.
RaiSim libraries and its Python wrapper are installed inside the Conda
environment. Conda activation hooks configure its library search path.
The existing license is at `/home/nmt/.raisim/activation.raisim`.

## Validation

- All 12 compiled environment modules imported in separate processes.
- CUDA tensor computation passed on the RTX 3070 Ti Laptop GPU.
- RaiSim constructed a world and completed a simulation step.
- The MANO demo environment initialized successfully.
- The demo's `--help` command and `pip check` passed.
- The pretrained MANO demo completed all 25 episodes and saved 140-frame
  motion recordings with finite values and nonzero hand motion under
  `raisimGymTorch/data_all/diverse_seq_npy`.
- The Windows viewer initializes on the RTX 3070 Ti. Its old resource paths
  pointed at an `artigrasp` installation and were replaced with GraspXL paths.
  The user confirmed the demo displays correctly after this correction;
  changing the default server address was unnecessary.

### Viewer freeze fix

The bundled Windows viewer could loop forever when a socket read returned
zero bytes partway through a frame (for example, when the simulation closed
its connection). A local fix in `Assembly-CSharp.dll` now reports the
disconnect through the viewer's existing error handler.

The original assembly reproduced the hang in a controlled test. The fixed
assembly passed normal-frame, interrupted-frame, and timeout tests. The
original DLL is backed up at `build/Assembly-CSharp.original.dll`; the patch
and test source are in `build/viewer-fix`.

The demo still ends normally after 25 sequences. Re-run the Python command
to generate another set; a finished demo is distinct from an unresponsive
viewer window.

## Optional requirements

### MANO training: installed

- `manotorch` 0.0.2 is installed from `/home/nmt/manotorch`, with its dependencies
  and the training scripts' `wandb` dependency.
- Existing models from `C:\Linux\GRAB\grab\mano_v1_2.zip` were extracted to
  `/home/nmt/manotorch/assets/mano`.
- `mano_amano.py` now expands the home-directory path correctly. Set
  `MANO_ASSETS_ROOT` to override that location if needed.
- GPU forward computation, GraspXL pose conversion, anatomy loss, and both
  MANO training scripts' `--help` entry points passed. Full training was not run.

To train without connecting to a Weights & Biases account:

```bash
conda activate graspxl
cd /mnt/c/Linux/GraspXL/raisimGymTorch
WANDB_MODE=offline python raisimGymTorch/env/envs/ours_fixed/runner.py
```

### Objaverse: 100-object subset installed

The official archive is now public at
https://huggingface.co/datasets/ethHuiZhang/GraspXL.
A working subset is installed at `rsc/surdf_group10_l`: 100 objects,
800 files including 200 floating/fixed-base URDF variants. All mesh references
were checked, and a sample object loaded successfully in GraspXL.
The full archive is approximately 49 GB and was not downloaded.

With the Unity viewer running and Auto-connect enabled:

```bash
conda activate graspxl
cd /mnt/c/Linux/GraspXL/raisimGymTorch
python raisimGymTorch/env/envs/ours_demo/objaverse_demo.py -group 10 -mode 2
```

### ShapeNet: installed

The supplied `ShapeNet.zip` was extracted into `rsc/large_scale_obj`.
It contains 3,992 objects, 7,984 URDFs, and 35,957 files in total
(approximately 5.2 GB uncompressed). The original ZIP is retained.

All extracted files passed ZIP checksum validation. All 23,952 URDF mesh
references resolve, and a sample ShapeNet object loaded successfully in
GraspXL. Full ShapeNet evaluation was not run.
