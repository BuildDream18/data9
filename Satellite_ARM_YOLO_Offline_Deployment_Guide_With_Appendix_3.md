**Satellite / ARM Computer Vision  
Offline Deployment Guide**

*YOLO26n-OBB • YOLO11n/s • Windows 11 • Offline Python • Training • Export • Storage*

Prepared October 2026. This document consolidates the previous YOLO11/offline-PC guidance and updates the recommendation for a new satellite/ARM project.

# 1. Executive recommendation

Best general starting point without knowing the exact flight computer: YOLO26n-OBB.

- Use OBB (oriented bounding boxes) for overhead targets that may appear at arbitrary rotations, including ships, aircraft, vehicles and buildings.

- Train and export on a development computer. The satellite should normally run inference only.

- For generic ARM CPUs, benchmark NCNN first and ONNX Runtime second. If the SoC has an NPU, use its supported vendor runtime/export when practical.

- Start with nano. Move to small only if the measured accuracy gain is worth the additional compute, RAM, storage and power.

- Python 3.12 is the recommended Windows development version. Treat Python 3.14 as an explicitly tested target, not an assumption.

- YOLO-family models are not inherently unsuitable for satellites. Model scale, OBB support, target runtime, quantization and aerial-domain training are the important factors.

# 2. Why YOLO26n-OBB

| **Model**   | **Input** | **DOTA mAP50-95** | **CPU ONNX\*** | **Fused params** | **FLOPs** |
|-------------|-----------|-------------------|----------------|------------------|-----------|
| YOLO26n-OBB | 1024      | 52.4              | 97.7 ms        | 2.4 M            | 14.8 B    |
| YOLO26s-OBB | 1024      | 54.8              | 218.0 ms       | 9.8 M            | 56.7 B    |
| YOLO26m-OBB | 1024      | 55.3              | 579.2 ms       | 21.2 M           | 184.9 B   |

\*Official reference benchmark, not an estimate for your ARM processor.

The nano model is attractive because the published DOTA accuracy gap from n to s is much smaller than the increase in parameters and FLOPs. OBB is especially appropriate for aerial imagery because rotated boxes follow an object's orientation instead of enclosing it in a horizontal rectangle.

Ultralytics OBB label format:

> class_index x1 y1 x2 y2 x3 y3 x4 y4

The coordinates are normalized to 0..1. Current YOLO26 OBB checkpoints are pretrained on DOTA.

# 3. What about YOLO11n and YOLO11s?

| **Model** | **Parameters** | **FLOPs @640** | **Approx .pt size** | **ARM view** |
|-----------|----------------|----------------|---------------------|--------------|
| YOLO11n   | 2.6 M          | 6.5 B          | ~5–6 MB             | Good         |
| YOLO11s   | 9.4 M          | 21.6 B         | ~18–20 MB           | Heavier      |

YOLO11 remains practical. For a new 2026 project, YOLO26 is the newer production family. Ultralytics reports a Raspberry Pi 5 ONNX benchmark of 7.79 FPS for YOLO26n versus 6.79 FPS for YOLO11n, with mAP 40.1 versus 39.5. That is a platform-specific reference, not a universal performance guarantee.

# 4. Recommended architecture

> Online Windows 11 development PC  
> Python 3.12 + Ultralytics + PyTorch  
> download / train / validate / export  
> \|  
> +-- best.pt  
> +-- ONNX  
> +-- NCNN  
> +-- optional vendor NPU format  
> \|  
> v  
> controlled transfer  
> \|  
> v  
> Offline ARM equipment  
> inference runtime + model  
> no training stack needed

# 5. Windows 11 development setup

> mkdir C:\satellite_yolo  
> cd C:\satellite_yolo  
> py -3.12 -m venv venv  
> venv\Scripts\activate  
> python -m pip install --upgrade pip  
> pip install -U ultralytics

Verify:

> python --version  
> python -c "import ultralytics; print(ultralytics.\_\_version\_\_)"  
> python -c "import torch; print(torch.\_\_version\_\_); print(torch.cuda.is_available())"  
> python -c "import torch; print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU only')"

