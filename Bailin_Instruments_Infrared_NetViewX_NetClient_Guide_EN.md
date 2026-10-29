# Bailin Instruments, Infrared Imaging, NetViewX.OCX and NetClient.dll

**Complete English setup, integration, acquisition, and troubleshooting guide**  
**Research date:** October 26, 2026  
**Starting environment:** Windows 11; optional Python 3.12; local authorized camera or instrument

> **Important finding:** “佰霖仪器” is the brand of **Henan Bailin Instrument Equipment Co., Ltd. (河南佰霖仪器设备有限公司)**. The company sells various inspection products, including industrial thermal-imaging equipment. Its official website describes the **BL-DT100 dual-spectrum thermal imaging/temperature monitoring unit**.
>
> However, I could **not verify** that `NetViewX.OCX` and `NetClient.dll` are supplied with any Bailin model. Those filenames appear together in older **Launch/朗驰 network camera and video-server manuals**. I could not verify an official public direct download for either file. Do **not** treat a same-named DLL from an unrelated website as compatible.
>
> This guide distinguishes confirmed facts from **conditional instructions** requiring your exact device model, firmware, SDK documentation, and authorized software package. There are **no fabricated DLL export names, COM methods, camera IP addresses, passwords, thermal calibration formulas, or download URLs**.

> **Priority update — object surface temperatures:** See **§12.4–12.11** for the full NetViewX.OCX + NetClient.dll workflow, including object selection, SDK capability checks, thermal-pixel mapping, executable ROI statistics, and calibration. Neither file has a verified public temperature API; the DLL can supply °C **only if your camera is radiometric and the matching OEM SDK documents that capability**.

## 1. What is 佰霖仪器?

- **Chinese legal/company name:** 河南佰霖仪器设备有限公司.
- **English:** Henan Bailin Instrument Equipment Co., Ltd.
- **Main categories shown on its site:** industrial online thermal imagers, industrial video endoscopes, water/environment testing instruments, construction testing and nondestructive testing tools.

The BL-DT100 listing describes thermal temperature monitoring and warning for settings such as charging areas, warehouses, and equipment monitoring. **It does not document that it uses NetViewX.OCX, NetClient.dll, RTSP, ONVIF, or a public radiometric SDK.** Ask for the **exact model manual** before selecting an integration method.

An industrial thermometer, USB endoscope, near-infrared surveillance camera, and network thermal camera have very different software interfaces. There is no one universal “Bailin software installer.”

## 2. What does “get infrared” mean?

| Requested output | Hardware required | What software can obtain |
|---|---|---|
| **IR night vision** (near-infrared illumination) | Camera sensitive to near-IR, often with IR LEDs | Typically black-and-white night video; **not temperature** |
| **Thermal image** (long-wave/other thermal IR) | Thermal-imaging sensor | Pseudocolor or grayscale thermal video/photo |
| **Actual temperature at a point/region** | A temperature-capable, calibrated/radiometric thermal device | Vendor-provided temperature values or metadata |
| **Raw thermal pixel values** | Device/firmware exposing raw radiometric output | Raw counts or radiometric array with vendor calibration |
| **Control of IR illuminator** | Supported camera with controllable IR LED/day-night mode | Device-specific control commands |

**Critical:** A colorful “thermal-looking” H.264/RTSP frame may contain **only rendered colors**. You cannot recover accurate °C temperatures from those colors without the manufacturer's radiometric information and calibration. A regular night-vision camera cannot measure heat just because it uses infrared light.

## 3. What are the two legacy files?

| Filename | Likely historical role | What it does **not** prove |
|---|---|---|
| `NetViewX.OCX` | Windows **ActiveX/COM video-viewing control**, historically installed for browser previews | No proof of a thermal sensor, raw radiometry, temperature API, or Bailin compatibility |
| `NetClient.dll` | Native dependency/network client/decoder associated with that legacy camera stack | Not necessarily a COM control; not necessarily a documented public SDK |
| `NetViewX.cab` | Old Windows Cabinet deployment package that may include OCX and related files | Version-specific; not guaranteed to be portable between camera brands/firmware |

Evidence: legacy Launch/朗驰 documentation names `NetClient.dll`, `NetViewX.cab` and `NetViewX.ocx` together and says the compatible network camera's own browser interface can supply a replacement playback component after a firmware update. That establishes a **legacy network video component**, not a Bailin infrared-temperature interface.

**Do not use** random “free DLL download” or registry-fixer websites. Executing or registering an unverified OCX/DLL can run third-party native code with your user or administrator privileges.

