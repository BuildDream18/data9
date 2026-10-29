# Luxand FaceSDK 9.0.1 — Complete Setup and Implementation Guide

**Prepared:** October 26, 2026  
**Requested product:** “Luxand 9.1”  
**Verified public release:** **Luxand FaceSDK 9.0.1**, released October 16, 2026  
**Primary walkthrough:** Windows 11 x64 and Python 3.12  
**Additional platforms:** Linux x86-64, Linux ARM/Zynq, C++, C#, browser and mobile

> **Version clarification:** Luxand's official public download, release-notes, and documentation sites currently identify **9.0.1**, not a public FaceSDK 9.1 release. This guide covers the verified 9.0.1 API. If you received a private 9.1 build, ask Luxand for its matching binaries, wrappers, documentation, and license.
>
> **Validation scope:** Code follows the documented 9.0.1 API but has not been executed with proprietary binaries or a license key here. Test all examples using your downloaded SDK.

## Contents

1. Product overview and capabilities
2. Hardware, OS, and library requirements
3. Official downloads and licenses
4. Windows 11 Python setup
5. Python photo detection
6. Python face comparison
7. Python webcam processing
8. Live GUI/Tracker recognition
9. Linux x86-64 and Linux ARM/Zynq setup
10. C++, C#, JavaScript, and mobile integration
11. Performance, data security, and offline deployment
12. Troubleshooting and acceptance checklist
13. Official sources

---

## 1. What is Luxand FaceSDK?

Luxand FaceSDK is a **commercial native face-detection and face-recognition SDK**. It is not a general YOLO object detector, not a Python-only PyTorch package, and not the same as Luxand's hosted cloud FaceAPI.

It can:

- Find one or many faces in photographs and video frames.
- Return face rectangles, detection confidence, rotation, and primary facial landmarks.
- Detect additional facial feature points.
- Extract compact biometric templates and compare two faces.
- Track faces over time, assign IDs, and associate names with IDs.
- Estimate age-related and expression-related facial attributes.
- Perform passive/active liveness checks; a **separate iBeta-certified liveness add-on** is available.

Typical flow: **image or camera → native FaceSDK library → face detection → optional template matching or tracking → application output and secure storage**.

Version **9.0** introduced updated face-detection/recognition models and additional web deployment options. Version **9.0.1** added tracker feature-point access, tracker fixes, upright stored face images, and lower default tracker recognition thresholds.

| Task | Main API/method |
|---|---|
| Find faces | `FSDK.Image.DetectMultipleFaces()` |
| Find one face | `FSDK.Image.DetectFace()` |
| Make a face template | `FSDK.Image.GetFaceTemplate()` |
| Compare templates | `FSDK.MatchFaces()` |
| Capture from camera | `FSDK.ListCameraNames()`, `FSDK.OpenVideoCamera()`, `Camera.GrabFrame()` |
| Recognize returning people in video | Tracker API |
| Save persistent tracker memory | Tracker persistence API |
| Evaluate liveness | Facial liveness attribute or separately licensed add-on |

**Face recognition is probabilistic.** A similarity score does not establish identity by itself and is not the probability that a person is who they claim to be.

### FaceSDK versus YOLO-OBB

Your earlier YOLO-OBB/Zynq work concerns detection of arbitrary, potentially rotated objects. Luxand FaceSDK specializes in **human faces and biometric identity**. It cannot replace a general satellite-object detector.

## 2. Supported systems and hardware

| Platform | Supported architecture | Important condition |
|---|---|---|
| Windows 7/8/10/11 and listed Server releases | x86 / x64 | Match process and DLL architecture |
| Linux | x86, x86-64, ARMv7, ARM64/AArch64 | **glibc 2.28 or newer** |
| macOS 10.13+ | x86-64 / Apple Silicon ARM64 | Matching `.dylib` |
| iOS 15+ | ARM64 device; supported simulators | iOS-specific build |
| Android 5.0+ (API 21+) | ARMv7, ARM64, x86, x86-64 | Android-specific binary |
| Modern browser | WebAssembly | Vendor WASM wrapper and browser license |

### Official CPU and RAM requirements

| Level | CPU | RAM |
|---|---|---|
| Vendor minimum | 1 GHz processor | 256 MB |
| Vendor recommended | Intel i7/i9/Xeon or AMD Ryzen class | 2 GB |
| Practical Windows 11 developer recommendation (my estimate) | 4+ modern cores | 8–16 GB |

The practical recommendation includes the operating system, editor, Python, and debugging tools; it is **not** Luxand's official minimum.

**GPU:** CPU execution is supported and an NVIDIA GPU is **not mandatory** for basic detection/recognition. Luxand documents optional GPU delegates for some operations, but benefit varies by task. Start in CPU mode even if your workstation has a GTX 1050 Ti. No blanket CUDA installation requirement is documented for the basic Python examples.

