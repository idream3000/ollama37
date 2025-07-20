# Manual Build Guide for Ollama37

This document provides comprehensive instructions for building Ollama37 from source on various platforms, specifically optimized for Tesla K80 and CUDA Compute Capability 3.7 hardware.

## Quick Build Options

### Docker Build (Recommended)

```bash
# Build ollama37 image for Tesla K80/Compute 3.7 support
docker build -f ollama37.Dockerfile -t ollama37 .
```

This Dockerfile uses a multi-stage build process:
1. **Stage 1 (Builder)**: Uses `dogkeeper886/ollama37-builder` base image with pre-installed CUDA 11.4, GCC 10, and CMake 4
2. **Stage 2 (Runtime)**: Creates a minimal Rocky Linux 8 runtime image with only the compiled binary and required libraries

The build process automatically:
- Configures CMake with GCC 10 and CUDA 3.7 support
- Compiles the C++ components with Tesla K80 optimizations  
- Builds the Go binary
- Creates a runtime image with proper CUDA environment variables

### Native Build

For native builds, you'll need to install the following prerequisites:

**Prerequisites:**
- Rocky Linux 8 (or compatible)
- `git` - For cloning the repository
- `cmake` - For managing the C++ build process
- `go` 1.24.2+ - The Go compiler and toolchain
- `gcc` version 10 - GNU Compiler Collection and G++
- CUDA Toolkit 11.4

**Quick Build Steps:**

```bash
# Clone repository
git clone https://github.com/dogkeeper886/ollama37
cd ollama37

# Configure and build
CC=/usr/local/bin/gcc CXX=/usr/local/bin/g++ cmake -B build
CC=/usr/local/bin/gcc CXX=/usr/local/bin/g++ cmake --build build

# Build Go binary
go build -o ollama .
```

---

## Detailed Installation Guides

### CUDA 11.4 Installation on Rocky Linux 8

**Prerequisites:**
- A Rocky Linux 8 system or container
- Root privileges
- Internet connectivity

**Steps:**

1. **Update the system:**
   ```bash
   dnf -y update
   ```

2. **Install EPEL Repository:**
   ```bash
   dnf -y install epel-release
   ```

3. **Add NVIDIA CUDA Repository:**
   ```bash
   dnf -y config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
   ```

4. **Install NVIDIA Driver (Version 470):**
   ```bash
   dnf -y module install nvidia-driver:470-dkms
   ```

5. **Install CUDA Toolkit 11.4:**
   ```bash
   dnf -y install cuda-11-4
   ```

6. **Set up CUDA Environment Variables:**
   ```bash
   # Create /etc/profile.d/cuda-11.4.sh
   echo "export PATH=/usr/local/cuda-11.4/bin:${PATH}" > /etc/profile.d/cuda-11.4.sh
   echo "export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64:${LD_LIBRARY_PATH}" >> /etc/profile.d/cuda-11.4.sh
   
   # Apply changes
   source /etc/profile.d/cuda-11.4.sh
   ```

**Verification:**
```bash
# Check CUDA compiler
nvcc --version

# Check driver
nvidia-smi
```

### GCC 10 Installation Guide

**Steps:**

1. **Update and Install Prerequisites:**
   ```bash
   dnf -y install wget unzip lbzip2
   dnf -y groupinstall "Development Tools"
   ```

2. **Download GCC 10 Source Code:**
   ```bash
   cd /usr/local/src
   wget https://github.com/gcc-mirror/gcc/archive/refs/heads/releases/gcc-10.zip
   ```

3. **Extract Source Code:**
   ```bash
   unzip gcc-10.zip
   cd gcc-releases-gcc-10
   ```

4. **Download Prerequisites:**
   ```bash
   contrib/download_prerequisites
   ```

5. **Create Installation Directory:**
   ```bash
   mkdir /usr/local/gcc-10
   ```

6. **Configure GCC Build:**
   ```bash
   cd /usr/local/gcc-10
   /usr/local/src/gcc-releases-gcc-10/configure --disable-multilib
   ```

7. **Compile GCC:**
   ```bash
   make -j $(nproc)
   ```

8. **Install GCC:**
   ```bash
   make install
   ```

9. **Post-Install Configuration:**
   ```bash
   # Create environment script
   echo "export LD_LIBRARY_PATH=/usr/local/lib64:\$LD_LIBRARY_PATH" > /etc/profile.d/gcc-10.sh
   
   # Configure dynamic linker
   echo "/usr/local/lib64" > /etc/ld.so.conf.d/gcc-10.conf
   ldconfig
   ```

### CMake 4.0 Installation Guide

1. **Install OpenSSL Development Libraries:**
   ```bash
   dnf -y install openssl-devel
   ```

