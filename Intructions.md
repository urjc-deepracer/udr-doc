# 🍓 Installing Raspbian with Raspberry Pi Imager

Step-by-step guide to install **Raspberry Pi OS (Raspbian)** on a microSD card using **Raspberry Pi Imager**.

---

## 🖥️ PART 1 — On the computer (PC)

### Step 1 — Insert the microSD into the PC

Before opening Raspberry Pi Imager, connect your microSD card to your computer:

- If your PC has an SD card slot, use a **microSD → SD adapter**.
- Otherwise, use a **USB card reader**.

---

### Step 2 — Download and install Raspberry Pi Imager

Download the official tool from the Raspberry Pi website:

👉 [https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)

Install it as you would any other program on your operating system.

---

### Step 3 — Open Raspberry Pi Imager

Once installed, open **Raspberry Pi Imager**. You will see the main screen with three buttons:

- **Raspberry Pi Device** → select your Raspberry Pi model
- **Operating System** → select the operating system
- **Storage** → select your SD card

![Raspberry Pi Imager main screen](https://cdn.lo4d.com/t/screenshot/800/raspberry-pi-imager.png)

---

### Step 4 — Select the operating system

Click **"Choose OS"** and select:

> For this project we used **Raspberry Pi OS (64-bit)**.  
> You can find it under **Raspberry Pi OS (other)** in the menu, or download it directly here:  
> 👉 [https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit)

![Selecting the OS in Raspberry Pi Imager](https://cdn.lo4d.com/t/screenshot/800/raspberry-pi-imager-2.png)

---

### Step 5 — Select the SD card

Click **"Choose Storage"** and select your microSD card from the list of detected devices.

> ⚠️ **Make sure you select the correct card** and not another hard drive or USB drive.

---

### Step 6 — Write the image to the SD card

Click **"Write"** and confirm that you want to erase the card.

![Writing process in Raspberry Pi Imager](https://cdn.lo4d.com/t/screenshot/800/raspberry-pi-imager-4.png)

The process will take between **5 and 15 minutes** depending on your card and computer speed.

When finished, Raspberry Pi Imager will automatically verify the write and display:

> ✅ **"Write Successful"**

---

### Step 7 — Remove the microSD from the PC

Once the write is complete:

1. Click **"Continue"** to close the window.
2. Safely eject the microSD card from the reader.

---

## 🍓 PART 2 — On the Raspberry Pi

### Step 8 — Insert the microSD into the Raspberry Pi

Locate the microSD slot on your Raspberry Pi. It is on the **underside** of the board.

Insert the microSD card with the **gold contacts facing down** and gently push until it clicks into place.

> ⚠️ Most models have **no locking mechanism** — the card simply holds by friction.

---

### Step 9 — Power on

Connect the **USB-C power cable** (or micro-USB on older models) to turn on the Raspberry Pi.

> 💡 The Raspberry Pi has **no power switch** — it turns on as soon as power is connected.

---

### Step 10 — First boot

On startup, the Raspberry Pi will boot from the microSD. You will see the **Raspberry Pi logo** and the system will begin loading.

- Create a username and password
- Select region and language
- Connect to a Wi-Fi network
- Update the system

---

## 🐳 PART 3 — Installing Docker on Raspberry Pi OS

### Step 11 — Update the system

Before installing Docker, make sure your system is fully up to date. Open a terminal and run:

```bash
sudo apt-get update && sudo apt-get upgrade -y
```

---

### Step 12 — Remove any old Docker versions

Remove unofficial Docker packages that may have been pre-installed:

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt-get remove $pkg
done
```

It is fine if `apt` reports that some packages are not installed.

---

### Step 13 — Add Docker's official repository

Install the required dependencies and add Docker's GPG key and repository:

```bash
# Install dependencies
sudo apt-get install -y ca-certificates curl

# Create the keyrings directory
sudo install -m 0755 -d /etc/apt/keyrings

# Add Docker's official GPG key
sudo curl -fsSL https://download.docker.com/linux/debian/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update the package list
sudo apt-get update
```

---

### Step 14 — Install Docker Engine

Install Docker Engine and the CLI:

```bash
sudo apt-get install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io
```

---

### Step 15 — Add your user to the Docker group

By default, Docker requires `sudo` to run. Add your user to the `docker` group to avoid this:

```bash
sudo usermod -aG docker $USER
```

Then reboot for the change to take effect:

```bash
sudo reboot
```

---

### Step 16 — Verify the installation

```bash
docker --version
docker run hello-world
```

If everything is set up correctly, the `hello-world` container will print a confirmation message and exit.

---

## 🤖 PART 4 — Running the ROS 2 Humble container

### Step 17 — Allow Docker to display graphical applications

Before running the container, allow Docker to access the X display server so that graphical tools like **RViz2** work correctly:

```bash
xhost +local:docker
```

> 💡 You need to run this command every time you reboot, before launching the container.

---

### Step 18 — Run the ROS 2 Humble container

```bash
sudo docker run -it \
  --name urjcracer \
  --net=host \
  --privileged \
  -v /dev:/dev \
  -v /run/udev:/run/udev:ro \
  -v /usr/lib/libcamera:/usr/lib/libcamera \
  -v ~/ros2_ws:/root/ros2_ws \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  ros:humble
```

What each flag does:

| Flag | Description |
|------|-------------|
| `--net=host` | Uses the host network directly, required for ROS 2 communication |
| `--privileged` | Grants the container access to all host devices |
| `-v /dev:/dev` | Mounts host devices (cameras, sensors, etc.) inside the container |
| `-v /run/udev:/run/udev:ro` | Gives the container read-only access to device event information |
| `-v /usr/lib/libcamera:/usr/lib/libcamera` | Shares the host's libcamera library with the container |
| `-v ~/ros2_ws:/root/ros2_ws` | Mounts your local ROS 2 workspace into the container |
| `-e DISPLAY=$DISPLAY` | Passes the display environment variable for graphical apps |
| `-v /tmp/.X11-unix:/tmp/.X11-unix` | Shares the X11 socket so graphical apps render on the host screen |

---

## 🔌 PART 5 — Installing pigpiod on the Raspberry Pi (host)

pigpiod must run **on the host** (the Raspberry Pi itself, outside Docker). The container communicates with it over the network on port `8888`. This is necessary because pigpiod needs direct access to the hardware GPIO pins, which only the host OS can provide.

### Step 11 — Install build dependencies

```bash
sudo apt-get update
sudo apt-get install -y unzip make gcc
```

---

### Step 12 — Download and build pigpio from source

```bash
wget https://github.com/joan2937/pigpio/archive/master.zip
unzip master.zip
cd pigpio-master
make
sudo make install
```

---

### Step 13 — Create the systemd service file

After building from source, the service file is not created automatically. Create it manually:

```bash
sudo nano /etc/systemd/system/pigpiod.service
```

Paste the following content:

```ini
[Unit]
Description=Daemon required to control GPIO pins via pigpio

[Service]
Type=forking
ExecStart=/usr/local/bin/pigpiod
Restart=always
ExecStop=/bin/systemctl kill pigpiod

[Install]
WantedBy=multi-user.target
```

Save and exit (`Ctrl+X`, then `Y`, then `Enter`).

---

### Step 14 — Enable pigpiod to start automatically on boot

```bash
sudo systemctl daemon-reload
sudo systemctl enable pigpiod
sudo systemctl start pigpiod
```

---

### Step 15 — Check that pigpiod is running

```bash
sudo systemctl status pigpiod
```

You should see output similar to:

```
● pigpiod.service - Daemon required to control GPIO pins via pigpio
     Loaded: loaded (/etc/systemd/system/pigpiod.service; enabled)
     Active: active (running) since ...
```

The key indicators are **`enabled`** (starts on boot) and **`active (running)`** (currently running).

---

### Step 16 — Using pigpiod from inside the Docker container

Since pigpiod runs on the host and the container uses `--net=host`, the container can reach it directly on `localhost:8888`. No extra configuration is needed inside the container.

To verify the connection from inside the container:

```bash
# Inside the Docker container
python3 -c "import pigpio; pi = pigpio.pi(); print('Connected:', pi.connected)"
```

If the output is `Connected: True`, pigpiod is reachable from the container.

> ⚠️ pigpiod must be running on the host **before** launching the Docker container.

---

## 🤖 PART 6 — Setting up the ROS 2 workspace inside the container

### Step 17 — Configure the bashrc

So that ROS 2 and the workspace are sourced automatically in every new terminal session inside the container:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

### Step 18 — Clone the project repositories into the workspace

```bash
cd ~/ros2_ws/src

git clone https://github.com/urjc-deepracer/udr-ros-esp.git
git clone https://github.com/urjc-deepracer/udr-ros-motors.git
git clone https://github.com/urjc-deepracer/udr-ros-rc.git
git clone https://github.com/urjc-deepracer/udr-ros-viewer.git
git clone https://github.com/urjc-deepracer/ros2_camera.git
git clone https://github.com/urjc-deepracer/udr-ros-coral.git
git clone https://github.com/urjc-deepracer/ros2_mpu6050.git
```

---

### Step 19 — Build the workspace

```bash
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build
source install/setup.bash
```

---

### Step 20 — Follow each package's specific installation steps

Some packages require additional dependencies or configuration steps that `rosdep` does not cover (drivers, Python packages, hardware setup, etc.). After building the workspace, read the `README.md` of each cloned package and follow its specific instructions:

| Package | README |
|---------|--------|
| `udr-ros-esp` | [README](https://github.com/urjc-deepracer/udr-ros-esp#readme) |
| `udr-ros-motors` | [README](https://github.com/urjc-deepracer/udr-ros-motors#readme) |
| `udr-ros-rc` | [README](https://github.com/urjc-deepracer/udr-ros-rc#readme) |
| `udr-ros-viewer` | [README](https://github.com/urjc-deepracer/udr-ros-viewer#readme) |
| `ros2_camera` | [README](https://github.com/urjc-deepracer/ros2_camera#readme) |
| `udr-ros-coral` | [README](https://github.com/urjc-deepracer/udr-ros-coral#readme) |
| `ros2_mpu6050` | [README](https://github.com/urjc-deepracer/ros2_mpu6050#readme) |

---

