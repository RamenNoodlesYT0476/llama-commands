# `llama-cli` Installation & Usage Guide

[![GitHub License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Platform Support](https://img.shields.io/badge/platform-macOS%20%7C%20Windows%20%7C%20Linux-lightgrey.svg)](#installation)
[![llama.cpp](https://img.shields.io/badge/powered%20by-llama.cpp-orange.svg)](https://github.com/ggml-org/llama.cpp)

A comprehensive guide for installing, configuring, and running local Large Language Models (LLMs) using `llama-cli` (`llama.cpp`) across **macOS**, **Windows**, and **Linux**.

---

## 📋 Table of Contents
- [Overview](#overview)
- [Installation](#installation)
  - [macOS](#1-macos-installation)
  - [Windows](#2-windows-installation)
  - [Linux](#3-linux-installation)
- [Quick Start](#quick-start)
- [CLI Command Reference](#cli-command-reference)
- [Hardware Performance Optimization](#hardware-performance-optimization)
- [Troubleshooting/Bugs](#troubleshooting/bugs)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

`llama-cli` is the command-line interface provided by the [`llama.cpp`](https://github.com/ggml-org/llama.cpp) ecosystem. It allows you to run quantized GGUF format models directly on consumer hardware with zero network latency and complete privacy.

---

## Installation

### 1. macOS Installation

The fastest method on macOS is using **Homebrew**, which automatically compiles `llama.cpp` with native **Apple Silicon Metal GPU acceleration**.

#### Prerequisites
- macOS 12.0 (Monterey) or newer
- Terminal (or standard alternatives like iTerm2 / Warp)

#### Step 1: Install Homebrew (if not installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### Step 2: Install `llama.cpp`
```bash
brew install llama.cpp
```

*Verification:*
```bash
llama-cli --version
```

---

### 2. Windows Installation

#### Option A: Windows Package Manager (Easiest)
1. Open **PowerShell** or **Command Prompt** as Administrator.
2. Run the installation command:
   ```powershell
   winget install llama.cpp
   ```
3. Restart your terminal window and verify:
   ```powershell
   llama-cli --version
   ```

#### Option B: Build from Source with CUDA (Best for NVIDIA GPUs)
1. **Install Prerequisites:**
   - [Git](https://git-scm.com/)
   - [CMake](https://cmake.org/)
   - [Visual Studio 2022](https://visualstudio.microsoft.com/) with **Desktop Development with C++** workload
   - [NVIDIA CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit)

2. **Clone & Compile:**
   ```powershell
   git clone https://github.com/ggml-org/llama.cpp
   cd llama.cpp
   cmake -B build -DGGML_CUDA=ON
   cmake --build build --config Release
   ```
   *The binary will be built at `.\build\bin\Release\llama-cli.exe`.*

---

### 3. Linux Installation

#### Option A: Package Manager
- **Homebrew for Linux:**
  ```bash
  brew install llama.cpp
  ```
- **Nix:**
  ```bash
  nix profile install nixpkgs#llama-cpp
  ```

#### Option B: Build from Source (Recommended for Dedicated GPUs)
1. **Install Build Dependencies (Debian / Ubuntu):**
   ```bash
   sudo apt update && sudo apt install -y build-essential cmake git
   ```

2. **Clone Repository:**
   ```bash
   git clone https://github.com/ggml-org/llama.cpp
   cd llama.cpp
   ```

3. **Build Binary Based on Hardware Acceleration:**

   | Target Hardware | Build Command |
   | :--- | :--- |
   | **CPU Only** | `cmake -B build && cmake --build build --config Release -j$(nproc)` |
   | **NVIDIA GPU (CUDA)** | `cmake -B build -DGGML_CUDA=ON && cmake --build build --config Release -j$(nproc)` |
   | **AMD GPU (ROCm/HIP)** | `cmake -B build -DGGML_HIPBLAS=ON && cmake --build build --config Release -j$(nproc)` |
   | **Universal GPU (Vulkan)** | `cmake -B build -DGGML_VULKAN=ON && cmake --build build --config Release -j$(nproc)` |

4. **Add Executable to Path (Optional):**
   ```bash
   sudo cp build/bin/llama-cli /usr/local/bin/
   ```

---

## Quick Start

You can run models directly from [Hugging Face](https://huggingface.co/) without manually downloading file weights first using the `-hf` flag:

```bash
llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c 4096 -cnv
```

To run a locally downloaded `.gguf` file:

```bash
llama-cli -m /path/to/your/model.gguf -c 4096 -cnv
```

---

## CLI Command Reference

Below is a breakdown of the core command-line flags accepted by `llama-cli`:

| Flag | Short / Long Name | Parameter | Description |
| :--- | :--- | :--- | :--- |
| `-c` | `--ctx-size` | `INTEGER` | Sets context window length in tokens (e.g., `-c 4096`). |
| `-b` | `--batch-size` | `INTEGER` | Sets logical batch size for prompt evaluation/prefill. |
| `-ub` | `--ubatch-size` | `INTEGER` | Sets physical compute micro-batch size for low-VRAM limits. |
| `-t` | `--threads` | `INTEGER` | Sets active CPU threads assigned to evaluation (match physical CPU cores). |
| `-ngl` | `--n-gpu-layers` | `INTEGER` | Sets the number of model layers to offload to GPU VRAM (e.g., `-ngl 99` for all). |
| `-cnv` | `--conversation` | *None* | Enables interactive conversation/chat mode in the terminal. |
| `-hf` | `--hf-repo` | `STRING` | Automatically fetches GGUF models directly from Hugging Face repository endpoints. |
| `-m` | `--model` | `PATH` | Direct filepath to a locally stored `.gguf` model file. |


General command: llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c # -b # -ub # -t # -cnv

Optimized command for MacBook Neo: llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c 8192 -b 128 -ub 64 -t 6 -cnv

---

## Hardware Performance Optimization

### Apple Silicon (M1 / M2 / M3 / M4)
Apple Silicon chips utilize **Unified Memory** with automatic Metal acceleration. Set CPU thread count (`-t`) equal to physical performance cores:

```bash
llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c 4096 -t 6 -cnv
```

### Discrete GPUs (NVIDIA / AMD)
To force all model layers onto your dedicated GPU VRAM, set the GPU offload count (`-ngl`) to a high number such as `99` (or `999`):

```bash
llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c 8192 -ngl 99 -cnv
```

---

## Troubleshooting/Bugs

<details>
<summary><b>1. Out of Memory (OOM) Errors or CUDA Allocation Failures</b></summary>

- Lower the context size (`-c 4096` to `-c 2048`).
- Reduce physical micro-batch size (`-ub 256` or `-ub 128`).
- Lower the number of offloaded GPU layers (`-ngl`) if your VRAM cannot fit the entire model.
</details>

<details>
<summary><b>2. Slow Inference Speed on Apple Silicon</b></summary>

- Ensure you built or installed `llama.cpp` with Metal enabled (Homebrew does this automatically).
- Do not set thread count (`-t`) higher than your physical CPU **Performance** core count (hyperthreads degrade token generation rate).
</details>

<details>
<summary><b>3. Command Not Found: llama-cli</b></summary>

- On Linux/macOS, ensure `/usr/local/bin` or `~/.brew/bin` is in your system `$PATH`.
- On Windows using CMake, run the executable directly from `.\build\bin\Release\llama-cli.exe` or add the directory to your System Environment Variables.
</details>

---

1. **Llama model not trained**: One of the most common issues is that the Llama model may not be trained or may not have been trained properly, leading to failed predictions or incorrect outputs.
2. **Model performance issues**: The Llama model may have performance issues, such as slow training times, high memory requirements, or frequent errors.
3. **Llama model not available**: The Llama model may not be available for use, resulting in errors or unexpected behavior.
4. **Token mismatch**: The Llama model requires a specific token type or format, and a mismatch may occur, leading to errors or unexpected behavior.
5. **Llama model not compatible with certain frameworks**: The Llama model may not be compatible with certain frameworks or libraries, such as TensorFlow, PyTorch, or Keras.
6. **Llama model not compatible with certain data formats**: The Llama model may not be compatible with certain data formats, such as CSV, JSON, or Avro.
7. **Llama model not able to handle large amounts of data**: The Llama model may struggle with large amounts of data, resulting in slow training times or errors.
8. **Llama model not able to handle high-dimensional data**: The Llama model may not be able to handle high-dimensional data, such as images or text data.
9. **Llama model not able to handle categorical data**: The Llama model may not be able to handle categorical data, resulting in errors or incorrect outputs.
10. **Llama model not able to handle out-of-distribution data**: The Llama model may not be able to handle data that is outside the training distribution, resulting in errors or incorrect outputs.

Some specific examples of issues users have reported include:

* **Llama model not recognizing image labels**: Users have reported that the Llama model is having trouble recognizing image labels, such as object detection or image classification.
* **Llama model not able to handle text data**: Users have reported that the Llama model is having trouble handling text data, such as sentiment analysis or language translation.
* **Llama model not able to handle high-dimensional data**: Users have reported that the Llama model is struggling with large amounts of data, resulting in slow training times or errors.

---

## Contributing

Pull requests and issues are welcome! Feel free to open a ticket to submit bug fixes, optimize command flags, or improve installation documentation.

## License

This guide is provided under the [MIT License](LICENSE). `llama.cpp` itself is licensed under the MIT License by ggerganov and contributors.
