# Environment Requirement

## Hardware
- Realtek Ameba Board (i.e. AmebaSmart RTL8730E VA7 or VA8 board)
- USB cable – Micro USB
- USB cable – Type-C USB
- Computer running Ubuntu 22.04 LTS or Virtual Machine with Ubuntu 22.04 LTS

### Virtual Machine
If using VMware Workstation, a USB Bluetooth adapter is preferred. If needed, install the specific USB Bluetooth adapter driver on the guest operating system.
Alternatively, you can attempt to enable Bluetooth adapter from your host computer. 
VM -> Settings -> USB Controller -> Uncheck box for Share Bluetooth devices with the virtual machine -> OK. 
Then, connect Bluetooth adapter to guest operating system. 
VM -> Removable Devices -> Intel® Wireless Bluetooth® -> Connect (Disconnect from Host).
In terminal, run `hciconfig` to ensure that Bluetooth adapter shows up, like hci0.

VMware exposes the host operating system (ethernet or WIFI) network as ethernet connection type on the guest operating system. By default, VMware will set the network adapter to use Network Address Translation (NAT). During Matter commissioning, mDNS protocol is used and the mDNS packets will fail or timeout if NAT is used. To fix this, we need to ensure that the guest operating system network is under the same subnet. This can be done by switching the network connection type to bridge.
VM- > Settings -> Network Adapter -> Choose Bridged: Connected directly to the physical network.
Afterwards, within the guest operating system, restart the network to refresh the IP address.

In terms of WIFI networking, VMware does not expose WIFI adapter from host operating system. In order to use WIFI on guest operating system, you may instead use a USB WIFI adapter.

| System              | VMware                             |
| ------------------- | ---------------------------------- |
| OS Name             | Ubuntu 22.04.4 LTS                 |
| OS Type             | 64-bit                             |
| Linux system kernel | 6.5.0-41-generic                   |
| Gnome Version       | 42.9                               |
| Windowing System    | Wayland                            |
| Virtualization      | VMware                             |
| Memory              | 8192MB (8GB)                       |
| Processors          | 12th Gen Intel® Core™ i7-1255U × 8 |
| Hard Disk (SCSI)    | 250GB                              |

The test Virtual Machine system used for Yocto Matter development is outlined in the table above.

