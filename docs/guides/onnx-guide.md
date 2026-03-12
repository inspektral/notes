# The complete guide to ONNX for ML engineers

**ONNX (Open Neural Network Exchange) is the universal interchange format that lets you train a model in any framework and deploy it on virtually any hardware — and understanding it is one of the most practical skills an ML engineer can develop.** This guide covers everything from ONNX's architecture and export workflows to runtime optimization and speech-specific deployment patterns. By the end, you'll know when ONNX is the right tool, how to use it, and how to avoid its pitfalls.

---

## What ONNX is and why it exists

ONNX defines **a common file format and operator set for representing machine learning models** — both deep learning and traditional ML. Originally named "Toffee" by Facebook's PyTorch team, it was renamed and jointly announced by **Facebook (now Meta) and Microsoft in September 2017**. IBM, Huawei, Intel, AMD, ARM, and Qualcomm quickly joined, and in November 2019 ONNX became a graduate-level project under the **LF AI & Data Foundation** (part of the Linux Foundation), where it remains under vendor-neutral, Apache 2.0–licensed governance.

ONNX solves two problems that plagued ML engineering before its creation. First, **framework interoperability**: a model trained in PyTorch can be exported to a universal `.onnx` file and consumed by any ONNX-compatible tool, eliminating the need to rewrite inference code when switching frameworks. Second, **deployment portability**: a single ONNX model runs on CPUs, NVIDIA GPUs, Intel accelerators, Apple Neural Engine, Qualcomm NPUs, AMD GPUs, FPGAs, and browsers — all without model changes. This eliminates vendor lock-in and lets teams pick the best framework for training independently from the best hardware for deployment.

As of March 2026, the latest stable release is **ONNX 1.20.1** (January 10, 2026), corresponding to approximately **opset 25 and IR version 11**. ONNX 1.21.0 is in release candidate stage. The companion inference engine, **ONNX Runtime (ORT) 1.24.3** (March 5, 2026), is maintained by Microsoft and released quarterly.

### The format under the hood

ONNX uses **Google Protocol Buffers (protobuf)** for serialization. A `.onnx` file contains a `ModelProto` with three independent version numbers: **IR version** (file format structure, changes rarely), **opset version** (operator definitions, increments each release), and **model version** (user-defined). The core of every model is a `GraphProto` containing **nodes** (operations like `Conv`, `MatMul`, `Softmax`), **inputs/outputs** (typed, shaped tensor definitions), and **initializers** (pre-trained weights stored as `TensorProto`).

Operators belong to domains: the default `ai.onnx` domain contains **~200+ deep learning operators** (currently at opset 25), while `ai.onnx.ml` covers traditional ML operators like tree ensembles and SVMs (opset 5). Recent opset additions reflect modern architecture needs — **opset 23 added `Attention`, `RMSNormalization`, and `RotaryEmbedding`** operators, directly targeting transformer deployment.

```python
import onnx
from onnx import __version__
from onnx.defs import onnx_opset_version

model = onnx.load("model.onnx")
print(f"ONNX version: {__version__}")
print(f"Default opset: {onnx_opset_version()}")
print(f"IR version: {model.ir_version}")
print(f"Nodes: {len(model.graph.node)}, Initializers: {len(model.graph.initializer)}")

# Validate structural correctness
onnx.checker.check_model(model)
```

---

## When ONNX helps and when it hurts

ONNX is not always the right choice. Understanding its tradeoffs prevents wasted effort.