**Camera:** Windows Media Foundation/DirectShow-compatible webcams and Linux V4L2 webcams are supported. Built-in IP camera support targets MJPEG. RTSP support is limited/preliminary, with separate vendor sample code.

## 3. Libraries and tools you need

### Windows 11 + Python

| Component | Required? | Reason |
|---|---|---|
| 64-bit Python (3.12 recommended for your workstation) | Yes | Script runtime |
| Python virtual environment | Recommended | Isolation |
| Entire official `fsdk/` Python wrapper directory | **Yes** | Python interface to FaceSDK |
| Official x64 `facesdk.dll` | **Yes** | Native licensed inference engine |
| Valid Luxand trial/production license key | **Yes** | SDK activation |
| `ctypes`, `os`, `sys`, `pathlib` | Built in | Used by wrapper/examples |
| Camera driver and permissions | For webcam examples | Image acquisition |
| Tkinter | For the official `*_tk.py` GUI demos | Window/UI support |
| Pillow | Optional/sample-dependent | Image display |
| OpenCV and NumPy | Optional | Custom video or image processing; **not needed** for core scripts |
| Visual Studio | Only if writing C++/C# | Build tools |

**Critical:** Do **not** assume `pip install facesdk` installs Luxand's proprietary FaceSDK. Luxand's official documentation says to copy the complete `fsdk` package and place a matching native binary in the architecture-dependent subdirectory.

### Other languages

| Language | SDK interface | Native dependency |
|---|---|---|
| C/C++ Windows | `LuxandFaceSDK.h` + `facesdk.lib` | `facesdk.dll` |
| C/C++ Linux | `LuxandFaceSDK.h` | `libfsdk.so` |
| C#/.NET | `FaceSDK.NET.dll` | Matching FaceSDK native binary |
| Java | Vendor Java wrapper/JAR and dependencies | Matching native binary |
| Android/Kotlin | Android wrapper | ABI-specific `.so` |
| Swift/iOS | Apple SDK libraries/frameworks | Device-compatible FaceSDK framework |
| Browser JS | Vendor WebAssembly API | `.wasm` and its companion files |

Luxand documents Python **3.5+**, but that does not guarantee every new Python minor version was individually tested. Python 3.12 is a good first attempt on your existing Windows system; if its particular sample fails, test a vendor-supported Python version.

## 4. Obtain the SDK and license

The 9.0.1 download page advertises approximately:

| Package | Download size |
|---|---:|
| All-in-One SDK | 674 MB |
| Windows binaries | 45 MB |
| Linux binaries | 97 MB |
| All-language wrappers | 83 MB |
| Windows Python sample | 45 MB |
| Linux Python sample | 97 MB |

Download the appropriate **platform SDK** and **Wrappers**, or the All-in-One archive. For the easiest first live application, download the separate **FaceSDK Python** sample as well. Extract them into a known folder such as `C:\SDKs\LuxandFaceSDK` and find the `Binaries` and `Wrappers` directories.

Licensing notes:

1. Request an evaluation key, or use a purchased commercial key.
2. Call `FSDK.ActivateLibrary(key)` **before** `FSDK.Initialize()`.
3. Some licenses are hardware-locked; Luxand supplies a hardware ID utility.
4. Commercial deployment and redistributing native binaries must match your purchased terms.
5. For disconnected servers/ARM devices, ask the vendor specifically about **offline and hardware-locked activation** before deploying.
6. Never include a real key in Git or publicly distributed client code.

## 5. Windows 11 + Python setup

### 5.1 Check your Python interpreter

Open PowerShell:

```powershell
py -0p
py -3.12 --version
py -3.12 -c "import sys,struct; print(sys.executable); print(struct.calcsize('P')*8, 'bit')"
```

The example expects **64 bit**. If you use Python 3.11, replace `-3.12` with `-3.11` in these commands.

### 5.2 Create an isolated project

```powershell
New-Item -ItemType Directory -Force C:\Projects\luxand-demo
Set-Location C:\Projects\luxand-demo
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe --version
```

Optional interactive activation:

```powershell
.\.venv\Scripts\Activate.ps1
```

If your policy blocks the activation script, keep calling `.\.venv\Scripts\python.exe` directly. You do **not** need to change machine-wide execution policy.

### 5.3 Copy the Python wrapper and native DLL

From the extracted Luxand package, copy the **whole** vendor `fsdk` Python package into your project. Then copy the x64 `facesdk.dll` from the SDK Windows binaries to the wrapper's `win64` directory.

Required layout:

```text
C:\Projects\luxand-demo\
  .venv\
  fsdk\
    __init__.py
    ...additional vendor Python modules...
    win64\
      facesdk.dll
  photo.jpg
  detect_faces.py
  match_faces.py
  camera_probe.py
```

Luxand's current Python wrapper expects:

