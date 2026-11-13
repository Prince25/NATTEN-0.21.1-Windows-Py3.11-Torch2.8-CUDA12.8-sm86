# Pre-Compiled NATTEN 0.21.1 Wheel for Windows (`win_amd64` | Python 3.11 | PyTorch 2.8.0+cu128 | RTX 30-Series / SM 8.6)

I wanted to use [Pixal3D](https://github.com/TencentARC/Pixal3D) inside ComfyUI. I found the package [**ComfyUI-Trellis2**](https://github.com/visualbruno/ComfyUI-Trellis2) package but had many issues installing dependencies, specifically Natten. I bit the bullet and build the wheel myself. Thus, here is a pre-built Windows binary wheel (`.whl`) for [**NATTEN 0.21.1**](https://github.com/SHI-Labs/NATTEN) (Neighborhood Attention Extension) with `libnatten` CUDA kernels compiled and enabled. Built specifically for Windows users running a NVIDIA RTX 30-series (Ampere) GPUs with Python 3.11, PyTorch 2.8.0, and CUDA 12.8.

**--> [Direct Download](https://github.com/Prince25/NATTEN-0.21.1-Windows-Py3.11-Torch2.8-CUDA12.8-sm86/raw/refs/heads/main/natten-0.21.1-cp311-torch28-cu128-sm86-win64.whl) <--**

---

## Compatibility & Build Specifications

Because PyTorch C++ and CUDA extensions link directly against specific runtime libraries and GPU compute architectures, your environment must match these specifications:

| Specification | Requirement / Build Target |
| :--- | :--- |
| **Operating System** | Windows 64-bit (`win_amd64`) |
| **NATTEN Version** | `0.21.1` |
| **Python Version** | Python 3.11 (`cp311`, built against CPython 3.11.13) |
| **PyTorch Version** | `2.8.0+cu128` (requires PyTorch `2.8.x` built with CUDA 12.8) |
| **CUDA Version** | CUDA 12.8 (`cu128`) |
| **GPU Architecture** | Compute Capability `8.6` (`SM 8.6` / Ampere: RTX 3050, 3060, 3070, 3080, 3090) |
| **Compiled Kernels** | `libnatten` enabled (`fmha`, `fna` forward/backward, and `reference` kernels) |

> **Note on GPU Support:** This wheel was compiled exclusively for `SM 8.6` (RTX 30-series). Users with RTX 20-series (`SM 7.5`), RTX 40-series (`SM 8.9`), or RTX 50-series (`SM 12.0`) GPUs will encounter a `CUDA error: no kernel image is available for execution on the device` at runtime.

---

## Installation

1. Download the `.whl` file (`natten-0.21.1-cp311-torch28-cu128-sm86-win64.whl`).
2. Activate your ComfyUI virtual environment (or target your StabilityMatrix / embedded Python executable) and run:
   ```bash
   pip install natten-0.21.1-cp311-torch28-cu128-sm86-win64.whl
   ```
3. Verify that Python loads `natten` along with the compiled CUDA extension (`HAS_LIBNATTEN` should return `True`):
   ```bash
   python -c "import torch, natten; print(torch.__version__, natten.__version__, natten.HAS_LIBNATTEN)"
   ```

---

## Why This Wheel Was Built From Source

### 1. Missing Pre-Built Windows Binaries for This Stack
Official NATTEN wheel releases primarily target Linux and specific upstream PyTorch/CUDA combinations, leaving Windows users on Python 3.11 and PyTorch `2.8.0+cu128` without a drop-in binary wheel. Custom ComfyUI nodes such as **ComfyUI-Trellis2** (for Microsoft's TRELLIS.2) require NATTEN, making a local source build necessary.

### 2. Build Fixes Applied During Compilation
Compiling NATTEN `0.21.1` from source on Windows using MSVC (`cl.exe`), NVCC (`nvcc.exe`), CMake, and Ninja required resolving two Windows-specific build issues:

* **MSVC C++ Alternative Keyword Error (`helpers.h`, line 28):** In `csrc/include/natten/helpers.h` at line 28, a `TORCH_CHECK` macro uses the C++ alternative operator token `not` (`not x.is_sparse()`). On Windows, MSVC (`cl.exe`) does not enable alternative operator keywords (`not`, `and`, `or`) by default without `/permissive-` or `<ciso646>`, causing compilation of the reference CUDA kernels (`autogen/src/cuda/reference/source_*.cu`) to fail with `error: identifier "not" is undefined`. Updating line 28 to:
  ```cpp
  TORCH_CHECK(!x.is_sparse(), #x " must be a dense tensor");
  ```
  resolved the compilation error cleanly while keeping all `reference`, `fna`, and `fmha` kernels intact for the final linker step.
* **Missing `python311.lib` Linker Path in Virtual Environments:** Windows Python virtual environments (including StabilityMatrix-managed environments) do not copy the `libs/` directory from the base Python installation into `venv\Scripts\libs`. During the final link stage (`link.exe`), CMake failed with `LINK : fatal error LNK1104: cannot open file 'python311.lib'`. Copying the `libs/` folder from the base CPython 3.11 installation into `venv\Scripts\libs` (or appending it to the `LIB` environment variable) allowed the `.pyd` extension to link properly.

---

## Attributions & Upstream Projects

All credit for the underlying software, CUDA kernels, and ComfyUI integrations belongs to their respective authors and maintainers:

* **NATTEN (Neighborhood Attention Extension):** Developed by [SHI-Labs](https://github.com/SHI-Labs) ([SHI-Labs/NATTEN](https://github.com/SHI-Labs/NATTEN) | [natten.org](https://natten.org)) and authored by Ali Hassani et al., released under the MIT License.
* **ComfyUI-Trellis2:** ComfyUI custom node wrapper ([visualbruno/ComfyUI-Trellis2](https://github.com/visualbruno/ComfyUI-Trellis2)) for Microsoft's [TRELLIS.2](https://github.com/microsoft/TRELLIS.2) 3D generation framework.
* **ComfyUI:** Node-based generative AI interface by [Comfy-Org/ComfyUI](https://github.com/Comfy-Org/ComfyUI).
* **StabilityMatrix:** Multi-platform package and environment manager by [LykosAI/StabilityMatrix](https://github.com/LykosAI/StabilityMatrix).
* **PyTorch & NVIDIA CUDA Toolkit:** Built against [PyTorch 2.8.0](https://pytorch.org/) and the [NVIDIA CUDA Toolkit 12.8](https://developer.nvidia.com/cuda-12-8-0-download-archive).