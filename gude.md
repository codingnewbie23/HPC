# Python Virtual Environment & JupyterLab Setup on Nibi HPC

This guide covers everything from creating a virtual environment to installing
PyVista and using your environment in JupyterLab via Open OnDemand (OOD).

---

## Prerequisites

- Active Alliance Canada account with Nibi access
- SSH access to Nibi: `ssh <username>@nibi.sharcnet.ca`
- Access to the Nibi OOD portal for JupyterLab

---

## Step 1 — Create your virtual environment

```bash
# Load the python module
module load python

# Create your venv (replace myvenv with your preferred name)
python -m venv /home/$USER/envs/myvenv

# Activate it
source /home/$USER/envs/myvenv/bin/activate
```

> **Important:** Always use the full venv pip path for all installs to
> ensure packages go into your venv and not the system environment.

---

## Step 2 — Install core packages

```bash
/home/$USER/envs/myvenv/bin/pip3 install --no-index --ignore-installed \
    ipykernel matplotlib numpy scipy pandas h5py
```

- `--no-index` tells pip to use Nibi's local wheelhouse instead of PyPI
- `--ignore-installed` forces packages to install into your venv even if
  they exist in the system environment

---

## Step 3 — Install PyVista and VTK

PyVista requires VTK, which is not available as a pip wheel on Nibi.
VTK is provided as a system module and must be manually linked to your
venv using a `.pth` file.

### 3a — Link VTK to your venv

```bash
# Load VTK module
module load vtk

# Find where VTK's Python bindings are located
python -c "import vtk; print(vtk.__file__)"
# Example output:
# /cvmfs/.../vtk/9.4.2/lib/python3.11/site-packages/vtk.py

# Create a .pth file pointing to that DIRECTORY (not the file itself)
# Replace the path below with the directory from the output above
echo "/cvmfs/soft.computecanada.ca/easybuild/software/2023/x86-64-v4/Compiler/gcc12/vtk/9.4.2/lib/python3.11/site-packages" \
    > /home/$USER/envs/myvenv/lib/python3.11/site-packages/vtk.pth

# Verify VTK is now visible to your venv
/home/$USER/envs/myvenv/bin/python -c "import vtk; print(vtk.__version__)"
```

### 3b — Install PyVista

```bash
# Install pyvista without vtk dependency resolution
/home/$USER/envs/myvenv/bin/pip3 install --no-index --no-deps pyvista

# Install pyvista's remaining dependencies
/home/$USER/envs/myvenv/bin/pip3 install --no-index --ignore-installed \
    pooch scooby imageio
```

> **Note:** Ignore the version conflict warning
> `pyvista 0.44.2 requires vtk<9.4.0` — the import works fine despite this.

---

## Step 4 — Register your venv as a Jupyter kernel

This step makes your venv available as a kernel in JupyterLab.
The `--user` flag installs the kernel spec to `~/.local/share/jupyter/kernels/`
which is where the OOD JupyterLab looks for kernels at launch.

```bash
python -m ipykernel install --user \
    --name=myvenv \
    --display-name "Python (myvenv)"

# Verify the kernel is registered
jupyter kernelspec list
# Should show myvenv under /home/$USER/.local/share/jupyter/kernels/myvenv
```

---

## Step 5 — Verify everything works

```bash
/home/$USER/envs/myvenv/bin/python -c "
import vtk;        print('vtk:       ', vtk.__version__)
import pyvista;    print('pyvista:   ', pyvista.__version__)
import matplotlib; print('matplotlib:', matplotlib.__version__)
import numpy;      print('numpy:     ', numpy.__version__)
import scipy;      print('scipy:     ', scipy.__version__)
import pandas;     print('pandas:    ', pandas.__version__)
import h5py;       print('h5py:      ', h5py.__version__)
"
```

---

## Step 6 — Launch JupyterLab via OOD

1. Go to the **Nibi OOD portal**
2. Click **Compute Node** → **Nibi JupyterLab**
3. Fill in the session request form:
   - Add `vtk` to the **Modules to load** field
   - Set your required CPU, memory, and time
4. Click **Launch**
5. Once status changes to **Running**, click **Connect to Jupyter**
6. Open your notebook → top right corner → **Change Kernel**
7. Select **"Python (myvenv)"**

> **Important:** `vtk` must be in the Modules to load field, otherwise
> the `.pth` file will not resolve and pyvista will fail to import.

---

## Step 7 — Verify in the notebook

Run this in a cell to confirm the correct environment is active:

```python
import sys
print(sys.executable)
# Should print: /home/<username>/envs/myvenv/bin/python

import subprocess
result = subprocess.run([sys.executable, '-m', 'pip', 'list'],
                       capture_output=True, text=True)
print(result.stdout)
# Should show your packages with their versions
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `pip` installs to system instead of venv | Always use `/home/$USER/envs/myvenv/bin/pip3` |
| Packages resolve to `/cvmfs/` not venv | Add `--ignore-installed` flag |
| `No module named pip` in venv | Run `python -m ensurepip --upgrade` |
| VTK not found by pip | Use `.pth` file to link system module (Step 3) |
| PyVista vtk version conflict warning | Safe to ignore if import succeeds |
| Notebook shows wrong kernel packages | Stop OOD session fully and relaunch |
| `mpi4py` / `h5py` install error | Load `mpi4py` module before activating venv |

---

## Quick Reference — Key Commands

```bash
# Activate venv
source /home/$USER/envs/myvenv/bin/activate

# Always install using full venv pip path
/home/$USER/envs/myvenv/bin/pip3 install --no-index --ignore-installed <package>

# Check what's installed in the venv
/home/$USER/envs/myvenv/bin/python -m pip list

# Check available system modules
module spider <package>

# List registered Jupyter kernels
jupyter kernelspec list
```