Your GTX 1050 Ti is useful for testing, export and limited training. At 1024-pixel OBB resolution, its VRAM can be restrictive. Use batch=1 or a smaller image size if necessary; a newer GPU is preferable for serious training.

# 6. Download model files completely while online

> from ultralytics import YOLO  
>   
> YOLO("yolo26n-obb.pt")  
> YOLO("yolo26s-obb.pt")  
> YOLO("yolo11n.pt")  
> YOLO("yolo11s.pt")

Keep explicit local copies:

> C:\satellite_yolo\models\\  
> yolo26n-obb.pt  
> yolo26s-obb.pt  
> yolo11n.pt  
> yolo11s.pt

Reference these local files in offline applications so the software does not attempt to download missing weights.

# 7. Python 3.12 versus Python 3.14

| **System**                | **Recommendation**          | **Reason**                                    |
|---------------------------|-----------------------------|-----------------------------------------------|
| Windows training/export   | Python 3.12                 | Conservative ecosystem choice                 |
| Offline Windows test PC   | 3.12 or 3.13                | Simpler wheel availability                    |
| Python 3.14 PC            | Only after wheel validation | Every dependency must have a compatible wheel |
| ARM + native NCNN         | No Python required          | Smaller production stack                      |
| ARM + Python ONNX Runtime | Target-specific             | Match supported ARM64 wheel/Python            |

Current Ultralytics metadata requires Python \>=3.8, but its PyPI classifiers currently enumerate versions through Python 3.13 rather than explicitly advertising 3.14. PyTorch may support a newer Python before the complete Ultralytics dependency chain does. Therefore a fully offline Python 3.14 installation should be built only after verifying every pinned package.

This corrects the earlier broad implication that Python 3.14 can simply be treated like Python 3.12 for the whole offline stack.

# 8. Build a fully offline Windows bundle

Download wheels on an Internet-connected machine using the same target Python version, OS and CPU architecture whenever possible.

> mkdir C:\satellite_yolo\offline_packages  
> py -3.12 -m pip download ultralytics -d C:\satellite_yolo\offline_packages

Also archive the exact CPU or CUDA PyTorch wheels that you tested. PyTorch build URLs/tags vary by version and CUDA release, so use the official PyTorch selector and pin the working versions.

Offline installation:

> py -3.12 -m venv C:\satellite_yolo\venv  
> C:\satellite_yolo\venv\Scripts\activate  
> pip install --no-index --find-links=D:\YOLO_OFFLINE\packages ultralytics

Disconnect networking and run the test again. The --no-index flag prevents pip from accessing an online package index.

# 9. Satellite/aerial dataset

DOTA is a strong OBB starting point. DOTA v1 contains aerial classes such as plane, ship, storage tank, bridge, large vehicle, small vehicle, helicopter and harbor. The official Ultralytics reference download is about 2 GB. Review its license before operational/commercial use; the documentation notes academic-use restrictions for DOTA v1.

Raw aerial images can exceed 10,000 pixels per side. Ultralytics documents tiling into overlapping 1024 × 1024 crops:

> from ultralytics.data.split_dota import split_trainval, split_test  
>   
> split_trainval(  
> data_root=r"D:\data\DOTAv1.0",  
> save_dir=r"D:\data\DOTAv1_split",  
> rates=\[0.5, 1.0, 1.5\],  
> gap=500,  
> )  
>   
> split_test(  
> data_root=r"D:\data\DOTAv1.0",  
> save_dir=r"D:\data\DOTAv1_split",  
> rates=\[0.5, 1.0, 1.5\],  
> gap=500,  
> )

# 10. Custom training dataset format

> satellite_dataset/  
> images/train/  
> images/val/  
> images/test/  
> labels/train/  
> labels/val/  
> labels/test/  
> satellite.yaml

Example satellite.yaml:

> path: D:/datasets/satellite_dataset  
> train: images/train  
> val: images/val  
> test: images/test  
>   
> names:  
> 0: ship  
> 1: aircraft  
> 2: vehicle  
> 3: building

Each OBB text label uses: class_id x1 y1 x2 y2 x3 y3 x4 y4. Build train/validation data that represents your actual sensor resolution/GSD, target sizes, illumination, viewing geometry, haze/clouds, seasons and backgrounds.

