# YOLO26-OBB on AMD Zynq: Feasibility and Implementation Guide

**Research date:** 2026-10-26  
**Applies to:** Ultralytics YOLO26 oriented-bounding-box (OBB) models n/s/m/l/x; AMD Zynq-7000, Zynq UltraScale+ MPSoC, and Kria K26 boards; other Zynq devices require a device-specific review.  
**Status:** Engineering assessment, not an on-board benchmark or proof of successful compilation.

## Executive answer

**Yes, YOLO26-OBB can run on appropriately equipped Zynq devices, but no unmodified YOLO26-OBB model should be assumed to run end-to-end on a Zynq DPU.**

- **YOLO26n-OBB:** Best first candidate for an ARM64 Zynq UltraScale+ MPSoC or Kria K26, particularly with a supported FPGA DPU and a modified/quantized network.
- **YOLO26s-OBB:** Reasonable second candidate on a sufficiently large Zynq UltraScale+ device; RAM, DPU resources, operator partitioning, and required latency decide feasibility.
- **YOLO26m-OBB:** Technically worth a feasibility experiment on a high-end configuration, but large at OBB's 1024-pixel reference resolution.
- **YOLO26l-OBB / YOLO26x-OBB:** Not sensible first choices for low-power, real-time Zynq deployment. They might execute on an adequately provisioned CPU at poor speed; do not equate that with practical FPGA/DPU support.
- **Zynq-7000 (e.g. XC7Z020):** A particularly constrained 32-bit Cortex-A9 family. Full YOLO26-OBB is an unfavorable target; use it only for carefully scoped offline/low-rate experiments or a custom neural accelerator project.

These recommendations are **inferences from model compute, chip architecture, and supported operators**. No independently validated YOLO26-OBB n/s/m/l/x FPS, memory, power, or accuracy numbers on specific Zynq boards were found in the official references reviewed. The precise chip part number, loaded DPU hardware design, DDR, operating system, and resolution are essential to a final yes/no decision.

## 1. All five YOLO26-OBB models

Ultralytics' published OBB figures refer to the **fused inference model at 1024 × 1024** and DOTAv1 test evaluation. They are not standard (non-OBB) YOLO26 numbers.

| Model | Parameters (million) | FLOPs (billion), 1024 | DOTAv1 test mAP50-95 (e2e) | CPU ONNX latency (ms), reference system | Theoretical weights FP32 / FP16 / INT8 (MB) |
|---|---:|---:|---:|---:|---:|
| YOLO26n-OBB | 2.4 | 14.8 | 52.4 | 97.7 | 9.6 / 4.8 / 2.4 |
| YOLO26s-OBB | 9.8 | 56.7 | 54.8 | 218.0 | 39.2 / 19.6 / 9.8 |
| YOLO26m-OBB | 21.2 | 184.9 | 55.3 | 579.2 | 84.8 / 42.4 / 21.2 |
| YOLO26l-OBB | 25.6 | 232.4 | 56.2 | 735.6 | 102.4 / 51.2 / 25.6 |
| YOLO26x-OBB | 57.6 | 520.1 | 56.7 | 1485.7 | 230.4 / 115.2 / 57.6 |

**Important limitations of this table:**

1. CPU ONNX latencies were measured by Ultralytics on its reference CPU environment; **they are not ARM Cortex-A53, Cortex-A9, KV260, ZCU104, or any other Zynq measurements**.
2. The calculated weight sizes are parameter count multiplied by 4, 2, or 1 byte. They exclude activations, workspaces, graph representations, runtime, OS, buffers, preprocessing, and postprocessing; do not use these as minimum DDR requirements.
3. Checkpoint files can be larger because training architecture and metadata may remain; actual exported file sizes depend on the export and optimizer.
4. Published accuracy comes from DOTA's evaluation procedure, not from your satellite images. At 640 or 512 input resolution, small-object recall may change greatly.
5. FLOPs refer to **1024 input resolution**, so do not confuse this table with 640-pixel standard-detection metrics.

Relative to n, model x uses **about 35.1 times** as many published FLOPs (520.1/14.8), for 4.3 percentage points higher published DOTAv1 mAP50-95.

## 2. What does "runs on Zynq" mean?

There are three materially different levels:

