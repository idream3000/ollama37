# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goal

This project (ollama37) exists to maintain support for NVIDIA Tesla K80 GPUs and other Compute Capability 3.7 hardware. The official Ollama release has deprecated support for these older GPUs, but this fork keeps them functional by:

- Maintaining sync with the official Ollama repository for latest features and fixes
- Preserving CUDA Compute Capability 3.7 support that was removed from upstream
- Providing a specialized build optimized for Tesla K80 and similar legacy hardware

This enables users with older NVIDIA GPUs to continue running modern LLMs locally without requiring hardware upgrades.

## CUDA 3.7 Support Implementation

CUDA Compute Capability 3.7 support is maintained in the following key locations:

- **`ml/backend/ggml/ggml/src/ggml-cuda/CMakeLists.txt:7`** - Core build configuration with `CMAKE_CUDA_ARCHITECTURES "37;50;61;70;75;80"`
- **`CMakePresets.json:24`** - "CUDA 11" preset includes "37" (CUDA 12 dropped 3.7 support)
- **`README.md:322`** - Tesla K80 optimization documentation
- **`docs/gpu.md:33`** - Building guidance for older GPUs

The project uses CUDA 11 toolchain to maintain compatibility with Tesla K80 and other Compute Capability 3.7 GPUs, as CUDA 12 officially dropped support for these architectures.

## Development Commands

### Building the Project
```bash
# Configure build (required on Linux/Intel macOS/Windows)
cmake -B build
cmake --build build

# For ROCm on Windows
cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
cmake --build build --config Release

# Build Go binary
go build -o ollama .
```

### Running Ollama
```bash
# Run development server
go run . serve

# Start server with built binary
./ollama serve
```

### Testing
```bash
# Run all tests
go test ./...

# Run tests with synctest (for Go 1.24 compatibility)
GOEXPERIMENT=synctest go test ./...

# Run integration tests (requires server running)
go test ./integration/...

# Run specific test package
go test ./server/...
```

### Docker
```bash
# Build standard image
docker build .

# Build with ROCm support
docker build --build-arg FLAVOR=rocm .

# Build ollama37 image for Tesla K80/Compute 3.7 support
docker build -f ollama37.Dockerfile -t ollama37 .
```

## Architecture Overview

Ollama is a local LLM server with Go backend and C++/CUDA acceleration:

### Core Components

**Entry Point**: `main.go` uses Cobra CLI framework, delegating to `cmd/` package for command handling.

**Server Layer** (`server/`): HTTP server built on Gin framework handling:
- REST API endpoints (`routes.go`)
- Model management (download, create, delete)
- Chat and generation endpoints
- Model scheduling and GPU resource management (`sched.go`)

**LLM Integration** (`llm/`): Abstracts language model backends with platform-specific implementations:
- `server.go` - LLM server process management
- `memory.go` - GPU memory management
- Platform-specific files for Darwin, Linux, Windows

**Model Layer** (`model/`): Handles model format conversion and tokenization:
- `models/` - Model-specific implementations (Llama, Gemma, etc.)
- `imageproc/` - Image processing for multimodal models
- Tokenizer implementations (BPE, SentencePiece)

**ML Backend** (`ml/backend/ggml/`): C++ acceleration layer built on GGML:
- CPU optimizations with SIMD
- CUDA GPU acceleration
- ROCm/HIP support for AMD GPUs
- Memory-mapped model loading

**Conversion Pipeline** (`convert/`): Converts models from HuggingFace/PyTorch formats to GGUF:
- Architecture-specific converters for different model families
- Safetensors and PyTorch tensor reading
- Quantization support

### Key Data Flow

1. **Model Loading**: Models downloaded/converted to GGUF format, stored locally
2. **Request Processing**: HTTP requests parsed, routed through server layer
3. **Model Scheduling**: GPU resources allocated, models loaded into memory
4. **Inference**: Requests forwarded to appropriate LLM backend process
5. **Response Streaming**: Generated tokens streamed back via HTTP

### GPU Acceleration

The project supports multiple acceleration backends:
- **CUDA**: NVIDIA GPU support via `ml/backend/ggml/ggml/src/ggml-cuda/`
- **Metal**: Apple Silicon native support
- **ROCm/HIP**: AMD GPU support
- **CPU**: Optimized CPU kernels with AVX/NEON

Libraries are dynamically loaded from:
- `./lib/ollama` (Windows)
- `../lib/ollama` (Linux) 
- `.` (macOS)
- `build/lib/ollama` (development)

### Configuration

- Environment variables prefixed with `OLLAMA_` (`envconfig/`)
- Model templates in `template/` directory
- Tool definitions in `tools/` for function calling

### Testing Structure

- Unit tests throughout codebase (`*_test.go`)
- Integration tests in `integration/` requiring running server
- Benchmark tests for performance validation
- Platform-specific test files for GPU/hardware features