**The strongest advantages** are portability (one model file runs everywhere), performance (ORT's graph optimizations can yield **2–3× speedups** over native framework inference), and ecosystem breadth (20+ hardware backends via Execution Providers, APIs in Python, C++, C#, Java, JavaScript, Swift, Rust, and more). ONNX is backed by **30+ major companies** including Microsoft, Meta, NVIDIA, Intel, and Qualcomm, with production deployments at Microsoft across Bing, Office, and Azure Cognitive Services. The single-runtime approach — one `InferenceSession` API across all languages and hardware — dramatically simplifies production deployment.

**The real disadvantages** are more nuanced. Cutting-edge operations (novel attention mechanisms, custom CUDA kernels) often lack ONNX operator equivalents, forcing custom op workarounds that require understanding the protobuf schema and operator registration. **Dynamic shapes**, while supported, cause issues with some converters and runtimes — a recurring pain point for speech and NLP models. Debugging is harder because ONNX graphs are opaque serialized binary; when the ONNX output diverges from framework output, pinpointing the error in a graph of hundreds of nodes is tedious even with visualization tools like Netron. Ecosystem fragmentation (multiple converters, multiple opset versions, multiple runtimes) demands careful version management. And complex models may suffer conversion fidelity issues or even performance degradation compared to natively optimized framework inference (TensorFlow XLA, `torch.compile`).

---

## ONNX vs. `torch.export` and `torch.compile`: choosing your deployment path

With PyTorch 2.x, ONNX is no longer the only game in town for getting a PyTorch model out of eager mode and into an optimized production format. Understanding when ONNX still wins — and when native PyTorch paths are better — is critical.

### The landscape has shifted

Before PyTorch 2.0, ONNX served as the primary bridge between PyTorch's dynamic eager execution and the static, optimized graphs that deployment runtimes expect. The workflow was: train in PyTorch → export to ONNX → run on ONNX Runtime (or convert further to TensorRT, OpenVINO, etc.).

PyTorch 2.0 changed this equation fundamentally. `torch.compile` uses TorchDynamo to capture Python bytecode and produce optimized computation graphs at runtime, with no export step at all. `torch.export` produces a serializable `ExportedProgram` — a frozen, ahead-of-time graph in PyTorch's own ATen operator set — which can then be handed to Torch-TensorRT, ExecuTorch (for mobile/edge), or consumed by other PyTorch-native runtimes. In fact, PyTorch's own ONNX exporter (`torch.onnx.export(dynamo=True)`) now uses `torch.export` internally as its first step before translating to ONNX operators — the two paths share the same graph capture engine.

### Head-to-head comparison

| Dimension | ONNX + ONNX Runtime | `torch.export` / `torch.compile` |
|---|---|---|
| **Runtime dependency** | ORT only (~50 MB wheel, no PyTorch needed) | Full PyTorch runtime required |
| **Cross-platform** | Python, C++, C#, Java, JS, Swift, Rust, mobile, browser | Python-first; C++ via `libtorch`; mobile via ExecuTorch |
| **Hardware backends** | 20+ Execution Providers (CUDA, TensorRT, OpenVINO, CoreML, DirectML, QNN, NNAPI…) | Inductor (CPU/CUDA), Torch-TensorRT, XLA; narrower EP coverage |
| **Operator coverage** | ONNX opset 25 (~200 ops); lags behind latest PyTorch ops by months | Full ATen operator set; same-day support for new PyTorch ops |
| **Dynamic shapes** | Supported but can be fragile; some EPs struggle | First-class support via `torch.export.Dim` |
| **Optimization tooling** | Graph opts, quantization (INT8/FP16), `onnxsim`, Olive pipelines | `torch.compile` with Inductor fusions; `torch.ao.quantization` |
| **Serialization** | `.onnx` file, fully portable | `ExportedProgram` serializable but PyTorch-ecosystem only |
| **Debugging** | Opaque protobuf graph; Netron for visualization | Source-level debugging, `TORCH_LOGS`, graph break reports |
| **Startup latency** | One-time session load; optimized graph reusable across runs | `torch.compile` recompiles each session (no serialization yet) |

### Performance: it depends on your model and hardware

On GPU with standard architectures (ResNet, BERT, GPT-2), benchmarks typically show ONNX Runtime with CUDA EP delivering inference ~20–30% faster than vanilla PyTorch eager mode. However, `torch.compile` with Inductor can match or exceed ORT performance on the same hardware, especially for transformer models where Inductor's fused attention kernels are highly optimized.

The performance picture changes dramatically when you factor in **hardware acceleration via Execution Providers**. ONNX → TensorRT EP can deliver 10–30% additional latency reduction through aggressive operator fusion and precision optimization — though setting this up adds complexity. The native PyTorch alternative, Torch-TensorRT, achieves similar speedups while staying entirely within the PyTorch ecosystem.

For **CPU-only deployment** (common in speech-on-edge scenarios), ONNX Runtime's MLAS library is exceptionally well-optimized and often outperforms PyTorch CPU inference by 2–4×. This is where ONNX still clearly shines.

### When to choose which

**Choose ONNX when:**
- You need to deploy without a PyTorch dependency (C++ services, mobile apps, browsers, embedded systems)
- Your target is non-NVIDIA hardware (Intel via OpenVINO EP, Qualcomm via QNN EP, Apple via CoreML EP, AMD via DirectML)
- You're building a multi-language stack (C#/.NET backend, Java Android app, JavaScript web app)
- CPU inference is your primary target
- You need a single artifact that runs identically across development, staging, and diverse production hardware

**Choose `torch.compile` / `torch.export` when:**
- You're deploying on GPU in a Python-first environment and want minimal friction
- Your model uses cutting-edge PyTorch operations that ONNX doesn't support yet
- You need data-dependent dynamic control flow (if/else on tensor values)
- You want same-day compatibility with the latest model architectures
- Debugging and iteration speed matter more than cross-platform portability

**The hybrid approach** is increasingly common: use `torch.compile` for rapid GPU-based iteration and A/B testing in Python, then export to ONNX when you need to ship to edge devices, non-Python environments, or heterogeneous hardware fleets. Since `torch.onnx.export(dynamo=True)` uses `torch.export` under the hood, the two paths are converging — think of ONNX as one of several possible output formats from PyTorch's unified export engine.

---

## The core components of the ONNX ecosystem

### ONNX Runtime: the inference engine

**ONNX Runtime (ORT)** is the high-performance, cross-platform inference engine that makes ONNX practical. Maintained by Microsoft and MIT-licensed, it provides the runtime that actually executes ONNX graphs with hardware-specific optimizations. ORT's key innovation is its **Execution Provider (EP) architecture**: pluggable hardware backends that claim the graph nodes they can accelerate, with automatic fallback.

When you create a session, ORT presents the model graph to each EP in priority order. Each EP claims nodes it handles optimally; unclaimed nodes cascade to the next EP. `CPUExecutionProvider` always serves as the final fallback and supports all operators. This means you can safely specify a GPU EP first — any unsupported ops automatically run on CPU.

| Category | Execution Provider | Notes |
|---|---|---|
| CPU | CPUExecutionProvider (MLAS) | Default, always available |
| CPU | OpenVINOExecutionProvider | Intel-optimized CPU/GPU/VPU |
| GPU | CUDAExecutionProvider | NVIDIA GPU (requires CUDA + cuDNN) |
| GPU | TensorrtExecutionProvider | NVIDIA TensorRT (best NVIDIA perf) |
| GPU | DirectMLExecutionProvider | Windows GPU (AMD/NVIDIA/Intel via DirectX 12) |
| GPU | ROCMExecutionProvider | AMD GPU |
| Mobile | CoreMLExecutionProvider | Apple Neural Engine/GPU/CPU |
| Mobile | NNAPIExecutionProvider | Android neural network API |
| Mobile | QNNExecutionProvider | Qualcomm NPU/DSP |
| Specialized | VitisAIExecutionProvider | Xilinx/AMD FPGA |

### Converters, Model Zoo, and optimization tools

**Converters** bridge each framework to ONNX. `torch.onnx.export()` handles PyTorch (with the newer Dynamo-based path now the default). `tf2onnx` converts TensorFlow/Keras models via CLI or Python API (note: it is currently seeking a new maintainer). `skl2onnx` converts scikit-learn models. The old `keras2onnx` is deprecated — use `tf2onnx` instead.

The **ONNX Model Zoo** (formerly at `github.com/onnx/models`) was deprecated in July 2025 and migrated to **`huggingface.co/onnxmodelzoo`**. It hosts pre-trained ONNX models across image classification (ResNet, EfficientNet), object detection (YOLO, SSD), NLP (BERT, GPT-2), and more.

**Optimization tools** include ORT's built-in graph optimizations (constant folding, operator fusion), **ONNX Simplifier** (`onnxsim`) for cleaning export artifacts, the `onnxruntime.quantization` module for INT8/FP16 quantization, and **Microsoft Olive** for end-to-end optimization pipelines. The **onnxconverter-common** library provides FP16 conversion and auto mixed-precision tools.

---

## How to export models to ONNX

### PyTorch export: the Dynamo path is now standard

Since PyTorch 2.5, the recommended export approach uses `torch.onnx.export(..., dynamo=True)`, which leverages `torch.export` and FX graph capture instead of the legacy TorchScript path. The Dynamo exporter handles a wider range of Python code, uses less memory during export, and produces cleaner graphs via `onnxscript` translation. As of PyTorch 2.9+, `dynamo=True` is the default.

```python
import torch
import torch.nn as nn

class SimpleModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(3, 16, 3, padding=1)
        self.relu = nn.ReLU()
        self.fc = nn.Linear(16 * 32 * 32, 10)

    def forward(self, x):
        x = self.relu(self.conv1(x))
        x = x.view(x.size(0), -1)
        return self.fc(x)

model = SimpleModel()
model.eval()  # Critical: always set to eval mode before export

example_input = (torch.randn(1, 3, 32, 32),)

# Dynamo-based export (recommended)
onnx_program = torch.onnx.export(
    model,
    example_input,
    "model.onnx",
    input_names=["input"],
    output_names=["output"],
    dynamic_axes={"input": {0: "batch_size"}, "output": {0: "batch_size"}},
    dynamo=True
)
```

For more precise dynamic shape control with the Dynamo exporter, use the `dynamic_shapes` parameter with `torch.export.Dim`:

```python
from torch.export import Dim
batch = Dim("batch", min=1, max=128)
dynamic_shapes = {"x": {0: batch}}

torch.onnx.export(model, (x,), "model.onnx", dynamo=True, dynamic_shapes=dynamic_shapes)
```

The **legacy TorchScript-based path** (with `dynamo=False`) still works but is deprecated. Its main limitation was the tracing-vs-scripting dilemma: **tracing** records a single execution path and loses data-dependent control flow, while **scripting** requires code to conform to TorchScript's restricted Python subset. The Dynamo exporter eliminates this distinction by hooking into Python's frame evaluation API.

### scikit-learn export via skl2onnx

```python
from sklearn.ensemble import RandomForestClassifier
from skl2onnx import to_onnx
import numpy as np

clf = RandomForestClassifier().fit(X_train, y_train)
onx = to_onnx(clf, X_train[:1].astype(np.float32))  # Pass sample for type inference
with open("rf.onnx", "wb") as f:
    f.write(onx.SerializeToString())
```

Important: scikit-learn uses float64 internally, but ONNX Runtime operates on **float32**. Always cast inputs to `float32` before inference.

### Common pitfalls and how to avoid them

**Dynamic axes are the #1 source of deployment failures.** Without them, the ONNX model only accepts the exact input shapes used during export. Always specify dynamic dimensions for batch size, sequence length, and any other variable dimension:

```python
dynamic_axes = {
    "input": {0: "batch_size", 1: "sequence_length"},
    "output": {0: "batch_size"}
}
```

**Unsupported operators** surface as export errors. Before debugging, try increasing the opset version — newer opsets support more operators. If an op truly isn't supported, you have three options: rewrite the model code to use ONNX-compatible ops, register a custom symbolic function (for the Dynamo exporter, use `custom_translation_table` with `onnxscript`), or fall back to ATen ops (requires runtime support). Use `torch.onnx.export(..., report=True)` to generate a diagnostic markdown report during export.

**Always validate the exported model** by comparing outputs between the original framework and ONNX Runtime:

```python
import numpy as np
import onnxruntime as ort

# PyTorch output
with torch.no_grad():
    pt_out = model(test_input).numpy()

# ONNX Runtime output
sess = ort.InferenceSession("model.onnx", providers=["CPUExecutionProvider"])
ort_out = sess.run(None, {sess.get_inputs()[0].name: test_input.numpy()})[0]

np.testing.assert_allclose(pt_out, ort_out, rtol=1e-03, atol=1e-05)
```

Small numerical differences (1e-5 to 1e-7) are normal. After validation, run `onnxsim model.onnx model_simplified.onnx` to clean up export artifacts — this is especially valuable before deploying to edge devices or hardware accelerators.

---

## How to run ONNX models for inference

### Python inference in five lines

```python
import onnxruntime as ort
import numpy as np

session = ort.InferenceSession("model.onnx",
    providers=['CUDAExecutionProvider', 'CPUExecutionProvider'])
input_name = session.get_inputs()[0].name
results = session.run(None, {input_name: np.random.randn(1, 3, 224, 224).astype(np.float32)})
```

Install with `pip install onnxruntime` (CPU) or `pip install onnxruntime-gpu` (GPU). These packages are mutually exclusive — uninstall one before installing the other. Since ORT 1.10, you must explicitly specify execution providers.

### Configuring session options for production

`SessionOptions` controls graph optimization, threading, memory management, and profiling:

```python
sess_options = ort.SessionOptions()

# Graph optimization (ORT_ENABLE_ALL is default and recommended)
sess_options.graph_optimization_level = ort.GraphOptimizationLevel.ORT_ENABLE_ALL

# Threading: intra-op parallelizes within operators, inter-op across operators
sess_options.intra_op_num_threads = 4    # Set to physical CPU core count
sess_options.inter_op_num_threads = 2    # Only effective in PARALLEL mode
sess_options.execution_mode = ort.ExecutionMode.ORT_SEQUENTIAL  # Default, best for most models

# Memory optimization
sess_options.enable_cpu_mem_arena = True   # Pre-allocates memory pools
sess_options.enable_mem_pattern = True     # Reuses allocation patterns

# Save optimized model for faster subsequent loads
sess_options.optimized_model_filepath = "model_optimized.onnx"

session = ort.InferenceSession("model.onnx", sess_options=sess_options,
    providers=['CUDAExecutionProvider', 'CPUExecutionProvider'])
```

For **offline optimization**, save the optimized model once, then load it in production with `ORT_DISABLE_ALL` to eliminate startup overhead. Use `ORT_PARALLEL` execution mode only for models with many independent branches; `ORT_SEQUENTIAL` is faster for most architectures.

### Execution provider configuration with options

```python
providers = [
    ('TensorrtExecutionProvider', {
        'device_id': 0,
        'trt_max_workspace_size': 2 * 1024 * 1024 * 1024,
        'trt_fp16_enable': True,
    }),
    ('CUDAExecutionProvider', {
        'device_id': 0,
        'arena_extend_strategy': 'kNextPowerOfTwo',
        'gpu_mem_limit': 4 * 1024 * 1024 * 1024,
        'cudnn_conv_algo_search': 'EXHAUSTIVE',
    }),
    'CPUExecutionProvider'
]
session = ort.InferenceSession("model.onnx", providers=providers)
```

### IO Binding eliminates GPU transfer overhead

For GPU inference, the default `session.run()` copies inputs to GPU and outputs back to CPU on every call. **IO Binding** keeps data on GPU, which can be the dominant optimization for fast models:

```python
# Create OrtValue directly on GPU
X_gpu = ort.OrtValue.ortvalue_from_numpy(input_data, 'cuda', 0)

io_binding = session.io_binding()
io_binding.bind_input('input', device_type='cuda', device_id=0,
    element_type=np.float32, shape=X_gpu.shape(), buffer_ptr=X_gpu.data_ptr())
io_binding.bind_output('output', 'cuda')

session.run_with_iobinding(io_binding)
output_gpu = io_binding.get_outputs()[0]  # Still on GPU
output_numpy = output_gpu.numpy()          # Copy to CPU only when needed
```

You can also bind directly from PyTorch CUDA tensors using `.data_ptr()`, enabling zero-copy interop between PyTorch and ORT.

### Other languages at a glance

**C++** uses `Ort::Env`, `Ort::Session`, and `Ort::Value` with explicit tensor creation via `Ort::Value::CreateTensor<float>()`. **C#** uses the `Microsoft.ML.OnnxRuntime` NuGet package with `InferenceSession` and `OrtValue`. **Java** uses the `com.microsoft.onnxruntime` Maven dependency with `OrtEnvironment` and `OrtSession`. **JavaScript** runs in browsers via `onnxruntime-web` (WebGPU, WebAssembly, or WebGL backends) and in Node.js via `onnxruntime-node` — both share the same `InferenceSession.create()` API. **React Native** has a dedicated `onnxruntime-react-native` package for mobile apps.

---

## How to optimize ONNX models

### Graph optimizations happen automatically

ORT applies graph optimizations at session load by default (`ORT_ENABLE_ALL`). **Basic optimizations** include constant folding, redundant node elimination (Identity, Dropout, etc.), and semantic-preserving fusions (Conv+BatchNorm, Conv+Add). **Extended optimizations** add complex fusions: MatMul+Add, GELU fusion, Layer Normalization fusion, Attention fusion, Skip Layer Normalization fusion — these are especially impactful for transformer models. **Layout optimizations** transform between NCHW and NCHWc for better CPU vectorization.

These optimizations run in "online mode" during session initialization. For production, use offline mode to pre-optimize once and avoid the startup cost.

### Quantization: dynamic, static, and QAT

Quantization maps FP32 values to INT8/UINT8, achieving ~**4× model size reduction** and faster integer arithmetic. ONNX Runtime's quantization module supports three approaches.

**Dynamic quantization** pre-quantizes weights offline and computes activation scales at runtime. It requires no calibration data and works best for transformer/RNN models:

```python
from onnxruntime.quantization import quantize_dynamic, QuantType

quantize_dynamic(
    model_input="model.onnx",
    model_output="model_int8.onnx",
    weight_type=QuantType.QInt8
)
```

**Static quantization** quantizes both weights and activations offline using a calibration dataset to determine activation ranges. It provides better hardware accelerator compatibility and no runtime overhead:

```python
from onnxruntime.quantization import quantize_static, CalibrationDataReader, QuantFormat, QuantType

class MyDataReader(CalibrationDataReader):
    def __init__(self, data, input_name):
        self.data, self.input_name, self.idx = data, input_name, 0
    def get_next(self):
        if self.idx >= len(self.data): return None
        result = {self.input_name: self.data[self.idx]}
        self.idx += 1
        return result

# Pre-process first: python -m onnxruntime.quantization.preprocess --input model.onnx --output model_prep.onnx
data_reader = MyDataReader(calibration_samples, "input")
quantize_static("model_prep.onnx", "model_static_int8.onnx", data_reader,
    quant_format=QuantFormat.QDQ, activation_type=QuantType.QUInt8, weight_type=QuantType.QInt8)
```

Use the **QDQ format** (QuantizeLinear/DequantizeLinear pairs) over QOperator — it's more flexible and works with more hardware backends including TensorRT and NNAPI.

**Quantization-Aware Training (QAT)** inserts fake quantization during training so the model learns to compensate for quantization error. QAT is done in the original framework (PyTorch's `torch.quantization.prepare_qat`), then the quantized model is exported to ONNX in QDQ format.

### FP16, BFloat16, and mixed precision in ONNX

Reduced-precision floating point is one of the most impactful optimizations for inference, but the two 16-bit formats — **Float16 (FP16)** and **BFloat16 (BF16)** — have fundamentally different properties and very different levels of ONNX ecosystem support.

#### Understanding the bit-level tradeoff

Both formats use 16 bits but allocate them differently:

| Property | FP32 | FP16 (IEEE 754) | BFloat16 |
|---|---|---|---|
| Sign bits | 1 | 1 | 1 |
| Exponent bits | 8 | 5 | 8 |
| Mantissa bits | 23 | 10 | 7 |
| Dynamic range | ±3.4 × 10³⁸ | ±65,504 | ±3.4 × 10³⁸ |
| Decimal precision | ~7 digits | ~3–4 digits | ~2–3 digits |
| Memory per element | 4 bytes | 2 bytes | 2 bytes |

The critical difference is **range vs. precision**. FP16 has 10 mantissa bits, giving it higher numerical precision (more significant digits), but its 5-bit exponent limits it to a narrow range — values larger than ~65,504 overflow to infinity, and very small gradients underflow to zero. This is why FP16 training requires **loss scaling** to keep gradient magnitudes in the representable range.

BFloat16 takes the opposite approach: it keeps FP32's full 8-bit exponent (identical dynamic range of ~10⁻³⁸ to ~10³⁸) while sacrificing mantissa precision to just 7 bits. This means BF16 can represent the same magnitude of numbers as FP32 — no overflow, no underflow — but with coarser granularity between adjacent values. Converting FP32 → BF16 is trivial (just truncate the lower 16 mantissa bits), whereas FP32 → FP16 can produce overflow errors.

#### What this means for inference

For **training**, BF16 has become the default for most large models because its FP32-equivalent range eliminates the need for loss scaling and avoids numerical instability — it's essentially a drop-in replacement for FP32 in most training pipelines. Google TPUs, NVIDIA Ampere+ GPUs, and Intel AMX all have native BF16 hardware support.

For **inference**, the picture is more nuanced:

- **FP16 is the mature, well-supported choice for ONNX inference.** ORT has comprehensive FP16 operator coverage across all major Execution Providers (CUDA, TensorRT, DirectML, CoreML). The conversion tooling is battle-tested. FP16's higher precision (10 vs. 7 mantissa bits) often produces more accurate inference results than BF16, because inference doesn't face the overflow/underflow issues that make training in FP16 difficult.

- **BF16 support in ONNX is still maturing.** While the ONNX spec has included BFloat16 as a data type since opset 13, and opset 22 significantly expanded BF16 operator coverage (adding BF16 variants for Conv, BatchNorm, MaxPool, and many others), **runtime support lags behind the spec**. As of early 2026, running BF16 Conv layers in ONNX Runtime with CUDA EP still produces errors for some operators — the operator kernels simply haven't been implemented yet. TensorRT has broader BF16 support as a listed data type, but not all operators accept it. CPU inference with BF16 requires hardware support (Intel AMX, ARM SVE2) and faces similar operator coverage gaps.

- **A practical issue with BF16 in ONNX:** NumPy does not natively support bfloat16, which creates friction when preparing inputs and reading outputs through the Python API. The ONNX library has added helper functions like `float32_to_bfloat16()`, but the workflow is less seamless than FP16.

#### Practical recommendation for speech engineers

For most ONNX speech deployments today, **FP16 is the pragmatic choice**. It halves model size and memory bandwidth, leverages GPU Tensor Cores (available since NVIDIA Volta), and has rock-solid ecosystem support. Use `onnxconverter-common` for conversion:

```python
from onnxconverter_common import float16
model_fp16 = float16.convert_float_to_float16(onnx.load("model.onnx"), keep_io_types=True)
onnx.save(model_fp16, "model_fp16.onnx")
```

The `keep_io_types=True` parameter is important — it keeps inputs and outputs as FP32, casting only internal operations to FP16. This avoids needing to convert your audio preprocessing pipeline to FP16.

If your model was **trained in BF16** (increasingly common with modern speech transformers), be aware that converting BF16-trained weights → FP16 for ONNX inference can introduce small numerical differences due to the different precision characteristics. In most speech tasks (ASR, enhancement, TTS), these differences are inaudible. However, if you're seeing quality regressions after FP16 conversion, try comparing against FP32 inference to isolate whether the issue is precision-related.

For **GPU deployment with NVIDIA Ampere or newer**, you could also keep the model in FP32 and let TensorRT EP or CUDA EP handle precision automatically via mixed-precision inference — this often produces the best quality/speed tradeoff without manual conversion.

For transformer models specifically, ORT's transformer optimizer provides combined optimization and FP16 conversion:

```python
from onnxruntime.transformers import optimizer
opt_model = optimizer.optimize_model("bert.onnx", model_type='bert', num_heads=12, hidden_size=768)
opt_model.convert_float_to_float16()
opt_model.save_model_to_file("bert_fp16.onnx")
```

### ONNX Simplifier cleans export artifacts

`onnxsim` performs constant propagation across the graph, replacing redundant operators with their constant outputs. PyTorch export often produces verbose graphs with excessive Shape/Gather/Unsqueeze operations that `onnxsim` eliminates:

```python
import onnx
from onnxsim import simplify

model = onnx.load("model.onnx")
model_simplified, check = simplify(model)
assert check, "Simplified model failed validation"
onnx.save(model_simplified, "model_simplified.onnx")
# Typical result: 30-50% fewer nodes with identical outputs
```

Run `onnxsim` after export, before quantization or deployment. It is especially critical when targeting edge runtimes like ncnn, TensorRT, or OpenVINO.

### Profiling identifies actual bottlenecks

```python
sess_options = ort.SessionOptions()
sess_options.enable_profiling = True

session = ort.InferenceSession("model.onnx", sess_options)
# Run inference...
profile_file = session.end_profiling()  # Returns path to JSON file

# Analyze: find the slowest operators
import json
with open(profile_file) as f:
    events = [e for e in json.load(f) if e.get('cat') == 'Node']
events.sort(key=lambda x: x.get('dur', 0), reverse=True)
for e in events[:10]:
    print(f"{e['name']}: {e['dur']}μs ({e.get('args', {}).get('provider', '?')})")
```

The profiling output uses Chrome Tracing format — open it in `chrome://tracing` for visual analysis. Look for operators consuming disproportionate time, Cast/MemcpyToHost nodes indicating unnecessary CPU-GPU transfers, and unfused operator sequences that should be fused. Use **Netron** (`netron.app`) to visualize and compare original vs. optimized graphs.

**The recommended optimization pipeline** is: Export → Simplify with `onnxsim` → Apply graph optimizations (automatic in ORT) → Quantize (dynamic for transformers, static for CNNs) or convert to FP16 for GPU → Tune threads and EP settings → Profile → Iterate.

---

## Speech-specific considerations for ONNX

Speech models present unique ONNX challenges: variable-length audio inputs, stateful streaming inference, multi-component pipelines, and custom signal processing operations. This section covers the practical patterns that make speech ONNX deployment work.

### Exporting speech models: architecture matters

**Whisper** is an encoder-decoder model that must be split into separate ONNX files for efficient inference. The encoder processes a fixed-size mel spectrogram (`[batch, 80, 3000]` for 30 seconds), while the decoder autoregressively generates tokens. For practical performance, the decoder is further split into two variants: one without KV cache (first token) and one with KV cache (subsequent tokens). **Without KV caching, ONNX Whisper is ~4× slower than PyTorch** — including the cache as explicit inputs/outputs is essential.

The most practical export paths are **HuggingFace Optimum** (which handles the encoder/decoder/decoder_with_past split automatically) and **sherpa-onnx's export scripts** (which produce deployment-ready models with quantized variants):

```python
# HuggingFace Optimum approach
from optimum.onnxruntime import ORTModelForSpeechSeq2Seq
from transformers import AutoProcessor, pipeline

model = ORTModelForSpeechSeq2Seq.from_pretrained("optimum/whisper-tiny.en")
processor = AutoProcessor.from_pretrained("optimum/whisper-tiny.en")
pipe = pipeline("automatic-speech-recognition", model=model,
                tokenizer=processor.tokenizer, feature_extractor=processor.feature_extractor)
result = pipe("audio.wav")
```

**Wav2Vec2, HuBERT, and WavLM** export more cleanly since they're CTC-based encoder-only models. Optimum's `ORTModelForCTC` supports `wav2vec2`, `wav2vec2-conformer`, `hubert`, `wavlm`, `sew`, `unispeech`, and `data2vec_audio`. Export via `optimum-cli export onnx --model facebook/wav2vec2-base-960h output_dir`.

**TTS models** present their own patterns. **Piper TTS is entirely ONNX-native** — all voice models ship as `.onnx` files, based on the VITS architecture, optimized with `onnxsim`, and running faster than realtime on Raspberry Pi 4. It supports **35+ languages** across quality levels (x_low through high). Coqui TTS provides `export_onnx()` for VITS models but has known issues with multi-speaker export and static quantization.

**NVIDIA NeMo** models all support ONNX export via the `Exportable` mixin — Conformer-CTC, Conformer-Transducer, Parakeet, and Canary models export with a single `model.export('model.onnx')` call. For cache-aware streaming Conformers, set `model.set_export_config({'cache_support': 'True'})` before export.

### Streaming inference requires explicit state management

ONNX models are stateless — **hidden states, attention caches, and convolutional buffers must be passed as explicit inputs and received as outputs** at each inference call. This is fundamentally different from PyTorch, where RNN states can be stored as module attributes.

```python
# Streaming speech enhancement with LSTM state management
import onnxruntime as ort

session = ort.InferenceSession("enhancement_model.onnx")

# Initialize hidden states (zeros)
h = np.zeros((2, 1, 128), dtype=np.float32)
c = np.zeros((2, 1, 128), dtype=np.float32)

for chunk in audio_stream(chunk_size=512, sample_rate=16000):
    outputs = session.run(
        ["enhanced_audio", "h_out", "c_out"],
        {"input_audio": chunk, "h_in": h, "c_in": c}
    )
    enhanced, h, c = outputs  # Carry states to next chunk
    play(enhanced)
```

Typical chunk sizes vary by application: **Silero VAD uses 30ms chunks** (512 samples at 16kHz), **DTLN uses 32ms frames** with 8ms hop, and **streaming ASR uses 160–640ms chunks** depending on architecture. The real-time constraint is simple: computation must be faster than the chunk duration (RTF < 1.0).

For transformer-based decoders, the KV cache pattern is critical. HuggingFace Optimum creates separate `decoder_with_past_model.onnx` files that accept past key/values as inputs and return updated caches. NeMo's cache-aware streaming Conformers export with cache tensors for attention and convolutional contexts.

### Navigating variable-length inputs and custom ops

**Variable-length audio** is handled through dynamic axes during export (`dynamic_axes={"input": {0: "batch", 1: "time"}}`) combined with padding strategies. Whisper pads all audio to 30 seconds. CTC models require passing input length alongside audio for proper decoding. NeMo's Exportable automatically infers dynamic axes from model `input_types`.

**STFT/iSTFT operations** are the most common custom-op challenge in speech models. STFT was not in the ONNX spec until opset 17, and even then support varies. Options are: handle STFT as pre/post-processing outside the ONNX graph (most common), use `onnxruntime-extensions` for custom audio ops, or implement STFT as a combination of standard ONNX operators. **Complex number operations** must be decomposed into real/imaginary pairs since ONNX doesn't natively support complex tensors.

**Autoregressive generation** (beam search, temperature sampling) can be embedded as an ONNX op (Microsoft's approach for Whisper) or managed externally. **ONNX Runtime GenAI** now provides a dedicated generate API for autoregressive models that is increasingly preferred over the embedded beam search approach.

### The sherpa-onnx ecosystem for production speech

**Sherpa-ONNX** deserves special attention as the most comprehensive ONNX-based speech framework available. Maintained by the k2-fsa project (next-generation Kaldi), it uses ONNX Runtime exclusively — no PyTorch dependency at runtime. It supports speech-to-text (streaming and non-streaming), text-to-speech, speaker diarization, VAD, speech enhancement, and source separation across **12 programming languages** and virtually every platform including Android, iOS, Raspberry Pi, RISC-V, and various NPUs.

Supported model families include Whisper (all sizes plus distil variants), Zipformer transducer (streaming), NeMo CTC/Transducer, Paraformer, SenseVoice, VITS/Piper TTS, and Silero VAD. The latest release (v1.12.28, February 2026) added support for FireRedASR, FunASR Nano, Google MedASR, and Qualcomm QNN NPU acceleration.

**Silero VAD** is the de facto standard for ONNX-based voice activity detection — a ~260K parameter LSTM model distributed as a single `.onnx` file that processes 30ms chunks in under 1ms on CPU:

```python
from silero_vad import load_silero_vad, read_audio, get_speech_timestamps

model = load_silero_vad()
wav = read_audio('audio.wav')
timestamps = get_speech_timestamps(wav, model, return_seconds=True)
```

### Performance reality for speech workloads

Real-world benchmarks paint a clear picture. The ESPnet-ONNX paper reports **~2× speedup for ASR and SLU tasks** and **~1.3× speedup for TTS** versus PyTorch inference. Specific latency numbers are encouraging for edge deployment:

| Model | Latency | Hardware | RTF |
|---|---|---|---|
| Silero VAD | <1ms per 30ms chunk | CPU (any modern) | <0.03 |
| DTLN enhancement | ~1.13ms per 32ms block | 2012 MacBook Air | ~0.035 |
| DCCRN-E enhancement | 3.12ms per frame | Intel i5-8250U | ~0.1 |
| Piper TTS (medium) | Faster than realtime | Raspberry Pi 4 | <1.0 |
| Whisper tiny.en | ~2× faster than PyTorch | CPU | Varies |

**Quantization impact varies by task.** For ASR, INT8 dynamic quantization on Whisper shows **<0.5% absolute WER degradation** on LibriSpeech — practically negligible. For speech enhancement, DTLN with dynamic range quantization maintains high audio quality while becoming Raspberry Pi 3B+ capable. For TTS, quantization is less effective — CNN-heavy vocoders don't benefit much, and Piper uses FP32 models but achieves good edge performance through small model sizes.

**CPU vs. GPU selection** follows model size. Small models under 10M parameters (VAD, enhancement) run best on CPU — GPU data transfer overhead dominates for such fast models. Medium models (Whisper tiny/base, small Conformers) are practical on CPU with GPU providing 3–5× additional speedup. Large models (Whisper large, large Conformers) strongly benefit from GPU with FP16.

---

## Conclusion

ONNX has matured from a framework interchange experiment into a **production-grade deployment format that excels in specific, well-defined scenarios** — particularly cross-platform deployment, CPU-optimized inference, and heterogeneous hardware targeting. It is no longer the only path from training to production (PyTorch 2.x's `torch.compile` and `torch.export` have absorbed much of ONNX's original value proposition for GPU-centric, Python-first deployments), but it remains the strongest choice when you need to escape the PyTorch runtime entirely.

The practical workflow is now well-established: export with `torch.onnx.export(dynamo=True)` or Optimum, simplify with `onnxsim`, optimize with ORT's graph optimizations, quantize or convert to FP16 for your target hardware (staying with FP16 over BF16 until BF16 runtime support matures), and profile to verify. For speech specifically, the key insight is that **stateful streaming requires explicit state management** — ONNX models are fundamentally stateless, and every hidden state, attention cache, and convolutional buffer must flow through the graph as inputs and outputs.

The most overlooked opportunity in the ONNX speech ecosystem is **sherpa-onnx**: if you're building a speech application that needs to run on diverse platforms without PyTorch as a runtime dependency, it provides a battle-tested framework covering ASR, TTS, VAD, and speaker diarization across 12 programming languages. Combined with Silero VAD for activity detection and Piper for synthesis, you can build complete voice pipelines that run on hardware as modest as a Raspberry Pi — all powered by ONNX Runtime.