## 4. Where can you download NetViewX.OCX and NetClient.dll?

### Current verified download status

| File | Verified official standalone download | Best acquisition route |
|---|---|---|
| `NetViewX.OCX` | **None found** | Request matching signed OCX/CAB from the **actual camera/OEM support** |
| `NetClient.dll` | **None found** | Request the complete matching client/SDK package and dependencies from the OEM |
| `NetViewX.cab` | No public universal URL verified | **Only if your compatible device's authorized web interface actually offers it** |

Useful **official or authoritative starting points**, not direct DLL links:

### Safest acquisition procedure

1. Photograph the **device label** with manufacturer, exact model, hardware revision, serial number and firmware version (avoid publishing private identifiers).
2. Ask the supplier whether the device **actually uses these named files**.
3. Request an official **signed installer/CAB**, all companion DLLs, an SDK ZIP, header/API reference, and a compatible Windows/version/bitness statement.
4. If the vendor explicitly says the **device itself distributes its OCX**, connect to the **authorized device over an isolated local network**, open its own web UI, and use its *Download control/plugin* link. This was the workflow documented for certain old Launch cameras, **not proof that Bailin does this**.
5. Do not guess an endpoint. If offered, use only the **actual link published by your device**, and check source, version, and signature.
6. Preserve the whole package. OCX and DLL versions may need to match the device firmware.

### What to request from the vendor

- Camera model + firmware-specific client installer (x86 or x64).
- `NetViewX.OCX`, `NetClient.dll`, optionally `NetViewX.cab`, matching companion files.
- Authenticode signature or vendor-published **SHA-256** checksums.
- OCX **CLSID/ProgID**, type library (`.tlb`) or COM automation API.
- Native `NetClient.dll` C header (`.h`), calling conventions, structure layouts, examples.
- Supported video paths: **RTSP/ONVIF/USB/UVC**, stream URL format and codec.
- Thermal/radiometric temperature SDK, raw frame specification, units and calibration documentation.
- Licensing, redistribution permission, supported OS and safety notes.

## 5. Choose an architecture before installing anything

**Route A — Need only thermal video:** Try vendor PC app; if explicitly supported, use **RTSP/ONVIF** and OpenCV/VLC. You may not need the OCX/DLL at all.

**Route B — Need temperatures/temperature matrix:** Obtain the device's **radiometric SDK**, temperature stream/data format, or vendor export API. The old viewer OCX alone is not enough.

**Route C — Need only old browser preview:** Use a verified **matching** OCX/CAB, Windows COM registration, and an approved legacy host or **Edge IE mode**.

**Route D — Need vendor-specific functions in a C#/C++ app:** Obtain the complete client SDK and **real headers/COM type library** before calling the DLL.

**Route E — Need to turn IR LEDs/night mode on/off:** Use the camera's documented day/night/IR controls; this does not require raw thermal data.

## 6. Physical device setup for a Bailin thermal instrument

**These are general safe commissioning steps, not undocumented BL-DT100 button/menu instructions.**

1. **Identify the model.** Record exact SKU, hardware revision, firmware, interfaces, supported supply voltage, and manual.
2. **Power safely.** Use only the voltage/PoE type and wiring specified on the manufacturer label. Never guess a DC voltage or pinout.
3. **Mount correctly.** Ensure the field of view covers the thermal target, the lens is clean, and the device is in the permitted environment.
4. **Connect data.** Ethernet, USB, serial, Wi-Fi, or an included NVR may be used depending on the model.
5. **Configure networking.** Use the official discovery tool/manual to set an IP address; keep PC and device reachable on the same authorized LAN/VLAN. **No assumed factory IP/password.**
6. **Verify with the vendor application first.** Confirm live visible-light and thermal channels separately if the device is dual-spectrum.
7. **Temperature setup (if supported):** set emissivity, measurement ROI, reflected/background temperature, atmospheric/distance options, measurement range, and calibration according to the instrument manual.
8. **Alarm setup (if supported):** choose temperature region and threshold, dwell/hysteresis, notifications and recording; test with a controlled known-safe target.
9. **Export/test.** Save a normal image, a thermal image, any temperature table, and a short video sample. Check timestamps, units, metadata, and camera resolution.
10. **Document results.** Record firmware, software version, optical setup, emissivity, and test conditions.

Do **not** rely on an unverified alarm threshold for life-safety or fire protection.

## 7. Windows 11 preparation and binary verification

