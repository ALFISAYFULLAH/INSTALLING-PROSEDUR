````markdown
# 🛠️ Analog VLSI Design Flow (Sky130 PDK) – Installation Guide

Repository ini berisi panduan **lengkap dan terstruktur** untuk instalasi serta konfigurasi *open-source Analog VLSI Design Tools* yang terintegrasi dengan **SkyWater 130nm PDK**.

Panduan ini cocok untuk:
- Mahasiswa & peneliti IC design
- Pemula Analog VLSI
- Pengguna **Xschem + Ngspice + Magic**
- Sistem operasi **Ubuntu Linux**

---

## 📚 Daftar Isi
1. [Prasyarat Sistem](#1-prasyarat-sistem)
2. [Tools & Fungsinya](#2-tools--fungsinya)
3. [Instalasi Dependensi Sistem](#3-instalasi-dependensi-sistem)
4. [Instalasi EDA Tools](#4-instalasi-eda-tools)
5. [Setup Sky130 PDK (OpenLane)](#5-setup-sky130-pdk-openlane)
6. [Integrasi Tools dengan Sky130](#6-integrasi-tools-dengan-sky130)
7. [Menjalankan Contoh Desain](#7-menjalankan-contoh-desain)
8. [Hasil & Visualisasi](#8-hasil--visualisasi)
9. [Catatan & Tips](#9-catatan--tips)

---

## 1. Prasyarat Sistem

- **OS**: Ubuntu 20.04 / 22.04 (direkomendasikan)
- **Disk Space**: ≥ 10 GB
- **RAM**: ≥ 8 GB
- **Internet**: Wajib

📷 *Contoh environment Ubuntu*  
![Ubuntu Environment](./images/ubuntu.png)

---

## 2. Tools & Fungsinya

| Tool | Fungsi |
|-----|-------|
| **Xschem** | Schematic capture & netlist |
| **Ngspice** | Simulator SPICE |
| **GAW** | Waveform viewer |
| **Magic** | Layout editor VLSI |
| **Netgen** | LVS (Layout vs Schematic) |
| **KLayout** | Layout viewer (opsional) |

📷 *Analog IC Design Flow*  
![Analog Flow](./images/flow.png)

---

## 3. Instalasi Dependensi Sistem

### 3.1 Update Sistem

```bash
sudo apt update
sudo apt install -y build-essential gcc make perl git
````

---

### 3.2 Library Pendukung EDA

```bash
sudo apt install -y \
m4 tcsh csh \
libx11-dev libx11-xcb-dev \
tcl-dev tcllib tk-dev \
libcairo2-dev mesa-common-dev libglu1-mesa-dev \
libncurses-dev clang \
bison flex libreadline-dev gawk \
libffi-dev graphviz xdot pkg-config \
python3 libboost-system-dev \
adms autoconf automake libtool \
libxpm-dev libxaw7-dev libssl-dev \
libgtk-3-dev libboost-python-dev \
libboost-filesystem-dev zlib1g-dev \
xterm graphicsmagick ghostscript vim
```
---

## 4. Instalasi EDA Tools

```bash
mkdir -p ~/open_source/tools
cd ~/open_source/tools
```
---

### 4.1 Xschem (Schematic Editor)

```bash
git clone https://github.com/StefanSchippers/xschem.git
cd xschem
./configure
make -j$(nproc)
sudo make install
```

📷 *Xschem berjalan*
![Xschem](images/xschem-1.png)

---

### 4.2 Ngspice (Simulator)

```bash
git clone https://git.code.sf.net/p/ngspice/ngspice ngspice_git
cd ngspice_git
./autogen.sh --adms
mkdir release && cd release
./configure \
  --with-x \
  --enable-xspice \
  --enable-cider \
  --enable-openmp \
  --with-readline=yes \
  --disable-debug
make -j$(nproc)
sudo make install
```

📷 *Ngspice CLI*
![Ngspice](images/07_ngspice.png)

---

### 4.3 GAW (Waveform Viewer)

```bash
git clone https://github.com/StefanSchippers/xschem-gaw
cd xschem-gaw
aclocal
automake --add-missing
autoconf
./configure
make -j$(nproc)
sudo make install
```

📷 *GAW waveform*
![GAW](images/08_gaw.png)

---

### 4.4 Magic & Netgen

#### Magic

```bash
git clone https://github.com/RTimothyEdwards/magic
cd magic
./configure
make -j$(nproc)
sudo make install
```

📷 *Magic layout editor*
![Magic](images/09_magic.png)

---

#### Netgen

```bash
git clone https://github.com/RTimothyEdwards/netgen
cd netgen
./configure
make -j$(nproc)
sudo make install
```

📷 *Netgen LVS*
![Netgen](images/10_netgen.png)

---

## 5. Setup Sky130 PDK (OpenLane)

```bash
cd ~
git clone https://github.com/The-OpenROAD-Project/OpenLane
cd OpenLane
make
make test
```

📷 *OpenLane setup sukses*
![OpenLane](images/11_openlane.png)

---

## 6. Integrasi Tools dengan Sky130

```bash
mkdir -p ~/design/{xschem,magic,netgen}
```

📷 *Struktur folder design*
![Design Folder](images/12_design_folder.png)

---

### Xschem & Ngspice

```bash
cd ~/design/xschem
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/xschem/xschemrc
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/ngspice/spinit .spiceinit
```

📷 *Xschem terhubung Sky130*
![Xschem Sky130](images/13_xschem_sky130.png)

---

### Magic & Netgen

```bash
cd ~/design/magic
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/magic/sky130A.magicrc .magicrc

cd ~/design/netgen
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/*/sky130A/libs.tech/netgen/setup.tcl setup.tcl
```

📷 *Magic + Sky130*
![Magic Sky130](images/14_magic_sky130.png)

---

## 7. Menjalankan Contoh Desain

```bash
unzip cnt_down4.zip
cp -r cnt_down4 ~/OpenLane/designs/
cd ~/OpenLane
make mount
./flow.tcl -design cnt_down4
```

📷 *OpenLane flow running*
![Flow Running](images/15_flow_running.png)

---

## 8. Hasil & Visualisasi

📷 *Layout hasil OpenLane (hidden)*
![Hidden Layout](images/16_layout_hidden.png)

📷 *Layout setelah tekan `x`*
![Final Layout](images/17_final_layout.png)

---

## 9. Catatan & Tips

* Layout di Magic **default tersembunyi**
* Blok semua area → tekan **`x`**
* Build lama = normal
* Sky130 PDK **besar & berat**

---

## 🎉 Selesai

Environment **Analog VLSI Design berbasis Sky130** siap digunakan.

**GG 🚀**

```