**YOLO MODELS**

ARM CPU & Fully Offline Deployment Guide

YOLOv1 through YOLO26 • parameters • FLOPs • FP32 weights • model sizes • ARM hardware • NCNN/ONNX/TFLite • offline setup

| **Purpose: provide a practical reference for choosing and deploying YOLO-family object detectors on ARM64 CPU systems without Internet access. Official benchmark values are distinguished from practical deployment recommendations.** |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|

# 1. Executive Summary

For a new ARM CPU-only deployment, the most attractive starting points are YOLO26n, YOLO11n, YOLOv10n, YOLOv8n, and YOLOv5n. Nano/Tiny models are usually the correct first choice because ARM systems are constrained more by compute throughput and memory bandwidth than by storage.

| **Priority**                      | **Recommendation** | **Reason**                                                                 |
|-----------------------------------|--------------------|----------------------------------------------------------------------------|
| Best current starting point       | YOLO26n            | 2.4M params, 5.5B FLOPs, 40.9 COCO mAP; designed for efficient deployment. |
| Mature modern option              | YOLO11n            | 2.6M params, 6.5B FLOPs; broad Ultralytics tooling and export support.     |
| Efficient end-to-end family       | YOLOv10n           | 2.3M params, 6.7G FLOPs; strong efficiency.                                |
| Mature ecosystem                  | YOLOv8n            | 3.2M params, 8.7B FLOPs; extensive examples and export paths.              |
| Older / very constrained hardware | YOLOv5n            | Small, mature, widely deployed; useful where compatibility matters.        |

# 2. Core Terms You Need to Understand

## 2.1 Parameters

Parameters are learned numeric values (weights and biases) stored inside the neural network. A model listed as 2.3M parameters contains about 2.3 million learned values. More parameters generally increase representational capacity, but also increase weight storage and often memory use.

## 2.2 FLOPs / GFLOPs

FLOPs describe the amount of arithmetic performed by a forward pass. Model tables often show billions of operations (G or B) for a fixed input such as 640×640. FLOPs are not a direct FPS measurement, but they are one of the strongest first-order indicators of CPU workload.

## 2.3 Approximate FP32 weights alone

FP32 stores each parameter as a 32-bit (4-byte) floating-point value. A quick estimate is: parameters × 4 bytes. Therefore a 2.3M-parameter model contains about 9.2 MB of raw FP32 parameter data. This is not total runtime RAM: activations, image tensors, buffers, libraries, and the inference engine require additional memory.

| **Example**      | **Value**                  |
|------------------|----------------------------|
| 2.3M parameters  | ≈ 2,300,000 learned values |
| FP32 raw weights | 2.3M × 4 bytes ≈ 9.2 MB    |
| FP16 raw weights | 2.3M × 2 bytes ≈ 4.6 MB    |
| INT8 raw weights | 2.3M × 1 byte ≈ 2.3 MB     |

## 2.4 mAP

mAP (mean Average Precision) summarizes detection accuracy across object classes and IoU thresholds. COCO mAP50-95 is stricter and more informative than mAP50 alone. Compare mAP only when dataset, evaluation method, and input assumptions are comparable.

## 2.5 Input resolution

Compute rises roughly with image area. 640×640 contains 409,600 pixels, while 320×320 contains 102,400 pixels. Dropping from 640 to 320 therefore reduces pixel count by 4×, although end-to-end speedup depends on the runtime and model.

# 3. YOLO Version Timeline and Families

