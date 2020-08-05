# HyperPixel 4.0 on Raspberry Pi 4B
Getting the Pimoroni HyperPixel 4.0 setup correctly and operational on a Raspberry Pi 4B turned out to be a voyage of discovery. I have captured the results here for myself and others. I have included my working config.txt (Raspian Buster) in the repo. This is specifically for the Pi 4B versions. Pi3 and earlier do not have these issues. It is highly recommended that you make a backup of your Octoprint configuration. This can be done from the web UI by going to settings, and clicking on **Backup & Restore**. Click on the **Create backup now** button and wait for completion. Copy the file from ~pi/.octoprint/data/backup/<backupfilename>.zip using your favorite transfer program. Try [WinSCP](https://winscp.net/) if you don't have something already installed. Mac and Linux folks have sftp at their disposal.

## Getting Ready
It is important to understand that Pimoroni uses *ALL* of the GPIO pins on the pi. Also the order of installation is important. You *can* install the screen after OctoDash is installed, but it can create situations of finding the where the video is going. In my case, since I use MobaXterm, OctoDash came up fullscreen on my 30" monitor. :satisfied:

Making the screen work is easiest if one starts with a fresh Octopi install and not changing **ANYTHING** other than what the installation instructions tell you do until after the screen is up and working.

## Step 1 - Install OctoPi
Follow the published instructions for getting OctoPi installed and running. https://github.com/guysoft/OctoPi

## Step 2 - Install the HyperPixel and driver (kernel 4.x systems) 
If you update your pi to the latest bits, you will find that the kernel version is now in the 5.4.x range. Run **uname -r** on the console or ssh session to get the kernel version. If it is 4.x, you are good to follow these instructions. If it says 5.x, you will need to follow the alternate Step 2 instructions below. According to the Pimoroni folks, the patch version below should work just fine on a Pi3. I haven't tested it so YMMV.
1. Plug the screen into the Pi and secure it using the provided stand-offs.
2. Power-up with a 5v3A power source and a high quality cable. The official pi power supply is recommended. Using a phone charger or other power supply may not provide enough power for the screen to operate properly.
3. Install the drivers using the one-line installation.<br>
```bash
curl -sSL https://get.pimoroni.com/hyperpixel4 | bash

```
4. Fixup the /boot/config.txt file to rotate the screen to landscape, and disable vc4-fkms-v3d dtbo.
   1. Add *:rotate,touchscreen-swapped-x-y* to the end of the first hyperpixel overlay line
   2. Comment out all *dtoverlay=vc4-fkms-v3d* lines in the file
   3. Add a line *display_lcd_rotate=3* at the end of the file
5. Reboot. <br>The screen should come on and display a console if running buster-lite or a desktop if running full Raspian.
6. Run raspi-config and change the boot option to boot into console with autologin to pi user.
7. Reboot.
## Alternate Step 2 - Intstall the HyperPixel and driver (kernel 5.x systems)
1. Install git
```
sudo apt install git
```
2. Clone this GitHub repository branch to your Pi:
```
git clone https://github.com/pimoroni/hyperpixel4 -b pi4
```
3.Run the installer:
```
cd hyperpixel4
sudo ./install.sh
sudo reboot now
```

4. Set the boot options to boot into console using autologin for pi.
```
sudo raspi-config
# choose 3 Boot Options
# choose B1 Desktop/CLI
# choose B2 Console Autologin Text console, automatically logged in as 'pip' user
# choose <ok><finish> and reboot if prompted.  
```
Running the **hyperpixel_rotate** command before you install **OctoDash** will fail because the rotation code uses X-windows modules to achieve its results. Wait until after OctoDash is installed before doing the rotation.

### Troubleshooting
If you only get a black screen with the backlight on, chances are that i2c, spi, or i2s are enabled in the config.txt. They *must* be commented out for the solution to work. Pimoroni directly manages those capabilities from their driver.

## Step 3 - Install OctoDash
1. Follow the published instructions for installing OctoDash<br>https://github.com/UnchartedBull/OctoDash. This will take a bit of time as it installs a big chunk of the X-windows system to acheived the desired environment. 
```
bash <(wget -qO- https://github.com/UnchartedBull/OctoDash/raw/master/scripts/install.sh)
```

## Step 4 - Rotation
I could not get the Hyperpixel rotation commands to work. So by following these steps you will get it in the right direction. For landscape the display_lcd_rotate=3 provides orientation for power and video on the "lower" side. Use display_lcd_rotate=1 if you need the power/display on the "upper" side.
1. Add *:rotate,touchscreen-swapped-x-y* to the end of the first hyperpixel overlay line
2. Add a line *display_lcd_rotate=3* at the end of the file
3. Comment out all *dtoverlay=vc4-fkms-v3d* lines in the file
4. reboot

### References

OctoPi Github - https://github.com/guysoft/OctoPi<br>
OctoDash Github - https://github.com/UnchartedBull/OctoDash<br>
Pimoroni Github - https://github.com/pimoroni/hyperpixel4<br>
> Relevant Issue: https://github.com/pimoroni/hyperpixel4/issues/39


### Tested on
Pi4B 1G, 4G Raspian 4.7 kernel - Passed<br>
Pi4B 1G, 4G Raspian 5.4 kernel - Use the Pimoroni fix at  https://github.com/pimoroni/hyperpixel4/tree/patch-pi4-i2c-fix.<br>

> Kudos to the Pimoroni team who have been in rapid response mode, working to get this resolved so quickly.
