# meta-g350-matter

A Yocto layer to integrate [Matter (connectedhomeip)](https://github.com/project-chip/connectedhomeip) on **MediaTek Genio 350 (G350)** platform.

This layer is a **fork of** [Ameba's `meta-realtek-matter`](https://github.com/Ameba-AIoT/meta-realtek-matter), customized to work with the **MTK G350** series.  
The key modification includes **disabling BlueZ** (not supported on G350) in favor of **Bluedroid**.

> ✅ Provides a minimal setup to run `chip-tool` on Genio 350 EVK.

---

## 🔧 Prerequisites

- Yocto BSP for MTK Genio 350:  
  [MTK Yocto BSP Guide](https://mediatek.gitlab.io/aiot/doc/aiot-dev-guide/master/sw/yocto/get-started/build-code.html)

- [connectedhomeip](https://github.com/project-chip/connectedhomeip) SDK cloned and initialized **outside of the Yocto project directory**:

```bash
git clone https://github.com/project-chip/connectedhomeip.git
cd connectedhomeip
./scripts/checkout_submodules.py --platform linux
```

---

## 🚀 Getting Started

### 1. Clone this layer into your Yocto project directory

```bash
cd /home/user/aiot-yocto/src
git clone https://github.com/FLHCore/meta-g350-matter.git
bitbake-layers add-layer meta-g350-matter
```

### 2. Run the environment setup script

Create a `setup-env.sh` file and paste the following content:

```bash
#!/bin/bash

set -e

export PROJ_ROOT=$(pwd)
export ROOTDIR=$PROJ_ROOT
export TEMPLATECONF="$PROJ_ROOT/src/meta-rity/meta/conf/"
source "$PROJ_ROOT/src/poky/oe-init-build-env"
export BUILD_DIR=$(pwd)
export MACHINE=genio-350-evk

CHIP_SDK_PATH_VAL="/home/ross/connectedhomeip"
CHIP_CONF="$BUILD_DIR/conf/local.conf"

add_conf_if_missing() {
    KEY="$1"
    VALUE="$2"
    if ! grep -q "^$KEY" "$CHIP_CONF"; then
        echo "$KEY = "$VALUE"" >> "$CHIP_CONF"
        echo "➕ Added $KEY to local.conf"
    else
        echo "✔ $KEY already set in local.conf"
    fi
}

echo ""
echo "🛠 Injecting Matter-related variables into local.conf..."
add_conf_if_missing "CHIP_SDK_PATH" "$CHIP_SDK_PATH_VAL"
add_conf_if_missing "CHIP_USE_GN" "1"
add_conf_if_missing "USE_MATTER_SDK" "1"

echo ""
echo "✅ Yocto MTK build environment ready"
echo "📁 BUILD_DIR = $BUILD_DIR"
echo "📦 MACHINE   = $MACHINE"
echo "🧱 SDK Path  = $CHIP_SDK_PATH_VAL"
echo "👉 You can now run:  bitbake rity-demo-image"
echo ""
```

run：

```bash
source ./setup-env.sh
```

This script will:
- Set up environment variables (`PROJ_ROOT`, `BUILD_DIR`, etc.)
- Source `oe-init-build-env`
- Inject Matter-specific settings to `local.conf`

### 3. Start building

```bash
export MACHINE=genio-350-evk
bitbake rity-demo-image
```

---

## 📂 Layer Structure

| Directory                                 | Description                                                                                                  |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `conf`                                   | Yocto layer configuration (`layer.conf`).                                                                    |
| `doc`                                    | Documentation in markdown format.                                                                            |
| `examples`                               | Ported Matter example applications. Each subfolder has its own README.                                       |
| `recipes-connectivity`                   | WiFi and Bluetooth driver overrides (Bluedroid instead of BlueZ).                                            |
| `recipes-devtools`                       | Development tool recipes needed by Matter applications.                                                      |
| `recipes-matter/`                        | Recipes to build Matter apps using the official Matter SDK.                                                  |
| `recipes-matter/matter-ported-examples`  | Recipes for ported Matter examples from `sources/matter/examples`.                                           |
| `recipes-matter/matter-custom-dac-examples` | Matter examples using custom DACs for commissioning.                                                     |

---

## ⚠️ Known Issues

- Some users report **terminal disconnection** when:
  - Build errors occur
  - Pressing `Tab` for autocompletion

If you encounter this:
- Try using a more stable terminal emulator like **tmux** or **screen**
- Or manually run each setup command in your shell