1. **Loads:** An ARM Linux process successfully opens the model and its tensors fit in memory.
2. **Infers:** The CPU and/or DPU actually computes correct rotated boxes for test images, with matching preprocessing and OBB decoding.
3. **Meets the mission:** End-to-end latency, FPS, model accuracy, DDR headroom, power, heat, boot time, reliability, and deployment constraints are acceptable.

A result at level 1 is not evidence of level 3. Also, Zynq's FPGA fabric is **not an AI accelerator by default**: the board must have a supported DPU bitstream/hardware design loaded, plus the matching runtime. A bare Zynq CPU cannot magically use programmable logic for ONNX inference.

## 3. Zynq and related AMD hardware families

| Family / example | Application CPU | AI-acceleration path | YOLO26-OBB verdict |
|---|---|---|---|
| **Zynq-7000**, e.g. Z-7020/ZedBoard | ARM Cortex-A9, ARMv7 / 32-bit | Legacy or custom DPU/FPGA design in some configurations; not the straightforward modern Zynq UltraScale+ flow | Avoid for a full-resolution real-time YOLO26-OBB project. An n CPU proof of concept would require an ARMv7-compatible inference runtime and enough RAM. |
| **Zynq UltraScale+ MPSoC**, e.g. ZCU104 (ZU7EV) | ARM Cortex-A53, ARM64 | DPUCZDX8G in FPGA programmable logic | **n recommended, s candidate; m high-risk, l/x poor fits**. Actual outcome depends on DPU architecture, operator partitioning, and board DDR. |
| **Zynq UltraScale+ MPSoC**, e.g. ZCU102 (ZU9EG) | ARM Cortex-A53, ARM64 | DPUCZDX8G | Stronger development platform for trying n/s and carefully measuring m; not a promise of m real-time inference. |
| **Kria K26**, e.g. KV260 or KR260 | Quad-core ARM Cortex-A53, ARM64 | DPU in programmable logic with a matching image/overlay | Very good development starting point for n and possible s. The KV260 starter kit has **4 GB DDR4**. |
| **Zynq UltraScale+ RFSoC**, device-dependent | ARM Cortex-A53 plus specialized RF hardware | Possible custom FPGA inference pipeline, but device resources and DPU support must be checked | Do not infer suitability from "RFSoC" alone. Review exact part, FPGA resources, DDR and runtime support. |
| **Versal AI Edge**, e.g. VEK280/VEK385 | Newer ARM plus AI Engines | Vitis AI NPU, **not the old Zynq DPU flow** | Often a more suitable AMD path if choosing new hardware for larger/newer YOLO architecture. This is **not Zynq**. |

**Recommended development boards:** start with **Kria KV260** if you want a compact ARM + FPGA vision prototype; choose **ZCU104** or **ZCU102** when testing or customizing the DPU implementation. Board suitability for actual flight hardware is a separate environmental/radiation qualification task.

## 4. Preliminary model-versus-device decision

**Legend:** Recommended = worth implementing first; Candidate = engineering work and testing required; Experimental = high technical/latency risk; Avoid = poor architecture match for this task. No entry is a measured compatibility certification.

| Platform and execution path | n-OBB | s-OBB | m-OBB | l-OBB | x-OBB |
|---|---|---|---|---|---|
| Zynq-7000 32-bit CPU | Experimental | Avoid | Avoid | Avoid | Avoid |
| Zynq UltraScale+ A53 **CPU only**, appropriate DDR | Candidate, likely slow | Experimental, likely slower | Not practical for real-time | Avoid | Avoid |
| Kria K26 **DPU** with model adaptations | Recommended | Candidate | Experimental | Avoid | Avoid |
| ZCU104 / ZCU102 **DPU** with model adaptations | Recommended | Candidate | Experimental on larger designs | Avoid | Avoid |
| Larger **custom** UltraScale+ design with substantial DDR, PL resources and accepted low FPS | Candidate | Candidate | Research experiment | Research experiment | Research experiment |

An unmodified model may fail at quantization/compilation or require excessive ARM CPU fallback even when the parameter count is small. The OBB angle/decode head makes this **more involved than ordinary YOLO26 horizontal-box detection**.

## 5. Why stock YOLO26-OBB cannot simply be compiled onto the Zynq DPU

The Zynq UltraScale+ DPUCZDX8G is optimized mainly for **INT8 convolutions** and a specific supported operator set.

