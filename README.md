```markdown
# 🛠️ Panduan Instalasi Analog VLSI Design Tools (Sky130 PDK)

Repository ini menyediakan panduan lengkap untuk melakukan instalasi dan konfigurasi *tools* desain analog berbasis open-source yang terintegrasi dengan **Skywater 130nm PDK**. Panduan ini ditujukan untuk sistem operasi **Linux (Ubuntu)**.

## 📋 Daftar Isi
1. [Prasyarat Sistem](#1-prasyarat-sistem)
2. [Daftar Tools & Kegunaan](#2-daftar-tools--kegunaan)
3. [Instalasi Dependensi](#3-instalasi-dependensi)
4. [Instalasi EDA Tools](#4-instalasi-eda-tools)
5. [Setup PDK (Sky130)](#5-setup-pdk-sky130)
6. [Integrasi & Konfigurasi Proyek](#6-integrasi--konfigurasi-proyek)

---

## 1. Prasyarat Sistem
* **OS**: Linux Ubuntu (Versi apa pun).
* **Disk Space**: Beberapa GB (PDK membutuhkan ruang yang cukup besar).
* **Koneksi Internet**: Dibutuhkan untuk melakukan *cloning* dari GitHub dan mengunduh paket.

---

## 2. Daftar Tools & Kegunaan
Berikut adalah ekosistem alat yang akan kita gunakan dalam alur desain analog:

| Tool | Fungsi |
| :--- | :--- |
| **Xschem** | Program *schematic capture* grafis untuk desain sirkuit. |
| **Ngspice** | Simulator SPICE open-source untuk simulasi elektronik. |
| **Gaw** | Penampil bentuk gelombang (*waveform viewer*) hasil simulasi. |
| **Magic** | Alat desain tata letak (*layout*) VLSI. |
| **Netgen** | Alat untuk perbandingan Netlist atau *Layout vs Schematic* (LVS). |
| **Klayout** | Alat desain dan penampil tata letak (opsional). |

---

## 3. Instalasi Dependensi
Sebelum mengompilasi *tools*, Anda wajib menginstal berbagai pustaka (*libraries*) pendukung agar proses instalasi berjalan lancar.

### 3.1 Update Sistem
Jalankan perintah berikut untuk memperbarui paket Ubuntu dan menginstal Git:
```bash
sudo apt-get update
sudo apt-get install build-essential gcc make perl dkms git -y

```

*Pastikan Git sudah terinstal dengan menjalankan `git --version*`.

### 3.2 Install Library EDA

Salin dan jalankan perintah panjang di bawah ini untuk menginstal semua kebutuhan dasar EDA tools. Proses ini mungkin memakan waktu.

```bash
sudo apt-get install m4 tcsh csh libx11-dev libx11-xcb-dev tcl-dev tcllib swig vim-gtk tk-dev libcairo2-dev mesa-common-dev libglu1-mesa-dev libncurses-dev build-essential clang bison flex libreadline-dev gawk libffi-dev git graphviz xdot pkg-config python3 libboost-system-dev vim adms autoconf automake libtool libxpm-dev libxaw7-dev libssl-dev libgtk-3-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev xterm graphicsmagick ghostscript --assume-yes

```

---

## 4. Instalasi EDA Tools

Kita akan membuat struktur direktori khusus untuk instalasi ini.

```bash
# Membuat direktori kerja di Home
mkdir -p ~/open_source/tools
cd ~/open_source/tools