# 11. Train YOLO26n-OBB

> from ultralytics import YOLO  
>   
> model = YOLO(r"C:\satellite_yolo\models\yolo26n-obb.pt")  
> model.train(  
> data=r"D:\datasets\satellite_dataset\satellite.yaml",  
> epochs=100,  
> imgsz=1024,  
> device=0,  
> batch=2,  
> workers=4,  
> project="runs_satellite",  
> name="yolo26n_obb_v1"  
> )

CLI equivalent:

> yolo obb train model=C:\satellite_yolo\models\yolo26n-obb.pt data=D:\datasets\satellite_dataset\satellite.yaml epochs=100 imgsz=1024 device=0 batch=2

On the GTX 1050 Ti, reduce batch to 1 and/or imgsz to 640 or 768 if you hit CUDA out-of-memory errors.

# 12. Validate and decide n versus s

> from ultralytics import YOLO  
> model = YOLO(r"runs_satellite\yolo26n_obb_v1\weights\best.pt")  
> metrics = model.val(data=r"D:\datasets\satellite_dataset\satellite.yaml")

- Compare n and s using the same held-out, mission-like validation set.

- Pay special attention to recall for small targets, not only aggregate mAP.

- Measure full pipeline latency: decode, tiling, preprocessing, inference, postprocessing and tile merging.

- Measure peak RAM, sustained power and thermal behavior on the actual ARM board.

- Prefer n when s does not materially improve mission-relevant detection.

# 13. Export for ARM

ONNX:

> from ultralytics import YOLO  
> model = YOLO(r"runs_satellite\yolo26n_obb_v1\weights\best.pt")  
> model.export(format="onnx", imgsz=1024)

NCNN:

> from ultralytics import YOLO  
> model = YOLO(r"runs_satellite\yolo26n_obb_v1\weights\best.pt")  
> model.export(format="ncnn", imgsz=1024)

Current YOLO26-OBB export support also includes RKNN, Qualcomm QNN, ExecuTorch, Hailo, IMX500, MNN and Xilinx formats. The exact ARM processor determines the best target. Ultralytics recommends NCNN for Raspberry Pi-class ARM CPU deployment, while ONNX Runtime officially supports Linux ARM64 CPU packages.

# 14. Quantization

- FP32: simplest compatibility baseline.

- FP16: can roughly halve weight storage and help on hardware with efficient FP16.

- INT8: can reduce storage/bandwidth and accelerate suitable NPUs/CPUs, but requires representative calibration and accuracy testing.

- Never accept quantization only because it is smaller; re-measure target-class recall and false positives.

# 15. Storage planning

| **Item**                           | **Planning size**               | **Notes**                        |
|------------------------------------|---------------------------------|----------------------------------|
| YOLO11n .pt                        | ~5–6 MB                         | Earlier model                    |
| YOLO11s .pt                        | ~18–20 MB                       | Earlier model                    |
| YOLO26n-OBB                        | Single-digit to low tens of MB  | Exact file/export varies         |
| YOLO26s-OBB                        | Several times n                 | 9.8M vs 2.4M fused params        |
| DOTA v1                            | ~2 GB                           | Official reference download      |
| CPU Python/PyTorch/Ultralytics env | ~1–3+ GB                        | Version dependent                |
| CUDA development stack             | Several GB                      | Much larger than model           |
| Training workspace                 | 20–50+ GB recommended           | Tiles, caches, runs, checkpoints |
| Minimal flight model/runtime       | Tens to hundreds of MB possible | Depends on OS/runtime            |

The neural-network file is not the main development-storage consumer. Dataset copies, tiled imagery, Python/PyTorch/CUDA, checkpoints and experiment outputs dominate. Reserve 20–50 GB or more on the development PC. For flight, measure the actual stripped runtime image rather than copying the training environment.

# 16. RAM, power and image-size considerations

Weight size is not equal to runtime RAM. RAM also includes activations, input/output buffers, runtime workspace, tile queues and the rest of your application. A 1024 × 1024 inference is substantially heavier than 640 × 640.

