# Klipper Thingino Wyze V3 Spotlight Control

The `[spotlight.cfg]` file enables the use of the Wyze v3 camera's Spotlight accessory on cameras flashed with the Thingino open-source firmware for Ingenic SoC cameras, integrating them into Klipper. 

The basis of the spotlight control would not be possible without the `gcode_shell_command`.

This script and the information here are provided for informational purposes and furnished without warranty. This guide describes in detail the implementation of the script and its associated requirements. Note that your environment may differ from that of the original author. Be advised that you should make an effort to understand the processes and concepts used here, and how they may differ from your own implementation and environment.

## Enable the G-code Shell Command

The `gcode_shell_command` extension allows Klipper to execute Linux shell commands or scripts directly from the printer configuration, enabling automation tasks like web API calls, file backups, or system monitoring without blocking the printer's motion queue. Created by [Arksine](https://github.com/Arksine) and installable via KIAUH, this feature requires defining a command in printer.cfg with a command, timeout, and verbose setting, then triggering it via `RUN_SHELL_COMMAND CMD=name`. 

More information on this can be found on the Kiauh repository:  
[https://github.com/dw-0/kiauh/blob/master/docs/gcode\_shell\_command.md](https://github.com/dw-0/kiauh/blob/master/docs/gcode_shell_command.md)

## Install via Kiauh

SSH into your host and navigate to the Kiauh directory  
```shell  
cd kiauh  
./kiauh.sh  
```  
In kiauh  
Select the option E for \[Extensions\]  
Select the option for G-Code Shell Command  
Once complete, use the back option and then quit Kiauh  
Restart Klipper:  
```shell  
sudo systemctl restart klipper  
```

## Manual Install

By default, the Kiauh G-code Shell Command Extension is packaged in Kiauh itself. If you’re looking to install it manually, you’ll need the entire contents from the following repository:

[https://github.com/dw-0/kiauh/tree/master/kiauh/extensions/gcode\_shell\_cmd](https://github.com/dw-0/kiauh/tree/master/kiauh/extensions/gcode_shell_cmd)

In a standard Kiauh install, this would be installed at:   
```shell  
\~/home/biqu/kiauh/kiauh/extensions/gcode\_shell\_cmd  
```  
After copying to the appropriate location, be sure to restart Klipper  
```shell  
sudo systemctl restart klipper  
```

### Warning

This extension may have a high potential for abuse if not used carefully\! Also, depending on the command you execute, high system load may occur, causing system instability. Use this extension at your own risk and only if you know what you are doing\!

#### Note

The use of this G-code Shell Command in conjunction with the script in this repo requires minimal processing. This means that it should not cause unexpected system load leading to print failures. Though note that your system may differ based on its system properties. 

## Add \[spotlight.cfg\] File to Host

On your Klipper host, locate the main directory where your `printer.cfg` file is. This is most commonly found at `~/printer_data/config/`.

### Via Git

1. SSH into your Klipper host and navigate to the directory where Klipper’s printer.cfg is located  
2. User curl to get the file:  
   `https://github.com/unknowndevice1447/thinginoWyzeSpotlight.git`

#### Manual Download

Use the Code option to download a .zip of the rpository.
`https://github.com/unknowndevice1447/thinginoWyzeSpotlight.git`

## Modify \[spotlight.cfg\] to Fit Your Needs

The script allows for multiple cameras by default. While it’s set up for two cameras, it can be extended to support many cameras. It can also be modified to use on one camera. Here, explore options for modifying the script to fit your needs. 

### Modify Your Camera’s IP and Klipper Hostname

In the spotlight.cfg file, you’ll want to modify a couple of lines to account for your Klipper device’s hostname and the IP address(es) of your cameras. 

Each Shell command definition has three available functions: High, Low, and Off. Each of these needs to have the `<hostname>` and `<cam_ip>` updated to reflect your environment. 

#### Modify Your Hosts Name

In the Shell Command sections, be sure to replace the `<hostname>` with the hostname of your Klipper machine.

#### Modify Your Camera’s Name

By default, the script is based on enumeration for simplification purposes. It assumes that you know your camera(s)' IP addresses to identify which camera is which in a multi-camera setup. You are free to modify the script to fit your needs.

To do this, you’ll need to modify the shell command:  
`\[gcode\_shell\_command \<camera\_name\>\_spotlight\_high\_cmd\]`

The corresponding Per-camera Macro:  
`\[gcode\_macro CAM1\_SPOTLIGHT\_HIGH\]
description: CAM1 spotlight full brightness  
gcode:  
    RESPOND MSG="CAM1 Spotlight: HIGH"  
    RUN\_SHELL\_COMMAND CMD=\<camera\_name\>\_spotlight\_high\_cmd`

And in the Combined Macros section:  
`\[gcode\_macro SPOTLIGHT\_HIGH\]  
description: All spotlights full brightness  
gcode:  
    \<camera\_name\>\_SPOTLIGHT\_HIGH`

### Multiple Cameras

By default, the .cfg was configured for two cameras. Following the standard set in Cam 1 and Cam 2, one could easily add more cameras if necessary. Do this with caution, as you should understand what you’re doing and how to make the appropriate changes, since this is supplied without warranty.

### Single Camera

If you wish to run a single camera, you may want to modify the initial \[spotlight.cfg\] file. Use a file editor of your choice to modify the configuration and remove the Cam 2 settings. You’ll also want to modify the use of “cam1” in the script. Removing “cam1” will simplify the overall process when adding the functions either in Klipper as a macro, or when adding to your slicer’s start or end G-code. If removing the secondary camera from this script and modify the naming, you should also remove the Combined macros as well. This will avoid any confusion when implementing this script. 

## SSH Access

Since the Tingino firmware enables the Wyze camera as an IP camera, we’ll need to ensure that your Klipper host can communicate with the camera. For this, we’ll need to ensure that SSH is available from the host to call another device. While you’re likely already leveraging SSH to access your Klipper host. While your Klipper host can accept SSH calls, it can’t make them unless you enable it to do so by generating an SSH key. 

The following is a general setup of passwordless SSH from your Klipper host to the Thingino camera. Please read carefully and take the time to understand this section, as this involves secure communications of devices on your network. The following makes no claims to be the most secure method. This is simply the method decided when generating this guide.

The following is intended to be run on the host running Klipper:  
```shell  
ssh-keygen \-t rsa \-b 4096 \-C "klipper-spotlight" \-f \~/.ssh/id\_spotlight \-N ""  
```  
What this does:

* `ssh-keygen` \- OpenSSH authentication key utility  
* `\-t rsa \-b 4096` \- Specifies the type of key to create as "ed25519" (the default)  
* `\-C "klipper-spotlight"` \- Provides a custom comment or label, which is appended to the end of the key  
* `\-f \~/.ssh/id\_spotlight` \- Specifies the filename of the key file and path  
* `\-N ""` \- Provides a null passphrase  
    * A passphrase is a secondary form of authorization for your SSH key and is required when using it, if set. It is suggested that this be left blank, as the script does not account for the storage or use of secondary secrets such as this.

Once you have SSH set up, manually push the public key to the camera. Note that `ssh-copy-id` does not work reliably with Thingino's SSH server. Use the manual method below instead. You’ll also have to enter the camera’s root password as authentication. If you’re using multiple cameras, you’ll need to use this process for each one. Be sure to update the `<camera_ip_address>` below:

```shell  
cat \~/.ssh/id\_spotlight.pub | ssh root@\<camera\_ip\_address\> \\  
'mkdir \-p /root/.ssh && cat \>\> /root/.ssh/authorized\_keys && \\  
chmod 600 /root/.ssh/authorized\_keys && chmod 700 /root/.ssh'  
```  

Now, verify it works without being asked for a password  

```shell  
ssh \-i \~/.ssh/id\_spotlight \-o PasswordAuthentication=no root@\<camera\_ip\_address\> 'spotlight\_ctl high  
```  

If successful, the light will turn on. You can now turn it off by running  

```shell  
ssh \-i \~/.ssh/id\_spotlight \-o PasswordAuthentication=no root@\<camera\_ip\_address\> 'spotlight\_ctl off'  
```

## Modify Your Printer Config

Add the following line to your printer.cfg to include this file. Place this near the bottom for fast and easy identification. Just be sure to place it above the auto-generated `SAVE_CONFIG` data.

`[include spotlight.cfg]`

## Spotlight Commands

The table below shows the default camera names. Be sure to adjust the camera names in the commands as necessary. 

| Scope | HIGH | LOW | OFF |  
| ---: | :---: | :---: | :---: |  
| CAM-1 only | `CAM1_SPOTLIGHT_HIGH` | `CAM1_SPOTLIGHT_LOW` | `CAM1_SPOTLIGHT_OFF` |  
| CAM-2 only | `CAM2_SPOTLIGHT_HIGH` | `CAM2_SPOTLIGHT_LOW` | `CAM2_SPOTLIGHT_OFF` |  
| Both cameras | `SPOTLIGHT_HIGH` | `SPOTLIGHT_LOW` | `SPOTLIGHT_OFF` |

## Macros in Klipper

By adding the `[include spotlight.cfg]` in your printer’s config, the spotlight commands from above will automatically be added to Klipper’s macros. If you use macros regularly, be sure to re-review your macros. You’ll likely want to reorder them, allowing you to turn the spotlight/s on or off with ease. 

## Modify Your Slicer Settings

Aside from adding the macro to manually trigger the spotlight/s, you can also add the triggers to your slicer’s Start or End G-code. Slicer settings and configurations may vary. Instead of providing individual instructions per slicer, I will simply offer the suggestion to add the combined `SPOTLIGHT_HIGH` and `SPOTLIGHT_OFF` to your Start and Stop G-Code. 

## Notes

### Power Considerations 

Power draw is something that should be considered seriously. Putting more than one camera on a low-powered SBC like the BTT CB1 or a Raspberry Pi will cause undesired effects. While the SBC may or may not error with power warnings, the camera’s may exhibit power fluctuations in the spotlight causing a strobe effect. Powering your camera/s separately is suggested. 

You'll also want to be careful powering multiple cameras from a single USB power block. Ensure that any multiple USB power block used can support the max draw from both the cameras and the spotlights combined. 
