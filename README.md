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
ollama pull llama3.2
ollama run llama3.2 "Why is the sky blue?"

# Interactive chat
ollama run gemma3
```

### Supported Models
All models from [ollama.com/library](https://ollama.com/library) including Llama 3.2, Gemma 3, Qwen 2.5, Phi-4, and Code Llama.

### REST API
```bash
# Generate response
curl http://localhost:11434/api/generate -d '{"model": "llama3.2", "prompt": "Hello Tesla K80!"}'

# Chat
curl http://localhost:11434/api/chat -d '{"model": "llama3.2", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Technical Details

### Tesla K80 Support
- **CUDA 3.7 Support**: Maintained via `CMAKE_CUDA_ARCHITECTURES "37;50;61;70;75;80"`
- **CUDA 11 Toolchain**: Compatible with legacy GPUs (CUDA 12 dropped 3.7 support)
- **Optimized Builds**: Tesla K80-specific performance tuning

### Recent Updates
- **v1.3.0** (2025-07-19): Added Gemma 3, Qwen2.5VL, latest upstream sync
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

