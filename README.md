# QEMU VM Setup Guide

This guide will help you set up and run virtual machines using QEMU at the command line, since we can't use virt-manager at school.

## Prerequisites

### On Your Host Machine
- Install SPICE viewer (for GUI): `sudo apt install virt-viewer`

### Inside Your VM (for GUI setup)
- Install SPICE guest agent: `sudo apt install spice-vdagent`
- This enables clipboard sharing and automatic window resizing

## Quick Start: Choose Your Path

**Do you already have a VirtualBox VM?**
- **YES** → Go to [Converting from VirtualBox](#converting-from-virtualbox)
- **NO** → Go to [Installing from ISO](#installing-from-iso-fresh-install)

**After converting or installing, choose:**
- **Headless (SSH only)** → [Minimal Setup](#minimal-setup-headless-ssh-only)
- **Desktop with GUI** → [Full Desktop Setup](#full-desktop-setup-gui--audio)

---

## Converting from VirtualBox

If you have an existing VirtualBox VM, convert it to qcow2 format:

```bash
# Convert VDI to qcow2
qemu-img convert -f vdi -O qcow2 your-vm.vdi ubuntu.qcow2

# Or convert VMDK to qcow2
qemu-img convert -f vmdk -O qcow2 your-vm.vmdk ubuntu.qcow2
```

After conversion, skip to either [Minimal Setup](#minimal-setup-headless-ssh-only) or [Full Desktop Setup](#full-desktop-setup-gui--audio).

---

## Installing from ISO (Fresh Install)

### Step 1: Create a disk image
```bash
qemu-img create -f qcow2 ubuntu.qcow2 40G
```

### Step 2: Boot from ISO to install
```bash
#!/usr/bin/env bash
qemu-system-x86_64 \
  -enable-kvm \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -drive file=ubuntu.qcow2,if=virtio,cache=writeback \
  -cdrom ubuntu-24.04-desktop-amd64.iso \
  -boot d \
  -vga virtio \
  -netdev user,id=net0 \
  -device virtio-net-pci,netdev=net0
```

**What this does:**
- `-cdrom ubuntu-24.04-desktop-amd64.iso` - mounts your ISO file
- `-boot d` - boots from CD/DVD drive (the ISO)
- Uses simple VGA for installation

### Step 3: Complete the installation
Install your OS in the window that appears.

### Step 4: After installation
Shut down the VM completely. Now choose either [Minimal Setup](#minimal-setup-headless-ssh-only) or [Full Desktop Setup](#full-desktop-setup-gui--audio) below for your regular use.

---

## Minimal Setup (Headless, SSH Only)

Perfect for servers, development environments, or if you only need terminal access.

### Create your launch script: `start-vm-minimal.sh`
```bash
#!/usr/bin/env bash
qemu-system-x86_64 \
  -enable-kvm \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -drive file=ubuntu.qcow2,if=virtio,cache=writeback \
  -nographic \
  -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::8443-:443 \
  -device virtio-net-pci,netdev=net0
```

### Make it executable and run
```bash
chmod +x start-vm-minimal.sh
./start-vm-minimal.sh
```

### Connect via SSH
```bash
ssh -p 2222 username@localhost
```

---

## Full Desktop Setup (GUI + Audio)

Complete desktop experience with graphics and sound.

### Create your launch script: `start-vm-full.sh`
```bash
#!/usr/bin/env bash
qemu-system-x86_64 \
  -enable-kvm \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 4 \
  -m 8G \
  -drive file=ubuntu.qcow2,if=virtio,cache=writeback \
  -vga none \
  -device virtio-gpu-pci,max_outputs=1,edid=on \
  -display none \
  -spice port=5931,disable-ticketing=on \
  -device virtio-serial \
  -chardev spicevmc,id=vdagent,name=vdagent \
  -device virtserialport,chardev=vdagent,name=com.redhat.spice.0 \
  -netdev user,id=net0,hostfwd=tcp::8443-:443,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0 \
  -device ich9-intel-hda \
  -device hda-duplex
```

### Make it executable and run
```bash
chmod +x start-vm-full.sh
./start-vm-full.sh
```

### Connect to the display
In another terminal:
```bash
remote-viewer spice://localhost:5931
```

---

## Tips

- Press `Ctrl+Alt+G` in the SPICE viewer to release mouse/keyboard capture.

---

Happy virtualizing! If you have questions or improvements, please share them.
