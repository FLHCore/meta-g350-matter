# Yocto Matter host machine requirements
- [Official Yocto System Requirements](https://docs.yoctoproject.org/4.0.19/ref-manual/system-requirements.html)
- The following is the test system specifications that was used to build AmebaSmart Yocto Matter, consisting of:
	1. connectedhomeip
	2. AmebaSmart Yocto distro: ameba-full
	3. AmebaSmart Yocto distro: ameba-generic

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
	git clone -b v1.3-branch --recurse-submodules git@github.com:project-chip/connectedhomeip.git
	```
3. Navigate inside `connectedhomeip`
	```bash
	cd connectedhomeip
	```
4. Update `connectedhomeip` submodules
	```bash
	git submodule update --init --recursive
	```
5. Perform bootstrap
	```bash
	source scripts/bootstrap.sh
	```
6. Upgrade pip
	```bash
	python -m pip install --upgrade pip
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
	sudo apt install chrpath diffstat liblz4-tool zstd gawk python3-pycryptodome repo
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
	repo init -u https://github.com/Ameba-AIoT/ameba-linux-manifest -b ameba-3.1 -m ameba-3.1-matter.xml
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
- At `~/rtl8730e-linux-sdk-3.1-release/sources/matter/recipes-matter/matter/matter.inc`

## Matter recipe variables
1. `MATTER_APP_NAME`
	- Give a list of the matter example apps you want to build
	- Names should follow the exact directory names inside of `connectedhomeip/examples`
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

At `sources/matter/conf/layer.conf`, `IMAGE_INSTALL` variable, you can add the matter recipe you want to be included to the Yocto image.

For example, you can choose multiple matter recipes to install.
```bash
IMAGE_INSTALL += "matter matter-ported-examples matter-custom-dac-examples"
```

## Matter recipe types
There are multiple matter recipes that have been prepared for you to use and explore.
- `matter` recipe will add the Linux example matter applications from `connectedhomeip/examples/*/linux/`
- `matter-ported-examples` recipe will add ported example matter applications from `sources/matter/examples/`
- `matter-custom-dac-examples` recipe will add ported example matter applications that uses custom Device Attestation Certificate (DAC) for commissioning

For ported matter examples, a `README.md` is available at the directory of the ported matter example source code `sources/matter/examples/`, to guide you to use the example.


# Build Yocto Matter
> [!NOTE]
> [Yocto Matter setup](<#yocto-matter-setup>) should be completed first

The steps to build matter is the same as the one from AmebaSmart Yocto SDK
1. [Build AmebaSmart Yocto](<AmebaSmart Yocto.md#build-amebasmart-yocto>)

# Flashing image to board
[AmebaSmart Yocto Flashing to NAND](<AmebaSmart Yocto Flashing to NAND.md>)

# UART serial monitor
- Baudrate: `1500000`
- After flashing, reset the board by pressing `CHIP_EN` button
# Connect to WIFI
## ifupdown
- On target AmebaSmart, run the following commands
1. Add WIFI credentials
	```bash
	wpa_passphrase "WIFI_SSID" "WIFI_PASSWORD" >> /etc/wpa_supplicant.conf
	```
2. Turn off WIFI interface
	```bash
	ifdown wlan0
	```
3. Turn on WIFI interface
	```bash
	ifup wlan0
	```
# Running Yocto matter apps
> [!TIP]
> [Remove stale chip configuration](<#remove-stale-chip-configuration>) on target AmebaSmart before every new commissioning

1. Navigate to directory containing installed matter apps
	```bash
	cd /matter-application
	```
2. List installed matter apps
	```bash
	# ls
	air-quality-sensor-app         chip-ota-requestor-app
	chip-air-purifier-app          chip-refrigerator-app
	chip-all-clusters-app          chip-rvc-app
	chip-all-clusters-minimal-app  chip-tv-app
	chip-bridge-app                chip-tv-casting-app
	chip-dishwasher-app            contact-sensor-app
	chip-energy-management-app     lit-icd-app
	chip-lighting-app              network-manager-app
	chip-lock-app                  persistent_storage
	chip-microwave-oven-app        thermostat-app
	chip-ota-provider-app
	```
3. Run matter app
	```bash
	./chip-lighting-app --wifi --discriminator 3844
	```
4. Commission matter app to network using [chip-tool](https://project-chip.github.io/connectedhomeip-doc/guides/chip_tool_guide.html) on host machine
	```bash
	./chip-tool pairing ble-wifi 0x1213 "WIFI_SSID" "WIFI_PASSWORD" "20202021" "3844"
	```
	Alternatively, if the target board is already on same WIFI network as host machine, commission over network,
	```bash
	./chip-tool pairing onnetwork-long 0x1213 20202021 3844
	```
	
	```bash
	$ ./chip-tool pairing onnetwork-long 0x1213 20202021 3844 | grep "Commissioning complete"
	[1719452234.100503][2680:2683] CHIP:CTL: Commissioning complete for node ID 0x0000000000001213: success
	```
5. On the host machine, use [chip-tool](https://project-chip.github.io/connectedhomeip-doc/guides/chip_tool_guide.html) to perform cluster control
	```bash
	./chip-tool onoff on 0x1213 1
	```
	
	```bash
	$ ./chip-tool onoff read on-off 0x1213 1 | grep OnOff
	[1719452808.138994][2697:2700] CHIP:TOO:   OnOff: TRUE
	```
	
	```bash
	./chip-tool levelcontrol move-to-level 150 0 0 0 0x1213 1
	```
	
	```bash
	$ ./chip-tool levelcontrol read current-level 0x1213 1 | grep CurrentLevel
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