| YOLO26/OBB operation | Likely location for a Zynq UltraScale+ DPU design | Engineering implication |
|---|---|---|
| Convolution and supported pooling | FPGA DPU | Main target for acceleration. |
| Default **SiLU** activation | CPU if left unchanged | Replace with a DPU-compatible activation such as **Hard-Swish**; then retrain/fine-tune and verify accuracy. |
| **MatMul / Softmax** inside two YOLO26 attention stages | CPU subgraphs | Can cause costly DPU-to-CPU transitions. Consider attention-free architecture if needed. |
| Concat, some upsampling, Add | DPU if shapes/partitioning comply | Inspect compiler report; not guaranteed for every graph layout. |
| **TopK / GatherElements** in the optional NMS-free head | CPU | The NMS-free path still has non-DPU work. |
| Rotated-box decode, trigonometric processing, class filtering | Usually ARM CPU/custom implementation | Handle OBB outputs correctly; check cosine/sine and input/output transformations. |
| Rotated NMS for one-to-many outputs | ARM CPU or custom FPGA kernel | Do **not** replace rotated NMS with ordinary horizontal-box NMS. |

**Special OBB warning:** The Ultralytics **OBB26** head outputs **raw angle predictions** rather than applying the sigmoid used by older OBB heads. Its box decoding uses trigonometry (cosine and sine). Do not blindly copy YOLOv8-OBB or YOLO11-OBB decode code into a YOLO26-OBB application. Use a reference implementation and compare corner coordinates, not just rectangular boxes.

**NMS/export distinction:** According to Ultralytics, leaving the export's nms argument unset produces the native one-to-many/raw-output path for external postprocessing; explicitly setting nms=False selects the available one-to-one NMS-free head. In either case, inspect the *actual exported ONNX graph*, because the Zynq DPU will not accelerate every end-of-network operation. Choose the path that minimizes CPU work and passes accuracy checks. Avoid nms=True for the DPU pipeline.

## 6. Implementation method A — CPU-only proof of concept (simplest)

**Purpose:** Prove training weights, dataset, export, inference correctness, and OBB output. This is the right starting point even if you intend to add the FPGA DPU later.

### Hardware and software

- Development machine: ordinary Linux/Windows x86-64 computer, Python and Ultralytics. A GPU is helpful for training; export and basic tests can run on CPU.
- Target Zynq: preferably 64-bit ARM Linux, compatible glibc/Python and **sufficient free DDR**. ARM64 ONNX Runtime is supported; **ARMv7/32-bit Zynq-7000 may require a custom runtime build or a different native inference engine**.
- Use image input 640 × 640 or 512 × 512 for first feasibility tests; verify whether reduced resolution still resolves your smallest satellite targets. The published table uses 1024.

### A1. Install and run the baseline on the development computer

~~~bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install ultralytics onnxruntime onnx

# Initial functional test: needs internet on first use to fetch pretrained weights.
yolo obb predict model=yolo26n-obb.pt source=sample.jpg imgsz=640 device=cpu save=True

# Export a fixed-size ONNX graph for a single image.
yolo export model=yolo26n-obb.pt format=onnx imgsz=640 batch=1 dynamic=False opset=17 nms=False

# Check exported model on the host.
yolo obb predict model=yolo26n-obb.onnx source=sample.jpg imgsz=640 device=cpu save=True
~~~

**Windows note:** activate with your Windows venv's Scripts directory rather than the Linux source command. Use a Linux x86-64 host or VM/container for the later AMD toolchain steps. On a computer with no internet, transfer the model, compatible wheels/runtime, and dependencies ahead of time.

### A2. Inspect the ONNX operators on the host

~~~python
import collections
import onnx

model = onnx.load("yolo26n-obb.onnx")
onnx.checker.check_model(model)
print("ONNX input:", [(x.name, x.type.tensor_type.shape) for x in model.graph.input])
print("ONNX output names:", [x.name for x in model.graph.output])
print("Operator counts:")
for name, count in sorted(collections.Counter(n.op_type for n in model.graph.node).items()):
    print(name, count)
~~~

This confirms the actual graph's operators. It does **not** establish how AMD's compiler will partition or accelerate them.

### A3. Transfer to ARM64 Zynq Linux