**Use an isolated test VM or restricted lab PC for any old ActiveX control.** Do not register unknown files on your main workstation. Registration and loading execute native code.

### 7.1 Obtain the complete trusted package

Put only verified vendor files in, for example:

```text
C:\BailinTest\Vendor\
  NetViewX.ocx
  NetClient.dll
  NetViewX.cab          # only if vendor supplied it
  vendor-setup.exe      # only if vendor supplied it
  additional-deps\...
```

This is an **example target layout**, not a claim the files are already available.

### 7.2 Check signatures, versions and checksums (read-only)

PowerShell:

```powershell
$ocx = "C:\BailinTest\Vendor\NetViewX.ocx"
$dll = "C:\BailinTest\Vendor\NetClient.dll"
Test-Path $ocx
Test-Path $dll
Get-Item $ocx, $dll | Select-Object Name, Length, VersionInfo
Get-AuthenticodeSignature -FilePath $ocx
Get-AuthenticodeSignature -FilePath $dll
Get-FileHash -Algorithm SHA256 -Path $ocx
Get-FileHash -Algorithm SHA256 -Path $dll
```

Compare hashes/signatures against the **manufacturer's** reference. A checksum alone does not establish that an unknown source is trustworthy. A missing or invalid signature should be taken up with the vendor; do not respond by disabling SmartScreen or globally permitting unsigned ActiveX controls.

### 7.3 Determine x86 or x64

With Visual Studio's Developer Command Prompt:

```bat
dumpbin /headers "C:\BailinTest\Vendor\NetViewX.ocx"
dumpbin /headers "C:\BailinTest\Vendor\NetClient.dll"
dumpbin /dependents "C:\BailinTest\Vendor\NetViewX.ocx"
dumpbin /exports "C:\BailinTest\Vendor\NetClient.dll"
```

Look for `machine (x86)` versus `machine (x64)` and the file's dependent DLLs. **Export names alone do not reveal safe argument types or their semantics.** You must get the OEM header/manual before calling exported functions.

### 7.4 If the vendor supplies a CAB

For **inspection**, not automatic installation:

```powershell
New-Item -ItemType Directory -Force "C:\BailinTest\Extracted"
expand.exe -F:* "C:\BailinTest\Vendor\NetViewX.cab" "C:\BailinTest\Extracted"
```

Inspect its `.inf`, OCX/DLL dependencies, publisher and version. Prefer the vendor's **signed installer** where provided; a manual OCX copy may omit required steps.

## 8. Register a verified OCX — legacy route only

**Only after** you have confirmed this is a trusted, compatible self-registering COM/ActiveX control, and ideally within a disposable isolated test VM:

For a **32-bit** OCX on 64-bit Windows, open an **elevated** Command Prompt and run:

```bat
C:\Windows\SysWOW64\regsvr32.exe "C:\BailinTest\Vendor\NetViewX.ocx"
```

For a true **64-bit** OCX:

```bat
C:\Windows\System32\regsvr32.exe "C:\BailinTest\Vendor\NetViewX.ocx"
```

**Counterintuitive Windows naming:** `SysWOW64` contains 32-bit registration tools; `System32` contains 64-bit tools on x64 Windows.

**Do not blindly run `regsvr32` on `NetClient.dll`.** Ordinary native DLLs are loaded by another program and may have no `DllRegisterServer` export. Use the installer/SDK directions.

If COM registration reports that the module cannot load, check dependencies, paths and bitness. If `DllRegisterServer` is absent, the file may not be self-registering (or may be damaged); request the correct installer.

## 9. Can a browser use NetViewX.OCX?

- **Current browsers:** No conventional ActiveX support. A normal HTML/JavaScript page cannot load an OCX.
- **Legacy IE compatibility host:** Some managed Windows setups still let selected intranet sites run certain old **ActiveX controls** through an IE compatibility host. The site must be added by organization policy. That host is **not** a guarantee that a particular old control works.
- **Windows desktop COM host:** Possible if the control is compatible, correctly registered, and used through documented COM interfaces.

Recommended testing procedure:

1. Update Windows.
2. Confirm the source is a **trusted private LAN site**.
3. If the organization still provides a legacy IE compatibility host, add only that site through the approved site list or internal process.
4. Make sure the verified vendor control is installed.
5. Open only the corresponding camera's legitimate web page.
6. Verify that preview works; do not weaken global browser security settings or bypass publisher verification.

