# 🛠️ Analog VLSI Design Flow (Sky130 PDK) – Installation Guide

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04%20|%2022.04-orange.svg)
![Sky130](https://img.shields.io/badge/PDK-SkyWater%20130nm-brightgreen.svg)
![Sky130](images/diices.png)

Panduan **lengkap dan terstruktur** untuk instalasi serta konfigurasi *open-source Analog VLSI Design Tools* yang terintegrasi dengan **SkyWater 130nm PDK**.

</div>

---

# 🎯 Target Pengguna

- 🎓 Mahasiswa & peneliti IC design
- 🔰 Pemula Analog VLSI
- 💻 Pengguna **Xschem + Ngspice + Magic**
- 🐧 Sistem operasi **Ubuntu Linux**

---

# 📚 Daftar Isi

1. [Prasyarat Sistem](#1-prasyarat-sistem)
2. [Tools & Fungsinya](#2-tools--fungsinya)
3. [Instalasi Docker](#3-instalasi-docker)
4. [Instalasi Dependensi Sistem](#4-instalasi-dependensi-sistem)
5. [Instalasi EDA Tools](#5-instalasi-eda-tools)
6. [Setup Sky130 PDK (OpenLane)](#6-setup-sky130-pdk-openlane)
7. [Konfigurasi Environment Variables](#7-konfigurasi-environment-variables)
8. [Integrasi Tools dengan Sky130](#8-integrasi-tools-dengan-sky130)
9. [Menjalankan Contoh Desain](#9-menjalankan-contoh-desain)
10. [Hasil & Visualisasi](#10-hasil--visualisasi)
11. [Catatan & Tips](#11-catatan--tips)

---

# 1. Prasyarat Sistem

| Requirement | Spesifikasi |
|------------|-------------|
| **OS** | Ubuntu 20.04 / 22.04 (direkomendasikan) |
| **Disk Space** | ≥ 10 GB |
| **RAM** | ≥ 8 GB |
| **Internet** | Wajib |
| **Processor** | Multi-core (direkomendasikan) |

> 📷 *Contoh environment Ubuntu*  
> ![Ubuntu Environment](images/ubuntu.png)

---

# 2. Tools & Fungsinya

| Tool | Fungsi | Status |
|------|--------|--------|
| **Xschem** | Schematic capture & netlist | ✅ Core |
| **Ngspice** | Simulator SPICE | ✅ Core |
| **GAW** | Waveform viewer | ✅ Core |
| **Magic** | Layout editor VLSI | ✅ Core |
| **Netgen** | LVS (Layout vs Schematic) | ✅ Core |
| **Docker** | Container platform | ✅ Core |
| **KLayout** | Layout viewer | 🔶 Optional |

> 📷 *Analog IC Design Flow*  
> ![Analog Flow](images/flow.png)

---

# 3. Instalasi Docker

## 3.1 Hapus Versi Lama (Jika Ada)

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

## 3.2 Setup Repository Docker

```bash
# Install prerequisites
sudo apt update
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Setup repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## 3.3 Install Docker Engine

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 3.4 Verifikasi Instalasi

```bash
sudo docker run hello-world
```

## 3.5 Tambahkan User ke Docker Group (Optional)

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

> ⚠️ **Logout dan login kembali** agar perubahan group berlaku

## 3.6 Enable Docker di Startup

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

# 4. Instalasi Dependensi Sistem

## 4.1 Update Sistem

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y build-essential gcc make perl git
```

Git membutuhkan identitas untuk setiap commit.

```bash
git config --global user.name "Nama Kamu"
git config --global user.email "email@domain.com"
```

Cek konfigurasi:

```bash
git config --global --list
```

## 4.2 Library Pendukung EDA

```bash
sudo apt install -y \
    m4 tcsh csh \
    libx11-dev libx11-xcb-dev \
    tcl-dev tcllib tk-dev \
    libcairo2-dev mesa-common-dev libglu1-mesa-dev \
    libncurses-dev clang \
    bison flex libreadline-dev gawk \
    libffi-dev graphviz xdot pkg-config \
    python3 python3-pip libboost-system-dev \
    adms autoconf automake libtool \
    libxpm-dev libxaw7-dev libssl-dev \
    libgtk-3-dev libboost-python-dev \
    libboost-filesystem-dev zlib1g-dev \
    xterm graphicsmagick ghostscript vim
```

---

# 5. Instalasi EDA Tools

## 5.1 Persiapan Directory

```bash
mkdir -p ~/open_source/tools
cd ~/open_source/tools
```

## 5.2 Xschem (Schematic Editor)

```bash
git clone https://github.com/StefanSchippers/xschem.git
cd xschem
./configure
make -j$(nproc)
sudo make install
cd ..
```

> 📷 *Xschem berjalan*  
> ![Xschem](images/xschem-1.png)

## 5.3 Ngspice (Simulator)

```bash
git clone https://git.code.sf.net/p/ngspice/ngspice ngspice_git
cd ngspice_git
./autogen.sh --adms
mkdir release && cd release
../configure \
    --with-x \
    --enable-xspice \
    --enable-cider \
    --enable-openmp \
    --with-readline=yes \
    --disable-debug
make -j$(nproc)
sudo make install
cd ../..
```

> 📷 *Ngspice CLI*  
> ![Ngspice](images/ngspice-1.png)

## 5.4 GAW (Waveform Viewer)

```bash
git clone https://github.com/StefanSchippers/xschem-gaw
cd xschem-gaw
aclocal
automake --add-missing
autoconf
./configure
make -j$(nproc)
sudo make install
cd ..
```

> 📷 *GAW waveform*  
> ![GAW](images/gaw-1.png)

## 5.5 Magic (Layout Editor)

```bash
git clone https://github.com/RTimothyEdwards/magic
cd magic
./configure
make -j$(nproc)
sudo make install
cd ..
```

> 📷 *Magic layout editor*  
> ![Magic](images/magic-1.png)

## 5.6 Netgen (LVS Tool)

```bash
git clone https://github.com/RTimothyEdwards/netgen
cd netgen
./configure
make -j$(nproc)
sudo make install
cd ..
```

> 📷 *Netgen LVS*  
> ![Netgen](images/netgen-1.png)

---

# 6. Setup Sky130 PDK (OpenLane)

```bash
cd ~
git clone https://github.com/The-OpenROAD-Project/OpenLane
cd OpenLane
make pdk
make test
```

> ⏳ **Catatan**: Proses ini membutuhkan waktu lama dan koneksi internet stabil

---

# 7. Konfigurasi Environment Variables

## 7.1 Edit `.bashrc`

```bash
nano ~/.bashrc
```

## 7.2 Tambahkan di Akhir File

```bash
# ============================================
# Analog VLSI Design Environment Variables
# ============================================

# PDK Root Directory
export PDK_ROOT=/opt/pdk/share/pdk

# EDA Library Path
export LD_LIBRARY_PATH=/opt/eda/lib

# EDA Binary Path
export PATH=/opt/eda/bin:$HOME/.local/bin:$PATH

# ============================================
```

## 7.3 Reload Configuration

```bash
source ~/.bashrc
```

## 7.4 Verifikasi Environment Variables

```bash
echo $PDK_ROOT
echo $LD_LIBRARY_PATH
echo $PATH
```

> ✅ **Expected output**: Path yang sudah dikonfigurasi harus muncul

---

# 8. Integrasi Tools dengan Sky130

## 8.1 Setup Directory Desain

```bash
mkdir -p ~/design/{xschem,magic,netgen}
```

## 8.2 Xschem & Ngspice

```bash
cd ~/design/xschem
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/xschem/xschemrc 
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/ngspice/spinit .spiceinit
```

> 📷 *Xschem terhubung Sky130*  
> ![Xschem Sky130](images/xschem-2.png)

## 8.3 Magic

```bash
cd ~/design/magic
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/magic/sky130A.magicrc .magicrc
```

## 8.4 Netgen

```bash
cd ~/design/netgen
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/netgen/setup.tcl setup.tcl
```

> 📷 *Magic + Sky130*  
> ![Magic Sky130](images/magic-2.png)

---

# 9. Menjalankan Contoh Desain

## 9.1 Extract dan Copy Desain

```bash
unzip cnt_down4.zip
cp -r cnt_down4 ~/OpenLane/designs/
```

## 9.2 Jalankan OpenLane Flow

```bash
cd ~/OpenLane
make mount
./flow.tcl -design cnt_down4
```

> ⏳ **Build time**: Proses ini normal memakan waktu lama

---

# 10. Hasil & Visualisasi

## 10.1 Buka Layout di Magic

```bash
magic ~/OpenLane/designs/cnt_down4/runs/RUN_2025.12.31_12.01.18/results/final/mag/cnt_down4.mag
```

## 10.2 Visualisasi Layout

1. Di Magic window, **blok semua area**
2. Tekan **`x`** untuk menampilkan layout

> 📷 *Layout setelah tekan `x`*  
> ![Final Layout](images/layout.png)

---

# 11. Catatan & Tips

## 💡 Tips Umum

- ⚠️ Layout di Magic **default tersembunyi**
- 🖱️ Blok semua area → tekan **`x`** untuk menampilkan
- ⏰ Build lama adalah **normal**
- 💾 Sky130 PDK **besar & memakan storage**

## 🔧 Troubleshooting

| Masalah | Solusi |
|---------|--------|
| Docker permission denied | Tambahkan user ke docker group |
| Library not found | Cek `LD_LIBRARY_PATH` |
| PDK tidak terdeteksi | Verifikasi `PDK_ROOT` |
| Build error | Pastikan semua dependensi terinstall |

## 📖 Resources Tambahan

- [SkyWater PDK Documentation](https://skywater-pdk.readthedocs.io/)
- [OpenLane Documentation](https://openlane.readthedocs.io/)
- [Xschem Tutorial](http://repo.hu/projects/xschem/)
- [Magic VLSI](http://opencircuitdesign.com/magic/)

---

# 🎉 SELESAI!

Environment **Analog VLSI Design berbasis Sky130** siap digunakan untuk:

- ✅ Schematic design dengan Xschem
- ✅ Circuit simulation dengan Ngspice
- ✅ Layout design dengan Magic
- ✅ LVS verification dengan Netgen
- ✅ Full RTL-to-GDSII flow dengan OpenLane

**Happy Designing! 🚀**

---

<div align="center">

Made with ❤️ for Open Source IC Design Community

</div>