Benchmark the production runtime on the actual ARM SoC at the intended thread count and thermal/power limits. Satellite design also requires system-level reliability work outside the ML model: watchdogs, deterministic recovery, storage integrity, radiation/SEU strategy, telemetry, update/rollback strategy and validation under expected thermal conditions.

# 17. Offline ARM deployment patterns

| **Hardware type**                | **Preferred first test** | **Python onboard?**                      |
|----------------------------------|--------------------------|------------------------------------------|
| Generic ARM64 Linux CPU          | NCNN; then ONNX Runtime  | No for native NCNN; optional for ORT     |
| Raspberry Pi-class ARM           | NCNN                     | Not required                             |
| Rockchip RK35xx + NPU            | RKNN                     | Usually not required in final native app |
| Qualcomm + supported accelerator | QNN                      | Target/runtime dependent                 |
| NVIDIA Jetson ARM                | TensorRT                 | Python optional                          |
| Xilinx/AMD adaptive SoC          | Vitis/Xilinx export path | Usually native runtime                   |

Do not create a 'universal ARM wheel folder'. ARMv7, ARM64, Linux ARM64, Windows ARM64 and vendor NPU stacks are different targets. The model can be portable; compiled runtimes often are not.

# 18. Minimal offline inference test

For an offline Windows CPU test with the local .pt checkpoint:

> from ultralytics import YOLO  
>   
> model = YOLO(r"C:\satellite_yolo\models\yolo26n-obb.pt")  
> results = model(r"C:\satellite_yolo\test_satellite.jpg", conf=0.25, save=True)  
>   
> for result in results:  
> print(result.obb)

For flight, prefer testing the exported production format rather than relying on .pt/PyTorch if your final runtime is NCNN, ONNX Runtime or a vendor NPU.

# 19. Recommended end-to-end project sequence

- 1\. Define target objects and minimum detectable object size/GSD.

- 2\. Obtain/annotate representative aerial imagery using OBB labels.

- 3\. Establish YOLO26n-OBB as the baseline.

- 4\. Train at 640/768 first if GPU-limited, then test 1024 where small-target accuracy matters.

- 5\. Validate on a geographically and temporally separate holdout set.

- 6\. Export ONNX and NCNN; export the vendor NPU format if the final SoC supports one.

- 7\. Benchmark on the actual ARM equipment.

- 8\. Test FP16/INT8 only after establishing an FP32 accuracy baseline.

- 9\. Compare YOLO26s-OBB only if nano accuracy is inadequate.

- 10\. Freeze exact model/runtime versions and build a reproducible offline deployment package.

- 11\. Perform long-duration, thermal, power, corruption/restart and fault-recovery tests before operational deployment.

# 20. What to transfer to the offline systems

Offline Windows development/test bundle:

> YOLO_OFFLINE/  
> packages/ \# pinned wheels  
> models/ \# .pt and exported models  
> scripts/ \# training/export/test scripts  
> requirements-lock.txt  
> test_images/  
> README.txt

Minimal ARM flight bundle should instead resemble:

> flight_app/  
> bin/ \# native application/runtime  
> model/ \# one validated exported model  
> config/ \# classes, thresholds, tile settings  
> test_vectors/ \# small known-good self-test inputs  
> version.json \# exact model/runtime/application versions

# 21. Licensing and mission considerations

Ultralytics is distributed under AGPL-3.0 with commercial licensing options. Review the license implications for your product/deployment. Dataset licenses are separate from model/software licenses. Also treat onboard ML as one component of a larger flight-software assurance process; a successful desktop inference demo is not flight qualification.

# 22. Final recommendation for your hardware situation

On the Windows 11 + GTX 1050 Ti computer: use Python 3.12, download and archive YOLO26n-OBB plus YOLO11n/s if you want comparison baselines, prepare the dataset, fine-tune, validate and export.

On the second offline computer without a GPU: use a CPU environment mainly for reproducibility/inference tests. Prefer Python 3.12/3.13. If Python 3.14 is mandatory, first create and test a complete cp314 wheel bundle on an Internet-connected machine with the same OS/architecture.