**Do not invent a CLSID.** An HTML `<object>` tag requires the *real* CLSID and documented API. Ask the vendor or inspect the **verified** control with Microsoft's OLE/COM Object Viewer (`oleview.exe` from Windows SDK).

## 10. Inspect the COM interface or native SDK without guessing methods

### If the OCX exposes COM

1. Install the Windows SDK's **OLE/COM Object Viewer**.
2. Open the **verified** OCX/type library using that tool.
3. Record the CLSID, ProgID, interfaces, events, argument types, and method names.
4. Generate a .NET interop assembly if needed (e.g., Visual Studio COM reference tooling).
5. Build a **32-bit host** for a 32-bit in-process OCX.
6. Use only documented camera methods and error codes.

### If NetClient.dll exports ordinary C functions

1. Read the OEM `NetClient.h` or official SDK manual.
2. Determine export name, calling convention (`cdecl` vs `stdcall`), argument sizes, structure alignment, encoding, and resource lifecycle.
3. Match application process architecture to the DLL.
4. Only then write C/C++ or C# P/Invoke declarations, or Python `ctypes` bindings.
5. Test login, stream start, callback, disconnect and cleanup **in that order**, using the official sample.

**No runnable NetClient.dll infrared function call is provided here because no authoritative exported function signatures for your specific build could be verified.** Guessing them risks application crashes or incorrect thermal data.

## 11. Infrared video without legacy ActiveX: RTSP/ONVIF route

This route applies **only if the exact camera documents an RTSP stream or ONVIF profile**. An old Launch manual gives an RTSP example for a Launch model; **do not reuse its IP or URL as a Bailin default**.

### Step 1 — Find the real stream

From the device manual/vendor application, obtain:

- Camera host/IP and stream path for **thermal channel** versus RGB channel.
- Protocol and port.
- Authentication type and actual authorized credentials.
- Video codec/resolution.
- Whether the stream is thermal **rendered video only** or includes radiometric metadata.

### Step 2 — Test with VLC/ffplay

Paste the manufacturer-provided RTSP URL into your authorized video client. If you use `ffprobe`/FFmpeg, test it with the exact supplied URL, not guessed URL paths.

### Step 3 — Test with Python + OpenCV

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install opencv-python
$env:CAMERA_RTSP_URL = Read-Host "Enter your vendor-confirmed RTSP URL"
```

Create `thermal_video.py`:

```python
import os
import cv2

url = os.environ.get("CAMERA_RTSP_URL")
if not url:
    raise SystemExit("Set CAMERA_RTSP_URL using the actual camera manual.")

cap = cv2.VideoCapture(url, cv2.CAP_FFMPEG)
if not cap.isOpened():
    raise SystemExit("Unable to open this RTSP stream.")

try:
    saved = False
    for _ in range(150):
        ok, frame = cap.read()
        if not ok:
            break
        if not saved:
            if not cv2.imwrite("camera_frame.png", frame):
                raise RuntimeError("Could not save a video frame.")
            saved = True
        cv2.imshow("Camera video (not calibrated temperatures)", frame)
        if (cv2.waitKey(1) & 0xFF) == ord("q"):
            break
    if not saved:
        print("Connected but received no decodable frame.")
finally:
    cap.release()
    cv2.destroyAllWindows()
