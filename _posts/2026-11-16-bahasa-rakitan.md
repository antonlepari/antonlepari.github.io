---
title: "ASM yang tidak ASMR!"
date: 2026-11-16 00:10:00 +0700
permalink: /blog/bahasa-rakitan/
tags: [assembly, low-level, reverse-engineering]
---

Python, Java, dan C++ terasa seperti memberi perintah lewat penerjemah. Bahasa rakitan (*assembly*) berbeda: ia hampir berbicara langsung dengan CPU, satu instruksi demi satu instruksi. Artikel ini membahas apa itu assembly, cara kerjanya, dan contoh program pertamamu.

> ⚡ **Ringkasan cepat**
> - Assembly adalah bahasa **tingkat rendah** yang instruksinya hampir 1:1 dengan perintah CPU.
> - Ditulis dengan **mnemonic** (`MOV`, `ADD`, `JMP`), lalu diterjemahkan oleh **assembler**.
> - Terikat pada **arsitektur** prosesor (x86-64, ARM, RISC-V, dan lainnya).
> - Sulit ditulis, tetapi sangat berguna untuk **memahami komputer**, **reverse engineering**, dan **analisis keamanan**.

---

## 📚 1. Apa Itu Bahasa Rakitan?

**Bahasa rakitan (*assembly language*)** adalah bahasa pemrograman tingkat rendah yang dipakai untuk memberi instruksi langsung kepada prosesor (CPU). Alih-alih angka biner yang panjang, instruksinya ditulis dengan kode singkat bernama **mnemonic**:

| Mnemonic | Fungsi |
|----------|--------|
| `MOV` | Memindahkan data |
| `ADD` | Menjumlahkan data |
| `SUB` | Mengurangi data |
| `MUL` | Mengalikan data |
| `JMP` | Melompat ke instruksi lain |

Berbeda dengan Python, Java, atau C++, assembly punya hubungan yang sangat dekat dengan **arsitektur perangkat keras** komputer.

---

## 🕰️ 2. Sejarah Singkat

Pada masa awal komputer, pemrograman dilakukan dengan **bahasa mesin**, yaitu deretan angka biner. Bahasa rakitan diciptakan agar programmer tidak perlu menghafal kode biner yang panjang.

Perkembangannya berkaitan erat dengan arsitektur prosesor:

- **Intel x86**
- **x86-64 (AMD64)**
- **ARM**
- **MIPS**
- **RISC-V**

> 📌 Setiap arsitektur punya kumpulan instruksi sendiri yang disebut **Instruction Set Architecture (ISA)**. Karena itu, kode assembly x86 tidak bisa langsung dijalankan di ARM.

---

## ⚙️ 3. Bagaimana Cara Kerjanya?

Assembly menerjemahkan instruksi mnemonic menjadi bahasa mesin yang dipahami CPU. Prosesnya:

1. Programmer menulis kode assembly.
2. **Assembler** menerjemahkannya menjadi kode objek (*machine code*).
3. **Linker** menghubungkan kode objek dengan pustaka dan membuat program yang bisa dijalankan.
4. **CPU** mengeksekusi instruksi mesin tersebut.

```text
┌────────────────────────┐
│ Programmer             │
│ menulis kode Assembly  │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ Assembler              │
│ menerjemahkan kode     │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ Machine Code           │
│ bahasa mesin           │
└───────────┬────────────┘
            ▼
┌────────────────────────┐
│ CPU                    │
│ menjalankan instruksi  │
└────────────────────────┘
```

---

## 🧠 4. Komponen Penting

### A. Register

**Register** adalah penyimpanan kecil di dalam CPU yang diakses sangat cepat. Contoh pada x86-64:

| Register | Fungsi |
|----------|--------|
| `RAX` | Register umum, sering untuk hasil operasi |
| `RBX` | Register umum |
| `RCX` | Register umum, sering sebagai penghitung |
| `RDX` | Register umum |
| `RSI` | *Source Index* |
| `RDI` | *Destination Index* |
| `RSP` | *Stack Pointer* |
| `RBP` | *Base Pointer* |

### B. Mnemonic

**Mnemonic** adalah singkatan yang mewakili satu instruksi.

| Instruksi | Fungsi |
|-----------|--------|
| `MOV` | Memindahkan data |
| `ADD` | Penjumlahan |
| `SUB` | Pengurangan |
| `INC` | Menambah nilai satu |
| `DEC` | Mengurangi nilai satu |
| `CMP` | Membandingkan data |
| `JMP` | Melompat ke instruksi lain |
| `CALL` | Memanggil prosedur |
| `RET` | Kembali dari prosedur |

### C. Operand

**Operand** adalah nilai atau lokasi data yang dipakai instruksi. Contoh:

```nasm
MOV AX, 10
```

- `MOV` adalah **mnemonic**.
- `AX` adalah operand **tujuan**.
- `10` adalah operand **sumber**.

> 💡 Pada sintaks Intel (yang dipakai NASM), tujuan ditulis **lebih dulu**, lalu sumber. Sintaks AT&T membalik urutannya.

---

## 💡 5. Contoh Program

Contoh berikut memakai sintaks **NASM** untuk **x86-64 di Linux**.

### A. Penjumlahan

Menjumlahkan 5 dan 3, lalu mengembalikan hasilnya sebagai *exit code* program.

```nasm
section .text
global _start

_start:
    mov rax, 5        ; rax = 5
    mov rbx, 3        ; rbx = 3
    add rax, rbx      ; rax = rax + rbx  -> 8

    mov rdi, rax      ; exit code = hasil penjumlahan
    mov rax, 60       ; nomor syscall exit
    syscall           ; keluar dari program
```

