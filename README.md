# Autoware_Projects

# 🚗 Autoware + AWSIM Setup Guide

### Full Installation, Docker Runtime, Build, and Launch Instructions

This README provides a complete workflow for setting up **Autoware** with **AWSIM** using Docker, including workspace preparation, dependency import, GPU-accelerated Docker, and optional Jetson Orin optimization steps.

---

## 📦 1. Clone the Autoware Repository

```bash
git clone https://github.com/autowarefoundation/autoware.git
```

---

## 📁 2. Prepare the Autoware Workspace

```bash
cd autoware
mkdir src
vcs import src < autoware.repos
```

---

## 🛠️ 3. (Optional) Pull Nightly Development Code

Use the nightly repos if you want the latest development features.

```bash
vcs import src < autoware-nightly.repos
```

---

## 🐳 4. Pull the Autoware Docker Image (CUDA Enabled)

```bash
docker pull ghcr.io/autowarefoundation/autoware:universe-devel-cuda
```

---

## 🚀 5. Run the Autoware Docker Container

Replace `/your/autoware/path` with the absolute path to your workspace.

```bash
docker run -it \
    --name autoware_latest \
    --gpus all \
    --net=host \
    --privileged \
    -v /your/autoware/path/out/of/docker:/wc_ws \
    -w /wc_ws \
    -e DISPLAY \
    -e QT_X11_NO_MITSHM=1 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    ghcr.io/autowarefoundation/autoware:universe-devel-cuda bash
```

### Explanation of Key Options

| Flag                | Purpose                              |
| ------------------- | ------------------------------------ |
| `--gpus all`        | Enables NVIDIA GPU inside Docker     |
| `--net=host`        | Required for ROS 2 networking        |
| `--privileged`      | Needed for accessing devices/sensors |
| `-v /tmp/.X11-unix` | Enables GUI apps (RViz2, etc.)       |
| `-e DISPLAY`        | X11 display forwarding               |

---

## 🧱 6. Build Autoware

Inside the Docker container:

```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --symlink-install
```

---

## 🎮 7. Launch AWSIM (Simulator)

In a separate terminal:

```bash
cd /your/awsim/path
./AWSIM.x86_64
```

If needed:

```bash
chmod +x AWSIM.x86_64
```

---

## 🚦 8. Launch Autoware

Inside the Docker container:

```bash
source install/setup.bash
ros2 launch autoware_launch autoware.launch.xml
```

---

# ⚡ Jetson Orin Nano Notes (Highly Recommended)

Autoware + AWSIM works great on Jetson Orin, but requires a few optimizations.

---

## 🔧 A. Install NVIDIA Container Runtime

```bash
sudo apt install nvidia-container-runtime
```

Test that GPU passthrough works:

```bash
sudo docker run --rm --gpus all nvidia/cuda:12.2.0-base nvidia-smi
```

---

## 🧠 B. Increase Shared Memory (Prevents RViz2 and Autoware Crashes)

```bash
--shm-size=2g
```

Add it to your docker command:

```bash
docker run -it \
    --shm-size=2g \
    --name autoware_latest \
    --gpus all \
    --net=host \
    --privileged \
    ...
```

---

## 🌀 C. (Optional) Enable Real-Time Kernel

```bash
sudo apt install linux-image-rt
```

Improves performance for Autoware control stack.

---

# 📂 Recommended Folder Structure

```
autoware_ws/
 ├── autoware/
 │    ├── src/
 │    ├── install/
 │    ├── build/
 │    └── log/
 └── awsim/
      ├── AWSIM.x86_64
      └── …
```

---

# 🧪 Troubleshooting

### **No GUI / RViz does not launch**

* Ensure X11 forwarding is enabled
* Run on host: `xhost +local:docker`

### **Docker cannot access GPU**

* Install `nvidia-container-runtime`
* Verify using: `docker run --gpus all … nvidia-smi`

### **Build errors**

* Check dependencies: `apt-get update` before building
* Re-source: `source /opt/ros/humble/setup.bash`

---

# 🎉 You're Ready to Run Autoware + AWSIM!

If you want, I can also generate:

✅ `setup.sh` auto-installer
✅ Docker Compose version
✅ Jetson Orin optimized launch script
✅ A visual architecture diagram
✅ A LinkedIn post announcing your project

Just tell me!
