# ⚡ Portable Local AI Image Generator

A premium, fully self-contained, zero-configuration desktop environment for offline Stable Diffusion image generation. Run your favorite Safetensors and GGUF models with hardware acceleration on Windows, Linux, and macOS without the hassle of configuring Python dependencies, global environments, or complex backend installs.

---

## 🚀 Key Highlights

*   **100% Offline & Private:** Your prompts, configurations, and generated images stay entirely on your local machine.
*   **Zero System Footprint:** Everything (including Node.js runtimes, model files, and GPU backend libraries) is sandboxed inside the project folder. No global environment paths are altered.
*   **Automated GPU Acceleration:** Automatically detects and configures the optimal backend for your hardware:
    *   **NVIDIA:** Native CUDA configuration.
    *   **AMD Radeon:** ROCm setup or Vulkan fallback.
    *   **Apple Silicon:** Metal GPU framework.
    *   **Intel Arc:** Cross-vendor Vulkan.
    *   **Intel Core Ultra:** Dedicated OpenVINO NPU support.
*   **Integrated Model Library:** Paste any Hugging Face URL to download weights directly, or drag-and-drop local `.safetensors`, `.gguf`, or `.ckpt` files.
*   **Real-Time Hardware Telemetry:** Monitor your CPU, RAM, GPU, and VRAM utilization directly inside the clean web workspace.
*   **Metadata-rich Gallery:** Automatically saves generated PNGs alongside detailed parameter metadata JSONs.

---

## 📁 Repository Structure

```
Portable-Local-Image-Generator/
├── windows.bat                # Main double-click launcher (Windows)
├── linux.sh                   # Main terminal launcher (Linux)
├── mac.sh                     # Main terminal launcher (macOS)
├── LICENSE                    # MIT Open Source license
├── .gitignore                 # Pre-configured ignore paths for model files and output images
├── README.md                  # Beautifully crafted documentation
├── scripts/
│   ├── setup.ps1              # Automated GPU-detect and environment installer (Windows)
│   ├── setup.sh               # Automated GPU-detect and environment installer (Linux/macOS)
│   ├── reset.ps1              # Utility to reset runtime environments (Windows)
│   ├── reset.sh               # Utility to reset runtime environments (Linux/macOS)
│   └── serve.cjs              # UI web server and backend lifecycle manager
└── app/
    ├── frontend/              # UI source code (Vite + React)
    ├── models/                # Folder for weights (.safetensors, .gguf, .ckpt)
    └── outputs/               # Saved images and parameter metadata
```

---

## ⚡ Quick Start Guides

Select your operating system below to get started:

### 🪟 Windows
1. **Launch:** Double-click `windows.bat`. On the first run, this automatically downloads a portable Node.js runtime and configures the prebuilt GPU backend binaries.
2. **Add Models:** Drop your `.safetensors` or `.gguf` weights into the `app/models/` directory, or use the **Model Manager** tab in the UI.
3. **Generate:** Open `http://localhost:1420` in your web browser, pick your model, and start generating.

### 🐧 Linux
1. **Check Compatibility:** Prebuilt Linux backends require **glibc 2.38+** (Ubuntu 24.04+, Fedora 40+, etc.). Run `ldd --version` to check.
2. **Launch:** Open a terminal in the project directory and run:
   ```bash
   ./linux.sh
   ```
   - **NVIDIA Users:** Follow the prompt to set up **CUDA** (downloads prebuilt binaries, compiles from source as a fallback).
   - **AMD Users:** Run `./linux.sh --max-perf` to configure high-performance ROCm drivers.
   - **Intel NPU Users:** Install the Intel Linux NPU driver, then run `./linux.sh --setup-openvino`.
3. **Generate:** Navigate to `http://localhost:1420` in your web browser.

### 🍎 macOS
1. **Compatibility:** Optimized for **Apple Silicon (M1, M2, M3, M4 or newer)** via Metal GPU acceleration. Intel Macs are not officially supported.
2. **Launch:** Run the launcher script from the terminal:
   ```bash
   ./mac.sh
   ```
3. **Generate:** Navigate to `http://localhost:1420` in your web browser.

---

## 🖥️ GPU Compatibility Matrix

### Windows
| GPU Vendor | Backend | Acceleration | Notes |
| :--- | :--- | :--- | :--- |
| **Nvidia** | CUDA | ✅ Native | Maps `sd-cuda.exe` with Nvidia SDK 12 optimizations. |
| **AMD Radeon** | Vulkan | ✅ Native | Maps `sd-vulkan.exe` with Vulkan API acceleration. |
| **Intel Arc** | Vulkan | ✅ Native | Maps `sd-vulkan.exe` for Intel hardware. |
| **Integrated / None** | CPU | ⚠️ Fallback | Runs on logical CPU threads (slow). |