On the final ARM equipment: do not assume Python or PyTorch is needed. Begin with YOLO26n-OBB exported to NCNN for generic ARM CPU, or to the SoC's native NPU format. Only choose YOLO26s-OBB after real-device benchmarking shows acceptable latency, RAM and power.

# 23. Information still needed for a flight-specific final design

To replace the generic ARM recommendation with an exact deployment recipe, identify: ARM CPU/SoC model, architecture (ARMv7/ARM64), OS/distribution, RAM, storage, NPU/GPU/FPGA if any, power budget, camera/sensor resolution, expected image rate, target classes, minimum target pixel size, and required latency. With those values, the model format, quantization level, input/tile size, runtime and expected storage can be narrowed precisely.

# Appendix A. Technical Terms, Full Names, and Easy Meanings

B1-level glossary for the terms used in this guide. Some names (such as NCNN and TensorRT) are product names, not abbreviations with an official long form.

**1. AI**

> **Full name:** Artificial Intelligence
>
> **Easy meaning:** Computer technology that can learn patterns and make predictions.
>
> **In this project:** Used to recognize objects in satellite images.

**2. ML**

> **Full name:** Machine Learning
>
> **Easy meaning:** A type of AI that learns from examples.
>
> **In this project:** The detection model learns from labeled images.

**3. Computer vision**

> **Full name:** No abbreviation
>
> **Easy meaning:** Technology that helps computers understand pictures and videos.
>
> **In this project:** Find ships, planes, and other targets.

**4. YOLO**

> **Full name:** You Only Look Once
>
> **Easy meaning:** A family of fast object-detection models.
>
> **In this project:** Find objects in images with one model.

**5. YOLO11 / YOLO26**

> **Full name:** YOLO model families; 11 and 26 are version names
>
> **Easy meaning:** Different generations of YOLO models.
>
> **In this project:** Compare speed, accuracy, and hardware needs.

**6. n / s / m**

> **Full name:** Nano / Small / Medium
>
> **Easy meaning:** Model size labels. Nano is usually lighter; medium is usually heavier.
>
> **In this project:** Choose a size that fits the ARM device.

**7. OBB**

> **Full name:** Oriented Bounding Box
>
> **Easy meaning:** A rectangle that can rotate to follow the direction of an object.
>
> **In this project:** Mark angled ships, aircraft, and buildings.

**8. AABB**

> **Full name:** Axis-Aligned Bounding Box
>
> **Easy meaning:** A rectangle whose sides stay horizontal and vertical.
>
> **In this project:** Compare normal boxes with rotated OBB boxes.

**9. Bounding box**

> **Full name:** No abbreviation
>
> **Easy meaning:** A box showing where an object is in an image.
>
> **In this project:** Show the detected object location.

**10. Class / class ID**

> **Full name:** Object category / category number
>
> **Easy meaning:** The name or number for an object type, such as ship or plane.
>
> **In this project:** Tell the model which object it detected.

**11. Annotation / label**

> **Full name:** No abbreviation
>
> **Easy meaning:** Information added to an image to show the correct object and location.
>
> **In this project:** Prepare examples for training.

**12. DOTA**

> **Full name:** Dataset for Object deTection in Aerial images
>
> **Easy meaning:** A dataset of overhead images with labeled objects.
>
> **In this project:** Train and test aerial OBB detection.

**13. GSD**

> **Full name:** Ground Sample Distance
>
> **Easy meaning:** The ground distance represented by one image pixel.
>
> **In this project:** Understand how small an object looks in satellite imagery.

**14. Pixel**

> **Full name:** Picture element
>
> **Easy meaning:** One small point in a digital image.
>
> **In this project:** Measure image size and object size.

**15. Image resolution**

> **Full name:** No abbreviation
>
> **Easy meaning:** The number of pixels in an image, such as 1024 × 1024.
>
> **In this project:** Balance image detail against processing cost.

**16. Tiling**

> **Full name:** No abbreviation
>
> **Easy meaning:** Cutting a very large image into smaller pieces.
>
> **In this project:** Process large satellite images with limited memory.

**17. Overlap / gap**

> **Full name:** No abbreviation
>
> **Easy meaning:** Shared image area between nearby tiles; gap is a tiling setting.
>
> **In this project:** Reduce missed objects near tile edges.