| Interpreter platform | Exact library path **inside `fsdk`** |
|---|---|
| Windows 64-bit | `win64/facesdk.dll` |
| Windows 32-bit | `win32/facesdk.dll` |
| Linux x86-64 | `linux64/libfsdk.so` |
| Linux x86 32-bit | `linux32/libfsdk.so` |
| Linux ARM64 | `linux64_arm/libfsdk.so` |
| Linux ARMv7 | `linux32_arm/libfsdk.so` |
| macOS Apple Silicon | `osx_arm64/libfsdk.dylib` |
| macOS Intel | `osx_x86_64/libfsdk.dylib` |

The **source** Windows DLL is listed in Luxand's C++ guide under the equivalent of `Binaries\Windows\x64\facesdk.dll`. Do not mix x86 and x64 binaries, and do not mix 8.x wrappers with 9.0.1 DLLs.

Confirm installation:

```powershell
Test-Path .\fsdk\__init__.py
Test-Path .\fsdk\win64\facesdk.dll
.\.venv\Scripts\python.exe -c "from fsdk import FSDK; print('FaceSDK loaded')"
```

### 5.4 Set the license for this terminal session

```powershell
$env:LUXAND_LICENSE_KEY = Read-Host "Enter your Luxand license key"
```

This makes the key available to Python child processes without hard-coding it in script files. Keep the terminal/session secure.

### 5.5 Choose a test photo

Save a consented, well-lit image with one or more clearly visible faces as `photo.jpg` in the project directory.

## 6. Python example: detect faces in a photo

Create `detect_faces.py`:

```python
import os
import sys
from pathlib import Path

from fsdk import FSDK


def main():
    if len(sys.argv) != 2:
        raise SystemExit("Usage: python detect_faces.py photo.jpg")

    image_path = Path(sys.argv[1]).resolve()
    if not image_path.is_file():
        raise SystemExit(f"Image not found: {image_path}")

    key = os.environ.get("LUXAND_LICENSE_KEY")
    if not key:
        raise SystemExit("Set LUXAND_LICENSE_KEY first.")

    FSDK.ActivateLibrary(key)
    FSDK.Initialize()
    try:
        print("SDK version:", FSDK.GetVersionInfo())
        image = FSDK.Image(str(image_path))
        try:
            faces = image.DetectMultipleFaces()
            print("Faces detected:", len(faces))
            for index, face in enumerate(faces, start=1):
                print(
                    f"Face {index}: box={face.rect}, "
                    f"confidence={face.score:.3f}, "
                    f"angle={face.angle:.3f} radians"
                )
        finally:
            FSDK.FreeImage(image)
    finally:
        FSDK.Finalize()


if __name__ == "__main__":
    main()
```

Run:

```powershell
.\.venv\Scripts\python.exe .\detect_faces.py .\photo.jpg
```

Illustrative output **format** (not a measured execution):

```text
SDK version: ...9.0.1...
Faces detected: 1
Face 1: box=(120, 80, 300, 270), confidence=0.950, angle=0.010 radians
```

- `face.rect` is a bounding rectangle in pixel coordinates: left, top, right, bottom.
- `face.score` is face **detection confidence**, not identity-match probability.
- `face.angle` is the in-plane rotation in **radians**.
- `DetectMultipleFaces()` returns an empty list when no faces are found.
- Use `FreeImage` in loops; finalize only after all SDK work is complete.

**Current Python syntax:** `from fsdk import FSDK`. Older online snippets sometimes use `import FSDK` with an older wrapper and obsolete APIs.

## 7. Python example: compare two faces

Create `match_faces.py`:

```python
import os
import sys
from pathlib import Path

from fsdk import FSDK


def main():
    if len(sys.argv) != 3:
        raise SystemExit("Usage: python match_faces.py first.jpg second.jpg")

    paths = [Path(value).resolve() for value in sys.argv[1:]]
    for path in paths:
        if not path.is_file():
            raise SystemExit(f"Missing photo: {path}")

    key = os.environ.get("LUXAND_LICENSE_KEY")
    if not key:
        raise SystemExit("Set LUXAND_LICENSE_KEY first.")

    FSDK.ActivateLibrary(key)
    FSDK.Initialize()

    first = None
    second = None
    try:
        first = FSDK.Image(str(paths[0]))
        second = FSDK.Image(str(paths[1]))

        template_a = first.GetFaceTemplate()
        template_b = second.GetFaceTemplate()
        similarity = FSDK.MatchFaces(template_a, template_b)

        print(f"Similarity: {similarity:.4f} on a scale of 0 to 1")
        print("Do not make identity decisions without local threshold testing.")
    except FSDK.FaceNotFound:
        print("At least one image has no suitable face.")
    finally:
        if first is not None:
            FSDK.FreeImage(first)
        if second is not None:
            FSDK.FreeImage(second)
        FSDK.Finalize()


if __name__ == "__main__":
    main()
```

Run:

```powershell
.\.venv\Scripts\python.exe .\match_faces.py .\person_a.jpg .\person_b.jpg
```

