android-cluster-toolkit
=======================

The Android Cluster Toolkit helps organize and manipulate a collection of Android devices. It was designed to work with a collection of devices connected to the same host machine, either directly or via one or more tiers of powered USB hubs. The tools within can operate on single devices, a selected subset, or all connected devices at once.

## Requirements

 - A Ruby Interpreter (Tested with Ruby 1.9.3 and 2.0.0)
 - A Linux Host machine (YMMV on other OSes)
 - One or more Android devices
 - Open USB ports (one per device)

## Hardware Components

There are no special hardware components, though various hardware has been used with varying results.

### Good Hardware
D-Link DUB-H7 hubs seem to work well. 

### Bad Hardware
Although it looks cool and has a lot of ports, the Manhattan MondoHub is not recommended. It simply doesn't have enough power to support all ports being in use at the same time.

## Software Components

Several scripts comprise the Android Cluster Toolkit. These scripts are described below.

### Library Components

The toolkit contains three scripts that are included by the other scripts. These scripts each serve a particular purpose.

#### devices-orig.rb

This file contains a Ruby Hash that describes various properties of the devices within your Android cluster. A sample file is included as "devices-orig.rb.sample". This file is never changed by the toolkit, so this is a great place to put manual comments and so forth.

#### devices.rb

The devices.rb script is generated from the devices-orig.rb script. It contains the list of devices that are currently connected to the host machine (the ADB Server). It is created by the reconfig.rb script.

#### madb.rb

This file contains functionality that is shared between several of the scripts in the Android Cluster Toolkit. It is included by those scripts and currently only contains a few utility method implementations.

### Management Scripts

Dealing with a large collection of Android devices isn't painless. A couple of scripts in the toolkit ease the pain. These scripts take no arguments and just do their thing when you run them.

#### scan.rb

The scan.rb script exists to make adding new devices easier. Once you obtain a new device, plug it in and run the scan.rb script. If it finds any devices that are not in devices-orig.rb it will output suitable Hash entries for them. Add these entries to devices-orig.rb to make these devices accessible to the multi-device scripts. Make sure  to set a unique name for the device (perhaps the model number or slang name, ie. GT-I9505 or SGS4).

#### reconfig.rb

The reconfig.rb script automatically generates the Hash in devices.rb. When executed, it copies the entries in devices-orig.rb that are currently connected to the ADB Server. This way, you can avoid silly "no such device" output from the multi-device scripts.

### Single Device Scripts

Although there used to be more single device scripts, they were deprecated in favor of a more unified interface (described in the next section). That said, one single-device script remains.

#### shell.rb <device|serial>

This script executes a shell suitable for operating on the specified device. This is useful for maintenance or doing more focused research on a single device. When executed, it simply sets up the environment so that other ADB-aware tools will operate on the specified device. Once the environment is set up, the script spawns a shell for your pleasure. Exit the new shell to return to the original shell.

### Multi-device scripts

The Android Cluster Toolkit's best feature is its multi-device scripts. These scripts can be used to perform some action against one, several, or all of the devices in your Android cluster at once. The first argument to these scripts specifies which device(s) to act upon as follows:

1. "." (a literal period, without the quotes): Act upon all devices
2. One or more device names or serial numbers separated by commas (quote your spaces if necessary): Act upon only the selected devices.

The arguments that follow the device selector are specific to each script. In some cases they are option, but in other cases they are not.

#### mdo <device selector> <adb args>

This multi-device script enables you to run an arbitrary ADB command against the selected device(s). Any command supported by the ADB client should work (ie, push, shell, reboot, etc).

#### mcmd <device selector> <command and args>

This multi-device script enables you to run an arbitrary shell command on the selected device(s). For example:

<code>./mcmd . getprop ro.build.fingerprint</code>

This will list the build fingerprint of all connected devices.

#### mbb <device selector> <command and args>

During research, the need arose to use tools from the BusyBox binary in lieu of the default versions. For example, the ls(1) binary inside BusyBox offers color output, among other features whereas the default one (usually from toolbox) does not. This script accomplishes this goal by prefixing the specified command with "/data/local/tmp/busybox". Placing a suitable busybox binary in this location is up to you.

#### mpull <device selector> <path to pull>

Another common task is pulling a particular file or path from all devices so that the data can be inspected on the host machine. This script will pull the specified paths from all devices and store them into a device-specific sub-directory within the "devices" directory in the Android Cluster Toolkit directory. For example:

<code>./mpull sgs4 /proc/version</code>

This command will pull the kernel version information from /proc/version and save it to devices/sgs4/proc/version.

## Getting Set Up

There are only a few steps to getting up and running.

1. Clone this repository
2. Go into the directory
3. Connect all of the desired devices
4. Run the ./scan.rb script
5. Add the Hash entries to devices-orig.rb
6. Run ./reconfig.rb

The relevant commands are:

```
$ git clone https://github.com/jduck/android-cluster-toolkit.git
$ cd android-cluster-toolkit
$ ./scan.rb
$ $EDITOR devices-orig.rb  # set names for the devices and save
$ ./reconfig.rb
```

At this point you're ready to go. To confirm everything is working, run something against all the devices.

<code>./mcmd . getprop ro.build.fingerprint</code>