**18. Train / validation / test**

> **Full name:** No abbreviation
>
> **Easy meaning:** Train teaches the model; validation helps choose settings; test checks final performance.
>
> **In this project:** Evaluate the model fairly.

**19. Epoch**

> **Full name:** No abbreviation
>
> **Easy meaning:** One full pass through the training dataset.
>
> **In this project:** Control how long training runs.

**20. Batch size**

> **Full name:** No abbreviation
>
> **Easy meaning:** How many images the model processes together during training.
>
> **In this project:** Lower it when GPU memory is limited.

**21. Checkpoint / .pt**

> **Full name:** PyTorch checkpoint file (.pt)
>
> **Easy meaning:** A saved model with learned weights.
>
> **In this project:** Keep and load trained YOLO models.

**22. Weights**

> **Full name:** No abbreviation
>
> **Easy meaning:** Learned numbers inside a neural network.
>
> **In this project:** Store what the model learned.

**23. Inference**

> **Full name:** No abbreviation
>
> **Easy meaning:** Using a trained model to make predictions on new images.
>
> **In this project:** Run detection on the satellite or ARM device.

**24. Preprocessing**

> **Full name:** No abbreviation
>
> **Easy meaning:** Preparing images before they enter the model.
>
> **In this project:** Resize and normalize input images.

**25. Postprocessing**

> **Full name:** No abbreviation
>
> **Easy meaning:** Turning model outputs into useful detections.
>
> **In this project:** Filter boxes and combine results.

**26. Confidence / conf**

> **Full name:** Confidence score
>
> **Easy meaning:** A number showing how confident the model is about a detection.
>
> **In this project:** Filter weak predictions.

**27. Precision**

> **Full name:** No abbreviation
>
> **Easy meaning:** Of the reported detections, how many are correct.
>
> **In this project:** Measure false-alarm performance.

**28. Recall**

> **Full name:** No abbreviation
>
> **Easy meaning:** Of the real objects, how many the model finds.
>
> **In this project:** Measure how often small targets are missed.

**29. False positive**

> **Full name:** No abbreviation
>
> **Easy meaning:** A detection reported when the object is not really there.
>
> **In this project:** Count unwanted alarms.

**30. mAP**

> **Full name:** mean Average Precision
>
> **Easy meaning:** A common score that summarizes object-detection quality.
>
> **In this project:** Compare models on the same test data.

**31. mAP50–95**

> **Full name:** mean Average Precision at IoU thresholds from 0.50 to 0.95
>
> **Easy meaning:** A stricter detection score averaged over several box-overlap thresholds.
>
> **In this project:** Compare box accuracy across models.

**32. IoU**

> **Full name:** Intersection over Union
>
> **Easy meaning:** How much a predicted box overlaps the correct box.
>
> **In this project:** Score detection box quality.

**33. FPS**

> **Full name:** Frames Per Second
>
> **Easy meaning:** How many images or video frames can be processed each second.
>
> **In this project:** Measure detection speed.

**34. Latency**

> **Full name:** No abbreviation
>
> **Easy meaning:** How long one processing task takes.
>
> **In this project:** Check whether detection finishes in time.

**35. FLOPs**

> **Full name:** Floating-Point Operations
>
> **Easy meaning:** An estimate of the amount of math work a model needs; model tables may count operations differently.
>
> **In this project:** Compare model compute requirements.

**36. Parameters**

> **Full name:** No abbreviation
>
> **Easy meaning:** The adjustable numbers in a model.
>
> **In this project:** Estimate model size and complexity.

**37. ARM**

> **Full name:** Originally Advanced RISC Machines; now used as the Arm architecture name
>
> **Easy meaning:** A processor architecture common in low-power devices.
>
> **In this project:** Run the model on embedded equipment.

**38. ARMv7 / ARM64**

> **Full name:** ARM version 7 / 64-bit ARM architecture
>
> **Easy meaning:** Different ARM processor and software targets.
>
> **In this project:** Choose the correct runtime and packages.

**39. CPU**

> **Full name:** Central Processing Unit
>
> **Easy meaning:** The main processor that runs general computer instructions.
>
> **In this project:** Run detection without a special accelerator.