According to [Official Yocto System Requirements](https://docs.yoctoproject.org/4.0.19/ref-manual/system-requirements.html), 8GB RAM is the minimum needed. If possible, more RAM (32GB) is recommended for the host machine or Virtual Machine. Otherwise, you can compensate by [increasing swap memory](#increasing-swap-memory).

For other Virtual Machines, you can build the Matter environment and Projects on VM but BLE adapter may be unavailable so it’s impossible to perform [ble-wifi commissioning](#ble-commissioning). Alternatively, you could test with [wifi commissioning](#wifi-commissioning).

### Increasing swap memory
If your system does not have enough RAM, you may partially compensate by increasing swap memory (virtual page) to 32GB. Keep in mind that using swap memory results in slower build times as compared to using system RAM.

For Ubuntu 22.04, you can expand swap memory as follows:
1. Check current swap
	```bash
	grep Swap /proc/meminfo
	```
2. Turn off swap
	```bash
	sudo swapoff -a
	```
3. Create empty swapfile of 32GB
	```bash
	sudo dd if=/dev/zero of=/swapfile bs=1M count=32768
	```
4. Set swapfile permission
	```bash
	sudo chmod 0600 /swapfile
	```
5. Make swap
	```bash
	sudo mkswap /swapfile
	```
6. Turn on swap
	```bash
	sudo swapon /swapfile
	```
7. Check current swap
	```bash
	grep Swap /proc/meminfo
	```
8. Make swap size persistent on restarts
	```bash
	grep Swap /proc/meminfo
	```
	- Ensure the following line is present, otherwise, add it
	```bash
	/swapfile none swap sw 0 0
	```

# Yocto Matter setup
> [!NOTE] 
> Preferably connectedhomeip folder should be located on the same level as AmebaSmart Yocto SDK folder

```
/home
└── your_username
   ├── connectedhomeip
   └── rtl8730e-linux-sdk-3.1-release
```

## Setup connectedhomeip
1. Install requirements
	```bash
	sudo apt install git gcc g++ pkg-config libssl-dev \
	libdbus-1-dev libglib2.0-dev libavahi-client-dev ninja-build \
	python3-venv python3-dev python3-pip unzip \
	libgirepository1.0-dev libcairo2-dev libreadline-dev
	```
2. Git clone `connectedhomeip`
	```bash
	git clone -b v1.3-branch --recurse-submodules https://github.com/project-chip/connectedhomeip.git
	```
3. Navigate inside `connectedhomeip`
	```bash
	cd connectedhomeip
	```
4. Update `connectedhomeip` submodules
	```bash
	git submodule sync --recursive
	git submodule update --init --recursive --force
	```
5. Perform bootstrap
	```bash
	source scripts/bootstrap.sh
	```
6. Upgrade pip
	```bash
	pip install --upgrade pip
	```
7. Build tools (e.g. `chip-tool`, `chip-cert`) for host
	```bash
	gn gen out/host
	ninja -C out/host
	```
	- The Matter tools executables will be built in the directory `connectedhomeip/out/host/`

## Setup Yocto Matter
1. Open a new terminal shell
2. Install utility packages
	```bash
	sudo apt install chrpath diffstat liblz4-tool zstd gawk python3-dev python-is-python3 python3-pycryptodome repo
	```
3. Change shell to use bash
	```bash
	sudo dpkg-reconfigure dash
	```
	- Select `NO`
4. Create `/opt/rtk-toolchain/` directory
	```bash
	sudo mkdir -p /opt/rtk-toolchain
	sudo chown -R ${USER}:${USER} /opt/rtk-toolchain
	```
5. Get Realtek Yocto and Matter source
	```bash
	mkdir rtl8730e-linux-sdk-3.1-release
	cd rtl8730e-linux-sdk-3.1-release
	repo init -u https://github.com/Ameba-AIoT/ameba-linux-manifest -b ameba-3.1 -m default-matter.xml
	repo sync
	```
6. Run Yocto setup script
  	- This will add `ROOTDIR` as environment variable referring to Yocto directory
  	```bash
  	source envsetup.sh
  	```
  	- Choose AmebaSmart board: `rtl8730elh-va7` or `rtl8730elh-va8`
  	- Choose distro: `ameba-generic` or `ameba-full`
7. Add matter layer
  	- This will add matter layer to `conf/bblayers.conf` in build directory
  	```bash
  	bitbake-layers add-layer "${ROOTDIR}/sources/yocto/meta-realtek-matter"
  	```
8. (Optional) [Edit matter recipe settings](<#Edit-matter-recipe-settings>)

# Edit matter recipe settings
- At `sources/yocto/meta-realtek-matter/recipes-matter/<matter-recipe>/<matter-recipe>.inc`

## Matter recipe variables
1. `MATTER_APP_NAME`
	- Give a list of the matter example apps you want to build
	- Note that when removing matter apps to be built, commenting out a line with `#` does not work for `MATTER_APP_NAME`. Please delete the corresponding line of the matter app you want removed instead.

2. `CHIP_ROOT`
	- Full path to the root directory of connectedhomeip
	- Can make use of `TOPDIR` to refer to the Yocto build directory created after sourcing Yocto setup script

3. `MATTER_APP_SRC_DIR`
	- Full directory path to the Matter application source code
	- Can make use of `MATTER_APP` to refer to individual directory name inside `MATTER_APP_NAME`

4. `MATTER_BUILD_DIR`
	- Relative directory path where the Matter application will be built, relative to `MATTER_APP_SRC_DIR`

5. `MATTER_APP_TARGET_DIR`
	- Full directory path where the Matter application will be installed on the target device

6. `MATTER_GN_ARGS`
	- gn args to be passed to generate ninja build files

7. `MATTER_BUILD_FILENAME`
	- Build filename. By default, `BUILD.gn` should be at root of `MATTER_APP_SRC_DIR`

# Select Matter recipe
Matter recipes will build example matter application(s).

At `sources/yocto/meta-realtek-matter/conf/layer.conf`, `IMAGE_INSTALL` variable, you can add the matter recipe you want to be included to the Yocto image.

For example, you can choose multiple matter recipes to install.
```bash
IMAGE_INSTALL += "matter matter-ported-examples matter-custom-dac-examples"
```

## Matter recipe types
There are multiple matter recipes that have been prepared for you to use and explore.
- `matter` recipe will add the Linux example matter applications from `connectedhomeip/examples/*/linux/`
- `matter-ported-examples` recipe will add ported example matter applications from `sources/yocto/meta-realtek-matter/examples/`
- `matter-custom-dac-examples` recipe will add ported example matter applications from `sources/yocto/meta-realtek-matter/examples/` that uses custom Device Attestation Certificate (DAC) for commissioning

For ported matter examples, a README.md is available at each of the subdirectory of the ported matter example source code `meta-realtek-matter/examples/`, to guide you to use the example.


# Build Yocto Matter
> [!NOTE]
> [Yocto Matter setup](<#yocto-matter-setup>) should be completed first

The steps to build matter is the same as the one from AmebaSmart Yocto SDK
1. [Build AmebaSmart Yocto](<AmebaSmart Yocto.md#build-amebasmart-yocto>)

# Flashing image to board
[AmebaSmart Yocto Flashing to NAND](<AmebaSmart Yocto Flashing to NAND.md>)

# Serial Monitor

| Settings  | Value       |
| --------- | ----------- |
| Baudrate  | 1500000 bps |
| Data bits | 8           |
| Parity    | None        |
| Stop bits | 1           |
| Encoding  | UTF8        |

1. Connect USB port from computer to UART type-C on Ameba board
2. Open Putty or similar serial monitor tools and set the baudrate to 1500000 to be able to view serial log output
3. Press and release the reset button (CHIP_EN) to boot
4. Once boot has completed, you should see the following on the serial output monitor
	```bash
	...
	Running local boot scripts (/etc/rc.local)
	Welcome to RTL8730ELH
	    _____     _____       ____      ____
	   |  ___\ \ |  ___\ \  / ___  \  /  ___ \
	   | |   | | | |   | | / /    \ \ | |
	   | |--- /   \ ---| | | |    | |  \ --- \
	   | |         ____/ /  \ \__/ /   ____/ /
	   |_|        |____ /    \____/   |_____/
	
	```
5. You can now use usual Linux commands (e.g. ls, cat, mkdir, touch, cd, cp, mv, rm, find, echo, chmod) on the serial monitor

# Run Yocto matter apps
> [!TIP]
> [Remove stale chip configuration](<#remove-stale-chip-configuration>) on target AmebaSmart before every new commissioning

1. Navigate to directory containing installed matter apps
	```bash
	cd /matter-application
	```
2. List installed matter apps
	```bash
	# ls
	chip-lighting-app
	thermostat-app
	```
3. Run matter app
	```bash
	./chip-lighting-app --wifi --discriminator 3840
	```
4. Look for the following line in the output logs and open the link in a web browser to see the QR code for commissioning
	```bash
	CHIP:SVR: https://project-chip.github.io/connectedhomeip/qrcode.html?data=MT%3A-24J042C00KA0648G00
	```
	- You should observe the following generated QR code,

		![QR code for commissioning with discriminator 3840](<images/commissioning_qr_code_discriminator_3840.png>)

		The QR code can be scanned by a commissioner (smartphone running Google Home, Apple HomeKit, Samsung SmartThings, or Home Assistant).

# Commissioning
To commission using Google Home, Apple Homekit, Samsung SmartThings, or Home Assistant, consult the vendor’s guide.

We shall demonstrate how to commission using [chip-tool](https://project-chip.github.io/connectedhomeip-doc/guides/chip_tool_guide.html).

## Commissioning Methods
Ameba only supports BLE and WIFI commissioning methods.

### BLE Commissioning
Commission matter app to network using [chip-tool](https://project-chip.github.io/connectedhomeip-doc/guides/chip_tool_guide.html) on host machine
```bash
./chip-tool pairing ble-wifi 0x1234 "WIFI_SSID" "WIFI_PASSWORD" "20202021" "3840"
```

### WIFI Commissioning
- If the target board is already on same WIFI network as host machine running chip-tool, commission over network,
	```bash
	./chip-tool pairing onnetwork-long 0x1234 20202021 3840
	```
- Alternatively, commission with QR code.
	```bash
	./chip-tool pairing code 0x1234 MT:-24J042C00KA0648G00
	```

- When commissioning is successful, you should observe the following:
	```bash
	CHIP:TOO: Device commissioning completed with success
	```

### Decommission
- You can decommission a commissioned device using the unpair command
	```bash
	./chip-tool pairing unpair 0x1234
	```

- When decommissioning is successful, you should observe the following
	```bash
	CHIP:TOO: Device unpair completed with success: 0000000000001234
	```

# Controlling Commands
On the host machine, use chip-tool to perform cluster control. Refer to [chip-tool](https://project-chip.github.io/connectedhomeip-doc/guides/chip_tool_guide.html) for full usage guide.

Below are some examples of chip-tool commands to control clusters.
```bash
./chip-tool onoff on 0x1234 1
```

```bash
$ ./chip-tool onoff read on-off 0x1234 1 | grep OnOff
[1719452808.138994][2697:2700] CHIP:TOO:   OnOff: TRUE
```

```bash
./chip-tool levelcontrol move-to-level 150 0 0 0 0x1234 1
```

```bash
$ ./chip-tool levelcontrol read current-level 0x1234 1 | grep CurrentLevel
[1719452898.654921][2710:2713] CHIP:TOO:   CurrentLevel: 150
```

# Remove stale chip configuration
```bash
rm -rf /tmp/chip*
```

# Yocto Matter directories
## Installed matter application
### Compiled matter applications
- `~/rtl8730e-linux-sdk-3.1-release/build_rtl8730elh-va8-generic/tmp/work/cortexa32hf-neon-rtk-linux-gnueabi/matter/0.1-r0/image/`
### Stripped matter applications
- `~/rtl8730e-linux-sdk-3.1-release/build_rtl8730elh-va8-generic/tmp/work/cortexa32hf-neon-rtk-linux-gnueabi/matter/0.1-r0/package`
## Logs
- `~/rtl8730e-linux-sdk-3.1-release/build_rtl8730elh-va8-generic/tmp/work/cortexa32hf-neon-rtk-linux-gnueabi/matter/0.1-r0/temp/`

# Yocto Troubleshooting
[AmebaSmart Yocto Troubleshoot](<AmebaSmart Yocto Troubleshoot.md>)