```

Run:

```powershell
.\.venv\Scripts\python.exe .\thermal_video.py
```

**Result:** You may save/display thermal-looking frames **if the selected stream is actually from the thermal channel**. The saved PNG is **not** a temperature matrix. OpenCV may require a supported FFmpeg backend and working authentication/codec.

## 12. How to obtain actual temperature (°C) or raw thermal pixels

This is a separate integration path:

1. **Confirm the hardware is radiometric.** Some devices output an image or alarm event but no per-pixel temperature matrix.
2. Ask for the **radiometric SDK** or documented metadata/temperature export endpoint.
3. Ask whether the SDK returns **°C, Kelvin, centi-Kelvin, raw detector counts, 16-bit arrays, or already corrected values**.
4. Obtain the **width/height, row stride, endian order, frame timestamp, and calibration version**.
5. Obtain formulas and settings for emissivity, ambient/reflected temperature, transmission, distance, and relevant sensor calibration.
6. Use the exact vendor functions to acquire synchronized temperature data and images.
7. Validate temperatures against a suitable known reference and the manufacturer accuracy specification.
8. Only then save to CSV/database or implement high-temperature alarms.

**Do not assume** a generic rule such as “pixel / 100 = °C.” Such conversion depends on the actual device and firmware.

An example *application-owned output schema* (not a vendor API claim):

```json
{
  "device_model": "FROM_DEVICE",
  "captured_at_utc": "YYYY-MM-DDTHH:MM:SSZ",
  "thermal_width": 256,
  "thermal_height": 192,
  "temperature_unit": "C",
  "pixel_temperature_array": "ONLY_IF_VENDOR_SUPPORTS_RADIOMETRIC_EXPORT",
  "emissivity": "FROM_MEASUREMENT_CONFIGURATION"
}
```

The 256×192 values above are merely a **schema example** and **not BL-DT100 specifications**.

### 12.4 The exact capability required from NetClient.dll

**Target result:** select a physical object in the `NetViewX.OCX` preview and read that object's **measured surface temperature** via `NetClient.dll`. Merely reading image RGB values is not thermometry.

| Actual device + SDK capability | Can it yield °C? | Role of the two files |
|---|---|---|
| Calibrated **radiometric** thermal sensor; vendor documents NetClient temperature-frame or spot/ROI API | **Yes**, when appropriately configured/calibrated | OCX previews; DLL supplies calibrated measurements |
| Radiometric sensor; DLL supplies only spot/ROI values | **Yes**, for the documented points/regions | OCX selects object; DLL queries native thermal coordinates |
| Thermal-looking video but DLL exposes only video/decoder functions | **No verified numerical API** | Both may show video; neither proves temperature export |
| Ordinary near-infrared CCTV with IR LED night vision | **No** | Camera hardware is not a thermal thermometer |

The historical manuals that mention these filenames describe **video playback**, not per-object thermal measurements. No legitimate exported temperature function signature for your specific `NetClient.dll` version has been verified. If its OEM SDK lacks radiometry, **do not invent `GetTemp()`, infer °C from false-color pixels, or label night vision as thermography**.

### 12.5 Implementation using BOTH components, in order

```text
Real radiometric thermal camera / verified thermal channel
  ├─ video → NetViewX.OCX → object selection (screen ROI)
  │                              └─ display/thermal geometry transform
  └─ calibrated measurements → NetClient.dll (ONLY if documented in OEM SDK)
                                 └─ point / ROI statistics OR temperature array
       match channel + frame timestamp + native thermal ROI
                   ↓
       object surface temperature: min / mean / max / p95 °C
                   ↓
       visible result, audit log, optional threshold alarm