```

### 4.1 Xschem

*Xschem adalah program schematic capture untuk memasukkan sirkuit elektronik secara interaktif*.

```bash
git clone [https://github.com/StefanSchippers/xschem.git](https://github.com/StefanSchippers/xschem.git)
cd xschem
./configure
make
sudo make install
# Verifikasi dengan mengetik: xschem
cd ..

```

### 4.2 Ngspice

*Ngspice adalah simulator SPICE open-source. Ini adalah alat berbasis Command Line Interface (CLI)*.

Ngspice akan diinstal dengan dukungan Xspice dan ADMS.

```bash
git clone [https://git.code.sf.net/p/ngspice/ngspice](https://git.code.sf.net/p/ngspice/ngspice) ngspice_git
cd ngspice_git
./autogen.sh --adms
mkdir release && cd release
./configure --with-x --enable-xspice --disable-debug --enable-cider --with-readline=yes --enable-openmp
make -j4
sudo make install
# Verifikasi dengan mengetik: ngspice
cd ../..

```

### 4.3 Gaw (Waveform Viewer)

*GAW digunakan untuk menampilkan bentuk gelombang output dari simulasi xschem*.

```bash
git clone [https://github.com/StefanSchippers/xschem-gaw](https://github.com/StefanSchippers/xschem-gaw)
cd xschem-gaw
aclocal && automake --add-missing && autoconf
./configure
make -j4
sudo make install
# Verifikasi dengan mengetik: gaw
cd ..

```

### 4.4 Magic & Netgen

#### Magic (Layout Tool)

*Magic adalah alat desain tata letak (layout) VLSI*.

```bash
git clone [https://github.com/RTimothyEdwards/magic](https://github.com/RTimothyEdwards/magic)
cd magic && ./configure && make && sudo make install
# Verifikasi dengan mengetik: magic &
cd ..

```

#### Netgen (LVS Tool)

*Netgen adalah alat untuk membandingkan netlist, proses yang dikenal sebagai LVS (Layout vs. Schematic)*.

```bash
git clone [https://github.com/RTimothyEdwards/netgen](https://github.com/RTimothyEdwards/netgen)
cd netgen && ./configure && make && sudo make install
cd ..

```

---

## 5. Setup PDK (Sky130)

Kita akan menggunakan **Google Skywater-130 (130 nm) PDK**. Kita akan menggunakan **OpenLane** untuk mengunduh dan menyiapkan data PDK ini, termasuk sel standar, model SPICE, dan simbol Xschem.

*Catatan: Proses ini membutuhkan unduhan yang besar dan waktu yang cukup lama*.

```bash
cd $HOME
git clone [https://github.com/The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)
cd OpenLane
make
make test

```

*Jika `make test` berhasil, PDK telah siap*.

---

## 6. Integrasi & Konfigurasi Proyek

Langkah ini sangat penting untuk menghubungkan *tools* dengan Sky130 PDK agar siap digunakan untuk mendesain.

### 6.1 Struktur Folder Desain

Buat direktori proyek di `$HOME`:

```bash
mkdir -p ~/design/xschem ~/design/magic ~/design/netgen

```

### 6.2 Konfigurasi Xschem & Ngspice

Hubungkan file konfigurasi (`.xschemrc` dan `.spiceinit`) dari PDK ke folder kerja Anda menggunakan *symbolic link*.

> **PENTING**: Pastikan path `/opt/pdk/...` di bawah ini sesuai dengan lokasi instalasi PDK di sistem Anda (biasanya hasil instalasi OpenLane). Jika berbeda, sesuaikan path-nya.

```bash
cd ~/design/xschem
# Link konfigurasi Xschem
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130A/libs.tech/xschem/xschemrc
# Link konfigurasi Ngspice
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130A/libs.tech/ngspice/spinit .spiceinit

```

*Sekarang, jika Anda menjalankan `xschem` dari folder ini, Xschem akan terintegrasi dengan perangkat Sky130*.

### 6.3 Konfigurasi Magic & Netgen

Lakukan hal yang sama untuk Magic dan Netgen.

```bash
# Setup Magic
cd ~/design/magic
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130A/libs.tech/magic/sky130A.magicrc .magicrc

# Setup Netgen
cd ~/design/netgen
ln -s /opt/pdk/share/pdk/ciel/sky130/versions/0fe599b2afb6708d281543108caf8310912f54af/sky130A/libs.tech/netgen/setup.tcl setup.tcl

```

---

**GG**.

```

```