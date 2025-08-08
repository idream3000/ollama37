# Syncing with Upstream Ollama

This document describes the process for syncing the ollama37 fork with the official ollama/ollama repository while preserving CUDA Compute Capability 3.7 support for Tesla K80 GPUs.

## Prerequisites

- Git configured with the upstream remote: `https://github.com/ollama/ollama.git`
- Understanding of which files contain CUDA 3.7 specific changes

## Key Files to Preserve

When merging from upstream, always preserve CUDA 3.7 support in these files:

1. **`ml/backend/ggml/ggml/src/ggml-cuda/CMakeLists.txt`** - Contains `CMAKE_CUDA_ARCHITECTURES "37;50;61;70;75;80"`
2. **`CMakePresets.json`** - Keep both "CUDA 11" (with arch 37) and "CUDA 12" presets
3. **`README.md`** - Maintain ollama37 specific documentation
4. **`docs/*.md`** - Keep our custom documentation

## Sync Process

### 1. Create a New Branch

```bash
git checkout main
git checkout -b sync-upstream-models
```

### 2. Add Upstream Remote (if not exists)

```bash
git remote add upstream https://github.com/ollama/ollama.git
```

### 3. Fetch Latest Changes

```bash
git fetch upstream main
```

### 4. Merge Upstream Changes

```bash
git merge upstream/main -m "Merge upstream ollama/ollama main branch while preserving CUDA 3.7 support"
```

### 5. Resolve Conflicts

Common conflict resolutions:

#### CMakePresets.json
- **Resolution**: Keep both CUDA 11 (with architecture 37) and CUDA 12 configurations
- **Example**: Preserve the "CUDA 11" preset with `"CMAKE_CUDA_ARCHITECTURES": "37;50;52;53;60;61;70;75;80;86"`

#### README.md and Documentation
- **Resolution**: Keep our version with `git checkout --ours README.md`
- **Reason**: Maintains ollama37 specific instructions and branding

#### Model Support Files
- **Resolution**: Accept upstream changes for new model support
- **Files**: `model/models/models.go`, new model directories
- **Example**: Accept new imports like `_ "github.com/ollama/ollama/model/models/gptoss"`

#### Backend/Tools Updates
- **Resolution**: Generally accept upstream improvements
- **Files**: `tools/tools.go`, `ml/backend/ggml/ggml.go`
- **Caution**: Verify no CUDA 3.7 specific code is removed

#### Test Files
- **Resolution**: Accept upstream test additions
- **Files**: `integration/utils_test.go`
- **Action**: Include new model tests in the test lists

### 6. Commit the Merge

```bash
git add -A
git commit -m "Merge upstream ollama/ollama main branch while preserving CUDA 3.7 support

- Added support for new [model name] from upstream
- Preserved CUDA Compute Capability 3.7 (Tesla K80) support
- Kept CUDA 11 configuration alongside CUDA 12
- Maintained all documentation specific to ollama37 fork
- [List other significant changes]"
```

### 7. Test the Build

Build with Docker to verify CUDA 3.7 support:

```bash
docker build -f ollama37.Dockerfile -t ollama37:test .
```

Or build manually:

```bash
CC=/usr/local/bin/gcc CXX=/usr/local/bin/g++ cmake -B build
CC=/usr/local/bin/gcc CXX=/usr/local/bin/g++ cmake --build build
go build -o ollama .
```

### 8. Test with Tesla K80

```bash
# Run the server
./ollama serve

# In another terminal, test a model
ollama run llama3.2:1b
```

### 9. Merge to Main

After successful testing:

```bash
git checkout main
git merge sync-upstream-models
git push origin main
```

## Troubleshooting

### Conflict in CUDA Architecture Settings
- Always ensure "37" is included in CMAKE_CUDA_ARCHITECTURES
- CUDA 11 is required for Compute Capability 3.7 support
- CUDA 12 dropped support for architectures below 50

### Build Failures
- Check that GCC 10 is being used (required for CUDA 11.4)
- Verify CUDA 11.x is installed (not CUDA 12)
- Ensure all CUDA 3.7 specific patches are preserved

### New Model Integration Issues
- New models should work automatically if they don't have specific CUDA version requirements
- If a model requires CUDA 12 features, document the limitation in README.md

## Recent Sync History

- **2025-08-08**: Synced with upstream, added gpt-oss model support
- **Previous syncs**: See git log for merge commits from upstream/main

## Notes

- This process maintains our fork's ability to run on Tesla K80 GPUs while staying current with upstream features
- Always test thoroughly before merging to main
- Document any model-specific limitations discovered during testing