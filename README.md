# Introduction
This is a layer that builds Matter applications using [connectedhomeip](https://github.com/project-chip/connectedhomeip). 
This layer depends and builds on top of [MTK's Yocto BSP layer](https://mediatek.gitlab.io/aiot/doc/aiot-dev-guide/master/sw/yocto/get-started/build-code.html).


It's a fork from [Ameba meta-realtek-matter](https://github.com/Ameba-AIoT/meta-realtek-matter).
This fork disable bluez because g350 use bluedroid.
It's a minimum support to run chip-tool on g350.

# Usage
To build and use this layer, download it into yocto project source.
in mtk yocto, it's /root/src/.
then
```
bitbake-layers add-layer [meta-g350-matter path]
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