**How it works:**

1. Load the two pictures.
2. Extract each picture's highest-confidence face template.
3. Compare the templates using `FSDK.MatchFaces`.
4. Receive a score between **0 and 1**, with higher meaning more similar.
5. Apply your own validated decision threshold **only after accuracy testing**.

For a group photo, use `DetectMultipleFaces()` and then `GetFaceTemplate(face)` to select the specific face instead of implicitly taking the highest-scoring one.

### Thresholds: an essential warning

A match score of **0.80 is not “80% probability of identity.”** Choose a threshold based on your images and acceptable **false acceptance rate (FAR)** and **false rejection rate (FRR)**. The vendor's Face Matching page shows example thresholds and explicitly warns that real cameras, poses and lighting differ from benchmark data. For a door lock, attendance product, or identity verification, perform local genuine/impostor-pair testing and consider MFA and liveness.

## 8. Python example: detect faces from a webcam

This uses Luxand's **native camera API**. It prints results for 100 frames; no OpenCV or Tkinter is required. Create `camera_probe.py`:

```python
import os
from fsdk import FSDK


def main():
    key = os.environ.get("LUXAND_LICENSE_KEY")
    if not key:
        raise SystemExit("Set LUXAND_LICENSE_KEY first.")

    FSDK.ActivateLibrary(key)
    FSDK.Initialize()

    capture_ready = False
    camera = None
    try:
        FSDK.InitializeCapturing()
        capture_ready = True

        names = FSDK.ListCameraNames()
        print("Available cameras:", names)
        if not names:
            raise RuntimeError("No compatible webcam detected.")

        camera = FSDK.OpenVideoCamera(names[0])
        for number in range(1, 101):
            frame = camera.GrabFrame()
            try:
                faces = frame.DetectMultipleFaces()
                print(
                    f"Frame {number}: {len(faces)} face(s), "
                    f"boxes={[face.rect for face in faces]}"
                )
            finally:
                FSDK.FreeImage(frame)
    finally:
        try:
            if camera is not None:
                camera.Close()
        finally:
            if capture_ready:
                FSDK.FinalizeCapturing()
            FSDK.Finalize()


if __name__ == "__main__":
    main()
```

Run:

```powershell
.\.venv\Scripts\python.exe .\camera_probe.py
```

If no cameras appear:

- In Windows Settings, allow camera access for **desktop apps**.
- Close Teams/Zoom/other apps that might exclusively use the camera.
- Confirm Windows Device Manager shows a working camera.
- Try the vendor camera demo.
- On Linux, check V4L2 device nodes and OS permissions.

On Windows, Luxand's camera initialization also initializes COM for the calling thread. Keep camera API calls and cleanup within an appropriate thread.

## 9. See a live graphical recognition demo

For a visual window with actual recognized identities, download the official **FaceSDK Python** sample archive from https://www.luxand.com/facesdk/download/ .

The vendor's Python sample set contains:

| Sample | Purpose |
|---|---|
| `LiveRecognition_tk.py` | Cross-platform Tkinter live face recognition |
| `LiveFacialFeatures_tk.py` | Cross-platform camera landmark tracking |
| `LiveRecognition.py` | Windows-specific live recognition |
| `FacialAttributes.py` | Age, gender, expression attributes for a still image |
| `LivePassiveLiveness.py` | Passive-liveness camera demo |
| `ActiveLiveness.py` | User-interaction liveness demo |
| `Lookalikes.py` | Similar-face database searching |
| `Portrait.py` | CLI facial crop |
| `trackerMemoryTool.py` | Tracker memory manipulation |

After extracting the sample, inspect its imports and license-key setup. A typical invocation from its directory is:

```powershell
.\.venv\Scripts\python.exe .\LiveRecognition_tk.py
```

Some samples depend on Tkinter, Pillow, or sample-specific modules. If an import fails, install **only the dependency shown by that sample's instructions**; the core SDK does not require installing NumPy, OpenCV, or TensorFlow.

### Tracker API: real-time recognition

For an application that recognizes people as they return to the camera:

1. Activate and initialize the SDK.
2. Create a tracker or load existing tracker memory.
3. Configure recognition and optional facial attributes.
4. Feed frames in order using Tracker API.
5. Obtain subject IDs and attach names after verified enrollment.
6. Save tracker memory when appropriate.
7. Free the tracker, camera, and image resources before finalization.

**Use the official live-recognition sample as the starting implementation.** Tracker IDs may be merged over time, so do not assume every temporary ID is a permanent person identity. Luxand's FAQ does **not** promise that one tracker-memory database can be shared concurrently by multiple devices; plan a central template-matching service or a supported merge workflow for multi-camera installations.

## 10. Linux x86-64 implementation

This section is for **Linux using glibc 2.28+**, not bare-metal Linux-less ARM.

### 10.1 Check architecture

```bash
uname -m
getconf GNU_LIBC_VERSION
python3 --version
```

