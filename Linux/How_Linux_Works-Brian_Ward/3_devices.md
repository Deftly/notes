# Devices
In this section we'll cover the kernel-provided device infrastructure in a functioning Linux system. The goal is to be about to extract information about the devices on a system in order to understand some rudimentary operations. Later sections will cover interacting with specific kinds of devices in more detail. 

## 3.1 Device Files
It's easy to manipulate most devices on a Unix system because the kernel presents many of the device I/O interfaces to user processes as files which are referred to as *device files* or *device nodes*. 

Device files are found in the */dev* directory. To get started working with devices consider the following command:
```Shell
$ echo blah blah > /dev/null
```

This sends stuff from the standard output to a file, but since */dev/null* is a device the kernel bypasses its usual file operations and uses a device driver on data written to this device. In the case of */dev/null* the kernel simply accepts the input data and throws it away.

Here are some example device files:
```Shell
$ ls -l /dev
brw-rw----.  1 root      disk      259,     0 May 19 08:02 nvme0n1
crw-rw-rw-.  1 root      root        1,     3 May 19 08:02 null
prw-r--r--.  1 root      root        0,     0 Mar 3  19:17 fdata
srw-rw-rw-.  1 root      root        0,     0 Dec 18 07:43 log
```

Note the first character of each line. If this character is `b`, `c`, `p`, or `s`, the file is a device. These letters stand for *block*, *character*, *pipe*, and *socket*.

#### Block device
Programs access data from a block device in fixed chunks. Disks, 


## 3.2 The sysfs Device Path

## 3.3 `dd` and Devices

## 3.4 Device Name Summary

### 3.4.1 Hard Disks: /dev/sd*

### 3.4.2 Virtual Disks: /dev/xvd*, /dev/vd*

### 3.4.3 Non-Volatile Memory Devices: /dev/nvme*

### 3.4.4 Device Mapper: /dev/dm-*, /dev/mapper/*

### 3.4.5 CD and DVD Drives: /dev/sr*

### 3.4.6 PATA Hard Disks: /dev/hd*

### 3.4.7 Terminals: /dev/tty*, /dev/pts/*, and /dev/tty

### 3.4.8 Serial Ports: /dev/ttyS*, /dev/ttyUSB*, /dev/ttyACM*

### 3.4.9 Parallel Ports: /dev/lp0 and /dev/lp1

### 3.4.10 Audio Devices: /dev/snd/*, /dev/dsp, /dev/audio, and More

### 3.4.11 Device File Creation

## 3.5 udev

### 3.5.1 devtmpfs

### 3.5.2 udevd Operation and Configuration

### 3.5.3 `udevadm`

### 3.5.4 Device Monitoring

## 3.6 In-Depth: SCSI and the Linux Kernel

### 3.6.1 USB Storage and SCSI

### 3.6.2 SCSI and ATA

### 3.6.3 Generic SCSI Devices

### 3.6.4 Multiple Access Methods for a Single Device