**40. GPU**

> **Full name:** Graphics Processing Unit
>
> **Easy meaning:** A processor good at doing many calculations in parallel.
>
> **In this project:** Speed up model training and some inference.

**41. NPU**

> **Full name:** Neural Processing Unit
>
> **Easy meaning:** A chip part designed to run neural networks efficiently.
>
> **In this project:** Run AI models using less energy on supported hardware.

**42. SoC**

> **Full name:** System on a Chip
>
> **Easy meaning:** One chip that contains several computer parts, often CPU, GPU, and other units.
>
> **In this project:** Identify the exact hardware and supported AI runtime.

**43. FPGA**

> **Full name:** Field-Programmable Gate Array
>
> **Easy meaning:** A chip whose digital logic can be configured for special tasks.
>
> **In this project:** Build specialized image-processing hardware.

**44. RAM**

> **Full name:** Random Access Memory
>
> **Easy meaning:** Temporary memory used while programs run.
>
> **In this project:** Hold image data, model buffers, and application data.

**45. VRAM**

> **Full name:** Video Random Access Memory
>
> **Easy meaning:** Memory used by a graphics card.
>
> **In this project:** Limit the training image size and batch size.

**46. CUDA**

> **Full name:** Compute Unified Device Architecture
>
> **Easy meaning:** NVIDIA technology for using compatible GPUs for computing.
>
> **In this project:** Accelerate PyTorch training on an NVIDIA GPU.

**47. PyTorch**

> **Full name:** No abbreviation
>
> **Easy meaning:** A software library for building and training AI models.
>
> **In this project:** Train and load YOLO checkpoints.

**48. Ultralytics**

> **Full name:** Company and software toolkit name
>
> **Easy meaning:** The toolkit used in this guide to train, test, and export YOLO models.
>
> **In this project:** Run the YOLO examples in the document.

**49. ONNX**

> **Full name:** Open Neural Network Exchange
>
> **Easy meaning:** A format for moving AI models between software tools.
>
> **In this project:** Export the model to a portable format.

**50. ONNX Runtime / ORT**

> **Full name:** Open Neural Network Exchange Runtime
>
> **Easy meaning:** Software that runs ONNX models.
>
> **In this project:** Test or deploy models on supported ARM systems.

**51. NCNN**

> **Full name:** ncnn (project name; not a standard acronym)
>
> **Easy meaning:** A lightweight neural-network inference framework, especially useful on mobile and embedded devices.
>
> **In this project:** Run exported models on ARM CPUs.

**52. RKNN**

> **Full name:** Rockchip Neural Network
>
> **Easy meaning:** Rockchip model format and software tools for supported processors.
>
> **In this project:** Deploy on compatible Rockchip NPUs.

**53. QNN**

> **Full name:** Qualcomm AI Engine Direct / Qualcomm Neural Network SDK terminology
>
> **Easy meaning:** Qualcomm software for running neural networks on supported Qualcomm hardware.
>
> **In this project:** Use supported Qualcomm accelerators.

**54. TensorRT**

> **Full name:** NVIDIA TensorRT (product name)
>
> **Easy meaning:** NVIDIA software that optimizes and runs AI models.
>
> **In this project:** Deploy on supported NVIDIA Jetson devices.

**55. ExecuTorch**

> **Full name:** Product name, not an acronym
>
> **Easy meaning:** A PyTorch-related runtime for running AI models on devices.
>
> **In this project:** Consider for supported embedded targets.

**56. Hailo**

> **Full name:** Company and accelerator platform name
>
> **Easy meaning:** AI accelerator hardware and related deployment software.
>
> **In this project:** Use a Hailo device if present.

**57. IMX500**

> **Full name:** Sony IMX500 sensor model name
>
> **Easy meaning:** An image sensor with built-in AI processing capabilities.
>
> **In this project:** Run supported AI tasks close to the camera.

**58. MNN**

> **Full name:** Mobile Neural Network
>
> **Easy meaning:** A lightweight framework for running neural networks on devices.
>
> **In this project:** Another possible deployment runtime.

**59. Xilinx / Vitis**

