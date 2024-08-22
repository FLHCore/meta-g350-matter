# Flashing to NAND for Yocto
- Use Windows Ameba ImageTool
- Requires flashing with USB mode
## USB mode
1. Connect UART type-C port to provide 5V power
2. Connect micro B USB port
3. Trigger `Download` mode
	1. Hold Download button
	2. Press and release reset EN pin
	3. Release Download button

# Flash using Windows ImageTool
1. At images directory, `~/rtl8730e-linux-sdk-3.1-release/images/`. Copy from Linux to Windows the following:
	1. `km4_boot_all.bin`
	2. `km0_km4_app.bin`
	3. `boot.img`
	4. `xxx.dtb` ([Finding dtb filenames](<#Finding-dtb-filenames>))
	5. `kernel.img`
	6. `rootfs.img`
	7. `userdata.img`
		
	- Below shows example commands to copy images from Ubuntu (in VMware) to Windows folder `~/Documents/Realtek/Ameba/AmebaSmart/tools/ameba/binaries/va-x-xxx/`
		- [setup shared folder in VMware](https://docs.vmware.com/en/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-D6D9A5FD-7F5F-4C95-AFAB-EDE9335F5562.html)
		- va8-generic
		```bash
		IMG_DIRECTORY="${HOME}/dev/rtl8730e-linux-sdk-3.1-release/build_rtl8730elh-va8-generic/tmp/deploy/images/rtl8730elh-va8/"
		IMG_DESTINATION="/mnt/hgfs/Documents_ext/Realtek/Ameba/AmebaSmart/tools/ameba/binaries/va-8-generic/"
		cp ${IMG_DIRECTORY}km4_boot_all.bin ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}km0_km4_app.bin ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}boot.img ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}rtl8730elh-va8-generic--5.4-r0-rtl8730elh-va8-20240629165442.dtb ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}kernel.img ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}rootfs.img ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}userdata.img ${IMG_DESTINATION}
		```
		- va8-full
		```bash
		IMG_DIRECTORY="${HOME}/dev/rtl8730e-linux-sdk-3.1-release/build_rtl8730elh-va8-full/tmp/deploy/images/rtl8730elh-va8/"
		IMG_DESTINATION="/mnt/hgfs/Documents_ext/Realtek/Ameba/AmebaSmart/tools/ameba/binaries/va-8-full"
		cp ${IMG_DIRECTORY}km4_boot_all.bin ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}km0_km4_app.bin ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}boot.img ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}rtl8730elh-va8-full--5.4-r0-rtl8730elh-va8-20240629171420.dtb ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}kernel.img ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}rootfs.img ${IMG_DESTINATION} && \
		cp ${IMG_DIRECTORY}userdata.img ${IMG_DESTINATION}
		```

2. Launch Windows ImageTool
3. Open Device Profile
	- AmebaSmart-VA7: [[AmebaSmart_Linux_NAND_128MB_VA7.rdev]]
	- AmebaSmart-VA8: [[AmebaSmart_Linux_NAND_256MB_VA8.rdev]]
4. Select respective `.bin` and `.img` that was built using Yocto `bitbake`
5. Click Download
# Finding dtb filenames
- At images directory, `~/rtl8730e-linux-sdk-3.1-release/images/`
	```bash
	find . -mindepth 1 ! -type l | grep .dtb
	```