1. Build/test on host first; copy the exported ONNX file, sample input and metadata to the board.
2. Verify architecture (aarch64), OS and memory; install a matching ARM64 ONNX Runtime. If a Python wheel is unavailable for the specific board environment, build ONNX Runtime or deploy its C/C++ runtime.
3. Implement the **same letterbox resize, channel ordering, normalization, bounding-box/angle decode, score thresholding, and scaling back to original pixels** as the Ultralytics reference.
4. Compare at least 20–50 representative images, including rotated and very small objects; then measure wall-clock latency and maximum resident memory on the board.
5. A high-level Python Ultralytics inference path is convenient for reference on the host but can be too heavy on a small embedded board. Prefer **ONNX Runtime C++** or **NCNN C++**, with your own verified OBB postprocess.

**Alternative ARM CPU runtime:** Ultralytics lists NCNN as an export format for YOLO26-OBB. Export on the host with:

~~~bash
yolo export model=yolo26n-obb.pt format=ncnn imgsz=640
~~~

Then run the exported NCNN model in a matching ARM build; confirm supported operators and output decoding on that specific target. NCNN is not a Zynq FPGA/DPU compiler: this path uses the CPU unless you implement another acceleration backend.

## 7. Implementation method B — FPGA/DPU acceleration on Zynq UltraScale+ or Kria

**Purpose:** Run as many supported INT8 network operations as possible on the FPGA DPU while the Cortex-A53 handles capture, preprocessing, unsupported operations, and OBB postprocessing.

### B0. Required tools and deployment flow

**Use the legacy Zynq DPU stack**, not the direct Ultralytics AMD Xilinx exporter intended for newer Versal AI Edge Gen 2 devices.

**Reference flow:**

~~~text
Train/fine-tune YOLO26n-OBB on a host
        |
        v
Adapt SiLU -> Hard-Swish, review attention and head
        |
        v
Calibrate / quantize to INT8 with AMD vai_q_pytorch
        |
        v
Compile for the exact DPU arch.json -> compiled .xmodel
        |
        v
Boot Zynq Linux with matching DPU bitstream, drivers, and VART
        |
        v
DPU/CPU graph execution -> decode oriented boxes on ARM -> evaluate
~~~

AMD's established Zynq UltraScale+ environment is documented with Vitis AI 3.0 board images for ZCU102, ZCU104 and KV260; the Ultralytics integration documentation describes the frozen/legacy DPU tooling as Vitis AI 3.5. **Pin a compatible AMD toolchain, quantizer, compiler, board image and DPU hardware design as a set.** Do not randomly combine newer Vitis AI NPU releases with legacy DPU bitstreams.

**Very important:** Ultralytics' current model.export(format="xilinx") output is a Quark-quantized ONNX flow for **Versal AI Edge Series Gen 2**, not a ready-to-run Zynq DPUCZDX8G xmodel.

### B1. Select DPU-capable hardware

- A supported Zynq UltraScale+ board, such as **ZCU104**, **ZCU102**, or **KV260**.
- Enough FPGA resources for your DPU configuration **plus** camera/IO/video custom logic. A DPU is configurable; using more DPU engines consumes more programmable-logic resources and may affect timing/power.
- A compatible SD/eMMC image and PetaLinux/Ubuntu layer with **VART**, the DPU driver, and the precise DPU bitstream or overlay.
- The DPU configuration's **arch.json**. It must match the loaded hardware fingerprint. A compiled xmodel from another DPU variant may not run.
- Host computer: Linux x86-64 with the matching Vitis AI Docker/conda environment. You train/quantize/compile on this host; the Cortex-A53 runs the result.

### B2. Adapt the OBB architecture to DPU-compatible activations

Start from Ultralytics' **OBB** YAML, not the standard detection YAML:

Copy it to a custom YAML (for example, yolo26n-obb-hswish.yaml) and retain the **OBB26** output head. Under the YAML parameters add the following, while retaining other parameters such as scales, reg_max and end2end:

~~~yaml
nc: 80
scale: n
activation: nn.Hardswish()
end2end: True
reg_max: 1
~~~

If your project has a different class count, train with the dataset-specific nc/classes. Validate that the model actually uses scale n before training.

Train/fine-tune on the development GPU:

~~~python
from ultralytics import YOLO

# The custom YAML is based on yolo26-obb.yaml and keeps OBB26 at its final layer.
model = YOLO("yolo26n-obb-hswish.yaml").load("yolo26n-obb.pt")
model.train(data="my_obb_dataset.yaml", epochs=100, imgsz=640, device=0)
model.val(data="my_obb_dataset.yaml", imgsz=640, device=0)
~~~

