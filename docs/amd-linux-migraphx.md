# AMD Linux MIGraphX Bring-Up

This repo now has enough hooks to test a custom ONNX Runtime build on AMD Linux:

- `inference.provider=auto|migraphx|rocm|cuda|directml`
- `inference.native_path=/path/to/custom/ort/natives`
- `-PonnxRuntimeVersion=...` to match the Java classes to the custom native build

## Current assessment

- Host machine: ROCm runtime sees the discrete RX 7900 XT as `gfx1100`.
- Project build: `./gradlew build` succeeds on this machine.
- Remaining blocker: the stock Java ORT dependencies published to Maven are CPU and CUDA only.
- Practical AMD path: build ONNX Runtime from source with MIGraphX enabled, then point the mod at those native libraries.

## Recommended ONNX Runtime target

Use an ONNX Runtime release branch that matches the installed ROCm/MIGraphX stack.

For this machine:

- ROCm installed: `7.2.x`
- Best match from the current ORT MIGraphX docs: `onnxruntime` `1.23.2`

## Expected custom build flow

Clone ONNX Runtime on the matching release branch:

```bash
git clone --recursive --branch rel-1.23.2 https://github.com/microsoft/onnxruntime.git
cd onnxruntime
```

Build with Java bindings and MIGraphX enabled:

```bash
./build.sh \
  --config Release \
  --build_shared_lib \
  --build_java \
  --parallel \
  --skip_tests \
  --use_migraphx \
  --migraphx_home /opt/rocm
```

The ORT docs also show the minimal MIGraphX build form as:

```bash
./build.sh --config Release --parallel --use_migraphx --migraphx_home /opt/rocm
```

## Terrain Diffusion test flow

After building custom ORT, build this mod against the matching Java API version:

```bash
./gradlew build -PonnxRuntimeVersion=1.23.2
```

Then set the mod config:

```properties
inference.device=gpu
inference.provider=migraphx
inference.native_path=/absolute/path/to/onnxruntime/build/output/native/libs
```

If the custom Java API does not expose `addMIGraphX`, this repo falls back cleanly and you can still try:

```properties
inference.provider=rocm
```

The mod now attempts reflective `addMIGraphX(int)` registration first when `inference.provider=migraphx` or Linux `auto` is used.

## Likely files needed in `inference.native_path`

The Java loader expects the ORT shared libraries in one directory. In practice this means the folder should contain at least:

- `libonnxruntime.so`
- `libonnxruntime4j_jni.so`
- the relevant provider library, such as a MIGraphX or ROCm provider `.so`
- any required dependent provider/shared libs produced by the ORT build

## Notes

- Official ORT Java install docs currently list Maven artifacts for CPU and CUDA, not a Java MIGraphX package.
- ORT Java supports loading native libraries from an explicit directory via `onnxruntime.native.path`, which this repo now maps from `inference.native_path`.
- ORT build docs recommend building from an official release branch for production-style use.