Expected architecture: `x86_64` for 64-bit Intel/AMD. If the glibc version is too old, use a compatible distribution or root filesystem; do not replace system glibc casually.

### 10.2 Create the project

```bash
mkdir -p "$PWD/luxand-demo"
cd "$PWD/luxand-demo"
python3 -m venv .venv
source .venv/bin/activate
python -V
```

Install the operating system's Python `venv` package if it is missing.

### 10.3 Install `fsdk` and the native Linux library

Copy the entire official `fsdk` directory from Luxand's wrapper package beside your Python scripts. Copy the **x86-64 `libfsdk.so`** from the downloaded Linux SDK into:

```text
luxand-demo/
  fsdk/
    __init__.py
    linux64/
      libfsdk.so
  detect_faces.py
  photo.jpg
```

Find the exact source file in your extracted release:

```bash
find /path/to/extracted/LuxandSDK/Binaries/Linux -type f -name libfsdk.so
mkdir -p ./fsdk/linux64
cp /path/to/correct/x86_64/libfsdk.so ./fsdk/linux64/libfsdk.so
file ./fsdk/linux64/libfsdk.so
ldd ./fsdk/linux64/libfsdk.so
```

The `/path/to/...` values are placeholders; use the real SDK archive layout. Use `ldd` only on the trusted library obtained directly from Luxand.

### 10.4 License and run

```bash
read -r -s -p "Luxand key: " LUXAND_LICENSE_KEY
export LUXAND_LICENSE_KEY
python -c "from fsdk import FSDK; print('Native FaceSDK import OK')"
python detect_faces.py photo.jpg
python match_faces.py person_a.jpg person_b.jpg
python camera_probe.py
```

**Headless Linux servers:** Image processing works without a graphical desktop. Camera capture needs an accessible supported camera/device/stream. GUI samples need a display and may require `python3-tk` and Pillow.

## 11. ARM Linux and Zynq implementation

Linux ARM support depends on the **actual processor, Linux architecture, glibc version, and native binary**.

| Hardware | CPU/system | Correct target to test | Important limitation |
|---|---|---|---|
| Raspberry Pi 4/5 running 64-bit Linux | AArch64 | Linux ARM64 | Check glibc, RAM, CPU performance |
| Zynq-7000 | Cortex-A9, ARMv7 | Linux ARMv7 | Legacy PetaLinux rootfs and modest CPU may limit support/FPS |
| Zynq UltraScale+ MPSoC | Usually Cortex-A53 with 64-bit Linux | Linux ARM64 | Some deployments still use a 32-bit userspace |
| Android ARM equipment | Android ABI | Android library | Do not use the ordinary Linux `.so` |
| Bare-metal or RTOS Zynq | No supported Linux userspace | No documented ready-to-run binary | Requires a separate vendor port |

On the device:

```bash
uname -m
getconf GNU_LIBC_VERSION
file /path/to/libfsdk.so
ldd /path/to/libfsdk.so
```

Documented wrapper destinations:

- **AArch64 Linux:** `fsdk/linux64_arm/libfsdk.so`
- **ARMv7 Linux:** `fsdk/linux32_arm/libfsdk.so`

Example on a compatible **AArch64** board:

```bash
mkdir -p ./fsdk/linux64_arm
cp /path/to/actual/ARM64/libfsdk.so ./fsdk/linux64_arm/libfsdk.so
python -c "from fsdk import FSDK; print('ARM64 FaceSDK loaded')"
python detect_faces.py photo.jpg
```

### Can FaceSDK use Zynq FPGA programmable logic?

**Not automatically.** A Linux ARM FaceSDK binary ordinarily executes on the **ARM processing system**. It is not a Vivado/Vitis FPGA bitstream and does not imply automatic acceleration in Zynq programmable logic. For FPGA acceleration you would need a vendor-supported delegate or an independently designed solution; this is **not** a documented one-click path.

### Practical ARM validation process

1. Identify the exact board and CPU (not merely “Zynq”).
2. Check the architecture and **glibc 2.28+** requirement.
3. Confirm RAM and that the matching vendor `.so` can be loaded.
4. Activate a target-compatible license.
5. First test **single photos** without a camera.
6. Process a representative set of at least 100 images and measure median/p95 latency.
7. Add a camera and measure sustained FPS, CPU usage, RAM and temperature.
8. Tune resolution, face patch size and threads.
9. Test reboot/persistence/offline behavior on the actual board.

The official **1 GHz/256 MB** minimum is not a guarantee of practical real-time performance on Zynq-7000. Confirm with real measurements and the vendor.

## 12. Implement FaceSDK using C++

### Windows Visual Studio setup

The SDK provides:

- Header: `Wrappers\C\LuxandFaceSDK.h`
- Windows x64 import library: `Binaries\Windows\x64\facesdk.lib`
- Windows x64 runtime library: `Binaries\Windows\x64\facesdk.dll`