Penjelasan:

- `mov rax, 5` memasukkan angka 5 ke `RAX`.
- `mov rbx, 3` memasukkan angka 3 ke `RBX`.
- `add rax, rbx` menjumlahkan keduanya, hasilnya (8) tersimpan di `RAX`.
- `mov rdi, rax` menyalin hasil ke `RDI`, tempat *exit code* dibaca.
- `mov rax, 60` dan `syscall` meminta sistem operasi mengakhiri program.

**Cara menjalankan:**

```bash
nasm -f elf64 jumlah.asm -o jumlah.o
ld jumlah.o -o jumlah
./jumlah
echo $?        # hasilnya: 8
```

### B. Perulangan Sederhana

```nasm
section .text
global _start

_start:
    mov rcx, 5        ; penghitung = 5

ulang:
    dec rcx           ; rcx = rcx - 1
    jnz ulang         ; ulangi selama rcx belum nol

    mov rax, 60
    xor rdi, rdi      ; exit code 0
    syscall
```

Penjelasan:

- `mov rcx, 5` mengisi penghitung dengan 5.
- `dec rcx` mengurangi nilainya satu per satu.
- `jnz ulang` melompat kembali ke label `ulang` selama hasilnya **belum nol** (*jump if not zero*).
- Setelah `RCX` mencapai nol, program lanjut ke bagian keluar.

---

## 🚀 6. Kelebihan

- **Performa tinggi:** instruksi CPU diatur secara langsung.
- **Kontrol perangkat keras:** register dan instruksi prosesor dikelola sendiri.
- **Ukuran efisien:** kode mesin yang dihasilkan bisa sangat ringkas.
- **Memahami komputer:** cara terbaik mempelajari arsitektur CPU dan memori.
- **Optimasi khusus:** berguna untuk mempercepat bagian program tertentu.

## ⚠️ 7. Kekurangan

- **Sulit dipelajari:** butuh pemahaman arsitektur komputer.
- **Kode rumit:** program sederhana pun bisa panjang.
- **Bergantung arsitektur:** kode x86 belum tentu jalan di ARM.
- **Sulit dipelihara:** kode panjang lebih sulit dibaca dan diperbaiki.
- **Pengembangan lambat:** banyak hal harus diatur manual.

---

## 🔍 8. Assembly vs Bahasa Tingkat Tinggi

| Aspek | Bahasa Rakitan | Bahasa Tingkat Tinggi |
|-------|----------------|-----------------------|
| Tingkatan | Rendah | Tinggi |
| Sintaks | Dekat dengan instruksi CPU | Lebih mudah dipahami manusia |
| Portabilitas | Bergantung arsitektur | Umumnya lebih tinggi |
| Pengelolaan memori | Lebih manual | Sering dibantu *runtime* |
| Contoh | NASM, MASM | Python, Java, C++ |
| Pengembangan | Relatif sulit | Relatif mudah |

---

## 🖥️ 9. Di Mana Assembly Dipakai?

- **Sistem operasi:** bagian tertentu kernel dan proses *booting*.
- **Embedded systems:** perangkat dengan sumber daya terbatas.
- **Firmware:** program tingkat rendah yang mengendalikan perangkat keras.
- **Keamanan komputer:** analisis kode mesin dan *reverse engineering*.
- **Optimasi performa:** mempercepat bagian tertentu program.
- **Pengembangan perangkat keras:** pengujian dan pengendalian prosesor.

> 🔐 **Bagi dunia keamanan siber**, membaca assembly adalah keterampilan dasar untuk *malware analysis*, *reverse engineering*, dan memahami kerentanan memori seperti *buffer overflow*.

---

## 📝 10. Istilah Penting

| Istilah | Pengertian |
|---------|------------|
| **CPU** | *Central Processing Unit*, pusat pemrosesan komputer |
| **ALU** | *Arithmetic Logic Unit*, bagian CPU untuk operasi aritmetika dan logika |
| **Register** | Penyimpanan kecil di dalam CPU |
| **Opcode** | Bagian instruksi yang menentukan operasi |
| **Operand** | Data atau lokasi data yang dipakai instruksi |
| **Assembler** | Program penerjemah bahasa rakitan |
| **Machine Code** | Instruksi dalam bentuk kode mesin |
| **Memory** | Tempat penyimpanan data dan instruksi |
| **ISA** | *Instruction Set Architecture*, rancangan kumpulan instruksi prosesor |

---

## 🎯 11. Kesimpulan

Bahasa rakitan adalah bahasa tingkat rendah yang memungkinkan programmer berinteraksi dekat dengan perangkat keras lewat instruksi yang dipahami prosesor. Lebih sulit dipelajari dibanding bahasa tingkat tinggi, tetapi tetap relevan untuk sistem operasi, firmware, *embedded systems*, optimasi program, dan analisis keamanan.

Mempelajari assembly berarti memahami bagaimana komputer bekerja dari tingkat instruksi paling dasar.

---

## 📖 12. Referensi Belajar

- [NASM Documentation](https://www.nasm.us/docs.php): dokumentasi assembler NASM.
- [Intel Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html): dokumentasi arsitektur dan instruksi Intel.
- **AMD64 Architecture Programmer's Manual**: dokumentasi arsitektur AMD64 (cari di situs AMD).
- [RISC-V Specifications](https://riscv.org/technical/specifications/): spesifikasi arsitektur RISC-V.

<div style="text-align:center; margin-top:2rem">

**💻 "Understand the Machine, Master the Code."**

*Belajar bahasa rakitan adalah langkah awal memahami komputer lebih dalam.*

</div>
