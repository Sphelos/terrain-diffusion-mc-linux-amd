# Linux AMD Release Notes

Release target: `terrain-diffusion-mc-2.1.0-linux-amd+1.21.11.jar`

## Purpose

This release packages the Linux AMD fork changes needed to run Terrain Diffusion on AMD GPUs under Linux through ROCm/MIGraphX.

## Included Fork Changes

- configurable GPU provider selection:
  - `auto`
  - `migraphx`
  - `rocm`
  - `cuda`
  - `directml`
- explicit `inference.native_path` support for custom ONNX Runtime native libraries
- dedicated Gradle build variant:
  - `-PuseMigraphx=true`
- patched ONNX Runtime Java binding support for `addMIGraphX()`

## Verified Setup

- OS: CachyOS Linux
- Kernel: `7.0.1-1-cachyos`
- CPU: AMD Ryzen 7 9800X3D
- GPU: AMD Radeon RX 7900 XT
- ROCm: `7.2.x`
- Java: Microsoft OpenJDK `21.0.7`
- Minecraft: `1.21.11`
- Fabric Loader: `0.19.2`
- Fabric API: `0.141.3+1.21.11`

## Required Config

```properties
inference.device=gpu
inference.provider=migraphx
inference.native_path=/absolute/path/to/onnxruntime-migraphx-native-directory
```

## Native Runtime Notes

This release expects external ONNX Runtime native libraries built with MIGraphX enabled. The jar alone is not enough.

The native directory used during testing contained:

- `libonnxruntime.so`
- `libonnxruntime4j_jni.so`
- `libonnxruntime_providers_migraphx.so`
- `libonnxruntime_providers_shared.so`

## Operational Notes

- `World Scale 2` is the recommended baseline.
- Higher world scale values smooth terrain more aggressively and can produce very large oceans.
- This release is intended for AMD Linux use, not as an official upstream-supported distribution.

## Credit

Original mod and integration work:

- [xandergos/terrain-diffusion-mc](https://github.com/xandergos/terrain-diffusion-mc)
- [xandergos/terrain-diffusion](https://github.com/xandergos/terrain-diffusion)