For a **Visual Studio x64** project:

1. Add the header directory to **C/C++ → Additional Include Directories**.
2. Add the import-library folder to **Linker → Additional Library Directories**.
3. Add `facesdk.lib` to **Linker → Input → Additional Dependencies**.
4. Put `facesdk.dll` in the built executable's output directory.
5. Ensure your Debug/Release configuration and library are **both x64**.
6. Activate the SDK, initialize, create/process images, release resources, and finalize.

Example `main.cpp`:

```cpp
#include <cstdlib>
#include <iostream>
#include "LuxandFaceSDK.h"

int main() {
    const char* key = std::getenv("LUXAND_LICENSE_KEY");
    if (key == nullptr) {
        std::cerr << "Set LUXAND_LICENSE_KEY first.\n";
        return 1;
    }

    if (FSDK_ActivateLibrary(const_cast<char*>(key)) != FSDKE_OK) {
        std::cerr << "Activation failed.\n";
        return 1;
    }
    if (FSDK_Initialize(const_cast<char*>("")) != FSDKE_OK) {
        std::cerr << "Initialization failed.\n";
        return 1;
    }

    HImage image = 0;
    int result = FSDK_LoadImageFromFile(&image, "photo.jpg");
    if (result == FSDKE_OK) {
        TFace face = {};
        result = FSDK_DetectFace(image, &face);
        if (result == FSDKE_OK) {
            std::cout << "Score: " << face.score << "\n"
                      << "Bounding box: (" << face.bbox.p0.x << ", "
                      << face.bbox.p0.y << ") to ("
                      << face.bbox.p1.x << ", "
                      << face.bbox.p1.y << ")\n";
        } else if (result == FSDKE_FACE_NOT_FOUND) {
            std::cout << "No face found.\n";
        } else {
            std::cerr << "Detection failed: " << result << "\n";
        }
        FSDK_FreeImage(image);
    } else {
        std::cerr << "Image load failed: " << result << "\n";
    }

    FSDK_Finalize();
    return (result == FSDKE_OK || result == FSDKE_FACE_NOT_FOUND) ? 0 : 1;
}
```

This is an **illustrative 9.0.1 C-API usage example**; inspect the actual downloaded header and compile it against the SDK release you use.

### Linux C++ build

Put the vendor `LuxandFaceSDK.h` in your compiler's header search path and use the correct architecture's `libfsdk.so`. A typical dynamic-library link command is:

```bash
g++ -O2 -I/path/to/SDK/Wrappers/C \
    main.cpp \
    -L/path/to/SDK/Linux/native-library-directory \
    -lfsdk -Wl,-rpath,/path/to/SDK/Linux/native-library-directory \
    -o face-demo
```

Replace placeholders with actual extracted paths. `-rpath` is one method of specifying the runtime library location; for distribution, prefer a controlled app-local path and the packaging layout allowed by your license. Do not assume that a Windows `.lib` file can link on Linux.

## 13. Implement using C#/.NET

Luxand provides:

- `Wrappers\dotNet\bin\FaceSDK.NET.dll`
- The platform-specific native `facesdk.dll` or `libfsdk.so`
- A maintained sample for Windows Forms and WinUI

### Setup

1. Install Visual Studio with .NET desktop development tools.
2. Create a compatible C# project and select the **x64** target if using x64 Windows native libraries.
3. Add a project reference to `FaceSDK.NET.dll`.
4. Place `facesdk.dll` next to the executable.
5. Add `using Luxand;` in source.
6. Activate the library before initialization; release native resources on shutdown.

The vendor documents .NET Framework 4.0 and .NET Standard 2.0 wrapper targets. Some .NET Standard 2.0 uses need `System.Drawing.Common`. Choose the wrapper that matches your target framework.

Simple C# **illustration** using the documented `CImage` object:

```csharp
using System;
using Luxand;

class Program
{
    static void Main()
    {
        string key = Environment.GetEnvironmentVariable("LUXAND_LICENSE_KEY");
        if (string.IsNullOrWhiteSpace(key))
            throw new InvalidOperationException("Missing Luxand license key.");

        int activated = FSDK.ActivateLibrary(key);
        if (activated != FSDK.FSDKE_OK)
            throw new Exception("FaceSDK activation failed: " + activated);

        int initialized = FSDK.InitializeLibrary();
        if (initialized != FSDK.FSDKE_OK)
            throw new Exception("FaceSDK initialization failed: " + initialized);

        try
        {
            using (CImage image = new CImage("photo.jpg"))
            {
                FSDK.TFace face = image.DetectFace();
                Console.WriteLine($"Face confidence: {face.score:F3}");
            }
        }
        finally
        {
            FSDK.FinalizeLibrary();
        }
    }
}
```

Use the **maintained sample** for a fully integrated GUI and iBeta setup:

That repository includes a WinForms project and a WinUI 3/.NET 8 project. Their iBeta liveness functionality also requires the relevant add-on and license. Verify the exact wrapper member names from your SDK download when compiling the illustration above.

## 14. JavaScript, HTML, browser, and mobile applications

### Browser HTML/JavaScript

FaceSDK can run in a browser through the vendor's **WebAssembly build and JavaScript wrapper**. It does **not** work by directly loading a native Windows DLL with regular browser JavaScript.

Implementation path:

1. Download the official WebAssembly package and sample.
2. Host the wrapper, WASM binary, and any required data files using an HTTP server.
3. Obtain the appropriate browser license (Luxand documents **domain-restricted keys**).
4. Allow webcam access with the browser's user-permission flow.
5. Initialize the WASM module and call the documented JS API.
6. Test browser compatibility, WebAssembly memory limits, latency, and permission errors.

The Python `fsdk` wrapper cannot be imported directly into a static HTML page, and a normal website cannot inspect arbitrary user files or cameras without browser permissions.

### Android and iOS

Use **ABI-matched** Android native libraries (ARMv7/ARM64/x86 variants) and the documented Java/Kotlin API, or the iOS XCFramework/static library with Objective-C/Swift. FaceSDK has Flutter and React Native wrappers and vendor examples too. Configure OS camera permission prompts and follow the platform lifecycle; do not copy a Linux `.so` into an Android project and expect it to load.

### Server or REST API

You can put a Python/C++/.NET FaceSDK worker behind your own authenticated API:

**client → HTTP API → image validation → FaceSDK worker → matched IDs/scores → response**.

This does **not** make FaceSDK itself a cloud service. If implemented on your server, inference still occurs on your server. Require authentication, rate limits, restricted image types/sizes, separated tenant data, and secure key management.

## 15. Face templates, tracker storage, and liveness

### Templates

The 9.0.1 technical specifications list **1040 bytes per face template**. Templates are sensitive biometric identifiers, not ordinary anonymous metadata.

A possible storage table:

| Field | Suggested representation | Meaning |
|---|---|---|
| `person_id` | UUID | Application identity |
| `template_version` | Text | SDK/model version used |
| `template_bytes` | Binary/BLOB | Luxand biometric template |
| `created_at` | Timestamp | Enrollment time |
| `consent_reference` | Reference | Record of authorization |
| `status` | Enum | Active or deleted/revoked |

Protect templates with encryption, access controls, tenant isolation, limited retention, and audited deletion. Avoid printing base64 template data or photos in logs.

### Migrating versions

- Version 9.0 introduced a different face detection/recognition pipeline and changed API structures compared with 8.x.
- Although template size remained 1040 bytes, **older templates should not be treated as compatible by default**.
- Version 9.0.1 has tracker fixes and conversion logic; do not rely on it to transparently preserve every older deployment without testing.
- Back up tracker memory, read the migration documentation, and verify recognized identities/thresholds before switching production versions.

### Passive and active liveness

- **Passive liveness** estimates the probability that the detected face is of a live subject. Luxand says it requires **RGB/color images** rather than grayscale images.
- **Active liveness** challenges the subject to perform actions (such as head movement in the provided demos).
- The **iBeta Certified Liveness Add-on** is a distinct package/solution. Do not describe ordinary passive liveness as equivalent to the certified add-on.
- Liveness is not foolproof and should be tested against the specific spoofing risks of your use case.

## 16. Performance optimization and capacity planning

1. **Measure first** on your actual hardware, not using another person's benchmark.
2. Reduce image resolution while keeping enough pixels for small/distant faces.
3. Tune `FaceDetectionPatchSize` and `FaceDetectionThreshold` after establishing baseline accuracy.
4. Tune `FaceDetectionNumThreads` for CPU cores, concurrency, and device thermals.
5. Use the Tracker API for continuous recognition across consecutive video frames.
6. Benchmark optional GPU delegates against CPU; different SDK stages benefit differently.
7. Release temporary frame images on **every frame** to avoid native-memory growth.
8. Measure median and p95 latency, peak RAM, CPU usage, actual FPS, and false accepts.
9. For ARM/Zynq, check sustained power/temperature throttling over longer runs.
10. Use an explicit face-matching threshold chosen from your own genuine/impostor test images.

Luxand's published detection benchmark uses particular settings and test systems. A figure from its Intel i7/RTX laptop benchmark should **not** be presented as guaranteed FPS on your GTX 1050 Ti, Raspberry Pi, or Zynq.

## 17. Offline installation checklist

For your offline Windows/Linux/ARM machine, prepare on an internet-connected computer:

- [ ] Official FaceSDK **9.0.1** binary package for the exact OS and CPU architecture.
- [ ] Matching official `fsdk` Python wrapper or C++/.NET wrapper.
- [ ] Python installer and a virtual environment plan (if using Python).
- [ ] Valid evaluation or production license, with **offline/hardware lock terms confirmed** by Luxand.
- [ ] Any native shared-library/runtime dependencies identified for your package.
- [ ] Optional GUI packages and Python wheels, but only if the chosen examples require them.
- [ ] Known-good test photos; script from sections 6–8; acceptance checklist.
- [ ] Local archive and file-integrity checks so you can reproduce the installation.
- [ ] Secure removable-media transfer process.

On the target:

1. Check architecture and runtime compatibility.
2. Copy SDK files and wrapper into the required folder layout.
3. Import the wrapper.
4. Activate the license.
5. Run the static-image test.
6. Run template comparison.
7. Run the camera check if appropriate.
8. Benchmark and record the exact software/hardware versions.

**Offline warning:** FaceSDK's local inference capabilities do not by themselves prove that every license type can be activated without network access. Confirm license requirements before building the offline device.

## 18. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `ModuleNotFoundError: No module named 'fsdk'` | Python package not in project/PYTHONPATH | Copy complete vendor `fsdk` folder beside script |
| “FaceSDK binary ... could not be found” | Native library absent from expected package subfolder | Check `fsdk/win64/facesdk.dll` or corresponding Linux/ARM path |
| Windows `[WinError 193]` / `BadImageFormatException` | x86/x64 mismatch | Match process interpreter/build to DLL architecture |
| Windows `[WinError 126]` | Missing dependent DLL or incorrect binary loading path | Inspect package and loader dependencies |
| `libfsdk.so` loading error | Wrong directory, architecture, permissions, or missing dependency | Use `file` and `ldd` on trusted vendor binary |
| `GLIBC_2.xx not found` | Linux rootfs older than SDK requirement | Use supported glibc 2.28+ rootfs |
| `FSDK.NotActivated` | Invalid/expired/hardware-mismatched key | Check entitlement, key, hardware ID and activation order |
| Wrapper imports but detection fails | Wrapper/binary version mismatch or no initialization | Use one release's binaries and call `ActivateLibrary` then `Initialize` |
| No faces | Lighting, blur, small/distant face, detector tuning | Test a clear front-facing image first |
| Camera list empty | Permissions or unsupported/busy camera | OS settings, V4L2/Media Foundation/DirectShow, vendor sample |
| Slow video | Too many pixels, conservative detection settings, slow CPU | Measure and tune resolution/patch size/threads |
| Memory growth | Frame images/trackers not freed | Release resources each iteration and shutdown |
| Incorrect identity matches | Thresholds, bad enrollment, low-quality images | Validate on representative genuine/impostor pairs |
| Old face templates rejected | Recognition model/version changes | Consult migration guide; re-enroll or convert as supported |
| GUI sample fails while still-image test works | Missing Tkinter/Pillow/sample dependency | Read that sample's imports and install relevant optional packages |

### Acceptance-test sequence

- [ ] Correct Python version and interpreter bitness.
- [ ] Matching native binary is in the wrapper directory.
- [ ] `from fsdk import FSDK` works.
- [ ] `FSDK.ActivateLibrary` succeeds with a valid key.
- [ ] `FSDK.Initialize` and `FSDK.GetVersionInfo` succeed.
- [ ] A known-good image returns one or more faces.
- [ ] Two known-good images return a matching similarity score.
- [ ] Camera enumerates and processes frames.
- [ ] Vendor live-recognition GUI starts.
- [ ] Native resources are released on shutdown.
- [ ] Accuracy/FPS/RAM/temperature meet application-specific requirements on the target.
- [ ] Version, license, and deployment procedure are documented.

## 19. Privacy, security, and licensing checklist

- Use facial biometrics only for legitimate purposes with suitable notice, authorization, and legal review.
- Secure enrolled templates, tracker files, keys and backups.
- Define biometric retention and deletion procedures.
- Restrict image uploads and use authentication/rate limiting for services.
- Do not log credentials, photos or face templates in plaintext.
- Do not treat a model score as proof of identity.
- Use liveness and additional authentication for higher-risk workflows.
- Calibrate for your camera environments and test for error rates across users.
- Confirm SDK redistribution, commercial usage, and hardware-lock terms with Luxand.

## 20. Verified official source links

The following **vendor sources** were checked on **October 26, 2026**.

## Recommended first path for your workstation

Since you already use **Windows 11 and Python 3.12**, begin with **FaceSDK 9.0.1 x64 + official `fsdk` wrapper + x64 `facesdk.dll`**:

1. Confirm that import and license activation succeed.
2. Run `detect_faces.py` on `photo.jpg`.
3. Run `match_faces.py` on two photos.
4. Run `camera_probe.py` on your integrated/USB webcam.
5. Run Luxand's `LiveRecognition_tk.py` to see live named recognition.
6. Only then migrate the image test to Linux ARM or your **exact** Zynq board.

**A successful installation** means the SDK reports its expected version, validates a real license, processes still images and camera frames without library errors, and releases native resources cleanly.