Changing SiLU to Hard-Swish changes inference numerics, so do not merely swap an activation in the exported graph and assume unchanged accuracy. Fine-tune and quantify the accuracy difference.

**Attention decision:**

- **Option 1: keep attention.** Let the compiler partition MatMul/Softmax operators to the CPU when possible. Use VART Graph Runner if CPU subgraph kernels exist. Measure the DPU/CPU transition cost.
- **Option 2: attention-free custom OBB architecture.** Replace layer 10's C2PSA with nn.Identity and disable attention in layer 22's C3k2 arguments, following the documented YOLO26 adaptation strategy. **Keep the final OBB26 head and its arguments [nc, 1]** (the generic detection example uses Detect, which would lose the angle head). Transfer compatible weights and retrain sufficiently because some attention weights no longer apply.
- **Option 3:** If these changes are uneconomical, benchmark an earlier DPU-friendly detection architecture and assess whether your OBB requirements can be preserved. Do not silently replace OBB with ordinary boxes if orientation is mission-critical.

For small satellite objects, changing architecture, image size or output head requires a new accuracy study.

### B3. Quantize with AMD's PyTorch quantizer

Vitis AI's **vai_q_pytorch** API uses a calibration pass, a test pass, then xmodel export. This is an **integration scaffold, not a turnkey YOLO26-OBB quantizer**: the network must first be wrapped/partitioned so its forward return signature and unsupported OBB/attention operations are accepted by the selected AMD tool version.

~~~python
import torch
from pytorch_nndct.apis import torch_quantizer
from ultralytics import YOLO

weights = "runs/obb/train/weights/best.pt"  # replace with actual fine-tuned weights
dummy = torch.randn(1, 3, 640, 640)

# IMPORTANT: Replace with a device-specific, verified DPU-compatible
# torch.nn.Module wrapper. A raw Ultralytics OBB network may not quantize
# or export correctly because of its heads, multiple outputs, and CPU ops.
net = YOLO(weights).model.cpu().eval()

qcal = torch_quantizer(
    "calib", net, (dummy,),
    output_dir="quantize_result",
    device=torch.device("cpu"),
)

# Implement an actual loader giving float32 [1,3,640,640] tensors,
# letterboxed/normalized exactly like inference.
for image_tensor in representative_calibration_batches():
    _ = qcal.quant_model(image_tensor)
qcal.export_quant_config()

# Reload the original FP32 model before the test-mode quantizer.
net2 = YOLO(weights).model.cpu().eval()
qtest = torch_quantizer(
    "test", net2, (dummy,),
    output_dir="quantize_result",
    device=torch.device("cpu"),
)
for image_tensor in representative_validation_batches():
    _ = qtest.quant_model(image_tensor)

# Only after the test graph is verified and the selected operators export:
qtest.export_xmodel(deploy_check=False)
~~~

The two data iterators shown above are **placeholders you must implement**; the raw model is intentionally identified as requiring a wrapper. The script demonstrates the correct AMD API lifecycle, not an already working OBB conversion. Run calibration in the **matching Vitis AI environment** and confirm that a quantized xmodel actually exists.

Suggested calibration set: several hundred representative images, covering your cameras/satellite scenes, seasons, lighting, small-object sizes and rotations. Use separate validation images; check quantized OBB mAP and angle errors against FP32.

### B4. Compile for your exact FPGA DPU

Within the matching AMD Vitis AI compiler container, after successful quantizer xmodel export:

~~~bash
vai_c_xir \
  -x /project/quantize_result/YOUR_QUANTIZED_MODEL.xmodel \
  -a /project/board/arch.json \
  -o /project/build/compiled \
  -n yolo26n_obb
~~~

The two source paths above are **placeholders**, not universal filesystem locations. Use the name of the xmodel generated by your quantizer and the **arch.json from your actual board bitstream**. Inspect all warnings, unsupported operations and DPU/CPU partitions. A successful command does not guarantee all layers are on the DPU or that the complete model has a runnable CPU fallback.

### B5. Run on the board with VART and ARM postprocessing

