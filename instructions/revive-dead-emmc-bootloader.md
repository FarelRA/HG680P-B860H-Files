# Revive Dead eMMC or Corrupt Bootloader for HG680P & B860H TV Boxes with Armbian

This guide details how to flash and configure a high-performance Linux server OS (Armbian) on old Indonesian TV boxes. This specific method uses a bootloader injection technique that forces the device to boot from the SD card, even if the internal storage is dead, the original Bootloader is corrupted, or Android OS is bricked.

---

## 🛠 Prerequisites & Downloads

### Hardware
* **TV Box:** HG680P or B860H (Amlogic S905X).
* **MicroSD Card:** 8GB+ (Class 10 recommended).
* **Card Reader:** A reliable USB card reader.
* **PC/Laptop:** Windows 10/11.

### Software
1.  **Bootcard Maker:** Tool to inject the magic bootloader.
    * [Download Bootcard Maker](https://raw.githubusercontent.com/FarelRA/HG680P-B860H-Files/refs/heads/main/tools/BootcardMaker.exe)
2.  **Custom Bootloader (`u-boot.bin`):** Pre-modified for HG680P/B860H compatibility.
    * [Download `u-boot.bin`](https://raw.githubusercontent.com/FarelRA/HG680P-B860H-Files/refs/heads/main/files/revive-dead-emmc-bootloader/u-boot.bin)
3.  **AutoScript (`aml_autoscript`):** Essential script to trigger the boot process.
    * [Download `aml_autoscript`](https://raw.githubusercontent.com/FarelRA/HG680P-B860H-Files/refs/heads/main/files/revive-dead-emmc-bootloader/aml_autoscript)
4.  **Rufus:** The tool to burn the OS image.
    * [Download Rufus](https://rufus.ie)
5.  **Armbian Image (ophub Amlogic Build):**
    * [Download from GitHub](https://github.com/ophub/amlogic-s9xxx-armbian/releases)
    * *Selection Guide:* Open "Assets" on the latest release. Look for a file name like:
        `Armbian_xx.xx.x_amlogic_s905x_xxx_x.xx.x_server_xxxx.xx.xx.img.gz`

---

## 🚀 Step-by-Step Installation

### Phase 1: Burn the Image
1.  Extract the Armbian Image archive file you downloaded.
2.  Insert your MicroSD card into your PC.
3.  Open **Rufus** then:
    * **Device:** Ensure your SD Card is selected.
    * **Boot Selection:** Click "SELECT" and choose the Armbian `.img`.
    * **Start:** Click the "START" button.
    * **Confirmation:** Confirm the warning that all data on the SD card will be destroyed.
4.  Wait for the progress bar to reach 100% ("READY"). Close Rufus.

### Phase 2: Inject the Bootloader (Critical Step)
*This step writes a specific bootloader to the hidden sector of the SD card, allowing it to take over the TV box immediately upon power-up.*

1.  Open **Bootcard Maker**.
    * **Choose Disk:** Select your SD Card drive.
    * **To Partition and Format**: **YOU MUST *UNCHECK* THIS BOX.**
    	* *Reason:* If you leave it checked, it will erase the Armbian OS you just installed in Phase 1.
    * **Choose your bin files:** Click "Open" and select the `u-boot.bin` file you downloaded.
    * **Make:** Click the "Make" button.
2.  **Success:** Wait for a popup message saying "Success". Close the tool.

### Phase 3: Configure Boot Files
1.  Open your Windows File Explorer.
2.  Navigate to the SD Card. It should be named `BOOT`.

#### A. Setup the U-Boot
1.  In the root folder, look for a file named `u-boot-p212.bin`.
2.  **Copy** this file and **Paste** it 3 times in the same folder.
3.  **Rename** the 3 copies to these exact names:
    * `u-boot.ext`
    * `u-boot.sd`
    * `u-boot.usb`
4.  Overwrite if it ask to.

#### B. Install the Autoscript
1.  Copy the `aml_autoscript` file you downloaded.
2.  Paste it directly into the **root** folder of the SD card (the main folder where you see all the files).
3.  Overwrite if it ask to.

#### C. Configure Device Tree (DTB)
You must edit three text files to tell the system it is running on an S905X chip (P212 board).

**1. Edit `uEnv.txt`**
* Open `uEnv.txt` with Notepad.
* Find the line starting with `FDT=`. Change it to:
    ```ini
    FDT=/dtb/amlogic/meson-gxl-s905x-p212.dtb
    ```
* Save and Exit.

**2. Edit `boot.ini`**
* Open `boot.ini` with Notepad.
* Find the block discussing `devtype`. Ensure it points to the p212 dtb file:
    ```ini
    if test "${devtype}" = ""; then setenv devtype "/dtb/amlogic/meson-gxl-s905x-p212.dtb"; fi
    ```
* Save and Exit.

**3. Edit `extlinux/extlinux.conf`**
* Go into the `extlinux` folder.
* Rename the `extlinux.conf.bak` to `extlinux.conf`
* Open `extlinux.conf` with Notepad.
* Update the `FDT` line to match the p212 path:
    ```text
    FDT /dtb/amlogic/meson-gxl-s905x-p212.dtb
    ```
* Save and Exit.

### Phase 4: First Boot
1.  Safely Eject the SD card from Windows.
2.  Insert the SD card into the SD slot of your HG680P or B860H.
3.  Connect the LAN cable (Ethernet) to your router.
4.  Plug in the Power Adapter.
5.  **Wait:** Do not interrupt the power. The first boot takes 3-10 minutes as the script resizes the partition and sets up the system.
    * *Indicator:* If your box has an LED, the green LED may stay off for a moment before it back on again.
	* Check your Router's "Client List" page for a new device named `armbian`.