```

1. **Identify hardware and prove thermometry.** With official vendor software, measure a known target and confirm the numeric temperature is based on the actual thermal sensor. A pseudo-color video is insufficient.
2. **Get one matched OEM SDK release.** Require the real `NetClient.h` header, demo software, `NetViewX.OCX` type library/CLSID and compatible versions of all dependencies.
3. **Prepare the Windows host.** Follow §§7–10. The OCX host, native DLL and application must have compatible architecture (often x86 for old controls). Prefer an isolated lab VM.
4. **Open the OCX thermal preview** with its documented interface. If the viewer is displaying RGB/visible-light channel, switch to the authentic thermal channel.
5. **Connect using NetClient.dll's documented initialization/login/stream APIs.** It may need a second session; follow the vendor's sample. Never guess the DLL signatures.
6. **Start the documented radiometric data subscription/query.** Determine whether the DLL returns point °C, ROI max/mean/min, a per-pixel calibrated °C array, or a raw detector array requiring the OEM calibration routine. Record units, sensor width/height, status codes, frame timestamp and invalid-pixel convention.
7. **Select the object ROI** in the OCX preview (mouse click, rectangle or object detector). Map the coordinates to the native thermal measurement grid (§12.6).
8. **Query/synchronize:** Feed the mapped coordinates into the actual DLL ROI/spot API; OR extract those pixels from the matching calibrated temperature frame (§12.8). Reject a frame from the wrong channel, time or camera.
9. **Check validity and physics** (§12.9); present min/mean/max or documented spot temperature with source, UTC time, emissivity, measurement status and uncertainty.
10. **Close safely:** stop temperature callbacks, video preview and device sessions, then release native and COM resources, in the vendor's documented order.

**Stop condition:** If step 6 is unsupported, you cannot measure an object's temperature from these files alone. Request a radiometric SDK/firmware or use a different temperature-capable device.

### 12.6 Object ROI mapping: pixels in OCX versus calibrated pixels

A mouse click in a video window is not automatically a thermal-array coordinate. For an **unrotated, uncropped thermal image shown entirely in the OCX viewport with aspect-preserving scaling and black bars**, define:

- `Vw,Vh` = OCX **video viewport** width/height in display pixels (exclude toolbars).
- `Tw,Th` = native calibrated temperature-array width/height from the SDK.
- `vx,vy` = selected object point relative to viewport top-left.
- `scale = min(Vw/Tw, Vh/Th)`.
- `left = (Vw - Tw×scale)/2`, `top = (Vh - Th×scale)/2`.
- `tx = floor((vx-left)/scale)`, `ty = floor((vy-top)/scale)`.

**Only accept** `0 ≤ tx < Tw`, `0 ≤ ty < Th`. Convert both rectangle corners, using `[x1,y1,x2,y2)` with exclusive right/bottom edges.

**Example (geometry only, not a Bailin specification):** `V=1280×720`, `T=320×240`, scale 3, image 960×720 and 160-pixel left/right bars. A click at viewport `(640,360)` selects native thermal pixel **`(160,120)`**. A click at `(50,360)` lands in the black bar and must be rejected.

**Critical exceptions:** Zoom, pan, cropping, stretching, rotation, split-screen views or visual-light/thermal fusion change the mapping. For these, obtain the **inverse transform and OEM visible/thermal registration calibration**; never simply scale RGB boxes to the thermal grid. For YOLO/OBB detections, map the **polygon/segmentation mask** where possible, not only an enclosing box that might contain hot background. Keep boundary/invalid pixels out of the ROI.

### 12.7 Identify the genuine NetClient and NetViewX functions

Inspect **trusted** matched files with Visual Studio/Windows SDK:

```bat
dumpbin /exports "C:\BailinTest\Vendor\NetClient.dll"
dumpbin /headers "C:\BailinTest\Vendor\NetClient.dll"
dumpbin /dependents "C:\BailinTest\Vendor\NetClient.dll"
```

Use Windows SDK `oleview.exe` to inspect the actual OCX **CLSID/ProgID, type library, methods, properties and events**. Ask the OEM specifically for:

- **Temperature frame:** thermal width/height, stride, units, signed/unsigned format, invalid values, calibration, timestamp and callback contract.
- **Point temperature:** point query parameters in **native thermal coordinates**.
- **ROI temperature:** rectangle/polygon coordinate definition and returned min/max/mean.
- **Video integration:** preview event/callback and thermal channel/frame ID matching.
- **DLL ABI:** function prototypes from `NetClient.h`, `cdecl`/`stdcall`, pointer ownership, struct packing, Unicode/ANSI strings, lifecycle and error codes.

A name visible in `dumpbin` is **not** enough to write safe `ctypes` or P/Invoke bindings. Only the actual header/manual provides signatures. Here is the precise **binding sequence, NOT real method names**:

```text
OCX: [create trusted COM viewer with OEM CLSID; show thermal channel]
DLL: [initialize SDK; authenticate; start temperature capability]
OCX: [get selected object ROI and corresponding preview timestamp]
APP: [transform ROI to native thermal sensor coordinates]
DLL: [read synchronized temperature array OR query documented ROI °C]
APP: [reject invalid/stale data; compute object surface temperature]
UI:  [show °C and measurement provenance; optionally save results]
BOTH: [stop callbacks; logout; free DLL and COM resources]
```

### 12.8 Working Python ROI temperature calculator (downstream of the DLL)

This **runnable Python 3.12** example uses only the standard library. It processes data **after an OEM-authenticated NetClient.dll adapter provides a calibrated temperature matrix**. The example **does not load the DLL** or claim it exports JSON. Never feed it an OCX screenshot, H.264 color pixels, or uncalibrated raw sensor counts.

Define your **application-owned** JSON format (numbers below are *synthetic examples*, not device specifications):

```json
{
  "source": "VENDOR_CALIBRATED_NETCLIENT_SDK",
  "timestamp_utc": "2026-10-26T15:00:00Z",
  "width": 4,
  "height": 3,
  "unit": "C",
  "temperature_c": [27.0, 28.0, 29.0, 26.0,
                    27.5, 34.0, 36.0, 26.5,
                    28.0, 33.0, 35.0, 27.0],
  "roi_xyxy": [1, 1, 3, 3],
  "emissivity": 0.95
}
```

Save as `object_temperature.py`:

```python
import json
import math
import statistics
import sys