| **Generation**    | **Approx. era** | **What changed / why it matters**                                                                                                 |
|-------------------|-----------------|-----------------------------------------------------------------------------------------------------------------------------------|
| YOLOv1            | 2016            | Original single-stage "You Only Look Once" detector; unified detection into one network.                                          |
| YOLOv2 / YOLO9000 | 2016-2017       | Improved recall/accuracy, anchors, batch normalization, multi-scale training; YOLO9000 expanded class coverage.                   |
| YOLOv3            | 2018            | Multi-scale detection using a Darknet-53 backbone; became a widely adopted baseline.                                              |
| YOLOv4            | 2020            | Combined many training and architecture improvements for strong real-time performance.                                            |
| YOLOv5            | 2020+           | Ultralytics PyTorch family; very mature deployment ecosystem and compact Nano models.                                             |
| YOLOv6            | 2022            | Meituan industrial-focused detector with efficient deployment-oriented variants.                                                  |
| YOLOv7            | 2022            | Trainable bag-of-freebies approach; strong GPU-era real-time detector.                                                            |
| YOLOv8            | 2023            | Ultralytics multi-task family: detect, segment, pose, classify, OBB; easy export/deployment.                                      |
| YOLOv9            | 2024            | Programmable Gradient Information (PGI) and GELAN architecture; Tiny through E variants.                                          |
| YOLOv10           | 2024            | End-to-end / NMS-free direction and efficiency-oriented design.                                                                   |
| YOLO11            | 2024            | Ultralytics efficiency/accuracy refinement with broad multi-task support.                                                         |
| YOLO26            | 2026            | Ultralytics unified family with native end-to-end inference, lighter head, updated training recipe, and edge-oriented efficiency. |

Naming is not governed by one single organization. Different YOLO generations were produced by different research teams or companies, so version numbers do not represent one continuous official vendor lineage.

# 4. Detailed Detection Model Specifications

Unless noted, tables below use 640×640 detection benchmarks. “Approx FP32 weights” is calculated as parameter count × 4 bytes and is only the raw parameter payload. “ARM suitability” is a practical qualitative recommendation, not an official minimum requirement.

## YOLOv5

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| v5n       | 2.6M       | 7.7G/B    | 10.4 MB                 | 34.3              | 1-2 GB            | Excellent             |
| v5s       | 9.1M       | 24.0G/B   | 36.4 MB                 | 43.0              | 2-4 GB            | Good                  |
| v5m       | 25.1M      | 64.2G/B   | 100.4 MB                | 49.0              | 4 GB+             | Moderate / slow       |
| v5l       | 53.2M      | 135.0G/B  | 212.8 MB                | 52.2              | 4-8 GB            | Generally impractical |
| v5x       | 97.2M      | 246.4G/B  | 388.8 MB                | 53.2              | 8 GB+             | Generally impractical |

Numbers shown are the current Ultralytics YOLOv5u detection variants, not every historical YOLOv5 checkpoint.

## YOLOv6

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| v6-N      | 4.7M       | 11.4G/B   | 18.8 MB                 | 37.5              | 2-4 GB            | Good                  |
| v6-S      | 18.5M      | 45.3G/B   | 74.0 MB                 | 45.0              | 4 GB+             | Moderate / slow       |
| v6-M      | 34.9M      | 85.8G/B   | 139.6 MB                | 50.0              | 4 GB+             | Heavy                 |
| v6-L      | 59.6M      | 150.7G/B  | 238.4 MB                | 52.8              | 4-8 GB            | Generally impractical |

YOLOv6 figures come from the deployed-model benchmark in the Meituan repository. The project also provides legacy and P6 variants.

## YOLOv8

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| v8n       | 3.2M       | 8.7G/B    | 12.8 MB                 | 37.3              | 2-4 GB            | Good                  |
| v8s       | 11.2M      | 28.6G/B   | 44.8 MB                 | 44.9              | 2-4 GB            | Good                  |
| v8m       | 25.9M      | 78.9G/B   | 103.6 MB                | 50.2              | 4 GB+             | Moderate / slow       |
| v8l       | 43.7M      | 165.1G/B  | 174.8 MB                | 52.9              | 8 GB+             | Generally impractical |
| v8x       | 68.2M      | 257.8G/B  | 272.8 MB                | 53.9              | 8 GB+             | Generally impractical |

## YOLOv9

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| v9-T      | 2.0M       | 7.7G/B    | 8.0 MB                  | 38.3              | 1-2 GB            | Excellent             |
| v9-S      | 7.1M       | 26.4G/B   | 28.4 MB                 | 46.8              | 2-4 GB            | Good                  |
| v9-M      | 20.0M      | 76.3G/B   | 80.0 MB                 | 51.4              | 4 GB+             | Moderate / slow       |
| v9-C      | 25.3M      | 102.1G/B  | 101.2 MB                | 53.0              | 4-8 GB            | Heavy                 |
| v9-E      | 57.3M      | 189.0G/B  | 229.2 MB                | 55.6              | 8 GB+             | Generally impractical |