> **Full name:** Company/brand and development platform names
>
> **Easy meaning:** Tools and hardware technology for programmable chips, now under AMD.
>
> **In this project:** Target supported adaptive SoCs.

**60. FP32**

> **Full name:** 32-bit Floating Point
>
> **Easy meaning:** A common number format using 32 bits.
>
> **In this project:** Start with a simple accuracy baseline.

**61. FP16**

> **Full name:** 16-bit Floating Point
>
> **Easy meaning:** A smaller floating-point number format.
>
> **In this project:** Reduce model storage on supported hardware.

**62. INT8**

> **Full name:** 8-bit Integer
>
> **Easy meaning:** A number format using 8-bit whole numbers.
>
> **In this project:** Reduce model size and sometimes increase speed.

**63. Quantization**

> **Full name:** No abbreviation
>
> **Easy meaning:** Changing model numbers to a smaller or simpler format.
>
> **In this project:** Save memory and power if accuracy remains good.

**64. Calibration**

> **Full name:** No abbreviation
>
> **Easy meaning:** Using sample data to set number ranges for some quantized models.
>
> **In this project:** Prepare an INT8 model.

**65. Runtime**

> **Full name:** No abbreviation
>
> **Easy meaning:** Software that actually executes a model.
>
> **In this project:** Run exported models on the target hardware.

**66. Export**

> **Full name:** No abbreviation
>
> **Easy meaning:** Converting a trained model into another file format.
>
> **In this project:** Create ONNX, NCNN, or vendor-specific models.

**67. OS**

> **Full name:** Operating System
>
> **Easy meaning:** Main software that manages the computer, such as Windows or Linux.
>
> **In this project:** Choose compatible installation files.

**68. CLI**

> **Full name:** Command-Line Interface
>
> **Easy meaning:** A way to control software by typing commands.
>
> **In this project:** Train or export YOLO from a terminal.

**69. venv**

> **Full name:** Virtual environment
>
> **Easy meaning:** An isolated place for a Python project packages.
>
> **In this project:** Avoid package conflicts.

**70. pip**

> **Full name:** Python package installer (commonly called pip)
>
> **Easy meaning:** A tool for installing Python packages.
>
> **In this project:** Install Ultralytics and other libraries.

**71. Wheel / .whl**

> **Full name:** Python wheel package
>
> **Easy meaning:** A ready-to-install Python package file.
>
> **In this project:** Install dependencies without internet access.

**72. cp314**

> **Full name:** CPython 3.14 compatibility tag
>
> **Easy meaning:** A tag used to identify Python 3.14-specific package builds.
>
> **In this project:** Check whether offline packages match Python 3.14.

**73. PyPI**

> **Full name:** Python Package Index
>
> **Easy meaning:** An online directory of Python packages.
>
> **In this project:** Download needed packages before going offline.

**74. YAML / .yaml**

> **Full name:** YAML Ain’t Markup Language
>
> **Easy meaning:** A readable text format for settings and structured data.
>
> **In this project:** Define dataset paths and object classes.

**75. SEU**

> **Full name:** Single-Event Upset
>
> **Easy meaning:** A change in an electronic bit caused by a particle or radiation event.
>
> **In this project:** Plan fault protection for space hardware.

**76. Watchdog**

> **Full name:** No abbreviation
>
> **Easy meaning:** A safety timer that detects a stuck program and can trigger recovery.
>
> **In this project:** Improve reliability during unattended operation.

**77. Telemetry**

> **Full name:** No abbreviation
>
> **Easy meaning:** Status and measurement data sent from a remote system.
>
> **In this project:** Monitor the satellite equipment.

**78. Rollback**

> **Full name:** No abbreviation
>
> **Easy meaning:** Returning to an earlier working software version.
>
> **In this project:** Recover from a bad update.

**79. AGPL-3.0**

> **Full name:** GNU Affero General Public License version 3.0
>
> **Easy meaning:** A software license with rules about using, modifying, and sharing software.
>
> **In this project:** Review licensing before product deployment.

**80. GB / MB**

> **Full name:** Gigabyte / Megabyte
>
> **Easy meaning:** Units used to describe file size and storage.
>
> **In this project:** Plan storage for models, data, and software.
