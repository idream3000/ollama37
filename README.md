# Ollama37 🚀

**Tesla K80 Compatible Ollama Fork**

Run modern LLMs on NVIDIA Tesla K80 and other CUDA Compute Capability 3.7 GPUs. While official Ollama dropped legacy GPU support, Ollama37 keeps your Tesla K80 hardware functional with the latest models and features.

## Key Features

- ⚡ **Tesla K80 Support** - Full compatibility with CUDA Compute Capability 3.7
- 🔄 **Always Current** - Synced with upstream Ollama for latest models and fixes  
- 🛠️ **Optimized Build** - CUDA 11 toolchain for maximum legacy GPU compatibility
- 💰 **Cost Effective** - Leverage existing hardware without expensive upgrades

## Quick Start

### Docker (Recommended)
```bash
# Pull and run
docker pull dogkeeper886/ollama37
docker run --runtime=nvidia --gpus all -p 11434:11434 dogkeeper886/ollama37
```

### Docker Compose
```yaml
services:
  ollama:
    image: dogkeeper886/ollama37
    ports: ["11434:11434"]
    volumes: ["./.ollama:/root/.ollama"]
    runtime: nvidia
    restart: unless-stopped
```
```bash
docker-compose up -d
```

## Usage

### Run Your First Model
```bash
# Download and run a model
ollama pull gemma3
ollama run gemma3 "Why is the sky blue?"

# Interactive chat
ollama run gemma3
```

### Tesla K80 Multi-GPU Example
```bash
# GPT-OSS utilizes both GPUs automatically
ollama pull gpt-oss
ollama run gpt-oss "Explain the advantages of dual GPU inference"

# Monitor GPU usage
nvidia-smi -l 1  # Shows ~94%/74% utilization on dual K80s
```

### Supported Models
All models from [ollama.com/library](https://ollama.com/library) including Llama 3.2, Gemma3n, Qwen 2.5, Phi-4, Code Llama, and **GPT-OSS** (multi-GPU optimized for Tesla K80).

### REST API
```bash
# Generate response
curl http://localhost:11434/api/generate -d '{"model": "gemma3, "prompt": "Hello Tesla K80!"}'

# Chat
curl http://localhost:11434/api/chat -d '{"model": "gemma3, "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Technical Details

### Tesla K80 Support
- **CUDA 3.7 Support**: Maintained via `CMAKE_CUDA_ARCHITECTURES "37;50;61;70;75;80"`
- **CUDA 11 Toolchain**: Compatible with legacy GPUs (CUDA 12 dropped 3.7 support)  
- **Multi-GPU Optimization**: GPT-OSS runs efficiently across dual K80 GPUs with 13,12 tensor-split
- **Memory Management**: Enhanced VMM pool with granularity alignment and progressive fallback

### Tesla K80 Memory Improvements (v1.4.0)

This release includes major stability improvements for Tesla K80 dual-GPU systems:

#### **VMM Pool Crash Fixes**
- **Issue**: `cuMemAddressReserve` failures causing `CUDA_ERROR_INVALID_VALUE` crashes
- **Solution**: Memory granularity alignment and progressive fallback (4GB → 2GB → 1GB → 512MB)
- **Result**: Stable memory allocation with 93.8%/74.0% GPU utilization on dual K80s

#### **Multi-GPU Model Switching**
- **Issue**: Scheduler deadlocks when switching between multi-GPU (GPT-OSS) and single-GPU (Llama 3.2) models  
- **Solution**: Enhanced conflict detection and proper unload sequencing in scheduler
- **Result**: Seamless gpt-oss ↔ llama3.2 switching with 4-17s load times

#### **Silent Inference Failures**
- **Issue**: Models loaded successfully but failed to generate output after model switching
- **Solution**: Critical `cudaSetDevice()` validation - fail fast instead of silent failures
- **Result**: Self-healing system with automatic recovery, no system reboots required

These improvements enable **robust production use** of Tesla K80 hardware for LLM inference with model switching capabilities that rival modern GPU setups.

### Recent Updates
- **v1.4.0** (2025-08-10): GPT-OSS multi-GPU support, critical Tesla K80 memory fixes, robust model switching
- **v1.3.0** (2025-07-19): Added Gemma3n, Qwen2.5VL, latest upstream sync  
- **v1.2.0** (2025-05-06): Qwen3, Gemma 3 12B, Phi-4 14B support

## Building from Source

### Docker Build
```bash
docker build -f ollama37.Dockerfile -t ollama37 .
```

### Manual Build
For detailed manual compilation instructions including CUDA 11.4, GCC 10, and CMake setup, see our [Manual Build Guide](docs/manual-build.md).

## Contributing

Found an issue or want to contribute? Check our [GitHub issues](https://github.com/dogkeeper886/ollama37/issues) or submit Tesla K80-specific bug reports and compatibility fixes.

## License

Same license as upstream Ollama. See LICENSE file for details.

## Advanced Usage

### Custom Models
```shell
# Import GGUF model
ollama create custom-model -f Modelfile

# Customize existing model
echo 'FROM llama3.2
PARAMETER temperature 0.8
SYSTEM "You are a helpful Tesla K80 expert."' > Modelfile
ollama create tesla-expert -f Modelfile
```

### CLI Commands
```shell
ollama list              # List models
ollama show llama3.2     # Model info  
ollama ps               # Running models
ollama stop llama3.2    # Stop model
ollama serve            # Start server
```

### Libraries & Community
- [ollama-python](https://github.com/ollama/ollama-python) | [ollama-js](https://github.com/ollama/ollama-js)
- [Discord](https://discord.gg/ollama) | [Reddit](https://reddit.com/r/ollama)

See [API documentation](./docs/api.md) for complete REST API reference.

