## 🚀 Overview

Implementation of YOLO-CPP by Mecatron.

- **⚡ YOLOs-CPP**: https://github.com/Geekgineer/YOLOs-CPP
- **⚡ ros2_yolos_cpp**: https://github.com/Geekgineer/ros2_yolos_cpp.git

**ros2_yolos_cpp** brings the blazing speed and unified API of [YOLOs-CPP](https://github.com/Geekgineer/YOLOs-CPP) to the robot operating system. It provides composable, lifecycle-managed nodes for the entire YOLO family (v5, v8, v11, v26, etc.).

### Key Features
- **⚡ Zero-Copy Transport**: Optimized for high-throughput image pipelines using `rclcpp::Subscription`.
- **🔄 Lifecycle Management**: Full support for `configure`, `activate`, `deactivate`, `shutdown` transitions.
- **🛠️ Composable Nodes**: Run multiple models in a single container for efficient resource usage.
- **📦 All Tasks Supported**: Detection, Segmentation, Pose, OBB, and Classification.
- **🏗️ Production Ready**: CI/CD tested, strictly typed parameters, and standardized messages (`vision_msgs`).

---

## 📥 Installation

### Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| C++ Compiler | C++17 | GCC 9+, Clang 10+, MSVC 2019+ |
| CMake | ≥ 3.16 | |
| OpenCV (CPP) | ≥ 4.5 | Core, ImgProc, HighGUI |
| ONNX Runtime | ≥ 1.16 | Auto-downloaded by build script |

### Requirement Check

```bash
# Check GCC version
g++ --version

# Check cmake
cmake --version

# OpenCV (CPP)
pkg-config --modversion opencv4 2>/dev/null || pkg-config --modversion opencv
```
### CUDA & cuDNN Requirement for ONNX Runtime
- CUDA 12.x is the default version for ONNX Runtime GPU packages in PyPI
- Other versions of CUDA please refer to [ONNX_CUDA](https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html)

| ONNX Runtime Version | CUDA  | cuDNN |
|----------------------|-------|-------|
| 1.20.x               | 12.x  | 9.x   |

#### CUDA 12.8 (WSL) & cuDNN 9.19.0
[CUDA](https://developer.nvidia.com/cuda-12-8-0-download-archive)
```bash
# Clone
wget https://developer.download.nvidia.com/compute/cuda/12.8.0/local_installers/cuda_12.8.0_570.86.10_linux.run
sudo sh cuda_12.8.0_570.86.10_linux.run


# Create symlink (required for most tools to find CUDA)
sudo ln -s /usr/local/cuda-12.4 /usr/local/cuda

# Add to ~/.bashrc
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc

# Check installation
nvcc --version 
```
[cuDNN](https://developer.nvidia.com/cudnn-downloads)

```bash
# Install
wget https://developer.download.nvidia.com/compute/cudnn/9.19.0/local_installers/cudnn-local-repo-ubuntu2204-9.19.0_1.0-1_amd64.deb
sudo dpkg -i cudnn-local-repo-ubuntu2204-9.19.0_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-ubuntu2204-9.19.0/cudnn-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cudnn9-cuda-12


### Installation of ONNX Runtime

```bash
# Clone
git clone https://github.com/Geekgineer/YOLOs-CPP.git
cd YOLOs-CPP

# Build (auto-downloads ONNX Runtime)
./build.sh 1.20.1 0   # CPU build 
./build.sh 1.20.1 1   # GPU build (requires CUDA - pick this)
```

### Build the ros2_yolos_cpp package
```bash
# Create workspace
cd ros2_ws/src

# Clone package
git clone https://github.com/Geekgineer/ros2_yolos_cpp.git

# Install dependencies
cd ros2_ws
rosdep update && rosdep install --from-paths src --ignore-src -y

# Build (Release mode recommended for performance)
colcon build --packages-select ros2_yolos_cpp --cmake-args -DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

### YOLO-CPP Package
‼️The YOLO-CPP package is used to build the onnxruntime, you may remove it when installation of the build is done (temporary)

---

## 🛠️ Usage

This package provides a launch file for each task. You **must** provide paths to your ONNX model and (optionally) labels file.

### 0. USB CAM Testing (WSL)
```bash
ros2 run usb_cam usb_cam_node_exe --ros-args \
  -p video_device:=/dev/video0 \
  -p image_width:=640 \
  -p image_height:=480 \
  -p framerate:=30.0 \
  -p pixel_format:=mjpeg2rgb \
  -p io_method:=mmap \
  -r __ns:=/camera
```

### 1. Object Detection
Publishes `vision_msgs/Detection2DArray` with bounding boxes and class IDs.

```bash
ros2 launch ros2_yolos_cpp detector.launch.py \
    model_path:=src/ros2_yolos_cpp/models/yolo11n.onnx \
    labels_path:=src/ros2_yolos_cpp/models/coco.names \
    use_gpu:=true \
    image_topic:=/camera/image_raw
```


### 2. Instance Segmentation
Publishes `vision_msgs/Detection2DArray` and a synchronized mask image.

```bash
ros2 launch ros2_yolos_cpp segmentor.launch.py \
    model_path:=/path/to/yolo11n-seg.onnx \
    labels_path:=/path/to/coco.names \
    image_topic:=/camera/image_raw
```

### 3. Pose Estimation
Publishes `vision_msgs/Detection2DArray` with keypoints.

```bash
ros2 launch ros2_yolos_cpp pose.launch.py \
    model_path:=/path/to/yolo11n-pose.onnx \
    image_topic:=/camera/image_raw
```

### 4. Oriented Bounding Boxes (OBB)
Publishes custom `ros2_yolos_cpp/OBBDetection2DArray` with rotated bounding boxes.

```bash
ros2 launch ros2_yolos_cpp obb.launch.py \
    model_path:=/path/to/yolo11n-obb.onnx \
    labels_path:=/path/to/dota.names \
    image_topic:=/camera/image_raw
```

### 5. Image Classification
Publishes `vision_msgs/Classification`.

```bash
ros2 launch ros2_yolos_cpp classifier.launch.py \
    model_path:=/path/to/yolo11n-cls.onnx \
    labels_path:=/path/to/imagenet.names \
    image_topic:=/camera/image_raw
```
### Starting ros2_yolo_cpp
- ros2_yolos_cpp is manged by life cycle
- After launching the launch file:

#### To Activate:
```bash
# Step 1: Configure (loads model, allocates GPU memory)
ros2 lifecycle set /yolos_detector configure

# Step 2: Activate (STARTS INFERENCE → publishes detections)
ros2 lifecycle set /yolos_detector activate
```
#### To Deactivate:
```bash
# 1. Deactivate → stops inference (active → inactive)
ros2 lifecycle set /yolos_detector deactivate

# 2. Cleanup → releases GPU/memory (inactive → unconfigured)
ros2 lifecycle set /yolos_detector cleanup

# 3. Shutdown → fully terminates node (unconfigured → finalized)
ros2 lifecycle set /yolos_detector shutdown
```

---

## ⚙️ Configuration

Nodes can be configured via launch arguments or a YAML parameter file. See `config/default_params.yaml` for a template.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model_path` | string | **Required** | Absolute path to `.onnx` model file. |
| `labels_path` | string | "" | Path to text file with class names (one per line). |
| `use_gpu` | bool | `false` | Enable CUDA acceleration (requires GPU build). |
| `conf_threshold` | double | `0.4` | Minimum confidence score to output detection. |
| `nms_threshold` | double | `0.45` | IoU threshold for Non-Maximum Suppression. |
| `image_topic` | string | `/camera/image_raw` | Topic to subscribe for input images. |
| `publish_debug_image` | bool | `true` | Publish standard `sensor_msgs/Image` with visualizations. |

### Topics

| Node Type | Subscriptions | Publications |
|-----------|---------------|--------------|
| **Detector** | `~/image_raw` | `~/detections` (Detection2DArray)<br>`~/debug_image` (Image) |
| **Segmentor** | `~/image_raw` | `~/detections` (Detection2DArray)<br>`~/masks` (Image)<br>`~/debug_image` |
| **Pose** | `~/image_raw` | `~/detections` (Detection2DArray)<br>`~/debug_image` |
| **OBB** | `~/image_raw` | `~/detections` (OBBDetection2DArray)<br>`~/debug_image` |
| **Classifier** | `~/image_raw` | `~/classification` (Classification)<br>`~/debug_image` |

---

## 🐳 Docker

Run the stack without installing dependencies locally.

```bash
# Build Docker image
docker build -t ros2_yolos_cpp .

# Run with GPU support
docker run --gpus all -it --rm \
    -v /path/to/models:/models \
    ros2_yolos_cpp \
    ros2 launch ros2_yolos_cpp detector.launch.py model_path:=/models/yolov8n.onnx
```