### Linux
| GPU Vendor | Primary Backend | Fallback | Notes |
| :--- | :--- | :--- | :--- |
| **NVIDIA** | CUDA / Vulkan | Vulkan / CPU | Auto-detects NVIDIA. Prompt-driven CUDA setup. |
| **AMD Radeon** | ROCm | Vulkan | ROCm provides best AMD performance when host ROCm drivers are available. |
| **Intel Arc** | Vulkan | CPU | Cross-vendor Vulkan support. |
| **Intel Core Ultra NPU** | OpenVINO NPU | CPU | Requires Intel NPU drivers and `./linux.sh --setup-openvino`. |
| **Integrated / None** | CPU | — | Runs on logical CPU threads (slow). |

### macOS
| Hardware | Primary Backend | Fallback | Notes |
| :--- | :--- | :--- | :--- |
| **Apple Silicon (M1+)** | Metal | CPU | Uses Darwin arm64 stable-diffusion.cpp backend. |

---

## ⏱️ Performance Benchmarks (20 Steps)

Estimated generation times for a single **512x512** image:

| OS | Hardware / Acceleration | Avg. Generation Time |
| :--- | :--- | :--- |
| **Windows** | NVIDIA RTX (CUDA) | **~10 seconds** |
| **Windows** | NVIDIA GTX (Vulkan) | **~30 seconds** |
| **Windows** | AMD Radeon / Intel Arc (Vulkan) | **~89 seconds** |
| **Linux** | NVIDIA RTX (CUDA) | **~10 seconds** |
| **Linux** | AMD Radeon (ROCm) | **~15 - 30 seconds** |
| **Linux** | Intel Core Ultra (OpenVINO NPU) | **~15 - 40 seconds** |
| **macOS** | Apple Silicon M-Series (Metal GPU) | **~12 - 25 seconds** |
| **macOS** | Apple Silicon ANE (Apple NPU) | **~10 - 18 seconds** |
| **All OS** | CPU Fallback | **150 - 300+ seconds** |

---

## 🛠️ Troubleshooting & Fixes

> [!TIP]
> **Reset Environment:** If a setup fails or you want to reinstall clean dependencies, run `scripts/reset.ps1` (Windows) or `scripts/reset.sh` (Linux/macOS). This will preserve your models and generated images.

> [!WARNING]
> **Port Conflicts:** The web UI runs on port `1420` by default. The backend attempts port `8080` first, then auto-selects a free port if busy.

> [!IMPORTANT]
> **GLIBC Version Error (Linux):** Prebuilt binaries require glibc 2.38+. If you are on an older system, you will need to compile the backend binaries from source (see compilation guide below).

---

## 🔨 Building Backends From Source

If prebuilt binaries are incompatible with your system configuration, you can compile them manually:

### Requirements
- `git`, `cmake`, `make` (or `ninja`), and a C++17 compiler (`g++` / `clang++`).
- For **CUDA**: NVIDIA CUDA Toolkit (`nvcc` in PATH).
- For **Vulkan**: Vulkan SDK / loader and drivers.
- For **ROCm**: AMD ROCm development libraries.
- For **macOS Metal**: Apple Command Line Tools / Xcode.

### Manual Build Instructions
```bash
# 1. Clone the upstream repository
git clone https://github.com/leejet/stable-diffusion.cpp.git
cd stable-diffusion.cpp
mkdir build && cd build

# 2. Configure for your backend (choose ONE)
cmake .. -DSD_BUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release                      # CPU only
cmake .. -DSD_CUDA=ON -DSD_BUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release         # CUDA
cmake .. -DSD_VULKAN=ON -DSD_BUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release       # Vulkan
cmake .. -DSD_HIPBLAS=ON -DSD_BUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release      # ROCm
cmake .. -DSD_METAL=ON -DSD_BUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Release        # macOS Metal

# 3. Build the binary
cmake --build . --config Release -j$(getconf _NPROCESSORS_ONLN 2>/dev/null || sysctl -n hw.ncpu)

# 4. Copy and rename the binary into the project directory
# Rename the server binary to match what serve.cjs expects (e.g. sd-cpu, sd-vulkan, sd-cuda, sd-rocm, or sd-metal)
```

---

## 📝 License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file. It bundles [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) (MIT License). Model weights are subject to their respective creators' licenses.
