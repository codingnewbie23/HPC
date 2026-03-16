# Installing PyVista on Nibi HPC

PyVista requires VTK, which is not available as a pip wheel on Nibi.
Instead, VTK is provided as a system module and must be manually linked
to your venv using a `.pth` file.

## Step 1 — Activate your venv
```bash
source /home/$USER/envs/myvenv/bin/activate
```

## Step 2 — Link VTK to your venv
```bash
# Load VTK module
module load vtk

# Find where VTK's Python bindings are located
python -c "import vtk; print(vtk.__file__)"
# Example output:
# /cvmfs/.../vtk/9.4.2/lib/python3.11/site-packages/vtk.py

# Create a .pth file pointing to that DIRECTORY (not the file itself)
echo "/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v4/Compiler/gcc12/vtk/9.4.2/lib/python3.11/site-packages" \
    > /home/$USER/envs/myvenv/lib/python3.11/site-packages/vtk.pth

# Verify VTK is now visible to your venv
/home/$USER/envs/myvenv/bin/python -c "import vtk; print(vtk.__version__)"
```

## Step 3 — Install PyVista
```bash
# Install pyvista without vtk dependency resolution
/home/$USER/envs/myvenv/bin/pip3 install --no-index --no-deps pyvista

# Install pyvista's remaining dependencies
/home/$USER/envs/myvenv/bin/pip3 install --no-index --ignore-installed \
    pooch scooby imageio
```

## Step 4 — Verify
```bash
/home/$USER/envs/myvenv/bin/python -c "
import vtk; print('vtk:    ', vtk.__version__)
import pyvista; print('pyvista:', pyvista.__version__)
"
```

## Step 5 — OOD JupyterLab

Add `vtk` to the **Modules to load** field in the OOD session request
form. Without this, the `.pth` file will not resolve and pyvista will
fail to import in the notebook.

> **Note:** Ignore the version conflict warning
> `pyvista 0.44.2 requires vtk<9.4.0` — the import works fine despite this.
