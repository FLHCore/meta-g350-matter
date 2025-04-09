# Introduction

This is a Yocto layer for building Matter applications using [connectedhomeip](https://github.com/project-chip/connectedhomeip).  
It is designed to work with [MediaTek's Yocto BSP layer](https://mediatek.gitlab.io/aiot/doc/aiot-dev-guide/master/sw/yocto/get-started/build-code.html).

This layer is forked from [Ameba's meta-realtek-matter](https://github.com/Ameba-AIoT/meta-realtek-matter).  
In this fork, BlueZ support is disabled because the G350 platform uses Bluedroid instead.

This provides a minimal setup to run `chip-tool` on the G350.

# Usage
To build and use this layer, clone it into your Yocto project source directory.  
For MTK Yocto, this is typically: /root/src/
Then, add the layer:

```bash
bitbake-layers add-layer [path-to-meta-g350-matter]
```

Note:
You must also clone the connectedhomeip project outside of your Yocto project directory.
Make sure the connectedhomeip project can be built independently before continuing.

```bash
git clone https://github.com/project-chip/connectedhomeip.git
cd connectedhomeip
./scripts/checkout_submodules.py --platform linux
```

# Matter Layer Structure

| Directory                                 | Description                                                                                                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| conf                                      | Contains the matter layer configuration file “layer.conf”, which contains configuration settings for the matter layer.                                                   |
| doc                                       | Contains markdown formatted documentations for using the matter layer.                                                                                                   |
| examples                                  | Contains the source code for ported matter application examples. Follow the included README.md within each example subdirectory to use the examples.                     |
| recipes-connectivity                      | Contains specific WIFI and Bluetooth driver changes (overriding the base Ameba Yocto SDK) for Matter applications to be able to perform commissioning.                   |
| recipes-devtools                          | Contains development tool recipes required to build matter recipes.                                                                                                      |
| recipes-matter                            | Contains matter recipes to build matter example applications.                                                                                                            |
| recipes-matter/matter                     | Contains recipes that will add the Linux example matter applications from connectedhomeip/examples/*/linux/.                                                             |
| recipes-matter/matter-custom-dac-examples | Contains recipes that will add ported example matter applications from sources/matter/examples/ that uses custom Device Attestation Certificate (DAC) for commissioning. |
| recipes-matter/matter-ported-examples     | Contains recipes that will add ported example matter applications from sources/matter/examples/.                                                                         |