## YOLOv10

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| v10n      | 2.3M       | 6.7G/B    | 9.2 MB                  | 38.5              | 1-2 GB            | Excellent             |
| v10s      | 7.2M       | 21.6G/B   | 28.8 MB                 | 46.3              | 2-4 GB            | Good                  |
| v10m      | 15.4M      | 59.1G/B   | 61.6 MB                 | 51.1              | 4 GB+             | Moderate / slow       |
| v10b      | 19.1M      | 92.0G/B   | 76.4 MB                 | 52.5              | 4-8 GB            | Heavy                 |
| v10l      | 24.4M      | 120.3G/B  | 97.6 MB                 | 53.2              | 4-8 GB            | Heavy                 |
| v10x      | 29.5M      | 160.4G/B  | 118.0 MB                | 54.4              | 8 GB+             | Generally impractical |

## YOLO11

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| 11n       | 2.6M       | 6.5G/B    | 10.4 MB                 | 39.5              | 1-2 GB            | Excellent             |
| 11s       | 9.4M       | 21.6G/B   | 37.6 MB                 | 47.0              | 2-4 GB            | Good                  |
| 11m       | 20.1M      | 68.1G/B   | 80.4 MB                 | 51.5              | 4 GB+             | Moderate / slow       |
| 11l       | 25.3M      | 87.2G/B   | 101.2 MB                | 53.4              | 4 GB+             | Heavy                 |
| 11x       | 56.9M      | 195.3G/B  | 227.6 MB                | 54.7              | 8 GB+             | Generally impractical |

## YOLO26

| **Model** | **Params** | **FLOPs** | **Approx FP32 weights** | **COCO mAP50-95** | **Practical RAM** | **ARM CPU**           |
|-----------|------------|-----------|-------------------------|-------------------|-------------------|-----------------------|
| 26n       | 2.4M       | 5.5G/B    | 9.6 MB                  | 40.9              | 1-2 GB            | Excellent             |
| 26s       | 9.5M       | 20.9G/B   | 38.0 MB                 | 48.6              | 2-4 GB            | Good                  |
| 26m       | 20.4M      | 68.4G/B   | 81.6 MB                 | 53.1              | 4 GB+             | Moderate / slow       |
| 26l       | 24.8M      | 86.8G/B   | 99.2 MB                 | 55.0              | 4 GB+             | Heavy                 |
| 26x       | 55.7M      | 194.4G/B  | 222.8 MB                | 57.5              | 8 GB+             | Generally impractical |

YOLO26 published Params/FLOPs values refer to the fused inference model; pretrained training checkpoints can report higher counts before unused training branches are removed/fused.

## 4.8 YOLOv7 notes

YOLOv7 is historically important and remains usable, but the official repository primarily emphasizes GPU throughput and larger configurations. For new CPU-only ARM deployments, newer Nano/Tiny models from YOLOv8/v9/v10/11/26 are generally easier to justify. YOLOv7 export code supports ONNX and CPU export workflows, but deployment requires more manual integration than current Ultralytics families.

# 5. ARM CPU Hardware Guidance

| **Hardware class**         | **Typical characteristics**            | **Recommended YOLO class**                          | **Suggested resolution** |
|----------------------------|----------------------------------------|-----------------------------------------------------|--------------------------|
| Very weak ARM              | 4× Cortex-A53/A55, ~1-2 GB RAM, no NPU | Nano/Tiny only: v5n, v9-T, v10n, 11n, 26n           | 320-416                  |
| Raspberry Pi 4 class       | 4× Cortex-A72, 2-8 GB RAM              | Nano primarily; Small only if low FPS is acceptable | 320-512                  |
| Raspberry Pi 5 class       | 4× Cortex-A76, 4-16 GB RAM             | Nano strongly recommended; Small is viable          | 416-640                  |
| High-end embedded ARM      | A78/A710/A720-class, 6-12 cores, 8+ GB | Nano/Small; Medium when latency permits             | 416-640                  |
| ARM server / Apple Silicon | Many high-performance cores, 16+ GB    | Nano through Medium/Large depending SLA             | 640+                     |