def measure(frame):
    if frame.get("source") != "VENDOR_CALIBRATED_NETCLIENT_SDK":
        raise ValueError("No verified calibrated SDK temperature source.")
    if frame.get("unit") != "C":
        raise ValueError("Expected vendor-converted degrees Celsius.")

    w, h = int(frame["width"]), int(frame["height"])
    temperatures_c = frame["temperature_c"]
    if w <= 0 or h <= 0 or len(temperatures_c) != w * h:
        raise ValueError("Bad calibrated thermal array dimensions.")
    x1, y1, x2, y2 = map(int, frame["roi_xyxy"])
    if not (0 <= x1 < x2 <= w and 0 <= y1 < y2 <= h):
        raise ValueError("ROI is outside the thermal array.")

    mask = frame.get("mask")  # Optional w*h object segmentation mask
    if mask is not None and len(mask) != w * h:
        raise ValueError("Invalid ROI mask length.")

    samples, selected = [], 0
    for y in range(y1, y2):
        for x in range(x1, x2):
            i = y * w + x
            if mask is not None and not mask[i]:
                continue
            selected += 1
            val = temperatures_c[i]
            if val is None:  # OEM invalid pixel must be normalized to None
                continue
            t = float(val)
            if math.isfinite(t) and t > -273.15:
                samples.append(t)

    if not selected or len(samples) / selected < 0.90:
        raise ValueError("Insufficient valid target pixels.")
    ordered = sorted(samples)
    p95 = ordered[math.ceil(len(ordered) * 0.95) - 1]
    return {
        "min_c": ordered[0],
        "mean_c": statistics.fmean(ordered),
        "max_c": ordered[-1],
        "p95_c": p95,
        "valid_pixels": len(ordered),
        "roi_xyxy": [x1, y1, x2, y2],
        "timestamp_utc": frame.get("timestamp_utc"),
        "emissivity": frame.get("emissivity")
    }


if __name__ == "__main__":
    if len(sys.argv) != 2:
        raise SystemExit("Usage: py -3.12 object_temperature.py calibrated_frame.json")
    with open(sys.argv[1], encoding="utf-8") as f:
        print(json.dumps(measure(json.load(f)), indent=2))