2. **Download CMake Source Code:**
   ```bash
   cd /usr/local/src
   wget https://github.com/Kitware/CMake/releases/download/v4.0.0/cmake-4.0.0.tar.gz
   ```

3. **Extract the Archive:**
   ```bash
   tar xvf cmake-4.0.0.tar.gz
   ```

4. **Create Installation Directory:**
   ```bash
   mkdir /usr/local/cmake-4
   ```

5. **Configure CMake:**
   ```bash
   cd /usr/local/cmake-4
   /usr/local/src/cmake-4.0.0/configure
   ```

6. **Compile CMake:**
   ```bash
   make -j $(nproc)
   ```

7. **Install CMake:**
   ```bash
   make install
   ```

### Go 1.24.2 Installation Guide

1. **Download Go Distribution:**
   ```bash
   cd /usr/local
   wget https://go.dev/dl/go1.24.2.linux-amd64.tar.gz
   ```

2. **Extract the Archive:**
   ```bash
   tar xvf go1.24.2.linux-amd64.tar.gz
   ```

3. **Post Install Configuration:**
   ```bash
   echo 'export PATH=$PATH:/usr/local/go/bin' > /etc/profile.d/go.sh
   source /etc/profile.d/go.sh
   ```

## Complete Ollama37 Compilation Guide

**Prerequisites:**
All components installed as per the guides above:
- Rocky Linux 8
- Git
- CMake 4.0
- Go 1.24.2
- GCC 10
- CUDA Toolkit 11.4

**Compilation Steps:**

1. **Navigate to Build Directory:**
   ```bash
   cd /usr/local/src
   ```

2. **Clone the Repository:**
   ```bash
   git clone https://github.com/dogkeeper886/ollama37
   cd ollama37
   ```

3. **CMake Configuration:**
   Set compiler variables and configure the build system:
   ```bash
   CC=/usr/local/bin/gcc CXX=/usr/local/bin/g++ cmake -B build
   ```

4. **CMake Build:**
   Compile the C++ components:
   ```bash
   CC=/usr/local/bin/gcc CXX=/usr/local/bin/g++ cmake --build build
   ```

5. **Go Build:**
   Compile the Go components:
   ```bash
   go build -o ollama .
   ```

6. **Verification:**
   ```bash
   ./ollama --version
   ```

## Tesla K80 Specific Optimizations

The Ollama37 build includes several Tesla K80-specific optimizations:

### CUDA Architecture Support
- **CMake Configuration**: `CMAKE_CUDA_ARCHITECTURES "37;50;61;70;75;80"`
- **Build Files**: Located in `ml/backend/ggml/ggml/src/ggml-cuda/CMakeLists.txt`

### CUDA 11 Compatibility
- Uses CUDA 11 toolchain (CUDA 12 dropped Compute Capability 3.7 support)
- Environment variables configured for CUDA 11.4 paths
- Driver version 470 for maximum compatibility

### Performance Tuning
- Optimized memory management for Tesla K80's 12GB VRAM
- Kernel optimizations for Kepler architecture
- Reduced precision operations where appropriate

## Troubleshooting

### Common Issues

**CUDA Version Conflicts:**
```bash
# Check CUDA version
nvcc --version
# Should show CUDA 11.4

# If wrong version, check PATH
echo $PATH
# Should include /usr/local/cuda-11.4/bin
```

**GCC Version Issues:**
```bash
# Check GCC version
/usr/local/bin/gcc --version
# Should show GCC 10.x

# If build fails, ensure CC and CXX are set
export CC=/usr/local/bin/gcc
export CXX=/usr/local/bin/g++
```

**Memory Issues:**
- Tesla K80 has 12GB VRAM - adjust model sizes accordingly
- Monitor GPU memory usage with `nvidia-smi`
- Use quantized models (Q4, Q8) for better memory efficiency

### Build Verification

After successful compilation, verify Tesla K80 support:

```bash
# Check if ollama detects your GPU
./ollama serve &
./ollama run llama3.2 "Hello Tesla K80!"

# Monitor GPU utilization
watch -n 1 nvidia-smi
```

## Performance Optimization Tips

1. **Model Selection**: Use quantized models (Q4_0, Q8_0) for better performance on Tesla K80
2. **Memory Management**: Monitor VRAM usage and adjust context sizes accordingly  
3. **Temperature Control**: Ensure adequate cooling for sustained workloads
4. **Power Management**: Tesla K80 requires proper power delivery (225W per GPU)

## Docker Alternative

If manual compilation proves difficult, the pre-built Docker image provides the same Tesla K80 optimizations:

```bash
docker pull dogkeeper886/ollama37
docker run --runtime=nvidia --gpus all -p 11434:11434 dogkeeper886/ollama37
```

This image includes all the optimizations and dependencies pre-configured for Tesla K80 hardware.