Practical RAM values in this guide are planning estimates. Actual peak memory depends on runtime, precision, static/dynamic shapes, number of threads, preprocessing, post-processing, camera buffers, and operating system services.

# 6. Which Runtime Should You Use on ARM?

| **Runtime**     | **ARM CPU fit**                         | **Advantages**                                                                                     | **Considerations**                                                                 |
|-----------------|-----------------------------------------|----------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| NCNN            | Excellent                               | Lightweight C++ runtime, strong ARM/mobile focus, SIMD-friendly, no Python required in production. | Export support and operator compatibility should be tested for your exact model.   |
| ONNX Runtime    | Excellent / very good                   | Portable, mature, straightforward Python/C/C++ integration; good general CPU backend.              | Wheel/platform availability and graph optimizations vary by ARM OS/Python version. |
| TensorFlow Lite | Very good                               | Strong mobile/embedded footprint and INT8 workflows.                                               | Conversion path may need extra validation depending on YOLO generation.            |
| OpenCV DNN      | Good                                    | Simple C++ integration and minimal stack if OpenCV already exists.                                 | Performance and operator coverage vary by model/export.                            |
| PyTorch .pt     | Usually least attractive for deployment | Convenient for development/training.                                                               | Large dependency footprint and higher memory overhead on constrained ARM systems.  |

# 7. Precision: FP32 vs FP16 vs INT8

| **Precision** | **Bytes/parameter** | **Pros**                                                                    | **ARM CPU note**                                                |
|---------------|---------------------|-----------------------------------------------------------------------------|-----------------------------------------------------------------|
| FP32          | 4                   | Maximum compatibility and baseline accuracy.                                | Good baseline; larger bandwidth/storage.                        |
| FP16          | 2                   | Half raw weight storage; often faster on hardware with strong FP16 support. | Some ARM CPUs do not gain as much as GPUs/NPUs; benchmark.      |
| INT8          | 1                   | Smallest common weight storage; can greatly improve CPU/NPU throughput.     | Requires quantization support/calibration; accuracy may change. |

# 8. Offline Deployment Architecture

Recommended workflow: prepare everything on an Internet-connected machine, export the models, collect architecture-correct dependencies, then transfer a complete bundle to the target ARM64 machine.

