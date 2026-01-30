# LiteRT Image Segmentation with QNN

Real-time image segmentation Android app powered by LiteRT (Google's new TensorFlow Lite runtime) with Qualcomm Neural Network (QNN) acceleration.

## Features

- **Automatic NPU→GPU→CPU fallback** ⚡ - Intelligently selects best accelerator
- **Real-time segmentation** using camera or gallery images
- **21-class segmentation**: person, car, cat, dog, bicycle, and more
- **Visual backend indicators** - See which accelerator is active (NPU ⚡, GPU 🔥, CPU ⚙️)
- **QNN acceleration** for optimized inference on Qualcomm devices
- **JIT compilation**: On-device model compilation for optimal performance

## Performance

**Verified on Samsung S25 Ultra (Snapdragon 8 Elite):**

| Backend | Status | Inference Time | Auto-Fallback |
|---------|--------|----------------|---------------|
| **NPU** | ✅ Working | **~85ms** ⚡ | **1st Priority** |
| GPU     | ⚠️ Variable | ~40-50ms | 2nd Priority |
| CPU     | ✅ Working | ~130ms | Final Fallback |

> **Note:** The app automatically tries NPU first, then GPU, then CPU. No manual selection needed!

## Supported Devices

Qualcomm Snapdragon devices with NPU support:
- SM8450 (Snapdragon 8 Gen 1)
- SM8550 (Snapdragon 8 Gen 2)
- SM8650 (Snapdragon 8 Gen 3)
- SM8750 (Snapdragon 8 Elite)
- SM8850 (Future)

## Quick Start

### Prerequisites

- Android Studio
- Android device with Qualcomm SoC and NPU support
- Android API level 31+

### Build and Run

1. Clone the repository:
```bash
git clone https://github.com/carrycooldude/image-segmentation-qnn-litert.git
cd image-segmentation-qnn-litert
```

2. Download QNN runtime libraries (automatically handled by build script):
```bash
bash fetch_qualcomm_library.sh
```

3. Build and install:
```bash
./gradlew installDebug
```

## How It Works

### Automatic Backend Selection

The app implements intelligent fallback logic at startup:

1. **Attempts NPU initialization** - Best performance on Snapdragon devices
2. **If NPU fails → tries GPU** - Good performance, broader compatibility  
3. **If GPU fails → uses CPU** - Guaranteed to work on all devices

```kotlin
// Automatic fallback on app launch
initSegmenterWithFallback()  // NPU → GPU → CPU
```

### Visual Indicators

The bottom sheet shows which backend is currently active:
- **NPU ⚡** - Fastest, using Qualcomm AI Engine
- **GPU 🔥** - Fast, using GPU acceleration
- **CPU ⚙️** - Compatible, CPU-only inference

You can also manually override the backend selection from the dropdown menu.

## Technical Details

- **Model**: 256x256x3 RGB input → 6-channel output (21 classes)
- **Runtime**: LiteRT 2.0.0-alpha
- **Acceleration**: Qualcomm AI Engine Direct via QNN
- **Mode**: JIT (Just-In-Time) compilation
- **Architecture**: Dynamic feature modules for on-demand runtime loading

## QNN Runtime Versions

The app includes Qualcomm runtime libraries for multiple QNN versions:
- v69 (SM8450)
- v73 (SM8550)
- v75 (SM8650)
- v79 (SM8750)
- v81 (SM8850)

Device SoC is automatically detected and the appropriate runtime is loaded.

## Project Structure

```
android_jit/
├── app/                           # Main application
│   ├── src/main/assets/          # Model file
│   └── src/main/java/            # Kotlin source code
├── litert_npu_runtime_libraries/ # QNN runtime modules
│   ├── qualcomm_runtime_v69/
│   ├── qualcomm_runtime_v73/
│   ├── qualcomm_runtime_v75/
│   ├── qualcomm_runtime_v79/
│   └── qualcomm_runtime_v81/
└── fetch_qualcomm_library.sh     # Download Qualcomm SDK libs
```


## References

- [LiteRT Documentation](https://ai.google.dev/edge/litert)
- [LiteRT NPU Guide](https://ai.google.dev/edge/litert/next/npu)
- [Qualcomm AI Engine Direct](https://ai.google.dev/edge/litert/next/qualcomm)
