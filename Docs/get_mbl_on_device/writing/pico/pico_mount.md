## Mounting the device

1. For PICO device: Run the USB OTG (on the go) Loader, using the following commands on Linux:<!--is this specific to pico? I don't think I saw it in the other ones-->

    ```
    cd pico-imx6-imx6ul-imx7_otg-installer_20171101
    sudo linux/imx_usb pico-imx7d_bootbomb_20170112.imx
    ```

    <span class="notes">Make sure you run the command as root using sudo, otherwise the `imx_usb` loader will report `main:Could not open device vid=0x... pid=0x... err=-3`.</span>

    You should see output on the PC that ends with:

    ```
    loading binary file(pico-imx7d_bootbomb_20170112.imx) to 877ff400, skip=0, fsize=259c00 type=aa

    <<<2464768, 2464768 bytes>>>
    succeeded (status 0x88888888)
    jumping to 0x877ff400
    ```

    On the console output from the device, it should end with:

    ```
    [    1.107086] Mass Storage Function, version: 2009/09/11                   
    [    1.112243] LUN: removable file: (no medium)                                 
    [    1.124192] Number of LUNs=1                                                 
    [    1.631131] configfs-gadget gadget: high-speed config #1: c                  
    ```
