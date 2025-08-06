# 📦 Learning Linux & Shell Scripting For DevOps - VirtualBox Environment

## Course Overview

**Target Audience:** Computer Science Students  
**Prerequisites:** Basic computer knowledge, Windows OS familiarity  
**Environment:** VirtualBox + Ubuntu 24.04.2 LTS  
**Duration:** 40+ hours of hands-on learning  

This comprehensive course uses a **local VirtualBox environment** to provide full control over a Linux system. Perfect for students who want persistent storage, offline access, and advanced system administration capabilities.

---

## 🖥️ Environment Setup - VirtualBox + Ubuntu

### Why Choose VirtualBox Environment?

✅ **Full Control** - Complete system administration access  
✅ **Offline Learning** - Work without internet once set up  
✅ **Persistent Storage** - All work saved locally  
✅ **Professional Simulation** - Mimics real-world server environments  
✅ **Advanced Features** - Access to kernel modules, hardware passthrough  

### Minimum System Requirements
- **CPU:** Dual-core 2.0 GHz or higher (quad-core recommended)  
- **RAM:** 8 GB minimum (16 GB recommended)  
- **Disk Space:** 80 GB free  
- **OS:** Windows 10/11, macOS, or Linux host  

---

## 1️⃣ Download and Install VirtualBox

1. Open your browser and navigate to: https://www.virtualbox.org/wiki/Downloads  
2. Under "VirtualBox X.Y.Z platform packages", click **Windows hosts** (or macOS/Linux hosts)  
3. Download the latest version (7.1.10 as of this guide)  
4. Run the installer as Administrator  
5. Accept default settings and proceed through the wizard  
6. Click **Finish** to complete installation  

**Verify Installation:**
```bash
# Start VirtualBox from Start Menu (Windows) or Applications (macOS)
# You should see the "Oracle VM VirtualBox Manager" window
```

---

## 2️⃣ Download Ubuntu 24.04.2 LTS ISO

1. Visit: https://ubuntu.com/download/desktop  
2. Click **Download Ubuntu 24.04.2 LTS** (approx. 5.7 GB)  
3. Save the ISO file to a known location (e.g., *Downloads*)  

> **Tip:** Prefer the Desktop version for GUI; Server edition is available for advanced users.

---

## 3️⃣ Create a New Virtual Machine

1. In VirtualBox Manager, click **New**  
2. **Name:** `Ubuntu-DevOps-Lab`  
3. **Machine Folder:** Leave default or choose custom path  
4. **Type:** Linux  
5. **Version:** Ubuntu (64-bit)  
6. Click **Next**  

**Memory Size:**
- Allocate **4 GB (4096 MB)** minimum  
- Allocate **8 GB (8192 MB)** if you have 16 GB RAM  

**Hard Disk:**
1. Select **Create a virtual hard disk now** → **Create**  
2. **VDI (VirtualBox Disk Image)** → **Next**  
3. **Dynamically allocated** → **Next**  
4. **Size:** **80 GB** (50 GB minimum) → **Create**  

---

## 4️⃣ Configure VM Settings

1. Select `Ubuntu-DevOps-Lab` → **Settings**  
2. **System → Processor:** Set **2–4 CPUs** (green bar)  
3. **Display → Screen:** Video Memory **128 MB**  
4. **Storage:**  
   - Under *Controller: IDE*, click **Empty**  
   - Click the disc icon → **Choose a disk file**  
   - Select the Ubuntu ISO you downloaded  
5. **Network:** Ensure **Adapter 1** is **NAT** for internet access  
6. Click **OK** to save  

---

## 5️⃣ Install Ubuntu

1. Select VM → **Start**  
2. When Ubuntu boots, select **Install Ubuntu**  
3. Choose language and keyboard layout  
4. **Installation Type:** *Normal installation* + *Download updates*  
5. **Erase disk and install Ubuntu** (affects only virtual disk)  
6. **User Setup:**  
   - **Name:** DevOps Student  
   - **Username:** student  
   - **Password:** ●●●●●●●●  
   - Enable **Log in automatically** (optional)  
7. Click **Continue** → Wait 15–30 minutes  
8. Click **Restart Now** when prompted  
9. Remove ISO if asked → Press **Enter**  

---

## 6️⃣ Post-Installation Optimization

### 6.1 Install Guest Additions

1. VM menu → **Devices → Insert Guest Additions CD image**  
2. Run installer when prompted:  
```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
cd /media/$USER/VBox_GAs_*/
sudo ./VBoxLinuxAdditions.run
```  
3. Reboot VM  

### 6.2 Update System & Install Tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git tree vim nano htop neofetch
```

### 6.3 Create Course Directory Structure

```bash
mkdir -p ~/linux-devops-lab/{scripts,exercises,projects,backups,logs,config}
cd ~/linux-devops-lab
```

### 6.4 Configure Bash Environment

```bash
cat >> ~/.bashrc << 'EOF'
# DevOps Course Aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias cls='clear'

# Course directory shortcut
export COURSE_HOME="$HOME/linux-devops-lab"
alias course='cd $COURSE_HOME'

export PS1='\[\033[01;32m\]devops@vm\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
EOF

source ~/.bashrc
```

---

## 7️⃣ Verify Environment

```bash
# System information
neofetch

# Check tools
git --version
curl --version

# Test script execution
echo -e '#!/bin/bash\necho "VirtualBox environment ready!"' > test.sh
chmod +x test.sh
./test.sh
```

Expected output: `VirtualBox environment ready!`

---

## 🚀 Ready for the Course

You now have a fully configured **Ubuntu 24.04.2 LTS VM** running inside VirtualBox. Proceed to:

1. **Module 2:** Linux Fundamentals  
2. **Module 3:** Essential Commands  
3. **Module 4:** Shell Scripting Basics  
4. Continue through all 7 modules  

All course content assumes your working directory is:
```bash
cd ~/linux-devops-lab
```

---

## 🛠️ Troubleshooting Tips

| Issue | Possible Fix |
|-------|--------------|
| *VT-x/AMD-V not available* | Enable virtualization in BIOS/UEFI |
| *Black screen after boot* | Increase video memory to 128 MB; enable 3D acceleration |
| *Network not working* | Switch **Adapter 1** to **Bridged Adapter** |
| *Low performance* | Allocate more RAM/CPUs in **Settings → System** |
| *Screen resolution stuck* | Reinstall Guest Additions; reboot VM |

---

## 🎓 Next Steps

- **Snapshot your VM** before starting labs (`VM → Take Snapshot`)  
- **Backup VM folder** regularly  
- **Sync scripts to GitHub** from within VM  
- Explore **shared folders** for host ↔ guest file transfer  

---

## 🏁 You're All Set!

Your VirtualBox environment is ready. Dive into the **Linux & Shell Scripting for DevOps** course and start building your automation skills.

Happy Learning & Automating! 🚀