1. Boot an image containing the matching DPU hardware design and VART libraries.
2. Ensure the target DPU fingerprint agrees with arch.json and compiled xmodel.
3. Copy compiled xmodel, metadata, label names and test images to the board.
4. Use **VART Graph Runner** if the compiled model contains both CPU and DPU subgraphs; register missing CPU/custom operators if required. Alternatively, execute a DPU subgraph and implement the remaining stages manually.
5. Query tensor shapes, layouts and **fix_point** quantization factors at runtime. Convert input to the right INT8 representation; dequantize outputs correctly. Do not assume float tensors or generic NCHW layout.
6. Decode **oriented bounding boxes**, preserving angle convention, 4-point corners and coordinate transformations. Apply rotated NMS when using the one-to-many path, or the appropriate TopK/selection stage for the one-to-one NMS-free path.
7. Validate output against host Ultralytics on the exact same images and class list.
8. Benchmark the **entire** pipeline, not just DPU kernel execution.

**Illustrative software boundaries:**

~~~text
Camera / image DMA
      |
      v
ARM: letterbox, BGR-to-RGB, normalize, quantize
      |
      v
FPGA DPU: supported convolutions / feature extraction / supported heads
      |
      +--> ARM: attention/other unsupported subgraphs, if present
      |
      v
ARM: dequantize, class filtering, OBB angle + rotated-box decode
      |
      v
ARM: rotated NMS or one-to-one selection; restore original coordinates
      |
      v
Coordinates, corner polygons, labels, confidence
~~~

**Key risk:** Many DPU-to-CPU partitions may cost more than anticipated. The optimal solution may be a modified/attention-free n-OBB model with relatively few CPU fallback stages, not an unmodified larger model.

## 8. Implementation method C — custom FPGA / high-end embedded design

Use this only if method B cannot meet the required accuracy or latency.

1. **Hardware:** choose a specific Zynq UltraScale+ with PL DSPs/BRAM/URAM and DDR bandwidth that can support your model's working set and video capture pipeline.
2. **Partition:** map backbone/neck convolutions to the DPU or your own FPGA HLS/RTL kernels. Keep angle handling, unsupported attention and OBB postprocessing on ARM or implement them in custom PL kernels.
3. **Precision:** test INT8 or mixed fixed-point precision. Keep sensitive box/angle stages in enough precision to preserve rotated IoU and small-object localization.
4. **DDR and DMA:** pipeline camera capture, feature-map traffic and output transfers; profile bandwidth and contention between PL and PS.
5. **Architecture:** to obtain greater speed, favor model/attention pruning, smaller class count, tiling and INT8 calibration before attempting m/l/x.
6. **Verification:** compare bit-exact or tolerance-bounded outputs with FP32 on a validation suite, including difficult rotation angles.

This is a **custom hardware/software engineering project**. Merely having enough space for INT8 weights does not prove all intermediate feature maps or operators fit the FPGA resource budget.

## 9. Satellite and aerial image deployment considerations

If the final application is on a satellite, drone, or other ARM-based observation device:

- **Resolution versus small objects:** The reference benchmark uses 1024. Running 512 or 640 helps compute, but ships/vehicles/buildings may become too small. Compare per-class recall at target size.
- **Tiling:** Process a large image in overlapping tiles (for example, 640 × 640), then transform 4-corner OBB coordinates to full-frame coordinates and merge duplicates using **rotated** overlap logic.
- **Orientation:** Verify angle units (radians), long-edge/short-edge conventions and any wraparound. Compare corners, not just angle scalars.
- **Offline operation:** Stage the model, board bitstream, runtime, packages, fonts/labels as needed, and calibration data before deployment. First use of Ultralytics pretrained names may trigger downloads on the connected training host.
- **Reliability:** Use watchdogs, bounded memory queues, thermal throttling checks, recovery on failed inference and reproducible builds.
- **Spaceflight warning:** Commercial development boards such as KV260/ZCU104 are not automatically radiation-qualified flight computers. Assess radiation, ECC, single-event effects, vibration, thermal vacuum, mission safety, and power separately.

## 10. Practical board and resource selection

The figures below are **development planning heuristics**, not official minimums, validated model requirements or guarantees:

