# Terrain Diffusion MC Linux AMD Fork

Terrain Diffusion MC for AMD GPUs on Linux through ROCm/MIGraphX.

## Upstream Credit

All core Terrain Diffusion Minecraft integration work comes from the original project and its contributors:

- Original Minecraft mod: [xandergos/terrain-diffusion-mc](https://github.com/xandergos/terrain-diffusion-mc)
- Original Terrain Diffusion models/project: [xandergos/terrain-diffusion](https://github.com/xandergos/terrain-diffusion)

This fork exists to document and package an AMD/Linux-oriented bring-up. Upstream authors deserve the credit for the original mod, model integration, and most of the codebase.

## Fork Scope

This fork adds and documents:

- Linux AMD GPU provider selection via `migraphx`
- explicit ONNX Runtime native library override support
- a dedicated `linux-amd` build variant
- notes for a tested ROCm/MIGraphX setup

It does not replace upstream. If you want the original project and its normal release stream, use the upstream repository.

## Verified Setup

- OS: CachyOS Linux
- Kernel: `7.0.1-1-cachyos`
- GPU: AMD Radeon RX 7900 XT (`gfx1100`)
- ROCm: `7.2.x`
- Java: Microsoft OpenJDK `21.0.7`
- Minecraft: `1.21.11`
- Fabric Loader: `0.19.2`
- Fabric API: `0.141.3+1.21.11`

## Build Matrix

| Build | Supports | Setup required |
| --- | --- | --- |
| **Linux AMD** | Linux + AMD GPU + ROCm/MIGraphX | Custom MIGraphX-enabled ONNX Runtime natives |
| **Windows** | Windows with DirectML-capable GPU | None beyond the bundled DirectML runtime |
| **CUDA** | Linux/Windows with NVIDIA GPU | CUDA + cuDNN |
| **CPU** | Any machine | None, but very slow |

For AMD on Linux, use the `linux-amd` jar from this fork's releases, not the CPU jar and not the upstream release artifacts.

## Requirements

- Minecraft with [Fabric](https://fabricmc.net/) and [Fabric API](https://modrinth.com/mod/fabric-api)
- Java 21
- For the Linux AMD path:
  - an AMD GPU supported by ROCm/MIGraphX
  - a working ROCm userspace install
  - custom ONNX Runtime native libraries built with MIGraphX enabled

Memory requirements vary with world scale and chunk generation pressure. On the tested machine:

- GPU VRAM used by inference: roughly `1.5-2 GB`
- Minecraft RAM allocation used for testing: `12 GB`

## Quick Start

1. Install the `linux-amd` jar in your Minecraft `mods/` folder.
2. Launch once so the mod can create its config and download models.
3. Edit `config/terrain-diffusion-mc.properties`.
4. Set:

```properties
inference.device=gpu
inference.provider=migraphx
inference.native_path=/absolute/path/to/onnxruntime-migraphx-native-directory
```

That directory contained:

- `libonnxruntime.so`
- `libonnxruntime4j_jni.so`
- `libonnxruntime_providers_migraphx.so`
- `libonnxruntime_providers_shared.so`

## Configuration

The mod config file is `config/terrain-diffusion-mc.properties`.

Relevant inference settings in this fork:

```properties
# "cpu", "gpu", or "auto"
inference.device=gpu

# "auto", "migraphx", "rocm", "cuda", or "directml"
inference.provider=migraphx

# Optional path to custom ONNX Runtime native libraries
inference.native_path=

# Offload inactive models between stages
inference.offload_models=true

# Validate existing model files
validate_model=true

# Explorer web UI port
explorer.port=19801

# Chunk generation region size in blocks
tile_size=256
```

## World Scale Notes

Terrain Diffusion worlds expose a `World Scale` setting from `1` to `6`.

- `2` is the default and the most balanced setting in this codebase.
- Higher values are not "more detail". They generate at native model resolution and then upsample more aggressively.
- Above `4`, very broad smoothing can produce large ocean-heavy starts. That is expected from the current scaling logic, not specifically an AMD/Linux issue.

## Dependencies

Primary runtime/build dependencies involved in this fork:

- Minecraft `1.21.11`
- Fabric Loader `0.19.2`
- Fabric API `0.141.3+1.21.11`
- ONNX Runtime `1.23.2`
- ROCm/MIGraphX `7.2.x`

The Linux AMD path in this fork relies on a custom ONNX Runtime Java/native build because the stock Maven Java artifacts do not ship as a ready-made MIGraphX package for this workflow.

## Building From Source

Standard upstream variants still work:

```bash
./gradlew build -PuseDml=true
./gradlew build -PuseCuda=true
./gradlew build -PuseCpu=true
```

Linux AMD build for this fork:

```bash
./gradlew build -PuseMigraphx=true -PonnxRuntimeMigraphxVersion=1.23.2-migraphxfix
```

This expects a matching patched ONNX Runtime Java artifact to be available through `mavenLocal()`.

## Linux AMD Notes

Detailed notes for the AMD/Linux port are in [docs/amd-linux-migraphx.md](docs/amd-linux-migraphx.md).

## Common Issues

### No configured GPU provider is available

This usually means one of these is wrong:

- the jar does not include the MIGraphX-capable Java bindings
- `inference.native_path` does not point at the custom ORT native library directory
- the native library directory is missing provider `.so` files
- ROCm/MIGraphX is not installed or not loadable on the machine

### Massive ocean starts at high world scale

This is an effect of the current upsampling-based scale system. Reduce `World Scale` back to `2` or `3` if you want less smoothing and more compact terrain features.

### Out-of-memory during world generation or model load

Increase Minecraft RAM allocation and keep world scale moderate. The model files themselves are large, and higher scale values change chunk-generation behavior in ways that can still stress memory and CPU.

## Release Notes

Release-specific notes for this fork live in [docs/releases/linux-amd-2.1.0.md](docs/releases/linux-amd-2.1.0.md).
