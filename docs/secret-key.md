# Analisis Statis: Secret Key (code.exe)

**Sumber:** crackmes.one
**Author:** RR7
**Platform:** Windows
**Arsitektur:** x86
**Bahasa:** C/C++ (compiled dengan GCC/MinGW-w64)

## 1. Hash

| Algoritma | Nilai |
|---|---|
| SHA256 | `F444F0EF793DB4E9320B04D618AFF6FCDA7CF1502B4C3CE5F4BF0B71C72E2273` |
| MD5 | `CCE9B5407F12FF9F4625EEB24A300932` |

## 2. Tipe File

- **Format:** Portable Executable (PE)
- **Arsitektur:** x86
- **Compiler:** GCC 16.1.0 (dikonfirmasi dari string `"GCC: (Rev5, Built by MSYS2 project) 16.1.0"`)
- **Toolchain:** MinGW-w64 via MSYS2 — bukan Visual Studio/MSVC maupun Visual Basic
- **Runtime model:** Universal CRT (UCRT), ditandai dependency ke `api-ms-win-crt-*.dll`

## 3. Strings (Hasil Ekstraksi)

Output `strings` pada binary ini jauh lebih panjang dibanding `abu_crackme_v1`
(ratusan baris) karena membawa banyak debug symbol, path compiler developer,
dan metadata struct Windows internal bawaan MinGW. Berikut string yang
signifikan setelah difilter:

| String | Kategori | Keterangan |
|---|---|---|
| `Enter Your Number :` | Pesan program | Prompt input pertama |
| `Enter The Secret Key :` | Pesan program | Prompt input kedua (muncul jika angka pertama benar) |
| `Congratulations, you have completed the challenge!` | Pesan program | Muncul saat kedua validasi lolos |
| `KERNEL32.dll` | Runtime dependency | Windows API dasar (critical section, VirtualProtect, dll) |
| `api-ms-win-crt-runtime-l1-1-0.dll` | Runtime dependency | Universal CRT (Windows 10+ API set) |
| `api-ms-win-crt-stdio-l1-1-0.dll` | Runtime dependency | Fungsi I/O standar (printf, scanf, fflush) |
| `api-ms-win-crt-math-l1-1-0.dll` | Runtime dependency | Fungsi matematika C |
| `api-ms-win-crt-heap-l1-1-0.dll` | Runtime dependency | Alokasi memori (malloc, calloc, free) |
| `api-ms-win-crt-string-l1-1-0.dll` | Runtime dependency | Fungsi string (strlen, strncmp) |
| `GCC: (Rev5, Built by MSYS2 project) 16.1.0` | Compiler info | Konfirmasi compiler & toolchain |
| `Mingw-w64 runtime failure` | Runtime dependency | Konfirmasi binary dibuild pakai MinGW-w64 |
| `code.c` | Source file | Nama file source asli sebelum dikompilasi |
| `main` | Fungsi | Entry point program |
| `printf`, `scanf` | Fungsi C standar | Fungsi I/O yang dipakai program (sesuai temuan decompile) |

## 4. Import Table

| DLL | Fungsi yang Diimpor (contoh) |
|---|---|
| `KERNEL32.DLL` | `GetLastError`, `VirtualProtect`, `VirtualQuery`, `Sleep`, `EnterCriticalSection`, `LeaveCriticalSection`, `InitializeCriticalSection`, `DeleteCriticalSection`, `TlsGetValue`, `SetUnhandledExceptionFilter` |
| `API-MS-WIN-CRT-STDIO-L1-1-0.DLL` | `__stdio_common_vfprintf`, `__stdio_common_vfscanf`, `fflush`, `setvbuf` |
| `API-MS-WIN-CRT-RUNTIME-L1-1-0.DLL` | `_cexit`, `_exit`, `_initterm`, `_initterm_e`, `_set_app_type`, `_set_invalid_parameter_handler`, `abort`, `exit` |
| `API-MS-WIN-CRT-HEAP-L1-1-0.DLL` | `calloc`, `free`, `malloc` |
| `API-MS-WIN-CRT-STRING-L1-1-0.DLL` | `strlen`, `strncmp` |
| `API-MS-WIN-CRT-MATH-L1-1-0.DLL` | (fungsi matematika terkait `_matherr` handler) |
| `API-MS-WIN-CRT-LOCALE-L1-1-0.DLL` | `_configthreadlocale` |
| `API-MS-WIN-CRT-ENVIRONMENT-L1-1-0.DLL` | `__p__environ` |
| `API-MS-WIN-CRT-PRIVATE-L1-1-0.DLL` | `__setusermatherr`, fungsi internal CRT lainnya |

![Import Table dari Ghidra Symbol Tree](screenshots/secret-key-import-table.png)

## 5. Kesimpulan

Binary ini adalah program C sederhana yang dikompilasi menggunakan
**GCC 16.1.0 via toolchain MinGW-w64/MSYS2**, bukan Visual Studio (MSVC)
maupun Visual Basic seperti binary sebelumnya (`abu_crackme_v1`). Hal ini
terlihat jelas dari string compiler yang tertanam langsung di binary serta
pola dependency ke API set modern **Universal CRT** (`api-ms-win-crt-*.dll`),
yang merupakan model API tervirtualisasi khas aplikasi Windows 10+ hasil
kompilasi MinGW modern.

Import table menunjukkan binary ini bergantung pada banyak modul kecil
`api-ms-win-crt-*` — pola khas UCRT yang memecah fungsi C standar
(stdio, string, heap, math) menjadi DLL-DLL terpisah, alih-alih satu
`msvcrt.dll` monolitik seperti binary C lawas.

Dari sisi proteksi, validasi program ini **sangat sederhana**: hanya
membandingkan dua nilai integer literal (`0x21` dan `0x66` dalam heksadesimal,
setara 33 dan 102 desimal) tanpa enkripsi, obfuscation, atau operasi
matematis kompleks apa pun. Ini bertolak belakang dengan `abu_crackme_v1`
yang memakai kombinasi fungsi trigonometri sebelum validasi — menjadikan
binary ini contoh baik untuk kategori crackme level pemula (entry-level),
sesuai skor kualitas rendah (1.0/4.0) yang tercatat di crackmes.one.

**Catatan:** ukuran output `strings` binary ini jauh lebih besar
dibanding `abu_crackme_v1` meski logikanya lebih sederhana — ini karena
binary hasil kompilasi GCC/MinGW membawa metadata debug (DWARF: `.debug_info`,
`.debug_line`, dll) dan path source developer secara default, berbeda
dengan binary VB6 yang cenderung lebih ringkas stringnya.