| Goal | Suggested starting hardware | What must be proved |
|---|---|---|
 Correctness / first demo | Linux x86-64 host; then K26 or other ARM64 board CPU | ONNX/NCNN model loads, correct OBB results, RAM/latency measured |
 Best initial Zynq implementation | **KV260 4 GB**, YOLO26n-OBB, optimized DPU version | DPU deployment, CPU fallback overhead, rotated output fidelity |
 More DPU experimentation | **ZCU104 / ZCU102**, n-OBB and s-OBB | Correct arch.json, PL resource fit, sustained performance |
 High-accuracy but low-rate batch inspection | High-end Zynq UltraScale+ with adequate DDR; explore s/m | Whether full pipeline latency is acceptable |
 New design targeting large modern models | Evaluate **Versal AI Edge** or a more AI-focused SoC | NPU operator coverage, model export, performance-per-watt |

For a practical development setup, 4 GB board DDR and a suitable boot/storage image are convenient starting points, but even 4 GB does **not** guarantee 1024 inference or large-model throughput. High-level Python dependencies can significantly increase memory usage compared with a native C++ runtime.

**No credible Zynq FPS range should be promised before measurement.** Peak theoretical TOPS does not capture unsupported operators, DDR transfers or rotated postprocessing.

## 11. Test and acceptance checklist

Before deciding that any particular n/s/m/l/x model "runs" on the board, record:

### Compatibility

- [ ] Exact AMD board and silicon part number, ARM architecture and Linux image.
- [ ] Exact DDR capacity and usable free RAM.
- [ ] DPU bitstream/overlay loaded; DPU variant/fingerprint recorded.
- [ ] Vitis AI quantizer, compiler, runtime and model versions pinned.
- [ ] ONNX operators and compiler CPU/DPU partition report saved.
- [ ] End-to-end execution succeeds with no unimplemented operators.

### Accuracy

- [ ] Host FP32 OBB baseline accuracy on **your** test set.
- [ ] Hard-Swish/attention-changed FP32 accuracy.
- [ ] INT8/mixed-precision OBB accuracy and per-class recall.
- [ ] Detection of difficult small, narrow and strongly rotated objects.
- [ ] Box corners/angles match host reference within agreed tolerance.
- [ ] Tiled-image output merges duplicates correctly.

### Performance

- [ ] p50/p95 end-to-end latency and sustained FPS after warmup.
- [ ] Separate camera/preprocess, DPU, CPU fallback and postprocess timings.
- [ ] Peak process RAM, DPU memory usage and available DDR headroom.
- [ ] CPU load, DDR bandwidth, PL utilization, FPGA timing closure.
- [ ] Power, temperature, thermal stability and error recovery.
- [ ] Offline reboot from a clean image works without downloading anything.

## 12. Recommended sequence for your project

**Phase 1 — Baseline (n only)**

1. Obtain yolo26n-obb.pt on the connected development machine.
2. Validate on representative aerial/satellite images at 1024, 640, and optionally 512.
3. Export to ONNX/NCNN and verify inference and 4-corner OBB results on the host.

**Phase 2 — Zynq CPU smoke test**

4. Move ONNX/NCNN and a small test set to the exact ARM64 board.
5. Implement/verify preprocessing and OBB postprocessing; record CPU-only latency/RAM.

**Phase 3 — Zynq DPU**

6. Adapt the OBB architecture for Hard-Swish and decide how to handle attention.
7. Re-fine-tune using mission images, quantize with matching Vitis AI, and compile for the board's arch.json.
8. Deploy via VART/Graph Runner, confirm partitions and match reference outputs.

**Phase 4 — Iterate**

9. Optimize n (resolution/tiling, graph partition, postprocess, buffer reuse).
10. Only if n's accuracy is insufficient and there is enough compute headroom, repeat for s.
11. Treat m as a research experiment. Do not start from l or x unless new hardware/low-rate requirements justify it.

### Final choice

For a general Zynq development project: **YOLO26n-OBB + Kria KV260 or ZCU104 + a Hard-Swish/quantized DPU path** is the best first experiment.

If selecting the processor is still open and high accuracy at 1024 pixels is non-negotiable, assess **Versal AI Edge** alongside high-end Zynq; it supports a different, more modern acceleration flow. For an existing Zynq design, provide the specific chip name (for example, **XCZU7EV**, **XCZU9EG**, or **XC7Z020**), DDR, FPGA/DPU design, input image size and target FPS before committing to a model.

## 13. Official sources (primary documentation)

**Document limitation:** No Zynq hardware was available for direct verification while preparing this guide. All final performance and compatibility decisions require compilation and measurement on the exact target board.