Internet PC / build machine  
\|  
\| download checkpoints + Python wheels + runtimes  
\| export .pt -\> NCNN / ONNX / TFLite  
v  
YOLO_ARM_OFFLINE/  
\|-- packages/  
\|-- models/  
\|-- runtimes/  
\|-- scripts/  
\|-- test_media/  
\`-- README_OFFLINE.txt  
\|  
v  
ARM64 target (no Internet required)

## 8.1 Recommended offline model library

models/  
YOLO11/  
yolo11n.pt  
yolo11n.onnx  
yolo11n_ncnn_model/  
yolo11s.pt  
yolo11s.onnx  
yolo11s_ncnn_model/  
YOLO26/  
yolo26n.pt  
yolo26n.onnx  
yolo26n_ncnn_model/  
yolo26s.pt  
yolo26s.onnx  
yolo26s_ncnn_model/

## 8.2 Export examples

\# Install on the connected build machine  
pip install ultralytics  
  
\# Export to NCNN  
yolo export model=yolo11n.pt format=ncnn imgsz=416  
yolo export model=yolo26n.pt format=ncnn imgsz=416  
  
\# Export to ONNX  
yolo export model=yolo11n.pt format=onnx imgsz=416  
yolo export model=yolo26n.pt format=onnx imgsz=416

Keep the original .pt checkpoints as archival/source models even if the final ARM application uses NCNN or ONNX.

## 8.3 Offline Python package repository

When the build machine and target have the same OS/architecture/Python ABI, packages can be downloaded into a local wheel directory and installed with --no-index. Cross-platform downloading is more delicate: x86 Windows wheels will not work on Linux ARM64.

\# Connected machine  
mkdir offline_packages  
pip download ultralytics -d offline_packages  
  
\# Offline target  
python3 -m venv yolo_env  
source yolo_env/bin/activate  
pip install --no-index --find-links=offline_packages ultralytics

For maximum reliability, build/test the wheel cache on an ARM64 Linux machine that matches the target distribution and Python version, or use a reproducible ARM64 container/chroot capable of obtaining the correct wheels.

# 9. Running YOLO11 and YOLO26 Offline

## 9.1 Python with Ultralytics + exported model

from ultralytics import YOLO  
  
model = YOLO('models/YOLO26/yolo26n_ncnn_model')  
results = model('test_media/test.jpg')  
for result in results:  
print(result.boxes)

## 9.2 Recommended first benchmark matrix

| **Model** | **Resolution** | **Runtime**  | **Precision**          | **Why test it**                               |
|-----------|----------------|--------------|------------------------|-----------------------------------------------|
| YOLO26n   | 320            | NCNN         | FP32/INT8 if supported | Maximum speed baseline                        |
| YOLO26n   | 416            | NCNN         | FP32/INT8              | Likely sweet spot for weak-to-mid ARM         |
| YOLO26n   | 640            | NCNN or ONNX | FP32                   | Accuracy baseline                             |
| YOLO11n   | 416            | NCNN         | FP32/INT8              | Mature modern comparison                      |
| YOLO11n   | 640            | ONNX         | FP32                   | Portable reference benchmark                  |
| YOLO26s   | 416            | NCNN         | FP32/INT8              | Check whether added accuracy is worth latency |

# 10. How to Benchmark Correctly on ARM

- Warm up the model for several runs before timing.

- Use batch size 1 for camera/real-time latency measurements.

- Measure preprocessing, inference, post-processing, and total end-to-end latency separately.

- Report median and p95 latency, not only average FPS.

- Pin or record thread count and CPU governor; thermal throttling can change results after several minutes.

- Benchmark the exact production resolution and precision.

- Watch peak RAM and CPU temperature during a sustained test.

- If using a camera, include capture and decode cost when measuring end-to-end system latency.

# 11. Model Selection Rules for ARM CPU

| **Situation**                                      | **Best starting choice**                                |
|----------------------------------------------------|---------------------------------------------------------|
| Need highest chance of real-time performance       | YOLO26n at 320 or 416, NCNN                             |
| Need mature modern tooling                         | YOLO11n                                                 |
| Need older compatibility / very constrained device | YOLOv5n                                                 |
| Need better accuracy and can accept lower FPS      | YOLO26s or YOLO11s                                      |
| CPU-only device below Cortex-A72 class             | Prefer Nano/Tiny + 320 input; consider INT8             |
| Medium/Large/X model on small SBC                  | Usually avoid unless FPS requirement is very low        |
| Production C++ application                         | Prefer NCNN/ONNX Runtime native API over Python/PyTorch |

# 12. Important Caveats

- FLOPs are architecture-level compute estimates, not a promise of FPS. Memory access patterns, SIMD utilization, operators, thread scheduling, and runtime implementation matter.

- Published CPU ONNX timings from Ultralytics are generally measured on x86 server CPUs, not your ARM board. Use them for relative comparison, not predicted ARM FPS.

- Raw FP32 weight size is not the same as .pt/.onnx file size and not the same as peak runtime RAM.

- Quantization can improve speed and memory efficiency, but calibration and accuracy verification are mandatory.

- YOLO model naming is fragmented across research groups. Always record the exact repository, checkpoint, commit/version, runtime, and export settings used in production.

# 13. Recommended Baseline for This Use Case

| **Start with YOLO26n at 416×416 using NCNN on ARM64, batch size 1. Keep YOLO11n as the comparison model. Benchmark 320, 416, and 640 resolutions; test INT8 only after establishing an FP32 accuracy baseline. Build one complete offline bundle containing models, exported runtimes, matching ARM64 dependencies, test media, and a benchmark script.** |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