```

To calculate measurements from the **real SDK output**, the OEM adapter must first save `calibrated_frame.json`. Then run:

```powershell
py -3.12 .\object_temperature.py .\calibrated_frame.json
```

For the example ROI, the measured synthetic values are `34,36,33,35` °C: **min 33, mean 34.5, max 36°C**. The 90% valid-pixel requirement is an **example quality gate**, not the camera's temperature accuracy guarantee. If the DLL supports **only spot/ROI measurements and not a temperature array**, use that documented SDK result directly; do not invent a matrix.

### 12.9 Physical measurement: avoid false temperatures

Thermal imaging usually estimates an object's **surface** temperature, not its internal/core temperature.

1. **Optics and target size:** Ensure the object occupies adequate true thermal pixels for the lens' spatial/spot measurement specification. Do not trust boundary and mixed pixels.
2. **Emissivity:** Match the actual surface material. Reflective/shiny metal often returns strong reflections; `0.95` is not a universal setting.
3. **Reflected apparent temperature, ambient, distance, humidity/transmission:** Configure as supported by the OEM measurement model. Ambient room temperature may not equal reflected temperature.
4. **Focus, warm-up and sensor correction:** Follow the manual's focus, NUC/FFC and calibrated measurement range.
5. **Obstructions:** Ordinary window glass and many plastics can block thermal IR; the sensor may be measuring the window itself.
6. **ROI purity:** Sample interior object pixels, mask background, distinguish center/mean/max/p95 and sensor saturation.
7. **Independent validation:** Compare known safe reference temperatures/contact sensor or a calibrated blackbody under controlled conditions, using the model's claimed uncertainty. Never alter energized surfaces just to improve emissivity.
8. **Timestamp:** ROI selection and temperature frame must refer to the **same physical target at compatible times**. Reject stale data and never confuse visible-light with thermal channels.
9. **Failure state:** Display **"temperature unavailable"**, not a guessed number, if calibration, channel identity, measured data, ROI mapping or synchronization is missing.

### 12.10 Validation and acceptance tests

| Test | Pass condition |
|---|---|
| **A. Hardware** | Temperature capable sensor produces numerical readings against a known warm/cool reference; near-IR CCTV fails |
| **B. NetClient** | Documented OEM function/callback returns calibrated unit, timestamp and error status—not only video |
| **C. NetViewX alignment** | The same chosen object maps to correct thermal points at center and near edges |
| **D. Reproducibility** | Multiple measurements stay within **the actual camera's accuracy specification**, under controlled settings |
| **E. Failure handling** | Missing radiometry or invalid/cropped ROI shows "unavailable" and prevents false alarms |

### 12.11 Information that unlocks real end-to-end OCX + DLL code

Ask the manufacturer to supply **`NetClient.h`, official temperature SDK samples, `NetViewX.OCX` type library/CLSID, and exact model/firmware compatibility**, including:

- Exported point/ROI/temperature-frame API **real prototypes**, structs, callbacks, status codes.
- Temperature unit and calibrated conversion method, raw data format, validity/saturation indicators.
- Thermal sensor frame dimensions, stream/channel ID, preview-to-thermal coordinate transform, time synchronization.
- Measurement parameters, accuracy/uncertainty, x86/x64 dependencies, license and supported OS.

**If the documented NetClient.dll has no temperature API, the correct outcome is that this specific pair cannot report real object temperatures.** A different radiometric firmware/device SDK is required; a fabricated SDK call is not a solution.

## 13. How to control infrared night mode

If the hardware is a **near-IR surveillance camera** rather than a thermal camera:

1. Verify there is a built-in IR illuminator and IR-cut filter.
2. Locate documented **Day/Night / Auto / IR LED** settings in the vendor application.
3. Test Auto mode under controlled lighting.
4. Check whether the camera has an authorized HTTP, ONVIF imaging, or vendor SDK control command for the feature.
5. Use only model-documented APIs; viewer DLL names do not determine IR-control commands.

Switching night mode does **not** turn an ordinary CCTV camera into a thermal thermometer.

## 14. Troubleshooting matrix

| Problem | Likely cause | Action |
|---|---|---|
| Cannot find trustworthy OCX/DLL download | Proprietary/old version-specific component | Obtain official package from exact OEM or matching device UI |
| OCX registration returns “module could not be loaded” | Missing dependency, wrong architecture, corrupted file | Inspect `dumpbin /dependents` and x86/x64 matching |
| `DllRegisterServer` not found | DLL is not a self-registering COM server | Do not register arbitrary `NetClient.dll`; use the vendor installer |
| `0x80040154 Class not registered` | Missing/wrong COM bitness or CLSID | Match 32-bit host to 32-bit OCX, confirm official CLSID |
| Browser shows blank preview | Browser cannot use ActiveX | Use modern video interface or approved Edge IE-mode route |
| IE mode refuses unsigned plugin | Publisher verification failure | Request a properly signed vendor installer; do not disable global protections |
| Preview shows video but no temperature | Non-radiometric stream/API | Ask for calibrated temperature SDK and actual thermal channel |
| Thermal image is colorful but wrong temperature | Colors are palette-mapped/AGC display | Do not derive °C from RGB pixels |
| Camera connection fails | Wrong actual device IP, subnet, creds, protocol or port | Check model manual and authorized network configuration |
| Python native integration crashes | Wrong `ctypes` argument types/calling convention/architecture | Get real C header; do not guess exports |
| Temperature incorrect | Emissivity, reflected temperature, distance, calibration mismatch | Follow vendor measurement/calibration instructions |

## 15. Information needed for an exact working integration

Please collect:

1. Photo of the device's **model label** (you can cover the serial number).
2. Firmware version, whether it has **Ethernet, USB, or serial** connection.
3. Your goal: **IR night video**, **thermal picture**, **real temperatures**, **raw thermal array**, or **IR LED control**.
4. Where you saw `NetViewX.OCX` and `NetClient.dll` (folder, installer, device page, manual screenshot).
5. Whether you already have either binary, a `.cab` package, SDK headers, or sample code.
6. Windows 11 64-bit confirmation and your intended language (Python/C#/C++/HTML).

### Chinese text to send the supplier

> 您好，我需要集成贵司设备的红外/热成像数据。请确认设备型号及固件对应的 Windows 客户端、NetViewX.OCX、NetClient.dll、NetViewX.cab 是否适用，并提供官方签名安装包、32/64 位版本、SDK 头文件、COM CLSID/ProgID、接口文档和示例。请说明是否支持 RTSP/ONVIF 热成像视频、逐像素温度矩阵/原始红外数据、测温单位及标定参数，以及离线使用方式。谢谢！

## 16. Source references

Sources reviewed October 26, 2026:

**Conclusion:** Identify the exact device first. Prefer a **documented modern video/thermal SDK interface**. Install `NetViewX.OCX` and `NetClient.dll` only when the manufacturer confirms that exact versions belong to your device and supplies a trusted distribution.
